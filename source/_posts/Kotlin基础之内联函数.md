---
title: Kotlin基础之内联函数
date: 2017-06-21 15:53:18
thumbnailImage: https://pic1.zhimg.com/v2-30253c279faba2e77120862dd54d49d4_r.jpg
tags: kotlin
---
## 内联函数

使用[`高阶函数`](https://kotlinlang.org/docs/reference/lambdas.html)会给运行时带来一些坏处：每个函数都是一个对象，捕获闭包（如：访问函数体内的变量），内存分配（函数对象或Class），虚拟调用引入的运行过载。
使用内联Lambda表达式在多数情况下可以消除这种过载。比如下面的函数就是这种情况下的很好的例子，`lock()`函数可以很容易地在调用点进行内联扩展。
```kotlin
lock(l){ foo() }
```
编译能够产生下面的代码，而不是创建一个函数对象参数，生成调用。
```kotlin
l.lock()
try {
    foo()
}
finally {
    l.unlock()
}
```
也是我们一开始想要的。
为了让编译器能够这样执行，需要用`inline`修饰符来标记`lock`函数
```kotlin
inline fun lock<T>(lock: Lock , body: () -> T): T{
  ...
}
```
`inline`修饰符既影响函数对象本身，也影响传入的Lambda参数：两者都会被内联到调用点。

编译预处理器对内联函数进行扩展，省去了参数压栈、生成汇编语言的CALL调用、返回参数、执行return等过程，从而提高了运行速度。
使用内联函数的优点，在函数被内联后编译器就可以通过上下文相关的优化技术对结果代码执行更深入的优化。
内联不是万能药，它以代码膨胀为代价，仅仅省去了函数调用的开销，从而提高程序的执行效率。
> 函数调用开销并不包括执行函数体所需要的开销，而是仅指参数压栈、跳转、退栈和返回等操作。如果执行函数体内代码的时间比函数调用的开销大得多，那么内联函数的效率收益会笑很多。另一方面每一处内联函数的调用都要拷贝代码，将使程序的总代码增大、消耗更多的内存空间。

## noinline
如果只需要在内联函数中内联部分Lambda表达式，可以使用`noinline`来标记不需要内联的参数。

```kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {
  // ...
}
```
内联Lambda只能在内联函数中调用或作为内联参数，但`noinline`的Lambda可随意使用。
> 没有内联函数参数和[`reified type parameters`](https://kotlinlang.org/docs/reference/inline-functions.html#reified-type-parameters)的内联函数，编译器会发出警告，因为内联这样的函数不见得有好处。

## 非局部返回
在Kotlin中可以使用正常、无条件的`return`退出有名和匿名函数，也意味需要使用一个标签来退出Lambda，在Lambda中禁止使用赤裸`return`语句，因为Lambda不能够使闭合函数返回。
```kotlin
fun foo(){
    ordinaryFunction{
        
        return // ERROR: can not make `foo` return here
    }
}
```
如果Lambda传入内联函数，则返回也是被内联，所以允许
```kotlin
fun foo(){
    inlineFunction {
        return // OK: the lambda is inlined
    }
}
```
这样的return（位于在Lambda中，但能够退出闭合函数）被称为非局部返回。Kotlin使用这种构造在有循环条件的闭合内联函数中
```kotlin
fun hasZeros(ints: List<Int>): Boolean{
    ints.forEach{
        if(it == 0) return true // returns from hasZeros
    }
    
    return false
}
```
> 一些内联函数可能不是从函数体中直接调用传入的Lambda参数，而是从其他的执行上下文，如本地对象或嵌套函数。在这些情况下，non-local 控制流则不允许出现在Lambda中。使用`crossinline`修饰符来标记。

```kotlin
inline fun f(crossinline body: () -> Unit) {
    val f = object: Runnable {
        override fun run() = body()
    }
    // ...
}
```
> 尚未在内联Lambda中支持break 和 continue 操作，Kotlin计划支持

## 具体化类型参数
有时需要访问传入函数中参数的类型

```kotlin
fun <T> TreeNode.findParentOfType(clazz: Class<T>): T? {
    var p = parent
    while (p != null && !clazz.isInstance(p)) {
        p = p.parent
    }
    @Suppress("UNCHECKED_CAST")
    return p as T?
}
```
在上述代码中，沿着树结构，使用反射来检查节点是否有指定类型。可以执行，但是调用点，并不优美
```kotlin
treeNode.findParentOfType(MyTreeNode::class.java)
```
实际上想要只是简单给函数传入一个类型，如：
```kotlin
treeNode.findParentOfType<MyTreeNode>()
```
内联函数支持具体化参数类型，因此可以这样写：
```kotlin
inline fun <reified T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p.parent
    }
    return p as T?
}
```
使用`reified`修饰符限制参数类型，可以在内联函数中访问，就像是普通的Class。因为函数是内联的，不在需要反射，像`!is`和`as`的普通操作符执行。也可以像上述说的那样调用`myTree.findParentOfType<MyTreeNodeType>()`

尽管反射在很多情况不需要，仍需要使用它来具体话参数类型。
```kotlin
inline fun <reified T> membersOf() = T::class.members

fun main(s: Array<String>) {
    println(membersOf<StringBuilder>().joinToString("\n"))
}
```
普通函数（未标记为内联）不可以有`reified`参数。不具有运行时表示的类型，不能用作具体化参数。
更多底层信息，参阅[`spec document`](https://github.com/JetBrains/kotlin/blob/master/spec-docs/reified-type-parameters.md)

## 内联属性
`inline`修饰符可以用在没有`Backing Filed`属性的访问函数。可以注解单独属性的访问函数。
```kotlin 
val foo: Foo
    inline get() = Foo()

var bar: Bar
    get() = ...
    inline set(v) { ... }
```
甚至可以注解整个属性，让属性访问函数都变为内联函数
```kotlin
inline var bar: Bar
    get() = ...
    set(v) { ... }
```
在调用时，内联访问函数与常规内联函数一样调用。