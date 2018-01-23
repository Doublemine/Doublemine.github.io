title: "Create an Android IME"
date: 2015-04-01 23:25:22
tags: android
categories: Android

---

## Creating an Input Method


>输入法编辑器（IME，有的地方简称IME）是一个使用户能够输入文字的控件。Android提供了一个可扩展的输入法框架，它允许应用程序为用户提供可替代的输入方法，如屏幕上的键盘甚至是语音输入。一旦安装,用户就可以在系统设置里选择他喜欢的输入法,这个设置可以在Android系统的所有的地方使用。当然，使用过程中，一次只有一种输入法被激活（用户选择的那个）。  



>想要给Android开发一个输入法应用，您的Android应用需要含一个继承自InputMethodService的类。此外，您通常需要创建一个“设置输入法”的Activity来设定应用的配置。您也可以定义一个作为系统设置的一部分来显示的设置界面。


​        


​        
本文包括以下内容：
- IME的生命周期。
- 在应用程序清单中的声明IME组件。
- IME的API。
- 设计一个IME的界面。
- 从IME向应用程序发送文本。
- 使用IME的变种。  

<!-- more -->
​    
>如果您以前没有开发过IME，建议先阅读《屏幕上的输入法》
>这一章的介绍。此外，您可以通过修改SDK中的软键盘（Soft Keyboard sample）实例代码来开始建立自己的输入法应用。
##The IME Lifecycle-IME的生命周期
IME的生命周期如下图：
​        
![生命周期图](http://developer.android.com/resources/articles/images/inputmethod_lifecycle_image.png)   



下面介绍了怎么根据输入法的生命周期来实现输入法的界面和代码
​    
#### Declaring IME Components in the Manifest-IME的组件清单中声明



>在Android系统中，IME是一个Android应用程序包含一个特殊的输入法服务。应用程序的清单文件必须申报服务，索取必要的权限，提供相匹配的action.view.InputMethod意图过滤器，并提供元数据定义输入法的特点。此外，提供了一个设置界面，允许用户修改输入法的行为，你可以定义一个“设置”的Activity，可以从系统设置中进入。



下面的代码片段声明了IME服务，这个服务需要通过BIND_INPUT_METHOD权限来声明系统允许它连接系统的IME（输入法服务）。还需要创建一个应答android.view.InputMethod的意图过滤器以及定义IME的元数据：
```xml
<!-- Declares the input method service -->
    <service android:name="FastInputIME"
        android:label="@string/fast_input_label"
        android:permission="android.permission.BIND_INPUT_METHOD">
        <intent-filter>
            <action android:name="android.view.InputMethod" />
        </intent-filter>
        <meta-data android:name="android.view.im" android:resource="@xml/method" />
    </service>
```


​    
>接下来的代码片段声明了设置IME的Activity。他有一个应答android.intent.action.MAIN的过滤器，这个过滤器表明这个Activity是IME应用的入口。

```xml
<!-- Optional: an activity for controlling the IME settings -->
    <activity android:name="FastInputIMESettings" 
        android:label="@string/fast_input_settings">
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
        </intent-filter>
    </activity>
```

>您也可以把设置输入法的功能直接放到输入法应用的界面上。

### The Input Method API-输入的API


>在android.inputmethodservice和android.view.inputmethod包下面可以找到开发IME所需的特殊的类。按钮事件类对于处理键盘字符的输入非常重要。


>一个输入法应用的核心部分是一个服务组件，这是一个继承自InputMethodService的类。除了继承通常的服务周期，这个类还有提供输入法界面、处理用户输入、发送文本到焦点区域的回调函数。 默认的，InputMethodService类提供了大部分管理输入法的状态、输入法的可见性以及与当前焦点区域交流的接口。



下面的类也很重要：  

    BaseInputConnection
   >定义了从输入法返回到应用后该应用接收输入法输入信息的渠道。您可以使用它来读取光标周围的文本，提交文本文本框内的信息， 发送原生的按钮事件到应用。应用程序应该继承，而不是实现InputConnection类的接口。

    KeyboardView
>查看扩展，呈现一个键盘和响应用户的输入事件。 键盘的一个实例，您可以定义在指定的键盘布局XML文件

## Designing the Input Method UI-设置输入法的界面



>一个输入法应用有两个主要的界面元素：候选字界面和输入界面。您只需要实现和您设计的输入法有关的元素即可。 

### Input view-输入法界面



>输入界面是用户输入通过按键，手写或者手势输入文本的界面。当一个自定义的输入法第一次显示的时候，系统将会执行onCreateInputView()方法。在您实现这方法的时候，您可以创建一个想要在输入法窗口显示的布局，然后返回给android系统。下面的代码片段是实现onCreateInputView() 方法的一个例子。

```java
  @Override 
    public View onCreateInputView() { 
        MyKeyboardView inputView = 
            (MyKeyboardView) getLayoutInflater().inflate( R.layout.input, null);
 
        inputView.setOnKeyboardActionListener(this); inputView.setKeyboard(mLatinKeyboard); 
 
        return mInputView; 
    } 
 
```


>在这个例子中，MyKeyboardView是一个继承KeyboardView的自定义接口后实现的一个实例，它呈现了一个键盘。如果您在创建一个QWERTY键盘，您可以参照软键盘的简单实例（ Soft Keyboard sample app）来学习怎么继承KeyboardView 类。


### Candidates view-候选字界面



>候选字界面是输入法显示候选字或者是联想字的界面。在IME的生命周期里，当IME准备显示候选字界面的时候，系统或调用onCreateCandidatesView()方法。在您实现这个方法的时候，请返回一个显示候选字或者联想字的界面，或者，如果您不行显示什么东西，那返回空。（注意，这个方法默认就是返回空，所以如果您不想提供候选字,那你不用实现这个方法。）



    实现候选字的实例请查看Soft Keyboard sample app工程。

## UI design considerations-界面设计要点



    这一节介绍输入法界面设计需要注意的要点：

### Handling multiple screen sizes-多分辨率支持



>输入法界面必须可以适合不同的分辨率，它也必须适应横屏和竖屏。在非全屏模式，要为应用留出足够的空间来显示文本输入区域和其他相关情况，从而是输入法界面占用的屏幕空间少于屏幕的一半。在全屏输入法模式下就不用考虑这个问题了。

### Handling different input types-处理不同的输入类型



>android文本输入域允许用户输入特定的文本类型，比如任何文本输入数字输入，URL输入，email输入，搜索字符串输入等。创建一个新的输入法的时候，您需要检测每一个输入框的类型并为它提供正确的输入，然而，您不必设置您的输入法来检查用户输入的文本是否符合指定的类型；这是那个输入框所在的应用程序的工作。



    这里是拉丁输入法的提供文本输入和数字输入的截图：

![文本数字输入截图](http://developer.android.com/resources/articles/images/inputmethod_text_type_screenshot.png)
![文本数字输入截图](http://developer.android.com/resources/articles/images/inputmethod_numeric_type_screenshot.png)



>当一个输入域获得焦点后,您的输入法就会开始运行。系统会调用onStartInputView()，传递一个含有输入类型和输入框其他属性的EditorInfo实体。在这个实体中，输入类型域含有输入域的输入类型。



>输入类型域是一个int类型，他含有一个标志各种输入类型的二进制数值。如果您想测试输入域类型，您可以使用TYPE_MASK_CLASS常量,如下所示
```xml
inputType & InputType.TYPE_MASK_CLASS 
```


    每一个二进制数值可以是下面几个值中的一个:

```xml
TYPE_CLASS_NUMBER
```
>输入数字的输入域。就像上面介绍中的截屏那样,拉丁输入法展示这样一种字母输入模版。

```xml
TYPE_CLASS_DATETIME
```
>一个输入时间和日期的输入域。

```xml
TYPE_CLASS_PHONE
```
>一个输入电话号码的输入域。

```xml
TYPE_CLASS_TEXT
```
>一个输入字符的输入域。

    在输入类型的文档里面有更多关于这些常量的描述。

>输入类型标志也可以是从输入域变化而来的其他常量，比如：
```xml
TYPE_TEXT_VARIATION_PASSWORD
```
>一个TYPE_CLASS_TEXT的变种，用来输入密码。输入的内容将显示为点点而不是真正的文本。

```xml
TYPE_TEXT_VARIATION_URI
```
>一个TYPE_CLASS_TEXT 的变种，用来输入web URL和其他统一资源定位符（URI）。

```xml
TYPE_TEXT_FLAG_AUTO_COMPLETE
```
>一个TYPE_CLASS_TEXT 的变种，用来输入应用程序通过词典，搜索或者其他功能可以自动补全的文本。

    当您测试这些变种标志的时候，记得使用正确的常量标志。这些常量标志在输入类型文档里面有详细说明。

>注意:在您的输入法中,如果需要输入信息到密码框,请确保文字处理的正确性，在您的输入界面和候选字界面都要隐藏密码文字。记住，不要在一个设备上储存密码。更多信息参阅设计的安全性（Designing for Security guide）这一节。

### Sending Text to the Application-向应用发送文本

>当用户使用您的输入法输入文本的时候，您可以通过使用发送个人按键事件或者在应用输入域的光标周围编辑文本的方式来应用发送文本。或者，您可以使用InputConnection 的一个实例来发送文本。您可以通过InputMethodService.getCurrentInputConnection()来获取这个实例。

#### Editing the text around the cursor-在光标周围编辑文本

>当您处理文本域存在的文本的时候，BaseInputConnection里有很多非常有用的方法：
```java
getTextBeforeCursor()
```
>返回光标所在位置前面的n个字符。
```java
getTextAfterCursor()
```
>返回光标所在位置后面的n个字符。
```java
deleteSurroundingText()
```
>删除光标所在位置周围的n个字符。
```java
commitText()
```
>把一个文本提交到文本域并把光标设置到新的位置。

    下面的代码片段展示了怎么使用“Hello”文本替换“Fell”的左边。

```java
InputConnection ic = getCurrentInputConnection();
 
    ic.deleteSurroundingText(4, 0);
 
    ic.commitText("Hello", 1);
 
    ic.commitText("!", 1);
```
#### Composing text before committing-在提交文本之前的撰写
>如果您的输入法通过需要各种计算或者各种按键的组合来生成字符或单词(比如中文),您可以先在文本域显示这个输入进度直到用户提交了这些文字，这样您可以替换已经完成文本的一部分。当您需要把它传送到InputConnection#setComposingText()时,你可以通过使用“间隔”来区别对待。

    下面的代码片段说明了怎么在输入域显示输入的进度。
```java
InputConnection ic = getCurrentInputConnection();
 
    ic.setComposingText("Composi", 1);
...
 
    ic.setComposingText("Composin", 1);
 
...
 
    ic.commitText("Composing ", 1);
```
    下面的截屏显示这如何呈现给用户

![截图](http://developer.android.com/resources/articles/images/inputmethod_composing_text_1.png)

![](http://developer.android.com/resources/articles/images/inputmethod_composing_text_2.png)
![](http://developer.android.com/resources/articles/images/inputmethod_composing_text_3.png)

#### Intercepting hardware key events-拦截实体键盘按钮事件

>尽管输入法窗口不会争夺输入焦点，但输入法还是会先接受来自实体键盘的输入并且可以选择忽略他们或者把他们转发给应用
>如果您想拦截实体键盘的输入，请重写onKeyDown() 和onKeyUp()方法，详情参考键盘输入应用。
>如果不想自己处理一些按键事件，记得调用按键的super()方法。

#### reating an IME Subtype-创建一个输入法亚种
>输入法亚种可以是输入法执行几种不同的输入模式也可以支持多种语言(比如搜狗输入法的中英文切换)。一个亚种可以是：

- 一种语言环境比如en_US或者fr_FR
- 一种输入模式比如声音键盘或者手写输入
- 其他输入样式，输入形式或者输入法的特殊属性，比如10按键或者qwerty键盘布局。



        基本上这些模式可以是键盘输入，语音输入等等。
        一个亚种也可以上面几种的混合。

>输入法切换窗口需要输入法亚种信息，这些信息可以从通知栏或者输入法设置里得到。这些信息也使得系统框架可以直接选择其中的一种特定的输入法亚种。当您创建输入法的时候，请使用亚种功能，因为他可以帮助用户曲别和切换不同的语言和不同的模式。


        你可以使用<subtype>在一个xml资源文件里定义亚种。下面的代码片段定义了两种输入法亚种：一种是美国英语语言环境，另一中是法国法语语言环境。

```xml

<input-method xmlns:android="http://schemas.android.com/apk/res/android"
        android:settingsActivity="com.example.softkeyboard.Settings"
        android:icon="@drawable/ime_icon"
    <subtype android:name="@string/display_name_english_keyboard_ime"
            android:icon="@drawable/subtype_icon_english_keyboard_ime"
            android:imeSubtypeLanguage="en_US"
            android:imeSubtypeMode="keyboard"
            android:imeSubtypeExtraValue="somePrivateOption=true"
    />
    <subtype android:name="@string/display_name_french_keyboard_ime"
            android:icon="@drawable/subtype_icon_french_keyboard_ime"
            android:imeSubtypeLanguage="fr_FR"
            android:imeSubtypeMode="keyboard"
            android:imeSubtypeExtraValue="foobar=30,someInternalOption=false"
    />
    <subtype android:name="@string/display_name_german_keyboard_ime"
            ...
    />
/>
```
>为了确保您的输入法亚种在界面上的标签正确，请使用%s来获取一个和亚种本地标签相同的亚种标签。如下面的两个代码片段：第一个代码片段展示了输入法xml文件的一部分：
```xml
<subtype
        android:label="@string/label_subtype_generic"
        android:imeSubtypeLocale="en_US"
        android:icon="@drawable/icon_en_us"
        android:imeSubtypeMode="keyboard" />
```
>下一个片段是输入法的strings.xml文件的一部分。这个字符串资源 “label_subtype_generic”被输入法界面用来定义亚种标签，他这样被定义：
```xml
<string name="label_subtype_generic">%s</string>
```
    这个设置了在任何英文语言环境使用“美式英语”或者在其他语言环境使用恰当的语言。

#### Choosing IME subtypes from the notification bar-通过通知栏选择输入法的亚种

>android系统可以管理输入法暴露的所有输入法亚种。输入法亚种被看作是他们所属输入法的一种模式。在通知栏，一个用户可以当前输入法的选择一种输入模式，就下面的截图：
>![](http://developer.android.com/resources/articles/images/inputmethod_subtype_notification.png)

![](http://developer.android.com/resources/articles/images/inputmethod_subtype_preferences.png)

#### Choosing IME subtypes from System Settings-通过系统设置选择输入法亚种

>用户可以在设置的"语言和输入"设置栏里设置输入法亚种。在软键盘例子中，InputMethodSettingsFragment.java 文件实现了在输入法设置中显示输入法亚种这个功能。请更多信息请参阅android SDK的软键盘例子。
>![](http://developer.android.com/resources/articles/images/inputmethod_subtype_settings.png)

### General IME Considerations-创建输入法应用的注意事项
>这里是您创建输入法应用需要注意的其他事项：
- 为用户提供一个可以在输入法界面直接设置输入法的方法。
- 由于输入法可能需要安装到设备上，请为用户提供一个方法来直接在输入界面切换不同的输入法
- 输入法界面需要反应迅速。提前载入或者只载入需要的大的界面资源，这样用户一点击文本框就可以看到。为了同样的目的，可以对输入法后续用到的资源和视图进行缓存。
- 相反的，当输入法窗口隐藏的时候，您应该释放为输入法分配的内存，这样应用可以有足够的内存来运行。如果输入法处于隐藏状态需要几秒钟，那您需要考虑使用一个延迟消息来释放资源。
- 请确保用户可以尽可能多的输入这种输入法需要输入的语言的字符。记住，用户可能需要输入用户名和密码，所以您的输入法必须提供各种不同的字符，使用户可以向设备输入密码。