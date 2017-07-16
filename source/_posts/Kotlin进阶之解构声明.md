title: Kotlin进阶之解构声明
date: 2017-07-16 15:28:39
thumbnailImage: https://pic1.zhimg.com/v2-30253c279faba2e77120862dd54d49d4_r.jpg
tags: kotlin
---

# 解构声明
Kotlin可以将一个对象解构为多个变量
```kotlin
val (name, age) = person 
```
这种语法被称为解构声明。解构声明一次创建多个变量。比如声明`name`和`age`两个新的变量，可以单独使用
```kotlin
println(name)
println(age)
```
解构声明最终编译为下面的代码
```kotlin
val name = person.component1()
val age = person.component2()
```
`component1()`和`component2()`函数是Kotlin中广泛使用的惯例原则的例子（如`+`、`*` 操作符，`for`循环等）。解构声明右侧能放任意对象，只要可以调用所需的组件函数，如组件`component3()`，`component4()`等等。

`componentN()`函数需要使用`operator`操作符标记，可以在解构声明中使用。

解构声明同样可以在`for`循环中使用
```kotlin
for ((a, b) in collection) { ... }
```
变量`a`和`b`为集合中元素`component1()`和`component2()`的值。

## 一个函数返回两个值范例
如果需要一个函数返回两个值，如：一个返回对象和一些排序状态，Kotlin可以通过声明一个[data类](https://kotlinlang.org/docs/reference/data-classes.html)并返回它的实例方式。
```kotlin
data class Result(val result: Int , val state: Status)

fun function(...): Result{
    // 计算
    return Result(result ,status)
}

// 现在可以使用这个函数了
val (result , status) = function(...)
```
因为解构声明自动为data类声明`componentN()`函数。

> 也可以使用标准类`Pair`（有返回Pair<Int , Status>的函数），但通常更好的方式就是使用自己命名的属性。

## 结构声明与Map范例
下面示例可能是迭代Map的好的方式
```kotlin
for((key , value ) in map){
    // ...
}
```
想要这样做，需要：
- 添加Map中值队列的`iterator()`函数
- 添加元素键值对的`component1()`和`component2()`函数

实际上，Kotlin标准库已经准备这些扩展：
```kotlin
operator fun <K, V> Map<K, V>.iterator(): Iterator<Map.Entry<K, V>> = entrySet().iterator()
operator fun <K, V> Map.Entry<K, V>.component1() = getKey()
operator fun <K, V> Map.Entry<K, V>.component2() = getValue()
```
因此可以自由在`for`循环中对Map使用解构声明（与data类集合一样）

## 未使用变量的下划线表示（从1.1开始）
如果不需要在解构声明中使用某个变量，可以使用下划线来替代
```kotlin
val (_, status) = getResult()
```

## 解构声明与Lambda表达式（从1.1起）
可以在Lambda表达式中使用解构声明，如果Lambda中有Pair类型参数（或`Map.Entry`等提供`componentN`函数的类型），可以将单独这个参数在圆括号中进行解构声明。
```kotlin
map.mapValues { entry -> "${entry.value}!" }
map.mapValues { (key, value) -> "$value!" }
```
> 注意声明两个参数与一个参数的解构声明不同

```kotlin
{ a -> ... } // 单个参数
{ a, b -> ... } // 两个参数
{ (a, b) -> ... } // 解构声明
{ (a, b), c -> ... } // 解构声明和单个参数组合
```
如果未使用组件某个解构参数，使用下划线替代属性名
```kotlin
map.mapValues { (_, value) -> "$value!" }
```
可以为整个解构参数或单个解构参数指定类型
```kotlin
map.mapValues { (_, value): Map.Entry<Int, String> -> "$value!" }

map.mapValues { (_, value: String) -> "$value!" }
```

## 参考

[JavaScript解构赋值](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)