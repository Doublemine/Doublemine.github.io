---
title: Gradle、Maven项目相互转换
tags:
  - Gradle
  - Maven
date: 2017-08-21 23:01:21
---


  {% note info%}
  在开发Android项目的时候，使用的是`Gradle`构建工具，喜欢它的灵活和方便，在转向Java后端开发的时候更多时候使用的是`Maven`构建工具，然而看着漫天的尖括号，心里实在是难受。虽然只是一个构建工具，本着折腾的心，我还是更认可和看好`Gradle`。然而很多时候你的队友并没有习惯去使用或者快速熟悉`Gradle`构建工具，那么这个时候就需要将`Gradle`项目转换为Maven项目了，或者将Maven项目转换为`Gradle`项目了。
  {%endnote%}

<!--more-->

### 安装Gradle／Maven

首先是安装构建工具，这个没啥好说的。

##### Windows

打开Powershell或者Cmder执行以下命令完成安装：

```powershell
choco install gradle
choco install maven
```

`choco`为windows下的一款包管理工具，可以方便安装管理配置一些常见的软件包，如果你没有安装`choco`的话，请移步：https://chocolatey.org/



#### Mac

打开Terminal，执行以下命令安装：

```shell
brew install gradle
brew install maven
```

### Maven to Gradle

需要特别说明的是，`Gradle`对`Maven`的支持是比较完善的，因此，转换也是非常的简单，在`pom.xml`文件所在的目录下执行：

```shell
gradle init     # 根据pom.xml内容生成对应的gradle配置
gradle build    # 开启gradle构建
```

-------

### Gradle to Maven

`Gradle`项目转`Maven`项目需要借助一个Gradle插件，在项目的`module`的`build.gradle`文件中加入以下配置即可：

```groovy
apply plugin: 'maven'
```

通过双击`Idea`的Gradle Tasks GUI:

![](https://ws1.sinaimg.cn/large/694830ebgy1fj5qcx3csoj20j20ee0ua.jpg)

或者执行命令来完成转换:

```shell
gradle install
```

完成之后，将会在当前Module项目的`build`目录下的`poms`文件夹下生成`pom-default.xml`，将其拷贝到项目的根目录下即可。

![](https://ws1.sinaimg.cn/large/694830ebgy1fj5pn78lbhj20ik0g4jsk.jpg)

- 参考文档：https://guides.gradle.org/migrating-from-maven/

---------------------

通过实际测试，这样的生成的`pom-default.xml`文件是不能用于直接`maven`构建的，因为生成的`pom-default.xml`文件中的`groupId`还需要我们手动指定下。这样显然是不清真的，于是我们可以在`build.gradle`文件中将其事先定义好，这样生成的pom文件就不用我们再手动更改了：

![](https://ws1.sinaimg.cn/large/694830ebgy1fj5pvzyw2xj20ni0dign1.jpg)

然而这样我们还是觉得麻烦，毕竟需要手动复制到项目根目录，再重新命名。我们还可以通过Hook Gradle中Maven插件的`install`Task来完成自动的复制和命名,编辑`build.gradle`:

```groovy
task convert2Maven {
    doLast {
        file("$buildDir/poms/pom-default.xml").renameTo(file("$rootDir/pom.xml"))
    }
}
install.dependsOn(convert2Maven)
```

此时，再执行`gradle install`这个task就可以看到gradle已经自动为我们在项目的根目录下生成好了`pom.xml`文件啦。