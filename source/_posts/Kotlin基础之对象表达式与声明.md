---
title: Kotlin基础之对象表达式与声明
date: 2017-07-06 10:27:24
thumbnailImage: https://pic1.zhimg.com/v2-30253c279faba2e77120862dd54d49d4_r.jpg
tags: kotlin
---
# 对象声明与表达式
有时需要创建类稍微修改的对象，不用显性声明类的子类。Java使用匿名内部类来处理，Kotlin使用对象表达式和对象声明概括。

## 对象表达式
创建继承某类的匿名类对象：
```kotlin
window.addMouseListener(object: MouseAdapter(){
    override fun mouseClicked(e: MouseEvent){
        // ...
    }
    
    override fun mouseEntered(e: MouseEvent){
        // ..
    }
})
```
如果超类有构造器，则需要传入合适的参数。多个超类在冒号之后使用逗号分隔
```kotlin
open class A(x: Int){
    public open val y: Int = x
}

interface B {...}

val ab: A = object : A(1) , B{
    override val y = 15
}
```
如果只需要一个对象，没有任何超类，则可以进行简化
```kotlin
fun foo(){
    val adHoc = object {
        var y: Int = 0
        var x: Int = 0
    }
    
    print(adHoc.x + adHoc.y)
}
```
> 用作类型的匿名对象，只能在局部和私有声明中。如果使用匿名对象作为public函数的返回值，或public属性的类型，则函数或属性的实际类型实际为匿名类的超类或Any（未声明任何超类）。匿名对象添加的成员不能被访问。

```kotlin
class C{
    // 私有函数，所以返回类型为匿名对象类型
    private fun foo() = object {
        val x: String = "x"
    }
    
    // Public函数，返回类型为Any
    fun publicFoo = object {
        val x: String = "x"
    }
    
    fun bar() {
        val x1: String = foo().x  // 有效
        val x2: String = publicFoo().x // 错误：无法引用x
    }
}
```

就像Java的匿名内部类，对象表达式中代码可访问闭合作用域中的变量（与Java不同是变量无须限制为final）

```kotlin
fun countClicks(window: JComponent){
    var clickCount = 0
    var enterCount = 0
    
    window.addMouseListener(object : MouseAdapter){
        override fun mouseClicked(e: MouseEvent){
            clickCount++
        }
        
        overried fun mouseEntered(e: MouseEvent){
            enterCount++
        }
    }
    
    // ...
}
```
## 对象声明
[单例](http://en.wikipedia.org/wiki/Singleton_pattern)是一个很有用的模式，Kotlin声明单例很简单
```kotlin
object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) {
        // ...
    }

    val allDataProviders: Collection<DataProvider>
        get() = // ...
}
```
这种形式被称为*对象声明*，名称跟在关键字`object`之后。与变量声明类似，对象声明非表达式，因此不能使用赋值语句的右侧。

引用对象，直接使用它的名称
```kotlin
DataProviderManager.registerDataProvider(...)
```
这样对象可以有超类
```kotlin
object DefaultListener : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }

    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
}
```
> 不能声明为局部对象（如直接在函数内部声明），但可以在其他对象声明中或非内部类中声明。

### 伴生对象

可以用`companion`关键字在类中声明对象
```kotlin
class MyClass {
    companion object Factory {
        fun create(): MyClass = MyClass()
    }
}
```
伴生对象成员可只使用类名作为限定符调用
```kotlin
val instance = MyClass.create()
```
伴生对象名可以忽略，使用`Companion`
```kotlin
class MyClass{
    companion object {
        
    }
}

val x = MyClass.Companion
```
> 注：即使伴生对象成员看起来像其他语言中的静态成员，在运行时这些仍然是真正对象的实例成员，并且可以实现接口

```kotlin
interface Factory<T>{
    fun create(): T
}

class MyClass {
    companion object : Factory<MyClass> {
        override fun create: MyClass = MyClass()
    }
}
```
然而在JVM上，如果使用`@JvmStatic`注解，伴生对象可以生成真正的静态成员和字段。

## 对象表达式与声明语义不同
1. 对象表达式使用时立即执行或初始化
2. 对象声明在第一次访问时，懒初始化。
3. 当对应类被加载时，则初始化伴生对象，符合Java静态初始化语义。

