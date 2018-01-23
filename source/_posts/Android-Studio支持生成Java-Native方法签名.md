---
title: Android Studio生成Java方法描述符
date: 2017-03-21 21:32:50
tags: 
 - Android
 - Android Studio
 - JNI
---



{% note info %}在日常的Android NDK开发中，会不可避免的用到C与Java代码相互调用的情况。Java调用C的方法还好，C调用Java的方法就比较麻烦了。需要编写看着就头疼的Java方法描述符才能正确的调用Java方法。 {% endnote %}



其中常见的Java方法域和描述符如下表所示：



|        Java类型         |           签名           |
| :-------------------: | :--------------------: |
|        Boolean        |           Z            |
|         Byte          |           B            |
|         Char          |           C            |
|         Short         |           S            |
|          Int          |           I            |
|         Long          |           J            |
|         Float         |           F            |
|        Double         |           D            |
| Fully-qualified-class | Lfully-qualified-class |
|        type[]         |         [type          |
|      Method type      |   (arg-type)ret-type   |



通过上述对照表，我们可以通过C代码查找一个为`String`类型的Java静态字段，例如:

```c
jfieldID staticJavaFieldId;
staticJavaFieldId = env->GetStaticFieldID(clazz, "mStaticField", "Ljava/lang/String;");
/**
* do something...
**/
```



<!--more-->

-----------------------------



借助`javap`我们可以很方便的得知一个`class`文件其中包含对应的描述符。如下：

```Java
$ javap -s -p com.xiamo.test.Message
  
Compiled from "Message.java"
public class com.xiamo.test.Message {
  private static final java.lang.String TAG;
    descriptor: Ljava/lang/String;
  public static final int CHECK_POINT;
    descriptor: I
  public static final int ERROR_NOT_SERVER;
    descriptor: I
  public int errorCode;
    descriptor: I
  public java.lang.String message;
    descriptor: Ljava/lang/String;
  public com.xiamo.test.Message();
    descriptor: ()V
```



但是每次需要查看对应类的方法描述符的时候都需要手动敲一次命令，这样显然不够清真。好在`Android Studio`为我们提供了`External Tools`。我们可以用它来自定义这个操作简化我们的双手。



### 设置External Tools



打开`Android Studio`的设置页面，在`Tools`选项卡中选中`External Tools`，如下图所示：



![](http://7xkj6q.com1.z0.glb.clouddn.com/image/jni/external%20tools.png "选择External Tools")



点击右侧区域的`+`新增一个`Tools`,在选卡中填入如下图所示的参数:



![](http://7xkj6q.com1.z0.glb.clouddn.com/image/jni/config.png "设置External Tools")



- `Name` 为你要设置的`External Tools`的名字，便于你自己标识就行，此处我设置为`JNI Descriptor Generator`
- `Program`为`Tools`执行的命令的路径，如果你需要替换为你自己JDK中的`javap`修改这个值就行，此处使用`Android Studio`自带的`JDK`路径，填入`$JDKPath$/bin/javap`
- `Parameters`为命令执行的参数，我们要获取方法描述符，所以设置为：`-s -p $FileClass$`
- `Working directory`为上述设置好的工具执行的目录，设置为`$ModuleFileDir$/build/intermediates/classes/debug`

点击保存，我们的`External Tools`就设置好啦。这个时候在`Tools`—>`External Tools`中就可以看到我们设置好的`Tools`了。需要注意的是这个时候点击改工具查看当前我们选中的Java源文件的文件操作符，是可能会报错找不到指定的class文件。



这是因为我们指定的`Working directory`中还没有生成class文件，选择`Build`选项中的`Make Project`，等待make完成，再次点击`Tools`—>`External Tools—>` `JNI Descriptor Generator` 即可生成对应Java源文件的文件描述符了。这样我们就可以愉快的调用使用C调用Java中的方法咯。