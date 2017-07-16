---
title: Android网络Client：Retrofit(一)
date: 2016-01-11 23:44:20
tags:
---
[Retrofit - A type-safe HTTP client for Android and Java](http://square.github.io/retrofit/)

[Retrofit分析-漂亮的解耦套路](http://www.jianshu.com/p/45cb536be2f4?utm_campaign=maleskine&utm_content=note&utm_medium=reader_share&utm_source=weixin&from=singlemessage&isappinstalled=0)
### Retrofit
[A type-safe HTTP client for Android and Java](http://square.github.io/retrofit/)
### 一、介绍
1.接口定义来定义HTTP API

	public interface GitHubService{
		@GET("users/{user}/repos")
		Call<List<Repo>> listRepos(@Path("user") String user);
	}

2.Retrofit对象生成GitHubService的实现

	Retrofit retorfit = new Retrofit.Builder()
			.baseUrl("https://api.github.com")
			.build();

	GitHubService service = retrofit.create(GitHubService.class);

3.创建GitHubService的实现后，发起同步或异步的网络请求，并返回Call对象

	Call<List<Repo>> repos = service.listRepos("octocat");

注解描述HTTP请求
	
	1.支持URL参数和查询参数动态可配
	2.支持对象转换成请求体（eg. JSON 、Protocol buffers）
	3.支持多种请求体和文件上传

### 二、API 声明
Annotations on the interface methods and its parameters indicate how a request will be handled.
以注解形式定义接口方法和对应的请求参数

1.请求方法

每个方法必须有一个对应的HTTP请求注解，提供请求的方法和对应的URL参数。五类注解：`GET`、`POST`、`PUT`、 `DELETE` 和`HEAD`。对应的资源URL请求地址在注解上指定。
	
	@GET("users/list")

你也可以在URL中指定查询参数

	@GET("users/list?sort=desc")

2.URL 处理

URL可以使用替换块动态改变方法的请求参数。替换参数为使用`{}`引用起来的字符串，并且要使用`@Path`注解的方式，用同样的字符串表示参数。

	@GET("group/{id}/users")
	Call<List<User>> groupList(@Path("id") int groupId);

也可以添加查询参数
	
	@GET("group/{id}/users")
	Call<List<User>> groupList(@Path("id") int groupId , @Query("sort") String sort);
复杂查询，参数可以为`Map`类型

	@GET("group/{id}/users")
	Call<List<User>> groupList(@Path("id") int groupId , @QueryMap Map<String , String> options);

3.请求体

`@Body`注解用来指定对象作为HTTP请求体

	@POST("users/new")
	Call<User> createUser(@Body User user);

使用`Retrofit`对象实例指定转换器来转换对象。
如果没有添加转换器，只能使用`RequestBody`

4.Form Encode编码数据And Multipart数据的请求
multipart/form-data : POST数据提交的方式，数据以multipart/form-data来编码。
Method 也可以发送form-encoded and multipart编码格式的数据
`@FormUrlEncoded`注解表示发送的数据格式为Form-encoded 格式。每个用注解`@Filed`表示的键值对包含指定键的名称和提供值的对象
	
	@FormUrlEncoded
	@POST("user/edit")
	Call<uSER> updateUser(@Filed("first_name") String first , @Field("last_name") String last);

`@Multipart`注解用来表示Method 为 Multipart请求。Parts使用`@Part`注解来表示。

	@Multipart
	@PUT("user/photo")
	Call<User> updateUser(@Part("photo") RequestBody photo , @Part("description") RequestBody description);

Multipart的Parts对象使用`Retrofit`的转换器或实现`RequestBody `来完成序列化

5.HEADER 请求操作

`@Headers`注解为方法指定静态headers
	
	@Headers("Cache-Control:max-age = 640000")
	@GET("widget/list")
	Call<List<Widget>> widgetList();

	@Headers({
			"Accept: application/vnd.github.v3.full+json" ,
			"User-Agent: Retrofit-Sample-App"
	})
	@GET("users/{username}")
	Call<User> getUser(@Path("username") String username);

  **注意：** Heads不必在每个方法上单独声明，可以在方法中引用相同名称的头部

`@Header`注解可以动态指定Header。与普通请求注解相同需要指定Header的参数，如果参数为null，则Header被忽略掉。另外，值和结果使用的时候会调用`toString`方法

	@GET("user")
	Call<User> getUser(@Header("Authorization") String authorization);

[OkHttp interceptor](https://github.com/square/okhttp/wiki/Interceptors)可以为需要添加Header的请求指定头部

6.同步和异步请求

`Call`可以执行同步或异步请求。每个实例只能使用一次，`clone()`方法可以创建一个新的`Call`。

**注意：** 在Android中，回调会在主线程中执行，在JVM中，回调会在调用Http Request请求的线程中执行。

### 三、Retrofit配置
`Retrofit` 通过定义的API Interfaces转变为回调对象。除了自带的默认的配置信息，开发者也可以自定义实现`Retrofit`的配置。

1.CONVERTERS 转换器
默认情况`Retrofit`只能把HTTP 体反序列化为[OkHttp](https://github.com/square/okhttp)的`ResponseBody`类型的对象，并且`@Body`只接受`ResponseBody`数据。
Converters 可以支持其他类型的数据。

(1).[Gson](https://github.com/google/gson): com.squareup.retrofit2:converter-gson

(2).[Jackson](http://wiki.fasterxml.com/JacksonHome): com.squareup.retrofit2:converter-jackson

(3).[Moshi](https://github.com/square/moshi/): com.squareup.retrofit2:converter-moshi

(4).[Protobuf](https://developers.google.com/protocol-buffers/): com.squareup.retrofit2:converter-protobuf
	
(5).[Wire](https://github.com/square/wire): com.squareup.retrofit2:converter-wire
	
(6).[Simple XML](http://simple.sourceforge.net/): com.squareup.retrofit2:converter-simplexml

例子：用基于Gson反序列化的`GsonConverterFactory `来生成`GitHubService`实例，

	Retrofit retrofit = new Retrofit.Builder()
			.baseUrl("https://api.github.com")
			.addConverterFactory(GsonConverterFactory.create())
			.build();

	GitHubService service = retrofit.create(GitHubService.class);

2.自定义CONVERTERS

如果要使用内容格式为如 `YAML`, `txt`等`Retrofit`不支持的或希望使用不同的库实现支持的内容的API，你可以创建继承`Converter.Factory`的自定义Converter，在建造自己的adapter时候指定一个自定义的Converter实例。

`Retrofit`地址 ： [Github](http://github.com/square/retrofit)

(1).Retrofit requires at minimum Java 7 or Android 2.3.

(2).proguard
	
	-dontwarn retrofit2.**
	-keep class retrofit2.** { *; }
	-keepattributes Signature
	-keepattributes Exceptions