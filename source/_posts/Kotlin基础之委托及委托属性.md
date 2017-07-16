---
title: Kotlin基础之委托及委托属性
date: 2017-07-06 10:27:37
thumbnailImage: https://pic1.zhimg.com/v2-30253c279faba2e77120862dd54d49d4_r.jpg
tags: kotlin
---
# 委托
## 类委托
[委托模式](https://en.wikipedia.org/wiki/Delegation_pattern)是替换继承的较好的设计模式，Kotlin天生支持委托模式，无须任何模板代码。类`Derived`可以继承`Base`接口，委托所有public方法给指定对象
```kotlin
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x)}
}

class Derived(b: Base) : Base by b

fun main(args: Array<String>){
    val b = BaseImpl(10)
    
    Derived(b).print() // 输出10
}

```
`Derived`的超类列表中的`by`语句表示`b`会内部存储在`Derived`中，编译器会为`b`生成接口`Base`所有方法。

> 复写可以与期望一样生效：编译器使用复写方法替换委托对象中的方法。如果在`Derived`中添加`override fun print() { print("abc") }`，程序则会输出`abc`，而不是`10`

## 委托属性
有一些普通类型属性，尽管可以在需要时每次手动实现，如果可以一次实现所有将会更好并放入到库中。包括：
1. 懒属性：只在第一次访问计算的值
2. 观察属性：监听属性变化的通知
3. 在map中存储属性，不是每个属性单个一个字段。

为了覆盖这些案例，Kotlin支持委托属性
```kotlin
class Example {
    var p: String by Delegate()
}
```
语法为：`val/var <property name>: <Type> by <expression>`。跟在`by`之后的表达式为`Delegate`，因为属性对应的`get()`和`set()`则被委托给它的`getValue`和`setValue`方法。属性委托不用实现任何接口，但是要提供`getValue`和`setValue`方法（var类型属性）。比如
```kotlin
class Delegate {
    operator fun getValue(thisRef: Any? , property: KProperty<*>): String{
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }
    
    operator fun setValue(thisRef: Any? , property: KProperty<*>, value: String){
        println("$value has been assigned to '${property.name} in $thisRef.'")
    }
}
```
当读取委托给实例`Delegate`的`p`，则会调用`Delegate`的`getValue()`方法，它的第一个参数为`p`的对象，第二个参数持有`p`自己的描述（可以使用它的属性名）。如
```kotlin
val e = Example()
println(e.p)
```
输出结果
```kotlin
Example@33a17727, thank you for delegating ‘p’ to me!
```
类似地，如果给`p`赋值，则调用`setValue()`函数，前两个参数一样，第三个为赋予的值
```kotlin
e.p = "NEW"
```
输出结果
```kotlin
NEW has been assigned to ‘p’ in Example@33a17727.
```
> 从Koltin 1.1开始，可以在函数中或代码块中声明委托属性。

## 标准委托
Kotlin标准库提供几种有用类型的工厂方法，

### 懒委托
`lazy()`函数输入lambda，返回`Lazy<T>`实例，可以作为实现懒属性的委托：第一次调用执行传入`lazy()`的lambda，并记住结果，之后调用get()，只返回记住的结果
```kotlin
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

fun main(args: Array<String>) {
    println(lazyValue)
    println(lazyValue)
}
```
输出结果
```kotlin
computed!
Hello
Hello
```
懒属性默认为同步赋值：只在一个线程中求值，所有线程都会看到相同的值。如果不需要同步初始化委托，那么多个线程可以同时执行，将`LazyThreadSafetyMode.PUBLICATION`作为参数传递给`lazy`函数。如果能够保证初始化在单线程中执行，那么可以使用`LazyThreadSafetyMode.NONE`模式，但不能保证线程安全和相关开销。

## 观察者
[Delegates.observable()](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/observable.html)有两个参数：初始值和修改处理Handler。每次给属性赋值时，都会调用handler（在赋值执行之后）。Handler有三个参数：被赋值的属性，旧值和新值
```kotlin
import kotlin.properties.Delegate

class User {
    var name: String by Delegate.observable("<no name>"){
        prop , old , new -> 
        println("$old -> $new")
    }
}

fun main(args: Array<String>){
    val user = User()
    user.name = "first"
    user.name = "second"
}
```
输出结果

```kotlin
<no name> -> first
first -> second
```
如果希望拦截赋值并禁止它，使用[vetoable()](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/vetoable.html)替换`observable`。传给`vetoable`的handler在给属性赋新值前执行。


## 在Map中存储新值
我们通常使用Map来存储属性值，在应用中很常见，如解析JSON或其他动态的事。可以使用map实例本身作为委托属性的委托者。
```kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}
```
构造器使用Map作为它的参数
```kotlin
val user = User(mapOf(
    "name" to "John Doe" , 
    "age"  to 25
))
```
委托属性按照属性名从Map中取值
```kotlin
println(user.name) // Prints "John Doe"
println(user.age)  // Prints 25
```
对于`var`属性，使用`MutableMap`来替换只读的`Map`
```kotlin
class MutableUser(val map: MutableMap<String, Any?>) {
    var name: String by map
    var age: Int     by map
}
```
## 局部委托属性（从1.1开始）
从1.1开始可以声明局部变量为委托属性，如：创建局部懒属性
```kotlin
fun example(compute: () -> Foo){
    val memoizedFoo by lazy(computeFoo)
    if (someCondition && memoizedFoo.isValid()) {
        memoizedFoo.doSomething()
    }
}
```
`memoizedFoo`属性只在第一次访问时进行计算，如果`someCondition`失败，则不会计算变量。

## 属性委托要求
下面总结委托对象的要求
1. 对于只读属性（val），委托对象提供名为`getValue`的函数，并带有下面几个参数
    - `thisRef` - 类型必须与属性对象的超类一致（对于扩展属性：扩展的类型）
    - `property` - 必须为`KProperty<*>`或其子类
    - 函数返回值必须与属性或其子类属性一致

2. 对于可变属性（var）, 委托对象需要额外提供名为`setValue`的函数，并带有下面几个参数
    - `thisRef` - 必须与`getValue()`一样
    - `property` - 与`getValue()`一样
    - 新值 - 类型必须与属性对象的超类一致
    
`getValue()`和（或）`setValue()`可能会作为委托对象的成员函数或扩展函数，当需要委托属性给对象时（没有这些函数）时，扩展函数会比较方便。这两种函数都需要使用`operator`关键字来标记。

委托类可以实现`ReadOnlyProperty`或`ReadWriteProperty`其中一个接口，包含需要的`operator`方法。Kotlin标准库声明了这些接口。
```kotlin
interface ReadOnlyProperty<in R, out T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
}

interface ReadWriteProperty<in R, T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
    operator fun setValue(thisRef: R, property: KProperty<*>, value: T)
}
```

## 转换规则
每个委托属性的内在机制：Kotlin编译器会生成一个辅助属性，并委托给为委托属性。例如：`prop`会生成`prop$delegate`隐藏属性，访问者的代码只委托给了附加的属性：
```kotlin
class C{
    var prop: Type by MyDelegate()
}

// 编译器会生成下面的代码
class C{
    private val prop$delegate = MyDelegate()
    var prop: Type
        get() = prop$delegate.getValue(this, this::prop)
        set(value: Type) = prop$delegate.setValue(this, this::prop, value)
}
```
Koltin编译器提供有关prop的所有的必要信息：第一个参数`this`是外部类C的引用，`this::prop`是`KProperty`类型的反射对象，描述`prop`。
> `this::prop`语法表示直接[绑定调用](https://kotlinlang.org/docs/reference/reflection.html#bound-function-and-property-references-since-11)代码中的应用，在Kotlin1.1后可用

## 提供委托（从1.1开始）

定义`provideDelegate`操作函数，可以继承创建对象给委托属性的逻辑。如果给在`by`右边使用的对象，定了成员函数或扩展函数`provideDelegate`，则创建委托属性时调用这个函数。

使用`provideDelegate`的一种情况就是：在创建属性时，检查属性一致性，不仅仅是在`getter`和`setter`中。

如：希望在绑定前，检查属性名，可以这样做
```kotlin
class ResourceLoader<T>(id: ResourceID<T>) {
    operator fun provideDelegate(
            thisRef: MyUI,
            prop: KProperty<*>
    ): ReadOnlyProperty<MyUI, T> {
        checkProperty(thisRef, prop.name)
        // create delegate
    }

    private fun checkProperty(thisRef: MyUI, name: String) { ... }
}

fun <T> bindResource(id: ResourceID<T>): ResourceLoader<T> { ... }

class MyUI {
    val image by bindResource(ResourceID.image_id)
    val text by bindResource(ResourceID.text_id)
}

```

`provideDelegate`参数与`getValue`一样
- `thisRef` - 类型必须与属性对象的超类一致（对于扩展属性：扩展的类型）
- `property` - 必须为`KProperty<*>`或其子类

在创建`MyUI`实例时，调用每个属性的`provideDelegate`方法，立即执行必要的验证

不能够拦截`property`和委托类的绑定操作，可以显性传入属性名来达到这个效果。

```kotlin
class MyUI {
    val image by bindResource(ResourceID.image_id, "image")
    val text by bindResource(ResourceID.text_id, "text")
}

fun <T> MyUI.bindResource(
        id: ResourceID<T>,
        propertyName: String
): ReadOnlyProperty<MyUI, T> {
   checkProperty(this, propertyName)
   // create delegate
}
```
在生成的代码中，`provideDelegate`方法用来初始化`prop$delegate`属性。对比上面`val prop: Type by MyDelegate()`未声明`provideDelegate`方式生成的代码：
```kotlin
class C {
    var prop: Type by MyDelegate()
}

// 编译器生成下面的代码
// 提供provideDelegate函数时
class C {
    // 调用 "provideDelegate" 创建 "delegate" 属性
    private val prop$delegate = MyDelegate().provideDelegate(this, this::prop)
    val prop: Type
        get() = prop$delegate.getValue(this, this::prop)
}
```
> `provideDelegate`方法只影响辅助属性的创建，不影响生成的getter和setter代码。

