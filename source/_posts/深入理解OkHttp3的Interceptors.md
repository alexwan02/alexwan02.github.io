---
title: 深入理解OkHttp3的Interceptors（1）
date: 2017-04-23 10:14:09
tags: okhttp3
categories: okhttp3
---
> Version : okhttp:3.6.0

`Interceptors`是OkHttp3整个框架的核心，包含了请求监控、请求重写、调用重试等机制。它主要使用责任链模式，解决请求与请求处理之间的耦合。
![img](https://raw.githubusercontent.com/wiki/square/okhttp/interceptors@2x.png)

## 1.1 责任链模式
将接收对象放入链中，按照链中顺序让多个对象处理请求。请求者不用知道具体是由谁处理。解决请求与接收之间的耦合，提高灵活性。
责任链负责对请求参数的解析，所有的扩展都是针对链中节点进行扩展。

OkHttp3的Interceptors的责任链
![img](http://res.cloudinary.com/dmfz9aun7/image/upload/v1492487313/wechat/Interceptors_uml.jpg)
## 1.2 OkHttp3链式流程
![img](http://res.cloudinary.com/dmfz9aun7/image/upload/v1492482119/wechat/OkHttp3_Interceptor.jpg)
## 2.1 RetryAndFollowUpInterceptor
`RetryAndFollowUpInterceptor`用于尝试恢复失败和重定向的请求，最多支持跟踪20次重定向。创建streamAllocation维护请求的Connections、Streams、Calls，类似中介者模式，之后交给BridgeInterceptor节点处理请求。
```java
  @Override public Response intercept(Chain chain) throws IOException {
    ...
    // 创建用于协调Connections、Streams、Call三者关系的streamAllocation
    streamAllocation = new StreamAllocation(
        client.connectionPool(), createAddress(request.url()), callStackTrace);
    // 重定向次数
    int followUpCount = 0;
    Response priorResponse = null;
    while (true) {
      // 无限循环
      ...
      Response response = null;
      boolean releaseConnection = true;
      try {
        response = ((RealInterceptorChain) chain).proceed(request, streamAllocation, null, null);
        releaseConnection = false;
      } catch (RouteException e) {
        // 连接路由失败，请求未发送
        ...
        releaseConnection = false;
        continue;
      } catch (IOException e) {
        // 与服务端通信失败，请求已发送
        ...
        releaseConnection = false;
        continue;
      } finally {
        // 释放资源
        if (releaseConnection) {
          ...
        }
      }
      // 记录上一次的响应，一般出现在重定向情况时。
      ...
      // 判断是否是重定向的响应
      Request followUp = followUpRequest(response);
      ...
      if (followUp == null) {
        ...
        // 正常响应直接返回
        return response;
      }
      ...
      // 检查是否能够继续重定向操作
      ...
      request = followUp;
      priorResponse = response;
    }
  }
```
`RetryAndFollowUpInterceptor`处理下层链中节点返回的响应和抛出的异常。
依据返回的响应或抛出的异常，进行检查和恢复操作
1. 关闭已建立的Socket连接
2. OkHttpClient是否关闭重连，默认开启重连
3. 请求是否已发送并且请求体不可重读，不可重连
4. 出现的致命的异常：请求协议异常、证书验证异常等
5. 是否有下一跳可尝试的路由。

```java
  private boolean recover(IOException e, boolean requestSendStarted, Request userRequest) {
    // 关闭Socket
    streamAllocation.streamFailed(e);
    // 如果Application层禁止重连，则直接失败
    if (!client.retryOnConnectionFailure()) return false;
    // 是否可以再次发送请求
    if (requestSendStarted && userRequest.body() instanceof UnrepeatableRequestBody) return false;
    // 致命异常则不可恢复
    if (!isRecoverable(e, requestSendStarted)) return false;
    // 没有可以再次尝试的路由
    if (!streamAllocation.hasMoreRoutes()) return false;
    return true;
  }
```

## 2.2 BridgeInterceptor
`BridgeInterceptor`是应用层与网络层节点的桥接，补全应用层请求的头部信息，调用之后网络与缓存数据处理，最后将响应返回给上层。

```java
  @Override public Response intercept(Chain chain) throws IOException {
    // 重写请求头部，填充必要的头部信息
    ...
    // 添加 "Accept-Encoding: gzip" header ，可以压缩请求数据
    ...
     requestBuilder.header("Accept-Encoding", "gzip");
    ...

    // 配置Cookie和代理信息
    ...
    requestBuilder.header("Cookie", cookieHeader(cookies));
    ...
    requestBuilder.header("User-Agent", Version.userAgent());
    ...
    // 把新构建的请求向下传递处理
    Response networkResponse = chain.proceed(requestBuilder.build());
    // 处理下层节点返回的响应，响应可能是缓存或者网络数据
    HttpHeaders.receiveHeaders(cookieJar, userRequest.url(), networkResponse.headers());

    Response.Builder responseBuilder = networkResponse.newBuilder()
        .request(userRequest);
    // 响应数据解压
    ...
    return responseBuilder.build();
  }
```
## 2.3 CacheInterceptor
本地缓存和网络缓存，默认无缓存。
1. 读取本地缓存，根据请求缓存策略构建网络请求和缓存响应。
2. 按照请求缓存策略，返回缓存或传递给`ConnectInterceptor`执行下一步数据操作。
3. 处理返回的网络响应数据的缓存操作。
```java
  @Override public Response intercept(Chain chain) throws IOException {
    // 读取本地磁盘缓存
    Response cacheCandidate = cache != null
        ? cache.get(chain.request())
        : null;

    ...
    // 缓存策略
    CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(), cacheCandidate).get();
    Request networkRequest = strategy.networkRequest;
    Response cacheResponse = strategy.cacheResponse;

    // 如果缓存未命中，则舍弃缓存
    ...

    // 禁止网络请求且不存在缓存，返回504，请求失败
    if (networkRequest == null && cacheResponse == null) {
      ...
      return ...;
    }

    // 禁止网络请求，缓存存在，返回响应到上一级
    if (networkRequest == null) {
      // return Cache
      ...
    }

    Response networkResponse = null;
    try {
      networkResponse = chain.proceed(networkRequest);
    } finally {
      // 处理网络缓存失败时，释放缓存流
    }

    // 本地存在缓存则检查响应状态码是否为304
    if (cacheResponse != null) {
      if (networkResponse.code() == HTTP_NOT_MODIFIED) {
        // update cache
        ...
        return response;
      } else {
        // close cacheResponse 
      }
    }
   
    // 构建新响应
    Response response = networkResponse.newBuilder()
        .cacheResponse(stripBody(cacheResponse))
        .networkResponse(stripBody(networkResponse))
        .build();

    if (HttpHeaders.hasBody(response)) {
      // 响应缓存
      CacheRequest cacheRequest = maybeCache(response, networkResponse.request(), cache);
      response = cacheWritingResponse(cacheRequest, response);
    }

    return response;
  }
```
## 2.4 ConnectInterceptor
1. 创建网络读写流必要的HttpCodec，用于请求编码和网络响应解码处理。
2. 复用或建立Socket连接RealConnection，用于网络数据传输。
3. 网络数据流具体处理细节交给CallServerInterceptor节点。
```java
  @Override public Response intercept(Chain chain) throws IOException {
    ... 
    StreamAllocation streamAllocation = realChain.streamAllocation();
    ...
    // 复用或创建新的RealConnection，并创建新的HttpCodec处理网络读写流。
    HttpCodec httpCodec = streamAllocation.newStream(client, doExtensiveHealthChecks);
    RealConnection connection = streamAllocation.connection();
    // 交给CallServerInterceptor处理网络流。
    return realChain.proceed(request, streamAllocation, httpCodec, connection);
  }
```
## 2.5 CallServerInterceptor
创建服务端的网络调用，向服务端发送请求并获取响应。
```java
@Override public Response intercept(Chain chain) throws IOException {
    // 获取需要写请求和读响应的HttpCodec
    ... 
    long sentRequestMillis = System.currentTimeMillis();
    // 向服务端发送头部请求
    ...
    Response.Builder responseBuilder = null;
    // 如果含有支持的方法请求体，则需要向服务端发送请求体
    if (HttpMethod.permitsRequestBody(request.method()) && request.body() != null) {
      // 发送请求体
      ...
    }
    // 结束请求
    httpCodec.finishRequest();
    
    // 如果头部响应未读取，则读取头部响应
    if (responseBuilder == null) {
      responseBuilder = httpCodec.readResponseHeaders(false);
    }

    // 构建响应体
    ... 
    return response;
  }
```
## 2.6 自定义Interceptor
`OkHttp3`除了默认5种Interceptor实现，还可以添加`Application层`与`Network层`的interceptor。
```java
@OkHttpClient
    /*
    * 添加应用层Interceptor
    */
    public Builder addInterceptor(Interceptor interceptor) {
      interceptors.add(interceptor);
      return this;
    }
    
    /*
    * 添加网络层Interceptor
    */
    public List<Interceptor> networkInterceptors() {
      return networkInterceptors;
    }
```
1. 应用层Interceptors

`Application Interceptors`对每个请求只调用一次，处理`BridgeInterceptor`返回的响应。可以不调用Chain.proceed()或多次调用Chain.proceed()。
2. 网络层Interceptors

`Network Interceptors`在`ConnectInterceptor`与`CallServerInterceptor`之间调用。涉及到网络相关操作都会经过`Network Interceptors`，因此可以在缓存响应数据的之前对响应数据进行预处理。与`Application Interceptors`不同的是不支持短路处理，必须且只能调用一次`Chain.proceed()`方法，保证链式调用唯一。

一个简单的LogInterceptors
```java
class LoggingInterceptor implements Interceptor {
  @Override public Response intercept(Interceptor.Chain chain) throws IOException {
    Request request = chain.request();

    long t1 = System.nanoTime();
    logger.info(String.format("Sending request %s on %s%n%s",
        request.url(), chain.connection(), request.headers()));

    Response response = chain.proceed(request);

    long t2 = System.nanoTime();
    logger.info(String.format("Received response for %s in %.1fms%n%s",
        response.request().url(), (t2 - t1) / 1e6d, response.headers()));

    return response;
  }
}
```
