---
title: Kotlin进阶之反射
date: 2017-07-17 09:38:51
tags: kotlin
---
# Reflection
反射是指计算机程序在运行时（Run time）可以访问、检测和修改它本身状态或行为的一种能力。Kotlin使函数和属性成为语言中`头等公民`，且以近似函数式或响应式方式内省属性和函数（如运行时属性名或类型；函数名或类型）
> 在Java平台上，需要使用反射特性的运行时组件为独立的`JAR`文件（`kotlin-reflect.jar`），目的是降低不需要使用反射的应用包的大小。如果要使用反射，首先需要保证项目中已添加相应的`.jar`文件。

## 类引用
获取运行时类的引用时反射最基本的特性。使用class字面量语法来获取对静态已知Kotlin类引用。

```kotlin
val c = MyClass::class
```
引用为[KClass](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html)类型的值

> Kotlin类引用与Java类引用不一样，获取Java类引用，使用`KClass`实例的`.java`属性
## 约束类引用
将对象作为`::class`语法的接收者，来引用指定对象的类。
```kotlin
val widget: Widget = ...
assert(widget is GoodWidget) { "Bad widget: ${widget::class.qualifiedName}" }
```
虽然接收者表达式的类型（`Widget`），但可获取确切的对象类引用（如：`GoodWidget`或`BadWidget`）。

## 函数引用
假设声明了具名函数
```kotlin
fun isOdd(x: Int) = x % 2 != 0
```
除了可以直接调用`isOdd(5)`，还可以使用`::`操作符将函数作为值传给另外一个函数。
```kotlin
val numbers = listOf(1, 2, 3)
println(numbers.filter(::isOdd)) // 输出[1 , 3]
```
这里`::isOdd`是函数类型（`(Int) -> Boolean`）的值

如果类型可以从上下文中推断出，`::`操作以重载的函数方式使用
```kotlin
fun isOdd(x: Int) = x % 2 != 0
fun isOdd(s: String) = s == "brillig" || s == "slithy" || s == "tove"
val numbers = listOf(1, 2, 3)
println(numbers.filter(::isOdd)) // 指isOdd(x: Int)函数
```
或者，以把方法存储到指定明确类型的变量中方式提供必要上下文信息。

```kotlin
val predicate: (String) -> Boolean = ::isOdd // 指向isOdd(x: String)
```

如果要使用类成员或扩展函数，则需要对函数进行修饰，如`String::toCharArray`表示`String`类型的扩展函数：`String.() -> CharArray`
### 范例：函数组合
假设有下面函数
```kotlin
fun <A, B, C> compose(f: (B) -> C, g: (A) -> B): (A) -> C {
    return { x -> f(g(x)) }
}
```

返回两个传入两个函数的组合：`compose(f, g) = f(g(*))`。 应用：

```kotlin
fun length(s: String) = s.length

val oddLength = compose(::isOdd, ::length)
val strings = listOf("a", "ab", "abc")

println(strings.filter(oddLength))  // 输出:"[a, abc]"
```
## 属性引用
Kotlin同样使用`::`操作符，访问Kotlin中属性。
```kotlin
var x = 1

fun main(args: Array<String>) {
    println(::x.get()) // 输出 "1"
    ::x.set(2)
    println(x)         // 输出 "2"
}
```
`::x`表达式赋值给`KProperty<Int>`类型的属性对象，可以使用`get`方法获取它的值，或者使用`name`属性检索属性名。具体信息查看[KProperty类文档](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-property/index.html)

对于可变属性（如：`var y = 1`），`::y`返回[KMutableProperty<Int>](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-mutable-property/index.html)类型的值，含有`set`方法。

属性引用可用在不需要参数的函数出
```kotlin
val strs = listOf("a", "bc", "def")
println(strs.map(String::length)) // 输出[1, 2, 3]
```

访问类成员属性
```kotlin
class A(val p: Int)

fun main(args: Array<String>){
    
    val prop = A::p
    println(prop.get(A(1))
}

```

对于扩展属性
```kotlin
val String.lastChar: Char
    get() = this[length - 1]
    
fun main(args: Array<String>){
    println(String::latChar.get("abc"))  // 输出 "c"
}
```
### 与Java反射交互
在Java平台上，标准库包含与Java反射对象相互映射的反射类（查看包`kotlin.reflect.jvm`）。如：查找backing字段或充当Kotlin属性的getter函数的Java方法。
```kotlin
import kotlin.reflect.jvm.*

class A(val p: Int)

fun main(args: Array<String>){
    println(A::p.javaGetter) // 输出 "public final int A.getP()"
    println(A::p.javaField)  // 输出 "private final int A.p"
}
```
使用`.kotlin`扩展属性来获取对应Java类的Kotlin类。
```kotlin
fun getKClass(o: Any): KClass<Any> = o.javaClass.kotlin
```

## 构造器引用
Koltin可以像函数和属性一样来引用构造器，可以用在与构造器参数相同的函数类型对象处，并返回合适类型的对象。

使用`::`操作符和类名引用构造器。

下面的函数为使用无参函数作为函数参数，返回类型为`Foo`
```kotlin
class Foo

fun funtion(factory: () -> Foo){
    val x: Foo = factory()
}
```

使用`::Foo`（Foo类的无参构造器），则可以这样调用：
```kotlin
function(::Foo)
```

## 约束函数与属性引用（Kotlin 1.1）

引用可以指向特殊对象的实例方法
```kotlin
val numberRegex = "\\d+".toRegex();
println(numberRegex.matches("29"))  // 输出"true"

val isNumber = numberRegex::matches
println(isNumber("29"))   // 输出"true"
```

与直接调用`matches`相反，存储对函数的引用。引用与接收者绑定，可以直接在函数表达式出调用或使用。
```kotlin
val strings = listOf("abc" , "124" , "a70")
println(strings.filter(numberRegex::matches)) // 输出"[124]"
```
比较约束函数与对应未约束应用，约束引用有自己绑定的`接收者`，所以接收者类型不再是参数。

```kotlin
val isNumber: (CharSequence) -> Boolean = numberRegex::matches

val matches: (Regex, CharSequence) -> Boolean = Regex::matches
```
属性引用也可以约束
```kotlin
val prop = "abc"::length
println(prop.get()) // 输出"3"
```