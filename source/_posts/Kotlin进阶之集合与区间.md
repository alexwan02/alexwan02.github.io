---
title: Kotlin进阶之集合与区间
date: 2017-07-17 09:37:47
tags: kotlin
---
# 集合
与多数语言不一样，Kotlin区分可变与不可变集合（list，sets，maps等等）。精确控制什么时候可以编辑集合有利于减少BUG和设计良好的API。

在这之前，理解只读的可变集合与实际不变的集合的区别很有必要。两者创建都很容易，但是类型系统表达却不相同。

Kotlin的`List<out T>`类型是一个接口，提供只读操作如`size`，`get`等等。与Java相同，继承`Collection<T>`接口，`Collection<T>`继承`Iterable<T>`。`MutableList<T>`添加了修改数组的方法。同样模式还有`Set<out T>/MutableSet<T>`和`Map<K, out V>/MutableMap<K, V>`

List与Set的基础用法
```kotlin
val numbers: MutableList<Int> = mutableListOf(1, 2, 3)
val readOnlyView: List<Int> = numbers
println(numbers) // 输出"[1, 2, 3]"
numbers.add(4)   
println(numbers) // 输出"[1 , 2, 3, 4]"
readOnlyView.clear() // 不会通过编译

val strings = hashSetOf("a", "b", "c", "c")
assert(strings.size == 3)
```
Kotlin没有创建列表或Set的专用语法，可以使用如`listOf()`，`mutableListOf()`，`setOf()`，`mutableSetOf()`创建。在非性能严格的代码中，可以使用简单的[惯用语法](https://kotlinlang.org/docs/reference/idioms.html#read-only-map)实现:`mapOf(a to b, c to d)`。

> `readOnlyView`变量指向相同list，像基本List改变一样改变。如果唯一引用指向只读变量List，则认为集合整体不可变。

创建不可变集合的一个简单方法：
```kotlin
val items = listOf(1, 2, 3)
```
目前，`listOf`方法使用数组List实现，但在未来可能会返回存储效率更高的完全不可变的集合类型。

> 只读类型为[协变量](https://kotlinlang.org/docs/reference/generics.html#variance)，意味可是使用`List<Rectangle>`赋值给`List<Shape>`（假设`Rectangle`继承`Shape`）。在可变集合中则不被允许，因为在运行时，可变集合可能允许出错。

有时希望及时返回集合快照给调用者，保证集合不变。

```kotlin
class Controller {
    private val _items = mutableListOf<String>()
    val items: List<String> get() = _items.toList()
}
```
`toList`扩展函数只是List的副本，返回的List保证不会改变。

还应该熟悉List和Set的一些有用的扩展方法
```kotlin
val items = listOf(1, 2, 3, 4)
items.first() == 1
items.last() == 4
items.filter { it % 2 == 0 } // 返回[2 , 4]

val rwList = mutableListOf(1, 2, 3)
rwList.requireNoNulls()        // 返回 [1, 2, 3]
if (rwList.none { it > 6 }) println("No items above 6")  // 输出 "No items above 6"
val item = rwList.firstOrNull()
```

同样的还有其他如`排序`，`压缩`，`折叠`，`减少`等可能需要的实用方法。

Map采取相同模式，可以像如下实例化和访问。

```kotlin
val readWriteMap = hashMapOf("foo" to 1, "bar" to 2)
println(readWriteMap["foo"]) // 输出"1"
val snapshot: Map<String, Int> = HashMap(readWriteMap)
```

# 区间
使用`rangeTo`函数行成的区间表达式，可以使用`..`形式的操作符，并使用`in`或`!in`补全。Kotlin为任意比较类型定义的区间，对于整型基本类型则有自己最优化的实现。如：

```kotlin
if( i in 1..10){  // 与 1 <= i && i <= 10相等
    println(i)
}
```
整形类型区间（`IntRange`，`LongRange`，`CharRange`）有个额外的特性：可以循环迭代。编译器可以将区间转为近似Java的`for`循环，而没有额外的开销。

```kotlin
for ( i in 1..4) print(i) // 输出 "1234"
for (i in 1..4) print(i) //  无任何输出
```
如果希望逆序循环，可通过标准库中定义的`downTo`函数
```kotlin
for (i in 4 downTo 1) print(i) // 输出 "4321"
```
循环通过`step`函数来指定单循环步数
```kotlin
for(i in 1..4 step 2) print(i) // 输出"13"
for(i in 4 downTo 1 step 2) print(i) // 输出"42"
```
使用`until`函数创建不包括结尾元素的循环迭代操作
```kotlin
for( i in 1 until 10 ){ // i in [1 , 10) 不包括10
    println(i)
}
```

## 实现原理
区间实现库中的一个通用的接口：`ClosedRange<T>`

`ClosedRange<T>`表示数学意义上的闭合区间，为比较类型定义。有两个端点：`起始`和`结束`，两点都包含在区间内。主要操作符是`contains`，使用形式`in`和`!in`

整型数列(`IntProgression`，`LongProgression`，`CharProgression`)表示算数队列。数列使用`起始`元素，`末尾`元素和`非零步数`定义。首位元素为起始元素，子队列元素为前一个元素加上步数，末尾元素总要循环命中（除非队列为空）。

队列为`Iterable<N>`子类，N表示`Int`，`Long`，`Char`，所以可以在`for`循环和像`map`，`filter`等函数中使用。队列循环近似与Java中的索引`for`循环
```kotlin
for (int i = first; i != last; i += step) {
  // ...
}
```
对于整型，`..`操作符会创建实现`ClosedRange<T>`和`*Progression`接口的对象。如`IntRange`实现`ClosedRange<Int>`并继承`IntProgression`，因此所有为`IntProgression`定义的操作符都用于`IntRange`。`downTo()`和`step()`函数返回值为`*Progression`。

使用`fromClosedRange`函数构建的队列，定义在它们的伴生对象中：
```kotlin
IntProgression.fromClosedRange(start, end, step)
```
队列末尾元素用于计算查找`step为正数`时不超过`end`值的最大值，或`step为负数`是不小于`end`的最小值`（last -  first） % step == 0`。

## 工具函数
### rangeTo()
整型的`rangeTo()`操作符就是调用`*Range`的构造方法
```kotlin
class Int{
    //...
    operator fun rangeTo(other: Long): LongRange = LongRange(this, other)
    //...
    operator fun rangeTo(other: Int): IntRange = IntRange(this, other)
    //...
}
```
浮点数（`Double`，`Float`）没有定义`rangeTo`操作符，使用标准库提供的泛型`Comparable`类型替代。

```kotlin
public operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T>
```
这个函数的返回区间不能用于循环迭代。

### downTo()
为整型定义的`downTo()` 扩展函数，如：
```kotlin
fun Long.downTo(other: Int): LongProgression {
    return LongProgression.fromClosedRange(this, other.toLong(), -1L)
}

fun Byte.downTo(other: Int): IntProgression {
    return IntProgression.fromClosedRange(this.toInt(), other, -1)
}
```

### reversed()
为每个`*Progression`类定义`reversed()`扩展函数，返回反转的数列
```kotlin
fun IntProgression.reversed(): IntProgression {
    return IntProgression.fromClosedRange(last, first, -step)
}
```

### step()
为`*Progression`定义`step()`扩展函数，返回使用step值过滤后的数列。`step`值要求总要为正数，因此这个不会修改循环方向。
```kotlin
fun IntProgression.step(step: Int): IntProgression {
    if (step <= 0) throw IllegalArgumentException("Step must be positive, was: $step")
    return IntProgression.fromClosedRange(first, last, if (this.step > 0) step else -step)
}

fun CharProgression.step(step: Int): CharProgression {
    if (step <= 0) throw IllegalArgumentException("Step must be positive, was: $step")
    return CharProgression.fromClosedRange(first, last, if (this.step > 0) step else -step)
}
```

> 函数返回的数列末尾值可能会与原队列末尾值不同，保证`(last - first) % step == 0`。
```kotlin
(1..12 step 2).last == 11  // 队列 [1, 3, 5, 7, 9, 11]
(1..12 step 3).last == 10  // 队列 [1, 4, 7, 10]
(1..12 step 4).last == 9   // 队列 [1, 5, 9]
```