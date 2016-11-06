title: Android之Keystore文件签名
isupdate: false
date: 2016-07-03 20:58:16
updatetime:
tags:
 - Android
 - Android Studio
 - KeyStore
categories: Android
---

Android应用开发的时候会经常使用到第三方SDK，例如地图、推送、支付以及分享等。而这些第三方服务SDK为了应用不被滥用都会有签名验证机制。我们通过上传Android应用的签名来在服务端配置以验证应用的合法性。本文将讲述下在Android Stduio中常用的签名操作。


### 默认KeyStore

#### 默认KeyStore存储路径

在Android Stduio中系统默认内置了一个签名文件`debug.keystore`，用于我们在debug下的默认App签名。如果没有在Gradle文件中特殊指定，那么Android Studio将自动使用默认的`debug.keystore`文件为项目App生成Debug版本的签名。


<!-- more-->

 * 在Mac/Linux系统中，`debug.keystore`文件默认储存在`~/.android/`路径下。

 * 在Windows系统中，`debug.keystore`文件将默认存储在`C:\Users\{USERNAME}\.android\`路径下。

#### 获取默认KeyStore `SHA-1`

知道了Android Stduio 默认的`debug.keystore`之后，下一步我们将是要获取其指纹信息，以便于在第三方服务配置中填入Debug指纹信息。

 - 在Linux/Mac系统中，打开终端并输入以下命令:

```bash
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android
```

 - 在Windows系统中，在CMD中输入以下命令:

```bat
keytool -list -v -keystore "%USERPROFILE%\.android\debug.keystore" -alias androiddebugkey -storepass android -keypass android
```

回车执行之后，你将会看到类似下面的`debug.keystore`输出提示:

```bash
别名: androiddebugkey
创建日期: 2015-11-18
条目类型: PrivateKeyEntry
证书链长度: 1
证书[1]:
所有者: CN=Android Debug, O=Android, C=US
发布者: CN=Android Debug, O=Android, C=US
序列号: b15af1
有效期开始日期: Wed Nov 18 07:18:45 CST 2015, 截止日期: Fri Nov 10 07:18:45 CST 2045
证书指纹:
         MD5: FE:A1:9C:02:71:A2:DA:F9:7F:1C:2B:61:D7:65:89:44
         SHA1: 01:DF:58:7D:04:3E:76:B5:92:98:37:0E:DD:70:61:01:70:F5:C9:8E
         SHA256: 84:18:44:C2:BD:AD:5D:A8:88:A1:96:EF:A6:27:86:0A:36:44:31:38:F2:5F:B6:4E:F1:10:EE:93:D6:22:CD:59
         签名算法名称: SHA256withRSA
         版本: 3

扩展:

```

我们将其中的证书指纹填入到第三方服务DEBUG配置中即可。当然了，有的时候出于这样或者那样的原因考虑，我们并不想使用系统默认的KeyStore或者就想自己生成一个新的KeyStore，Debug环境与Release环境都使用同一个来减少配置的麻烦。这个时候我们就需要创建一个新的KeyStore文件了。

### 创建新的KeyStore

我们使用JDK自带的Keytool命令行工具即可完成KeyStore密钥库文件的创建，此处需要说明的是，Android Stduio中自带的图形化界面`KeyStore`生成工具生成的`.jks`文件与Keytool生成的`.keystore`文件在使用上没有任何区别。

#### 生成KeyStore

在终端中键入以下命令:

```bash
keytool -genkey -v -keystore {FILENAME.keystore} -alias {ALIAS} -keyalg RSA -validity {DURATION}
```

 - `{FILENAME.keystore}` 为生成的KeyStore的文件名
 - `{ALIAS}` 为生成的KeyStore文件的别名
 - `{DURATION}` 为该KeyStore文件的过期时间

下面将以生成一个test.keystore文件为示例:

```bash
keytool -genkey -v -keystore test.keystore -alias test -keyalg RSA -validity 365
```

键入以上命令将生成一个以RSA算法加密的有效期365天的名为`test.keystore`的文件，该KeyStore文件的alias为 test。回车确认执行该命令之后，将会要求输入密钥库口令以及一些基本的信息，根据提示输入无误之后将会在当前终端所在目录生成指定的KeyStore文件。完整的示例如下所示:

```bash

$ keytool -genkey -v -keystore test.keystore -alias test -keyalg RSA -validity 365
输入密钥库口令:  android
再次输入新口令: android
您的名字与姓氏是什么?
  [Unknown]:  Doublemine
您的组织单位名称是什么?
  [Unknown]:  Test
您的组织名称是什么?
  [Unknown]:  Test
您所在的城市或区域名称是什么?
  [Unknown]:  Test
您所在的省/市/自治区名称是什么?
  [Unknown]:  Test
该单位的双字母国家/地区代码是什么?
  [Unknown]:  Test
CN=Doublemine, OU=Test, O=Test, L=Test, ST=Test, C=Test是否正确?
  [否]:  y

正在为以下对象生成 2,048 位RSA密钥对和自签名证书 (SHA256withRSA) (有效期为 365 天):
         CN=Doublemine, OU=Test, O=Test, L=Test, ST=Test, C=Test
输入 <test> 的密钥口令
        (如果和密钥库口令相同, 按回车):
[正在存储test.keystore]

```

这样我们就有了一个全新的KeyStore文件可以用于Android的App签名，有了KeyStore文件下一步当然就是获取我们生成的KeyStore文件的指纹信息咯~

#### 获取KeyStore指纹信息

与获取默认的`debug.keystore`文件指纹信息类似，我们在终端中键入以下命令:

```bash
 keytool -v -list -keystore test.keystore -alias test -keypass android -storepass android
```

即可获取到我们生成的KeyStore指纹信息，有的同学已经看出来了，只要将上述命令中的几个参数替换下，即可查看任意KeyStore的指纹信息:

```bash
 keytool -v -list -keystore {FILENAME.keystore} -alias {ALIAS} -keypass {KEYPASSWD} -storepass {STOREPASSWD}
```

 - `{FILENAME.keystore}`为keystore文件名
 - `{ALIAS}`为KeyStore的别名
 - `{KEYPASSWD}`为KeyStore的密钥口令
 - `{STOREPASSWD}`为KeyStore的密钥库口令

### 应用KeyStore

前面我们忙活了大半天生成了KeyStore文件，并查看其指纹信息。但是如果我们不使用到我们的项目中，毕竟还是不会对我们的项目生效的~我们还需要在Gradle脚本中对其进行配置，我们的项目才会应用其KeyStore文件。

其中我们有两种较为普遍的方式在项目中配置我们的KeyStore文件，第一种比较简单粗暴，直接在gradle构建脚本中写入KeyStore信息,第二种则将KeyStore信息配置在一个单独的配置文件中，在gradle构建时动态读取。

#### 签名信息写入Gradle脚本

在Android Stduio中打开主moudle的`build.gradle`文件,在其中的`android`闭包中键入如下内容:

```gradle
signingConfigs {
    release {
      keyAlias 'test'
      keyPassword 'android'
      storeFile file('./keystore/test.keystore')
      storePassword 'android'
    }
  }
```
其中声明的release闭包中包含了`keyAlias`、`keyPassword`、`storeFile`、`storePassword`四个Property。其中含义分别为:

 - `keyAlias` keystore的`alias`
 - `keyPassword` KeyStore的密钥口令
 - `storeFile`为KeyStore的文件存放路径，可以为相对或者绝对路径，此处使用的为相对路径
 - `storePassword`为KeyStore的密钥库口令

以上的Gradle DSL将会作用于我们的项目的Release版本，当我们在终端中个输入:

```bash
gradlew assembleRelease
```

项目将会使用我们上面定义的`test.keystore`密钥库文件签名打包项目为Release发布版。

同样，如果我们不想使用默认的`debug.keystore`签名项目的Debug版本，我们亦可以重新生成一个KeyStore文件或者使用Release版本的签名该文件，放入debug闭包中即可:

```gradle
signingConfigs {
    release {
      keyAlias 'test'
      keyPassword 'android'
      storeFile file('./keystore/test.keystore')
      storePassword 'android'
    }
    debug {
     keyAlias 'test'
      keyPassword 'android'
      storeFile file('./keystore/test.keystore')
      storePassword 'android'
    }
  }
```

#### 签名信息写入配置文件

细心的同学可能发现了，虽然上面的把签名信息写入gradle脚本中比较方便省事，但是却在密钥文件的密钥密码泄露问题，任何能够看到此Moudle的build.gradle脚本的人都可以拿到KeyStore文件及其对应的密钥口令，可能会导致一些安全风险。因此在一些开源项目或者比较敏感的项目中，可能会存在类似的gradle配置:

在主moudle的build.gradle脚本的android闭包中:

```gradle
applicationVariants.all {  
    if (project.hasProperty('keyAlias') && project.hasProperty('storeFile') &&
    project.hasProperty('storePassword') &&
    project.hasProperty('keyPassword')) {
  android.signingConfigs.release.keyAlias = keyAlias
  android.signingConfigs.release.storeFile = file(storeFile)
  android.signingConfigs.release.storePassword = storePassword
  android.signingConfigs.release.keyPassword = keyPassword
} else {
  android.buildTypes.release.signingConfig = null
}
}
```

其中`Variants`翻译中文为`变种`,`applicationVariants.all`属性含义为`app plugin`下所有的`Variant`的配置信息，可以将其看作为一个总览，可以方便的访问所有对象。
[相关延伸阅读Gradle Plugin User Guide](http://tools.android.com/tech-docs/new-build-system/user-guide)，我们在其中通过`project.hasProperty`读取项目中的配置，并将其动态的赋值给`signingConfigs.release`下的相关属性。

然后我们通过在`gradle.properties`或者其它项目中能够被gradle的文件中定义以上属性并赋值即可：

```gradle
storeFile=./keystore/test.keystore
storePassword=android
keyAlias=test
keyPassword=android
```

这样我们在项目团队协作时，将`gradle.properties`文件忽略即可。

Enjoy IT!