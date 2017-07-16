---
title: RxJava读书笔记(一)
# 设置日志缩略图，可以是外链，也可以是相对路径
thumbnailImage: http://res.cloudinary.com/dmfz9aun7/image/upload/v1453950569/reactive/Rx_Logo_S.png
categories: RxJava
tags: RxJava
---
[RxJava Essentials CN](https://www.gitbook.com/book/yuxingxin/rxjava-essentials-cn/details)

[ReactiveX](http://reactivex.io/documentation/observable.html)

[ReactiveX文档中文翻译](https://mcxiaoke.gitbooks.io/rxdocs/content/Intro.html)

## 介绍
### 概述
RxJava是[ReactiveX (Reactive Extensions)](https://reactivex.io/)的Java版本：
-  a library for composing asynchronous and event-based programs by using observable sequences.
一个构成异步的、基于事件的用可观察的序列的程序库。

关于ReactiveX的更多信息，查看[Introduction to ReactiveX](http://reactivex.io/intro.html)

### 轻量级
RxJava是个轻量级的库，以单独的jar形式，集中观察者模式和相关高阶函数
### 多语言的实现
RxJava支持Java6及以上的基于JVM的实现的语言如：[Groovy](https://github.com/ReactiveX/RxGroovy)、[Clojure](https://github.com/ReactiveX/RxClojure)、[JRuby](https://github.com/ReactiveX/RxJRuby)、[Kotlin](https://github.com/ReactiveX/RxKotlin)、[Scala](https://github.com/ReactiveX/RxScala)。

RxJava适用多语言环境不仅仅是Java或Scala。设计遵守每种JVM-based语言的风格。

### RxJava库
以下的扩展库都可以适用RxJava
- [Hystrix](https://github.com/Netflix/Hystrix/wiki/How-To-Use#wiki-Reactive-Execution) 针对分布式系统的延迟和容错库
- [Camel RX](http://camel.apache.org/rx.html)使用RxJava提供重用[Apache Camel components, protocols, transports and data formats](http://camel.apache.org/components.html)的简单方式。
- [rxjava-http-tail](https://github.com/myfreeweb/rxjava-http-tail) 
- [mod-rxvertx - Extension for VertX](https://github.com/vert-x/mod-rxvertx)
- [rxjava-jdbc](https://github.com/davidmoten/rxjava-jdbc)
- [rtree](https://github.com/davidmoten/rtree)