---
title: Spring Boot使用SpringFox自动生成Api Doc
date: 2017-06-30 10:56:01
tags: 
 - Swagger
 - Spring Boot
 - Gradle
 - Kotlin
---

{% note info %}在做Android开发的时候，对于Api接口的对接有着深刻的体会：后端通过Markdown或者Word写好Api文档，然后通过类似Samba或者Dropbox这样的服务与移动端实现文档共享。有的时候因为接口出了问题，中间还得来回修改对接，效率低下不说，要是后端手抖写错参数而没有意识到，移动端埋头一顿调试。。。说多了都是泪。{%endnote%}

为了避免同时维护代码和文档来保持两者之间的同步而带来的额外负担，同事推荐了[`ApiDoc`](http://apidocjs.com/)来生成文档，虽然生成的文档界面比较清爽然而前提是必须得按照规定的语法写上详细的注释，才能生成对应的文档，虽然写注释本身是一件好事，不过有能够自动生成的方法为啥不使用呢?

<!--more-->

-----------

与`Apidoc`类似，`Swagger`也是一个用来文档化Resetful Api的项目，不过开源社区的支持应该是所有类似项目中最为完善的，因此除了可以使用[Swagger Editor](https://github.com/swagger-api/swagger-editor)来编写Api文档之外，你还可以使用其它对应的自动化生成工具，以此来避免同时维护文档和代码的麻烦：

- [Springfox](https://github.com/springfox/springfox) 是为Spring而打造的自动化生成接口文档的其中一个Java实现
- [Django Reset Swagger](https://github.com/marcgibbons/django-rest-swagger) 则是为Django而打造的Python实现。

这篇文章将从头创建一个Spring Boot项目并使用Springfox来生成对应的接口文档，来说明使用Springfox是多么的简单。首先创建Spring Boot项目：



### 创建Spring Boot项目

#### Eclipse

如果你是使用Eclipse的话，那么：

![](https://camo.githubusercontent.com/8caa3693b4268c095c001089313d687f647d551a/687474703a2f2f696d67322e77696b69612e6e6f636f6f6b69652e6e65742f5f5f636232303133303831393134323932382f6361726466696768742f696d616765732f7468756d622f352f35352f476f2d686f6d652d796f7572652d6472756e6b2e6a70672f35303070782d476f2d686f6d652d796f7572652d6472756e6b2e6a7067)

#### IntelliJ IDEA

我们使用IDEA的`Spring initializr`向导来简化初始化创建项目，如图所示：

![](https://ws1.sinaimg.cn/large/694830ebgy1fh3qqzjjr9j21gu0w4td1.jpg)

点击下一步根据个人的喜好来配置喜欢的JVM语言和构建工具，此处我选择`Kotlin`和`Gradle`，一切都是为了爽：

![](https://ws1.sinaimg.cn/large/694830ebgy1fh3qv1abohj21gu0w4jum.jpg)

点击下一步选择需要集成的依赖项，此处我们简单演示下Resetful Api文档生成，所以选择Web即可，如图：

![](https://ws1.sinaimg.cn/large/694830ebgy1fh45g74sdaj21gu0w4agg.jpg)

点击Next直至完成。这样，我们就完成了Spring Boot项目的创建了。

-------------

### 添加Springfox依赖

编辑根目录下的`build.gradle`文件，修改以下内容：

```groovy
dependencies {
    .... /*some depends...*/
    compile "io.springfox:springfox-swagger2:$springfoxVersion"
    compile "io.springfox:springfox-swagger-ui:$springfoxVersion"

}

ext {
    springfoxVersion = '2.7.0'
}

```

### 配置Springfox

Springfox通过`Docket`对象来定义生成的Api的一些属性，因此我们来创建一个Configure类来专门做Springfox的配置。创建一个`Swagger2Configure.kt`文件，并添加以下内容：

```kotlin
@Configuration
@EnableSwagger2
class Swagger2Configure {

    @Bean fun petApi(): Docket {
        return Docket(DocumentationType.SWAGGER_2)
                .apiInfo(generateApiInfo()) /*定制swagger ui显示的版本信息*/
                .useDefaultResponseMessages(false)
                .select()
                /*移除默认的Error Controller*/
                .apis(Predicates.not(RequestHandlerSelectors.basePackage("org.springframework.boot")))
                .paths(PathSelectors.any())
                .build()
    }

    private fun generateApiInfo(): ApiInfo {
        return ApiInfoBuilder().title("Spring Boot Api Doc")
                .contact(Contact("Doublemine", "https://notes.wanghao.work", "doublemine.w@gmail.com"))
                .description("This is a sample api doc description")
                .build()
    }

}
```

上述示例只演示了最基本的配置，如果想查看完整的示例解释，请移步[Configuration explained](http://springfox.github.io/springfox/docs/current/#configuration-explained),至此，Springfox的配置就完成了。就是这么简单。

---------------

### 创建接口

我们创建一个简单的UserController来模拟获取用户信息，`UserController.kt`：

```kotlin
@RequestMapping("/user/")
@RestController
open class UserController : BaseController() {
    @RequestMapping(value = "/info/{id}", method = arrayOf(RequestMethod.GET))
    fun getUserInfoById(@PathVariable id: Int): User {
        return User(id, "小白", 35)
    }
}
```

对应的Model`User.kt`:

```kotlin
data class User(val id: Int, var name: String, var age: Int)
```

至此就完成了简单的接口，接着我们启动项目并访问http://localhost:8080/swagger-ui.html ，一切正常的话，你将会看到以下页面：

![](https://ws1.sinaimg.cn/large/694830ebgy1fh4gacpr8wj21sk0su0wr.jpg)

一般来说这样已经能够满足我们的基本需要了，如果还需要更为详细的文档，Springfox也提供的注解来简化配置过程，我们接下来稍微修改下`UserController.kt`：

```kotlin
@Api(tags = arrayOf("用户信息"))
@RequestMapping("/user/")
@RestController
open class UserController : BaseController() {
    @ApiOperation("获取用户信息", notes = "根据用户Id在来查询用户信息")
    @RequestMapping(value = "/info/{id}", method = arrayOf(RequestMethod.GET))
    fun getUserInfoById(@PathVariable id: Int): User {
        return User(id, "小白", 35)
    }
}
```

我们重启项目查看下：

![](https://ws1.sinaimg.cn/large/694830ebgy1fh4gfu3btij21qo0qi786.jpg)

可以发现文档添加了对应的中文，要查看全部可用的注解以及其作用，请移步官方文档：

- [ Support for documentation from property file lookup](http://springfox.github.io/springfox/docs/current/#support-for-documentation-from-property-file-lookup)
- [Swagger-Core Annotations](https://github.com/swagger-api/swagger-core/wiki/Annotations-1.5.X)

简单集成使用到这里👌咯，后续再写一写生成静态文档相关的内容吧。Just for Fun！