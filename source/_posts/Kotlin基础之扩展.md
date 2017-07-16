---
title: Kotlin基础之扩展
date: 2017-07-06 10:26:45
thumbnailImage: https://pic1.zhimg.com/v2-30253c279faba2e77120862dd54d49d4_r.jpg
tags: kotlin
---
# 扩展
Koltin提供与C#和Gosu类似的扩展类的功能，无须继承类或类似装饰者的设计模式。通过称为`扩展`的特殊声明实现。

支持扩展函数和扩展属性

## 扩展函数
使用**接收者类型**在函数名前声明扩展函数（接收者类型为要扩展的类型）。

为`MutableList<Int>`添加`swap`函数
```kotlin
fun MutableList<Int>.swap(index1: Int , index2: Int){
    val tmp = this[index1] // 'this' 对应list
    this[index1] = this[index2]
    this[index2] = tmp
}
```
扩展函数中的`this`关键字对应`接收者对象`(在`.`之前的对象)。这样，就可以调用`MutableList<Int>`的`swap()`函数了。
```kotlin
val l = mutableListOf(1, 2, 3)
l.swap(0, 2) // 'this' inside 'swap()' will hold the value of 'l'
```
函数适应于任意`MutableList<T>`，因此也可以这样做：
```kotlin
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this' 对应list
    this[index1] = this[index2]
    this[index2] = tmp
}

```

在函数名前声明泛型类型，参考[函数泛型](https://kotlinlang.org/docs/reference/generics.html)

## 静态处理扩展
扩展实际不会修改扩展的类型。通过定义扩展，无须向类中插入新的成员，只要使用`点表达式`就可以给这个类型创建新的可调用函数。

需要强调的是，扩展函数是静态分发，也就是说，扩展函数不是接收者类型虚拟的。意味着扩展函数由表达式类型调用时函数来决定，而不是由运行时的运行结果类型决定是否被调用。如：
```kotlin
open class C

class D: C()

fun C.foo() = "c"

fun D.foo() = "d"

fun printFoo(c: C) {
    println(c.foo())
}

printFoo(D())

```
例子中输出结果为"c"，所以扩展函数只被声明时类调用。

如果类有成员函数与类的扩展函数有相同的名称和参数，则类成员函数具有较高的优先级。如：
```kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo() { println("extension") }
```
如果调用C类型的`c.foo()`，会输出"member"，而非"extension"。

然而，如果有相同名称，但是签名不同（参数不同），则扩展函数会复写成员函数。

```kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo(i: Int) { println("extension") }
```
调用`C().foo(1)`则会输出"extension"

## 可为空的接收者
扩展可用可为空的接收者定义。这样的扩展可被空值的对象变量调用，可在代码体中进行`this == null` 检查。所以Kotlin允许调用toString函数，而不用进行非空检查。
```kotlin
fun Any?.toString(): String {
    if (this == null) return "null"
    // after the null check, 'this' is autocast to a non-null type, so the toString() below
    // resolves to the member function of the Any class
    return toString()
}
```

## 扩展属性
与扩展函数相同，Kotlin还支持扩展属性
```kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

> 因为扩展没有向类中插入成员，因此扩展属性没有有效方法来拥有一个`backing`字段。这也就解释了对于扩展属性，初始化是不允许的，只能通过显性提供getter和setter方法来定义

```kotlin
val Foo.bar = 1 // 错。不允许为扩展属性进行初始化。
```

## 伴生对象
如果类中定义伴生对象，也可以为伴生对象定义扩展函数和扩展属性
```kotlin
class MyClass{
    companion object{ } // 称为伴生
}

fun MyClass.Companion.foo(){
    // ..
}
```
与伴生对象常规成员类似，可以只使用类名作为限定符来调用方法。
```kotlin
MyClass.foo()
```

## 扩展作用域
多数情况下在顶层定义扩展，如在包的根目录中
```kotlin
package foo.bar
 
fun Baz.goo() { ... } 
```
在声明的包外使用扩展，需要在调用出导入：
```kotlin
package com.example.usage
import foo.bar.goo  // 导入名称为"goo"的所有扩展
import foo.bar.*    // 导入foo.bar中所有扩展

fun usage(baz: Baz){
    baz.goo()
}
```
查看[Imports](https://kotlinlang.org/docs/reference/packages.html#imports)更多信息。

## 声明扩展作为成员
在一个类可以为另外一个类声明扩展，在这样扩展中，会有多个隐性接收者对象成员，无须修饰进行访问。声明扩展的类的实例被称为`分发接收者`，扩展方法对应的接收者类型实例被称为`扩展接收者`。
```kotlin
class D{
    fun bar() { ... }
}

class C{
    fun baz() { ... }
    
    fun D.foo() {
        bar()   // calls D.bar
        baz()   // calls C.baz
    }
    
    fun caller(d: D) {
        d.foo()   // call the extension function
    }
}
```
如果`分发接收者`和`扩展接收者`成员命名冲突，优先使用扩展接收者的成员。可以使用[限定符](https://kotlinlang.org/docs/reference/this-expressions.html#qualified)语法引用分发接收者成员。
```kotlin
class C{
    fun D.foo(){
        toString() // 调用D.toString()
        this@C.toString() // 调用C.toString()
    }
}
```
声明为成员的扩展，可声明为`open`，被子类复写。意味着对于分发接收者类型，函数分发是虚的；对于扩展接收者类型，函数分发是静态的。
```kotlin
open class D{
    
}

class D1 : D(){
    
}

open class C {
    open fun D.foo(){
        println("D.foo in C")
    }
    
    open fun D1.foo(){
        println("D1.foo in C")
    }
    
    fun caller(d: D){
        d.foo() // 调用扩展函数
    }
}

class C1 : C(){
    override fun D.foo(){
        println("D.foo in C1")
    }
    
     override fun D1.foo() {
        println("D1.foo in C1")
    }
    
}

C().caller(D()) // print "D.foo in C"
C1().caller(D()) // prints "D.foo in C1" - 分发接收者是虚拟处理的
C().caller(D1()) // prints "D.foo in C" - 扩展接收者是静态处理的

```
## 动机
在Java中，我们常用`*Utils`命名的类：`FileUtils`、`StringUtils`等等。`java.util.Collections`也一样。这些Utils类最让人讨厌的部分就是使用的代码像这样：
```java
// Java
Collections.swap(list, Collections.binarySearch(list, Collections.max(otherList)), Collections.max(list))
```
静态导入调用方法，省略类名。
```kotlin
// Java
swap(list, binarySearch(list, max(otherList)), max(list))
```
这样会有点好处，但是对于IDE的代码补全的帮助很少或没有帮助。如果能够像下面那样做，那么会更好：
```kotlin
// Java
list.swap(list.binarySearch(otherList.max()), list.max())
```
但是又不想实现list内部所有可能的方法，这也是扩展函数的有用的地方。