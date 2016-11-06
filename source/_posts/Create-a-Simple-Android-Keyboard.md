title: Create a Simple Android Keyboard
isupdate: false
date: 2015-09-04 15:12:11
updatetime:
tags: 
 - IME
 - Android
categories: Android
---

##前言

&emsp;&emsp;由于最近做的一个Android项目需要用到用户的输入一些字符，常规的输入法输入非常的不方便。因此有必要自定义一个输入法来完成这个过程。此处给出一个简单的输入法Demo
来帮助理解自定义输入法的一些实现过程。

 - **有关输入法的一些说明，请参见:[Create a Android IME](http://notes.xiamo.tk/2015-04-01-Create-an-Android-IME.html)**

<!--more-->

##准备工作

&emsp;&emsp;磨刀不误砍柴工，创建一个Android是需要一点点准备工作的,选择一个好的IDE能够提高我们编码的效率：
 - Android Studio（推荐）
 - Android SDK

本博文的编写环境为Android Studio，如果你还在使用Eclipse的话，转到Android Studio上来吧！如果对上述IDE的下载感到茫然的话，推荐一个国内Android开发的好网站：[http://www.androiddevtools.cn](http://www.androiddevtools.cn/)。

##创建一个Android项目

 - 打开Android Studio，创建一个新的Android项目，命名为`SimpleKeyboard`，如下图：

![](http://i3.piimg.com/37ae2bfdf945c18e.png "创建一个SimpleKeyboard项目")

 - 最小的Android支持版本我们在这里选择API 9也就是Android 2.3咯：

![](http://i3.piimg.com/f337c436d68dcc22.png "设置最小SDK支持版本")

 - 由于我们只是一个单独的键盘Demo，为了达到极致精简，还是不需要Activity了，因此我们选择`Add No Activity`：

![](http://i3.piimg.com/fed92e215ec17e4f.png "选择Add No Activity")

点击Finish，这样就完成了我们Android Keyboard项目的创建了。这个等待Gradle构建完成，因为我们这个项目是没有Activity的，因此我们需要按照如下图稍微配置一下：

![](http://i3.piimg.com/4c72f6d152c54dee.png)

![](http://i3.piimg.com/2195d3987d5dddd1.png)


##编辑AndroidManifest.xml文件

&emsp;&emsp;键盘在Android系统中被识别为一个输入法编辑器（IME），IME作为一个Service运行。只有在AndroidManifest.xml文件中通过`android.permission.BIND_INPUT_METHOD`权限声明的Service并且响应`android.view.im`这个元数据动作才能够被Android系统正确的识别为IME。因此，我们在`AndroidManifest.xml`文件中的application标签对中添加如下代码：

```xml
		<service
            android:name=".SimpleIME"
            android:label="@string/custom_ime"
            android:permission="android.permission.BIND_INPUT_METHOD">
            <meta-data
                android:name="android.view.im"
                android:resource="@xml/method" />
            <intent-filter>
                <action android:name="android.view.InputMethod" />
            </intent-filter>
        </service>
```

##创建method.xml

>&emsp;&emsp;在上面的Service声明中，meta-data标签声明引用了一个叫做`method.xml`的文件，如果没有这个文件，那么Android系统将不能够识别我们的Service为一个有效的IME Service。这个文件包含了有关输入法及其子类的详细信息。

在我们的Demo中，我们定义一个`subtyoe`来声明输入法的显示名称以及其语言环境：（如果没有res/xml目录的话，创建一个并将下面的内容添加到该目录的method.xml文件中）

```xml
<?xml version="1.0" encoding="utf-8"?>
<input-method xmlns:android="http://schemas.android.com/apk/res/android">
    <subtype
        android:imeSubtypeLocale="zh_CN"
        android:imeSubtypeMode="keyboard"
        android:label="tk_xiamo_notes" />
</input-method>
```

如果你对label中直接写死字符串这种做法比较有强迫症的话，可以将其抽取到Strings.xml文件中。

##定义键盘布局

&emsp;&emsp;我们的键盘布局比较简单，仅仅包含了一个KeyboardView，因此在`layout`目录中创建一个`keyboard.xml`文件，并添加一下内容：

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.inputmethodservice.KeyboardView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/keyboard"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_alignParentBottom="true"
    android:keyPreviewLayout="@layout/preview" />

```

其中需要说明的是：
 - `layout_alignParentBottom`属性设置为`true`来保证键盘会在设备屏幕的底端弹出而不是在其它什么地方弹出。
 - `keyPreviewLayout`属性用于按下按键时暂短的按键预览。这里我们由于图方便省事，就用一个TextView来预览吧~

在`layout`目录下创建一个preview.xml文件，并在其中添加如下内容：

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:background="#A6A6A6"
    android:textStyle="bold"
    android:textSize="30sp"
    >
</TextView>
```

##定义键盘按键

>键盘按键所代表的键值以及其位置等详细信息都被指定在一个xml文件中，每个独立的键盘按键至少都必须包含一下两个属性：



| 属性名 | 作用 |
|--------|--------|
|keyLabel     |    决定这个按键上显示的字符信息    |
|codes           |  决定这个按键的对应的字符信息所代表的键值|



**例如：定义一个字母A的按键那么它的`codes`属性值应该为：97，`keyLabel`属性值应该为A。**

如果一个按键关联了多个键值，那么点击该按键输出的字符依赖于敲击该按键的次数。

**例如：如果一个按键拥有三个键值：63、33、58:**



| 敲击该按键的次数 | 输出的字符 |
|--------|--------|
|    1次    |    ?    |
|2次|!|
|3次|:|



当然了，除了上述属性之外，一个按键还有一些其他的属性：



| 属性名 | 作用 |
|--------|--------|
|   keyEdgeFlags     |  可以设置的值有：right或者left，该属性通常用在一行最左边或者最右边的按键上用于表示按键的排布      |
|keyWidth| 定义一个按键的宽度，通常该宽度值被定义为一个百分比值|
|isRepeatable|如果这个属性被设置为true的话，长按被设置为该属性的按键将会在长按这段时间中多次重复该按键的动作。通过，在删除键或者空格键中设置该属性为true|


键盘上的按键通过Row标识为一组按键，比较推荐的做法是限制每组中最多10枚按键，这样的话，每个按键的宽度等于键盘宽度的10%。在本Demo中，按键高度被设置为60dp，这个数值可以任意调整。但是推荐不要低于48dp。


在`res/xml`目录中，创建一个`qwerty.xml`文件，并添加以下内容完成按键的定义：

```xml
<?xml version="1.0" encoding="utf-8"?>
<Keyboard xmlns:android="http://schemas.android.com/apk/res/android"
    android:horizontalGap="0px"
    android:keyHeight="60dp"
    android:keyWidth="10%p"
    android:verticalGap="0px"
    >
    <Row>
        <Key
            android:codes="49"
            android:keyEdgeFlags="left"
            android:keyLabel="1" />
        <Key
            android:codes="50"
            android:keyLabel="2" />
        <Key
            android:codes="51"
            android:keyLabel="3" />
        <Key
            android:codes="52"
            android:keyLabel="4" />
        <Key
            android:codes="53"
            android:keyLabel="5" />
        <Key
            android:codes="54"
            android:keyLabel="6" />
        <Key
            android:codes="55"
            android:keyLabel="7" />
        <Key
            android:codes="56"
            android:keyLabel="8" />
        <Key
            android:codes="57"
            android:keyLabel="9" />
        <Key
            android:codes="48"
            android:keyEdgeFlags="right"
            android:keyLabel="0" />
    </Row>
    <Row>
        <Key
            android:codes="113"
            android:keyEdgeFlags="left"
            android:keyLabel="q" />
        <Key
            android:codes="119"
            android:keyLabel="w" />
        <Key
            android:codes="101"
            android:keyLabel="e" />
        <Key
            android:codes="114"
            android:keyLabel="r" />
        <Key
            android:codes="116"
            android:keyLabel="t" />
        <Key
            android:codes="121"
            android:keyLabel="y" />
        <Key
            android:codes="117"
            android:keyLabel="u" />
        <Key
            android:codes="105"
            android:keyLabel="i" />
        <Key
            android:codes="111"
            android:keyLabel="o" />
        <Key
            android:codes="112"
            android:keyEdgeFlags="right"
            android:keyLabel="p" />
    </Row>
    <Row>
        <Key
            android:codes="97"
            android:keyEdgeFlags="left"
            android:keyLabel="a" />
        <Key
            android:codes="115"
            android:keyLabel="s" />
        <Key
            android:codes="100"
            android:keyLabel="d" />
        <Key
            android:codes="102"
            android:keyLabel="f" />
        <Key
            android:codes="103"
            android:keyLabel="g" />
        <Key
            android:codes="104"
            android:keyLabel="h" />
        <Key
            android:codes="106"
            android:keyLabel="j" />
        <Key
            android:codes="107"
            android:keyLabel="k" />
        <Key
            android:codes="108"
            android:keyLabel="l" />
        <Key
            android:codes="35,64"
            android:keyEdgeFlags="right"
            android:keyLabel="\#\@" />
    </Row>
    <Row>
        <Key
            android:codes="-1"
            android:keyEdgeFlags="left"
            android:keyLabel="↑" />
        <Key
            android:codes="122"
            android:keyLabel="z" />
        <Key
            android:codes="120"
            android:keyLabel="x" />
        <Key
            android:codes="99"
            android:keyLabel="c" />
        <Key
            android:codes="118"
            android:keyLabel="v" />
        <Key
            android:codes="98"
            android:keyLabel="b" />
        <Key
            android:codes="110"
            android:keyLabel="n" />
        <Key
            android:codes="109"
            android:keyLabel="m" />
        <Key
            android:codes="46"
            android:keyLabel="." />
        <Key
            android:codes="63,33,58"
            android:keyEdgeFlags="right"
            android:keyLabel="\?!:" />
    </Row>
    <Row android:rowEdgeFlags="bottom">
        <Key
            android:codes="44"
            android:keyEdgeFlags="left"
            android:keyLabel=","
            android:keyWidth="10%p" />
        <Key
            android:codes="47"
            android:keyLabel="/"
            android:keyWidth="10%p" />
        <Key
            android:codes="32"
            android:isRepeatable="true"
            android:keyLabel="SPACE"
            android:keyWidth="40%p" />
        <Key
            android:codes="-5"
            android:isRepeatable="true"
            android:keyLabel="Del"
            android:keyWidth="20%p" />
        <Key
            android:codes="-4"
            android:keyEdgeFlags="right"
            android:keyLabel="Done"
            android:keyWidth="20%p" />
    </Row>
</Keyboard>
```

##创建Service类

创建一个java类，命名为`SimpleIME.java`（与AndroidManifest.xml文件中的定义向对应）：
 - `SimpleIME`类应该继承至`InputMethodService`类。
 - `SimpleIME`类应该实现`OnKeyboardActionListener`接口，该接口包含了键盘被点击或者被按下时回调的一些函数。

创建完成之后，将以下内容添加到该文件中：

```java
package tk.xiamo.notes.simplekeyboard;

import android.inputmethodservice.InputMethodService;
import android.inputmethodservice.Keyboard;
import android.inputmethodservice.KeyboardView;
import android.media.AudioManager;
import android.view.KeyEvent;
import android.view.View;
import android.view.inputmethod.InputConnection;

public class SimpleIME extends InputMethodService implements KeyboardView.OnKeyboardActionListener {

    private KeyboardView keyboardView;
    private Keyboard keyboard;
    private boolean caps = false;//大小写转换
    @Override
    /**
     * onKey方法用于处理键盘与其他应用程序输入域进行交互。
     * getCurrentInputConnection 这个方法被用于获取其它应用程序的输入域，一旦我们获取到这个Connection对象，我们就可以使用以下方法了：
     * commitText ：输出一个或者多个字符到输入域中。
     * deleteSurroundingText ：从输入域中删除一个或者多个字符。
     * sendKeyEvent ：发送一个事件（如KEYCODE_ENTER）到外部的应用程序。
     */
    public void onKey(int primaryCode, int[] keyCodes) {
        InputConnection inputConnection = getCurrentInputConnection();
        playClick(primaryCode);
        switch (primaryCode) {
            case Keyboard.KEYCODE_DELETE:
                inputConnection.deleteSurroundingText(1, 0);
                break;
            case Keyboard.KEYCODE_SHIFT:
                caps = !caps;
                keyboardView.setShifted(caps);
                keyboardView.invalidateAllKeys();
                break;
            case Keyboard.KEYCODE_DONE:
                inputConnection.sendKeyEvent(new KeyEvent(KeyEvent.ACTION_DOWN, KeyEvent.KEYCODE_ENTER));
                break;
            default:
                char code = (char) primaryCode;
                if (Character.isLetter(code) && caps) {
                    code = Character.toUpperCase(code);
                }
                inputConnection.commitText(String.valueOf(code), 1);
        }
    }

    @Override
    /**
     * 当键盘被创建的时候，onCreateInputView方法将会被自动调用。所有的Service成员变量可以在该方法中被初始化。
     */
    public View onCreateInputView() {
        keyboardView = (KeyboardView) getLayoutInflater().inflate(R.layout.keyboard, null);
        keyboard = new Keyboard(this, R.xml.qwerty);
        keyboardView.setKeyboard(keyboard);
        keyboardView.setOnKeyboardActionListener(this);
        return keyboardView;

    }
    /**
     * @param keyCode 按键的键码
     *                对指定的按键播放不同的按键声音
     */
    private void playClick(int keyCode) {
        AudioManager am = (AudioManager) getSystemService(AUDIO_SERVICE);
        switch (keyCode) {
            case 32:
                am.playSoundEffect(AudioManager.FX_KEYPRESS_SPACEBAR);
                break;
            case Keyboard.KEYCODE_DONE:
            case 10:
                am.playSoundEffect(AudioManager.FX_KEYPRESS_RETURN);
                break;
            case Keyboard.KEYCODE_DELETE:
                am.playSoundEffect(AudioManager.FX_KEYPRESS_DELETE);
                break;
            default:
                am.playSoundEffect(AudioManager.FX_KEYPRESS_STANDARD);
        }
    }
    @Override
    public void onText(CharSequence text) {}
    @Override
    public void swipeLeft() {}
    @Override
    public void swipeRight() {}
    @Override
    public void swipeDown() {}
    @Override
    public void swipeUp() {}
    @Override
    public void onPress(int primaryCode) {}
    @Override
    public void onRelease(int primaryCode) {}
}

```

**需要特别说明的是：一旦用户在键盘上按下了一个按键，onKey方法将会带着被按下的按键的所代表的键值参数而被调用，基于键值的不同，将会执行以下动作：**



| 键值 | 指定的动作 |
|--------|--------|
|    KEYCODE_DELETE    |   将会使用deleteSurroundingText 方法删除光标左侧的一个字符。     |
|KEYCODE_DONE|将会激发一个KEYCODE_ENTER 事件。|
|KEYCODE_SHIFT|caps变量的值将会被改变并且通过setShifted方法更新键盘的状态。整个键盘都会被重新绘制来保证状态改变之后按键的label标签可以被更新。其中：invalidateAllKeys 方法将可以重绘所有按键。|
|普通键值|将会被简单的转换为一个字符然后发送到文本输入域中，如果caps变量被设置为true，那么按键字符都将被转换为大写。|

##编译运行

编译运行，在手机上运行的效果如下图：

![](http://i1.piimg.com/e15673195304af50.jpg "键盘运行效果")

