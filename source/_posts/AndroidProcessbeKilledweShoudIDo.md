title: 当Android应用被强杀之后
isupdate: false
date: 2016-09-10 19:00:20
updatetime:
tags: Android
categories: Android
---

Android应用在后台运行的时候很容易被强杀，尤其是在国内XX助手、XX管家、XX大师之类的应用大行其道之下。如果我们开发的应用没有被用户加入白名单，很大程度上会被系统因为内存不足或者用户主动给应用在后台运行的时候杀掉。这个时候问题就来了：

 - **如何保证我们的应用在被强杀之后用户再次回到应用来保证应用的稳定性而不Crash？**


为了说明上述问题，我们来简单模拟这样一个过程:


> APP --> ActivityA --> ActivityB -->ActivityC --> Pressed Home

假设APP在Activity C页面用户按下`Home`键应用退到后台运行。这个时候启动DDMS，选中该APP的进程，Kill。然后我们从运行APP历史列表中选中该APP并将其置于前台，这个时候回到该应用的界面还是Activity C。再点击返回按钮回到ActivityB，在某些性能比较差一点的机器上可能会出现短暂的黑屏然后才会显示出ActivityB。这是因为该Activity实例其实在Kill该APP进程的时候已经被销毁了，但是Android系统虽然销毁了Activity实例，却并没有销毁该APP的Activity栈。因此我们点击返回按钮还是会回到ActivityB。但是需要重新构建该ActivityB的实例。

这样看貌似并没有什么问题，然而事情并不会这么简单（废话，不然我写这篇博客干嘛。。），如果ActivityB中引用了静态变量并尝试获取其值的时候，这个时候是会出现NPE的。

<!--more-->

我们简单来总结下上述过程:

 1. 当应用在后台被Kill，整个APP进程都被销毁，所有变量都被清空，包括Application的实例。

 2. 虽然所有变量和实例都被销毁，但是Activity栈并没有被清空，所以我们回到应用还能得知页面的打开顺序。

 3. 当应用被强杀时，会自动调用`onSaveInstance`方法去保存一些核心变量，然而这在面对N多的页面的时候显然不是一件省心的事情，而且你也不能保证你的队友也会这么做。。

 4. 在某些性能比较低或者页面逻辑比较复杂的页面会黑屏是因为需要重建ActivityB的实例，也就是需要重走Activity的声明周期`OnCreate`，性能差点的机器上自然就会有短暂的黑屏了。



如果APP中没有静态变量的引用，那就不会出现NEP，但是一旦引用了静态变量，这个时候可能就比较危险了。（静态变量包括全局的登录状态，全局的用户配置、标志位之类的数据）当然了，如果你能将所有的静态变量修改到单例中去，并将其持久化，为NULL的时候再去取的话，原则上来说这样也可以避免NPE。然而要是这样做的话很大程度上会减缓开发的进度，而且指不定哪个队友就给你挖坑了。然后你怎么挂的都不知道。。

为了一劳永逸的解决这个问题，我们需要冷静下来思考一下:**既然APP被强杀了，为啥还要回到原先的页面中去而不是重走启动APP的流程?**

我们虽然不能阻止Android系统销毁实例却保存Activity栈，也不想多写那些持久化或者Cache静态变量代码（这将是一件费力而且不太讨好的事情）所以我们唯一能做的就是检测应用是否被强杀，并且在被强杀之后重走启动流程而不是回到原先的逻辑当中。

以下给出我的一种实现方式，如果你有更好的想法，欢迎和我交流:


```java

public abstract class BaseActivity extends AppCompatActivity {
  
  private static int FLAG;
  private final static int FLAG_NOT_INIT = 200;

  @Override protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    if (FLAG != FLAG_NOT_INIT) {
      if (this.getClass() == LauncherActivity.class) {
        FLAG = FLAG_NOT_INIT;
      } else {
        reLauncher();
      }
    }
  }
}

```

我们实例化上述场景，给出一种对应关系:

ActivityA --> 启动页

ActivityB --> 主页

ActivityC --> 详情页

其中将ActivityB的`launchMode`设置为`singleTask`，并且在BaseActivity中使用静态变量FLAG进行判断当前应用是否被强杀，如果被强杀则利用ActivityB的`launchMode`特性清空栈并重新初始化即可。



######有关Activity的启动模式相关文章，可参阅[Activity启动模式图文详解](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0520/2897.html)


