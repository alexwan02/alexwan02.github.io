---
title: Kotlin基础之高阶函数与Lambdas
date: 2017-06-21 15:53:18
thumbnailImage: https://pic1.zhimg.com/v2-30253c279faba2e77120862dd54d49d4_r.jpg
tags: kotlin
---
## 高阶函数
在数学和计算机科学中，高阶函数是至少满足下列一个条件的函数：
- 接受一个或多个函数作为输入
- 输出一个函数
定义一个高阶函数，参数为Lock类型对象和一个函数。函数执行时获取锁，运行函数参数，执行结束释放锁。

```kotlin
fun <T> lock(lock: Lock , body: () -> T){
  lock.lock()
  try{
    return body()
  } 
  finaly{
    lock.unlock()
  }
}
```
`body`为函数类型对象：`() -> T`。表示无参函数，并且返回值类型为T。

调用`lock`函数，可以传入另外一个函数作为参数（参考[`函数引用`](https://kotlinlang.org/docs/reference/reflection.html#function-references)）

```kotlin
fun toBeSynchronized() = sharedResource.operation()
val result = lock(lock, ::toBeSynchronized)
```
更加方便的方式是传入[`lambda表达式`](https://kotlinlang.org/docs/reference/lambdas.html#lambda-expressions-and-anonymous-functions)
```kotlin
val result = lock(lock, { sharedResource.operation() })
```
Lambda表达式
- Lambda表达式要用花括号包裹
- 参数在`->` 之前声明（参数可能省略）
- 执行内容跟在 `->` 之后（如果提供的话）

在Kotlin中，如果函数最后参数为函数类型，可以在括号外部传入Lambda表达式作为指定的参数

```kotlin
lock (lock) { sharedResource.operation() }
```

高阶函数的另外一个例子：`map()`

```kotlin
fun <T, R> List<T>.map(transform: (T) -> R): List<R> {
    val result = arrayListOf<R>()
    for (item in this)
        result.add(transform(item))
    return result
}
```
map 调用
```kotlin
val doubled = inits.map{ value > value * 2}
```

> 如果lambda是函数调用的唯一参数，花括号可以完全省略。

### it : 单个参数的隐式名称
如果函数只有一个参数，则可以忽略声明函数参数（与`->`一起），用`it`来替代：
```kotlin
ints.map{ it * 2}
```
因此可以写成查询表达式形式的代码
```kotlin
strings.filter { it.length == 5 }
       .sortBy { it }
       .map { it.toUpperCase() }
```
### 下划线替代未使用的参数
如果lambda参数未使用，可以使用下划线来替代
```kotlin
map.forEach { _, value -> println("$value!") }
```

### Lambdas中的解构（since 1.1）
Lambdas解构参考[`解构声明`](https://kotlinlang.org/docs/reference/multi-declarations.html#destructuring-in-lambdas-since-11)

## 内联函数
有时可使用内联函数来提升高阶函数性能，[`内联函数`](https://kotlinlang.org/docs/reference/inline-functions.html)

## Lambda表达式和匿名函数
Lambda表达式和匿名函数都是`函数常量`。用表达式作为参数，无须额外声明函数。
```kotlin
max(strings, { a, b -> a.length < b.length })
```
函数`max`是一个高阶函数，第二个参数是函数类型的值。等同于
```kotlin
fun compare(a: String, b: String): Boolean = a.length < b.length
```

### 函数类型
对于接受函数作为参数的高阶函数，要给参数执行函数类型。例如上面的`max`函数的定义
```kotlin
fun <T> max(collection: Collection<T>, less: (T, T) -> Boolean): T? {
    var max: T? = null
    for (it in collection)
        if (max == null || less(max, it))
            max = it
    return max
}
```
参数`less`的类型就是`(T, T) -> Boolean`。即：比较两个T类型的变量的大小的函数，如果第一个比第二个小则返回`true`

在代码中，`less`用作函数，调用时传入两个T类型的参数。

这个函数类型也可以有具名参数
```kotlin
val compare: (x: T, y: T) -> Int = ...
```

### Lambda表达式的语法

Lambda表达式的完整语法形式,比如函数类型常量
```kotlin
val sum = { x: Int, y: Int -> x + y }
```
Lambda表达式总以花括号包裹。完整语法形式中：
- 参数在圆括号中声明，参数类型可选
- 表达式体在`->` 符号之后。
- 如果返回类型不是`Unit`，表达式体中最后（可能为单个）的表达式被视为返回值。

如果忽略所有可选注解，则Lambda表达式为
```kotlin
val sum: (Int, Int) -> Int = { x, y -> x + y }
```
有单独一个参数的Lambda表达式很常见。如果Kotlin能够识别函数签名，则允许不声明这个唯一的参数，使用参数隐式声明`it`
```kotlin
ints.filter { it > 0 } //  常量类型为'(it: Int) -> Boolean'
```
Kotlin使用[`qualified return`](https://kotlinlang.org/docs/reference/returns.html#return-at-labels)语法，显式指定返回值类型。否则，隐式使用返回最后一个表达式的值。因此下面两段代码效果相同。
```kotlin
ints.filter {
    val shouldFilter = it > 0 
    shouldFilter
}

ints.filter {
    val shouldFilter = it > 0 
    return@filter shouldFilter
}
```
> 如果函数以另外一个函数作为最后一个参数，lambda表达式则可以在参数列表外使用。参考[`callSuffix`](https://kotlinlang.org/docs/reference/grammar.html#callSuffix)语法

### 匿名函数
上面Lambda表达式语法中，没有讲到关于Kotlin指定函数的返回类型能力。在多数情况中，没有必要执行函数返回类型，因为Kotlin可以自动推断。但是如果需要显式指定类型，可以通过`匿名函数`来实现。
```kotlin
fun(x: Int, y: Int): Int = x + y
```
匿名函数与常规函数声明看起来非常相似，除了忽略了函数名。函数体可以是表达式，也可以是代码块
```kotlin
fun(x: Int, y: Int): Int {
    return x + y
}
```

指定参数和返回值类型方式与常规函数相同，如果根据上下文可以推断出类型，可以省略参数类型。
```kotlin
ints.filter(fun(item) = item > 0)
```
返回值类型推断与正常函数相同：带有表达式体的匿名函数返回类型自动推断；带有代码块的匿名函数返回类型要显式指定（或默认为`Unit`）。

> 匿名函数总要用括号传参。在括号外传参的方式只适应于Lambda表达式

Lambda表达式和匿名表达式一个不同点是：非局部返回行为，即没有标签的返回声明语句从声明有`fun`关键字的函数中返回。
所以Lambda表达式中的返回语句会从封闭函数中返回，而匿名函数的返回语句返回匿名函数本身。

### 闭包
闭包（Closure），又称词法闭包（Lexical Closure）或函数闭包（function closures），是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。

Kotlin中Lambda表达式与匿名函数（[`本地函数`](https://kotlinlang.org/docs/reference/functions.html#local-functions)、[`对象表达式`](https://kotlinlang.org/docs/reference/object-declarations.html#object-expressions)亦同）可以访问它的闭包，如外部作用域中声明的变量。与Java不同，可以修改闭包中的变量。

```kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
print(sum)
```
### 带有接收者的函数常量
Kotlin可以调用带有接收者的函数常量。在函数常量的函数体中，可以不使用任何额外限定符，调用接收者的方法，这与扩展函数相似：允许访问函数体内部接收者的成员。其中一个例子[`Type-safe Groovy-style builders.`](https://kotlinlang.org/docs/reference/type-safe-builders.html)

这种函数常量类型实际是带有接收者的函数类型
```kotlin
sum : Int.(other: Int) -> Int
```
如果函数常量是接收者的一个方法，则可以直接调用。
```kotlin
1.sum(2)
```
匿名函数语法可以直接指定函数常量接收者类型。可以声明带有接收者的函数类型变量，并在之后调用。
```kotlin
val sum = fun Int.(other: Int): Int = this + other
```
当可以从上下文中推断接收者类型时，Lambda表达式可以用作带有接收者的函数常量。
```kotlin
class HTML {
    fun body(){...}
}

fun html(init: HTML.() -> Unit): Unit{
    val html = HTML() // create the receiver object
    html.init()
    return html
}

html {    // lambda with receiver begins here
   body() // calling a method on the receiver object 
}
```