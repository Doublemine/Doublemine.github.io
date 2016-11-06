title: RxJava、Retrofit接收Error Response Body
isupdate: false
date: 2016-07-02 22:10:53
updatetime:
tags: 
 - Android
 - Android Studio
 - RxJava
 - Retrofit
categories: Android
---

`RxJava`配合`Retrofit`能够大大简化Android项目中的网络请求代码量，使得逻辑更清晰，当然也可能会遇到一些问题。下面给出一种问题的解决方案。



### 需求

一个基本的RxJava配合Retrofit以及Lambda的网络调用看起来像这个样子的:

```java

 Subscription subscription = mApi.getSimpleApi()
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(response -> {

         //do Something

        }, throwable -> {

        //Ops Error

        });

```

当`Retrofit`中的网络请求返回码状态码为`200`时，执行`do Something`中的逻辑处理正常的
业务流程，但是当服务器返回状态码为`非200`时，将会执行`Ops Error`中的业务流程而不会
执行`do Something`中的业务逻辑。

<!-- more -->

这样本没有什么问题，一般我们会在错误处理逻辑中在UI中给出错误提示，像这样:

```java
Log.e("Ops", "Error:" + throwable.getMessage());
```

但是这样的话我们只能获取到一服务器的错误响应码以及对应的简短的响应码错误说明，一般情况下我们服务器
都会包装错误信息为一个JSON，客户端解析错误信息必要的时候动态展示在UI上以提示用户。如果我们要拿到这样的JSON，使用`throwable.getMessage()`这样做显然是不行的。是不是使用RxJava配合Retrofit只能拿到这样的错误Throwable信息呢？

显然不是的，其实服务器返回的错误信息`非200`响应码的`Response Body`JSON对象包含在这个`throwable`对象中，我们可以这样将其解析出来:

```java
throwable -> {
          if(throwable instanceof HttpException){
            HttpException httpException= (HttpException) throwable;
            try {
              String errorBody= httpException.response().errorBody().string();

		//TODO: parse To JSON Obj

            } catch (IOException e) {
              e.printStackTrace();
            }
          }
	 //Ops Print throwable

        });
```

在`Parse to JSON Obj`中将`errorBody`解析为JSON对象进行相应的处理即可。

然而，你不能让我每个地方都加上这样的一段代码吧，既然我们使用的是`RxJava`，我们可以让这种处理稍微看起来优雅点。以下以`Jackson`为例:

#### 自定义Action1

由于`RxJava`的错误异常处理接受一个参数，并且没有返回值，因此我们可以定义一个`Action1`来替代默认的Error Action:

```java
public abstract class ErrorAction implements Action1<Throwable> {

  @Override public void call(Throwable throwable) {
    call(ErrorMessage.handle(throwable));
  }

  public abstract void call(ErrorMessage error);
}
```


其中`ErrorMessage`为我们定义好的错误消息Model:

#### 定义Throwable Handle

```java
@JsonIgnoreProperties(ignoreUnknown = true) public class ResponseError {
  private final static int ERROR_CODE_IO_ERROR = 2072;
  private final static int ERROR_CODE_UN_KNOW = 2073;
  @JsonProperty("status_code") public int statusCode;
  public String message;

  public void setStatusCode(int statusCode) {
    this.statusCode = statusCode;
  }

  public void setMessage(String message) {
    this.message = message;
  }

  public static ResponseError handle(Throwable throwable) {
    ResponseError responseError = null;
    if (throwable instanceof HttpException) {
      HttpException exception = (HttpException) throwable;
      try {
        responseError = new ObjectMapper().readValue(exception.response().errorBody().string(),
            ResponseError.class);
      } catch (IOException e) {
        responseError = new ResponseError();
        responseError.setMessage(e.getLocalizedMessage());
        responseError.setStatusCode(ERROR_CODE_IO_ERROR);
      }
    } else {
      responseError = new ResponseError();
      responseError.setMessage(throwable.getMessage());
      responseError.setStatusCode(ERROR_CODE_UN_KNOW);
    }
    return responseError;
  }
}
```

这样，我们就完成了一个自定义`Action1`了，接下来我们便可以这样使用了:

```java

Subscription subscription = mApi.getSimpleApi()
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(response -> {

         //do Something

        },  new ErrorAction() {
              @Override public void call(ErrorMessage msg) {
			//Do Error               

              }
            });
```

其中在`Do Error`中拿到`ErrorMessage`对象，进行相应的对象操作即可~

Enjoy IT!





