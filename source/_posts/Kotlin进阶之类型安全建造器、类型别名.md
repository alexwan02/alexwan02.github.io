---
title: Kotlin进阶之类型安全建造器、类型别名
date: 2017-07-17 16:53:24
tags: kotlin
---
# 类型安全建造器
[建造器](概念)在*Groovy社区*非常流行，使用半声明方式来定义数据，有利于[生成XML](http://www.groovy-lang.org/processing-xml.html#_creating_xml)，[UI组件布局](http://www.groovy-lang.org/swing.html)，[描述3D场景](http://www.artima.com/weblogs/viewpost.jsp?thread=296081)等等。

多数情况，Kotlin允许类型检查建造器，比`Groovy`的动态类型实现更具有吸引力。


## 类型安全构建器范例
```kotlin
import com.example.html.*

fun result(args: Array<String>) = 
    html {
        head {
            title {+"XML encoding with Kotlin"}
        }
        
        body {
            h1 {+"XML encoding with Kotlin"}
            p  {+"this format can be used as an alternative markup to XML"}
            
            // 带有属性和文本内容元素
            a(href = "http://kotlinlang.org") {+"Kotlin"}
            
            // 混合内容
            p {
                +"This is some"
                b {+"mixed"}
                +"text. For more see the"
                a(href = "http://kotlinlang.org") {+"Kotlin"}
                +"project"
            }
            p {+"some text"}
            
            // 生成内容
            p {
                for (arg in args)
                    +arg
            }
        }
    }

```
可以[在线演示](https://try.kotlinlang.org/#/Examples/Longer%20examples/HTML%20Builder/HTML%20Builder.kt)这段代码

## 如何运转
这面介绍Kotlin类型安全建造器的实现机制。首先定义构建的Model，这里模拟HTML标签。`HTML`是描述`<html>`标签的类，定义了如像`<head>`和`<body>`的子标签。

回想如何定义Html
```kotlin
html{
  // ...
}
```

`html`实际上使用[lambda表达式](https://kotlinlang.org/docs/reference/lambdas.html)作为参数的函数。

函数的定义：
```kotlin
fun html(init: Html.() -> Unit): Html{
    val html = Html()
    html.init()
    return html
}
```
`html`的参数名为`init`的函数。`init`函数类型为`HTML.() -> Unit`：带有接收者的函数类型。也就意味着传入`HTML`类型实例的接收者给函数，并且可以在函数内部调用实例的成员。可以使用`this`关键字访问接收者

```kotlin
html{
    this.head{ /* ... */ }
    this.body{ /* ... */ }
}
```
（`head`和`body`都是`html`的成员函数）

`this`关键字可以省略。
```kotlin
html {
    head { /* ... */}
    body { /* ... */}
}
```

浏览上面定义的`html`函数体：创建新的`HTML`实例，调用参数传入的函数，进行初始化（在例子中调用`HTML`实例的`head`与`body`函数），之后返回`HTML`实例。这就是建造器真正执行的内容。

`HTML`类中定义的`head`和`body`函数与`html`相似。唯一不同的是它们都将建造器实例添加到`HTML`实例的子集。

```kotlin
fun head(init: Head.() -> Unit): Head{
    val head = Head()
    head.init()
    children.add(head)
    return head
}

fun body(init: Body.() -> Unit): Body{
    val body = Body()
    body.init()
    children.add(body)
    return head
}
```
实际上两个函数做相同事情，所以使用泛型函数`initTag`进行合并：
```kotlin
protected fun <T: Element> initTag(tag: T , init: T.() -> Unit): T{
    tag.init()
    children.add(tag)
    return tag
}
```
函数简化后：
```kotlin
fun head(init: Head.() -> Unit ) = initTag(Head() , init)
fun body(init: Body.() -> Unit ) = initTag(Body() , init)
```

之后可以使用它们来创建`<head>`和`<body>`标签了

那么如何添加文本内容到标签体中，就像开头介绍的那样。
```kotlin
html{
    head {
        title {+"XML encoding with Kotlin"}
    }
    // ...
}
```
因此只要在标签体重添加字符串，字符串前的`+`符号表示`unaryPlus()`操作符对应的函数。`unaryPlus()`是`TagWithText`抽象类（`Title`的父类）定义扩展成员函数。

```kotlin
fun String.unaryPlus(){
    children.add(TextElement(this))
}
```
所以前缀`+`实际上将`String`包裹到`TextElement`中，并添加`children`集合中，称为标签树合适的部分。

定义在包`com.example.html`中的这些函数在上面建造器例子定义导入。在小节最后给出了包定义的全部内容。

## 作用域控制：@DslMarker（Kotlin 1.1）
当使用DSL（Domain Specific Language特定领域语言）时，可能会遇到一个问题就是调用太多函数。调用所有Lambda表达式中隐式的接收者可能会有获取结果不一致问题，比如在另外一个`head`中的`head`标签
```kotlin
html{
    head {
        head {} // 禁止这样使用
    }
    
    // ...
}
```
在这个例子中，只有最近隐式接收者成员`this@head`必须为可用。`head()`是外部接收者`this@html`的成员，必须要合法调用。

为了解决这个问题，Kotlin 1.1引入了控制接收者作用域的特殊机制。

为了是编译开启作用域控制，需要使用相同注解标记去注解DSL使用的接收者类型。如，对于HTML建造器，声明`@HTMLTagMarker`注解
```kotlin
@DslMarker
annotation class HTMLTagMarker
```
使用`@DslMarker`注解的注解类被称为`DSL标记`

在DSL中，所有标签类都继承`Tag`类，只使用`@HtmlTagMarker`注解超类，编译器就会把所有继承的类当做一注解
```kotlin
@HtmlTagMarker
abstract class Tag(val name: String) { ... }
```

因此无须使用`@HtmlTagMarker`注解`Head`和`HTML`类

```kotlin
class HTML() : Tag("html"){ ... }
class Head() : Tag("head") { ... }
```

添加注解后，Kotlin编译就会知道哪个隐式接收者是相同DSL的一部分，只允许调用最近接收者的成员：

```kotlin
html{
    head {
        head {} // 错误：外部接收者成员
    }
    
    // ...
}
```

> 仍可以调用外部接收者的成员，但需要显式指定接收者

```kotlin
html{
    head {
        this@html.head{ } // 可以
    }
    // ...
}
```

## 包com.example.html函数完整定义
下面是包`com.example.html`完整定义，构建一个HTML树，大量使用了[扩展函数](https://kotlinlang.org/docs/reference/extensions.html)和[带有接收者的Lambda](https://kotlinlang.org/docs/reference/lambdas.html#function-literals-with-receiver)。

> @DslMarker注解只在Kotlin 1.1之后可用

```kotlin
package com.example.html

interface Element {
    fun render(builder: StringBuilder, indent: String)
}

class TextElement(val text: String) : Element {
    override fun render(builder: StringBuilder, indent: String) {
        builder.append("$indent$text\n")
    }
}

@DslMarker
annotation class HtmlTagMarker

@HtmlTagMarker
abstract class Tag(val name: String) : Element {
    val children = arrayListOf<Element>()
    val attributes = hashMapOf<String, String>()

    protected fun <T : Element> initTag(tag: T, init: T.() -> Unit): T {
        tag.init()
        children.add(tag)
        return tag
    }

    override fun render(builder: StringBuilder, indent: String) {
        builder.append("$indent<$name${renderAttributes()}>\n")
        for (c in children) {
            c.render(builder, indent + "  ")
        }
        builder.append("$indent</$name>\n")
    }

    private fun renderAttributes(): String {
        val builder = StringBuilder()
        for ((attr, value) in attributes) {
            builder.append(" $attr=\"$value\"")
        }
        return builder.toString()
    }

    override fun toString(): String {
        val builder = StringBuilder()
        render(builder, "")
        return builder.toString()
    }
}

abstract class TagWithText(name: String) : Tag(name) {
    operator fun String.unaryPlus() {
        children.add(TextElement(this))
    }
}

class HTML : TagWithText("html") {
    fun head(init: Head.() -> Unit) = initTag(Head(), init)

    fun body(init: Body.() -> Unit) = initTag(Body(), init)
}

class Head : TagWithText("head") {
    fun title(init: Title.() -> Unit) = initTag(Title(), init)
}

class Title : TagWithText("title")

abstract class BodyTag(name: String) : TagWithText(name) {
    fun b(init: B.() -> Unit) = initTag(B(), init)
    fun p(init: P.() -> Unit) = initTag(P(), init)
    fun h1(init: H1.() -> Unit) = initTag(H1(), init)
    fun a(href: String, init: A.() -> Unit) {
        val a = initTag(A(), init)
        a.href = href
    }
}

class Body : BodyTag("body")
class B : BodyTag("b")
class P : BodyTag("p")
class H1 : BodyTag("h1")

class A : BodyTag("a") {
    var href: String
        get() = attributes["href"]!!
        set(value) {
            attributes["href"] = value
        }
}

fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()
    html.init()
    return html
}
```

# 类型别名
类型别名可以为已有类型提供别名。在类型名过长时，可以引入一个不同的短名使用。

```kotlin
typealias NodeSet = Set<Network.Node>

typealias FileTable<K> = MutableMap<K, MutableList<File>>
```

函数类型的别名
```kotlin
typealias MyHandler = (Int, String, Any) -> Unit

typealias Predicate<T> = (T) -> Boolean
```

内部类和嵌套类别名
```kotlin
class A {
    inner class Inner
}
class B {
    inner class Inner
}

typealias AInner = A.Inner
typealias BInner = B.Inner
```

类型别名不会引入新类型。等价于对应的基本类型。当添加`typealias Predicate<T>`并使用`Predicate<Int>`时，Kotlin编译器会将它扩展为`(Int) -> Boolean`，所以需要泛型函数类型时，可以传入自己定义类型变量。
```kotlin
typealias Predicate<T> = (T) -> Boolean

fun foo(p: Predicate<Int>) = p(42)

fun main(args: Array<String>) {
    val f: (Int) -> Boolean = { it > 0 }
    println(foo(f)) // 输出 "true"

    val p: Predicate<Int> = { it > 0 }
    println(listOf(1, -2).filter(p)) // 输出 "[1]"
}
```