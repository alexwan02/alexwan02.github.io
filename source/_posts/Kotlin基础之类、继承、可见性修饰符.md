---
title: Kotlin基础之类、继承、可见性修饰符
date: 2017-07-03 15:48:46
thumbnailImage: https://pic1.zhimg.com/v2-30253c279faba2e77120862dd54d49d4_r.jpg
tags: kotlin
---
# 类与继承

## 类
在Kotlin中使用class声明类

```kotlin
class Invoice {
}
```
类声明由类名，类头部（指定类型参数，首要构造等）和用大括号包裹的类体组成。类头部和类体为可选；如果无类体，可省略花括号
```kotlin
class Empty
```

## 构造器
Kotlin的类可以有一个首要构造器和多个次要构造器。首要构造是类头部的一部分，跟在类名之后（参数类型可选）
```kotlin
class Person constructor(firstName: String) {
}
```
如果首要构造没有任何注解或可见修饰符，`constructor`关键字可省略。
```kotlin
class Person(firstName: String){
    
}
```
首要构造不能含有任何代码。初始代码可以放在初始块中，初始块使用`init`前缀修饰：
```kotlin
class Customer(name: String){
    init {
        logger.info("Customer initialized with value ${name}")
    }
}
```
> 首要构造的参数可以在init代码块中使用，也可以用来初始化类中声明的属性

```kotlin
class Customer(name: String) {
    val customerKey = name.toUpperCase()
}
```
事实上，Kotlin有个简明语法在首要构造器中初始话声明的属性
```kotlin
class Person(val firstName: String, val lastName: String, var age: Int) {
    // ...
}
```
与常规属性一样，声明在首要构造的属性可以是可变的（var）或只读（val）

如果构造器有注解或可见修饰符，在构则要加上`constructor`关键字。
```kotlin
class Customer public @Inject constructor(name: String) { ... }
```
更多查看[`可见性修饰符`](https://kotlinlang.org/docs/reference/visibility-modifiers.html#constructors)

## 次要构造器

类可以声明次要构造器，使用`custructor`前缀修饰
```kotlin
class Person {
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
```
如果类有首要构造，每个次要构造器都需要委托给首要构造或间接地通过其他次要构造器。使用`this`关键字完成委托。
```kotlin
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```
如果不是抽象类并且没有声明任何构造器，则生成一个无参首要构造。构造器可见性为public。如果不希望有public类型构造器，则需要声明一个空的非默认可见性的首要构造器
```kotlin
class DontCreateMe private constructor () {
}
```
> 在JVM上，如果首要构造所有参数都有默认值，编译器生成一个使用默认值的额外参数构造器。可以让kotlin和像`Jackson`或`JPA`这样创建类的实例的库一起使用。

    class Customer(val customerName: String = "")
    
## 创建类的实例
像常规函数一样调用构造器，来创建一个类的实例
```kotlin
val invoice = Invoice()

val customer = Customer("Joe Smith")

```
> kotlin 没有`new`关键字

查看关于创建[`嵌套类`](https://kotlinlang.org/docs/reference/nested-classes.html)的描述

## 类成员
类包含
- 构造器和初始块
- [函数](https://kotlinlang.org/docs/reference/functions.html)
- [属性](https://kotlinlang.org/docs/reference/properties.html)
- [嵌套和内部类](https://kotlinlang.org/docs/reference/nested-classes.html)
- [对象声明](https://kotlinlang.org/docs/reference/object-declarations.html)

## 继承
所有Kotlin类都有一个共通的超类Any。
```kotlin
class Example // Implicitly inherits from Any
```
`Any`不是`java.lang.Object`;除了`equals()`，`hashCode()`，`toString()`，没有其他的成员。详情查看[`Java协同`](https://kotlinlang.org/docs/reference/java-interop.html#object-methods)小节

在类头部的尾部来声明明确的父类类型
```kotlin
open class Base(p: Int)
class Derived(p: Int) : Base(p)
```
如果类有一个首要构造，则必须使用首要构造器初始化基类。

如果类没有首要构造器，则每个次要构造器使用`super`关键字初始化基类或委托给其他构造执行。
> 不同次要构造器可以调用不同基类构造

```kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)

    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```
类的`open`注解与Java的`final`意义相反：允许其他类继承本类。kotlin默认的所有类都是final类型，对应[`Effective Java`](http://www.oracle.com/technetwork/java/effectivejava-136174.html)中第17条：`要么为继承而设计，并提供文档说明，要么就禁止继承`。
## 复写方法
如上面说到，Kotlin 力求事情明确。与Java不同，Kotlin需要明确的可复写的成员（Koltin称为open）和复写
```kotlin
open class Base {
    open fun v() {}
    fun nv() {}
}
class Derived() : Base() {
    override fun v() {}
}
```
`Derived.v()`需要`override`，如果缺失，编译则给出抗议。如果函数没有`open`注解，如`Base.nv()`，则在子类中声明相同签名的方法，无论使用或不使用`override`注解都是非法的。在final类（没有使用`open`注解）中，禁止注解open成员。

标记为`override`的成员函数可以被子类复写。如果禁止复写，使用`final`关键字
```kotlin
open class AnotherDerived() : Base() {
    final override fun v() {}
}
```
## 复写属性
复写属性原理与复写方法相似；超类中声明的属性并在派生类中重新声明，必须使用`override`在前面标记，并且具有兼容的类型。每个声明的属性可以被有初始化操作或有`getter`方法的属性复写。
```kotlin
open class Foo {
    open val x: Int get { ... }
}

class Bar1 : Foo() {
    override val x: Int = ...
}
```
可以使用var属性来复写val属性，反过来不可。因为val属性本质上声明了getter方法，复写为`var`属性在派生类中声明了`setter`方法

> 可以在首要构造器中使用`override`关键字来声明属性

```kotlin
interface Foo {
    val count: Int
}

class Bar1(override val count: Int) : Foo

class Bar2 : Foo {
    override var count: Int = 0
}
```

## 复写规则
Koltin定义了实现继承的规则。如果类继承它直接超类的相同成员的多个实现，则必须复写这个方法并且提供自己实现。为了表示采用的继承实现父型，需要使用`super`修饰尖括号中的超类类型，如：`super<Base>`
```kotlin
open class A{
    open fun f(){ print("A") }
    fun a() { print("a") }
}

open class B{
    fun f(){ print("B") }
    funcb() { print("b") }
}

class C() : A() , B{
    // The compiler requires f() to be overridden:
    override fun f() {
        super<A>.f() // call to A.f()
        super<B>.f() // call to B.f()
    }
}
```
同时继承A和B，对于函数`a()`和函数`b()`没有问题，因为`C`只继承了这些函数一个实现。但是对于函数`f()`，有`C`的两个实现，因此必须要复写`f()`，并由`C`提供自己的显示来消除歧义。

## 抽象类
类和它一些成员可以声明为`abstract`。抽象成员在当前类没有具体实现。
> 不言而喻，抽象类或抽象函数无须使用open注解

可以使用抽象来复写非抽象的open函数

```kotlin
open class Base {
    open fun f() {}
}

abstract class Derived : Base() {
    override abstract fun f()
}
```

## 伴生对象
与Java，C++不同的是，Kotlin没有静态方法。在多数类中，建议使用包级函数替代。

如果需要写一个不用类的实例来调用的函数，但要通过访问类的内部信息（如：工厂方法），可以在类中写作为[`对象声明`](https://kotlinlang.org/docs/reference/object-declarations.html)成员。

更为特殊的情况，如果在类中声明[`伴生对象`](https://kotlinlang.org/docs/reference/object-declarations.html#companion-objects)，就可以只使用类的名称作为限定符来调用成员，就好像Java/C#中调用静态方法那样。

# 可见性修饰符

类、对象、接口、构造器、函数、属性和他们的`setter`都可以有可见性修饰符（Getters与属性相同的修饰符）。Kotlin有四种可见性修饰符：`private`，`protected`，`internal`和`public`。默认可见性为`public`。

## 包
函数，属性，和类，对象和接口都可以在上层中声明。如：直接在包中
```kotlin
// file name: example.kt
package foo

fun baz() {}
class Bar {}
```

- 如果没有指定任何可见性修饰符，默认为`public`，意味：声明随处可见
- 如果标记为`private`，则只在声明的文件中可见
- 如果标记为`internal`，则在相同[`模块`](https://kotlinlang.org/docs/reference/visibility-modifiers.html#modules)中可见
- 标记为`protected`，则不能为顶层访问
```kotlin
// file name: example.kt
package foo
private fun foo() {} // 在example.kt内部可访问
public var bar: Int = 5 // 在任何地方可见
 private set         // setter 方法只在example.kt中可见
internal val baz = 6 // 在相同module中可见
```
## 类与接口
对于类中声明的成员
- private：只在声明的类中可见（包括类所有的成员函数）
- protected：与private相似，但子类也可访问
- internal：相同模块的任何client都访问类的internal成员
- public：任何client都可访问类的public成员

> Java开发注意：在kotlin中，外部类无法访问内部类的私有成员

如果复写`protected`成员且没有显式指定可见修饰符，复写后的成员也具有`protected`可见性。
```kotlin
open class Outer {
    private val a = 1
    protected open val b = 2
    internal val c = 3
    val d = 4  // public by default
    
    protected class Nested {
        public val e: Int = 5
    }
}

class Subclass : Outer() {
    // a 不可见
    // b, c 和 d 可见
    // Nested 和 e 可见

    override val b = 5   // 'b' 为 protected
}

class Unrelated(o: Outer) {
    // o.a, o.b 不可见
    // o.c 和 o.d are 可见（相同模块）
    // Outer.Nested 不可见且 Nested::e 也不可见
}
```
## 构造器
使用下面的语法来指定类的首要构造器的可见性
> 需要显式添加`constructor`关键字

```kotlin
class C private constructor(a: Int) { ... }
```
类C的构造器为私有。默认的构造器都为`public`属性，也说明了只要可访问类，就能使用对应类的构造器。（如 internal类型的类，只在相同模块中访问）

## 局部变量
局部变量，函数和类没有可见性修饰符
## 模块
`internal`修饰符表示成员只能被相同模块访问。明确地说，Kotlin的模块是共同编译的一组文件：
- `IntelliJ IDEA`模块
- Maven或Gradle项目
- 使用Ant构建编译的文件