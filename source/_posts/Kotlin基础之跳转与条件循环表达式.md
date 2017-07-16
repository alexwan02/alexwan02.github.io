---
title: Kotlin基础之跳转与条件循环表达式
date: 2017-07-03 15:47:55
thumbnailImage: https://pic1.zhimg.com/v2-30253c279faba2e77120862dd54d49d4_r.jpg
tags: kotlin
---

# 跳转
Kotlin有三种结构的跳转表达式
- return. return默认返回最近的闭合函数或[`匿名函数`](https://kotlinlang.org/docs/reference/lambdas.html#anonymous-functions)
- break. 结束最近闭合循环
- continue. 执行最近闭合循环下一步

所有跳转表达式可在大的表达式中使用
```kotlin
val s = person.name ?: return
```
跳转表达式的类型为[`Nothing类型`](https://kotlinlang.org/docs/reference/exceptions.html#the-nothing-type)

## Break和Continue 标注
kotlin中任意表达式都可以用标注来标记，以`标识符`+`@`的形式来表示标注，如：`abc@` ，`fooBar@`都是有效的标注（[`参考语法`](https://kotlinlang.org/docs/reference/grammar.html#labelReference)）。把标注放在表达式后，即完成对表达式的标记。
```kotlin
loop@ for(i in 1..100){
  // ...
}
```
使用标注限制beak和continue
```kotlin
loop@ for(i in 1..100){
    for(j in 1..100){
        if(...) break@loop
    }
}
```
有限制标注的break表达式会跳转到标注对应的循环执行点。`continue`则执行对应循环的下一步迭代。

## 在标注处Return
使用函数常量，局部函数和对象表达式，Kotlin可以实现内嵌函数。returns使用标注可以从外部函数返回，其中一个用处就是从Lambda表达式中返回。
```kotlin
fun foo(){
  ints.forEach{
    if( it == 0 ) return
    print(it)
  }
}
```
`return表达式`从最近闭合函数中返回，如`foo`

> 非局部函数只支持传入[`内联函数`](https://kotlinlang.org/docs/reference/inline-functions.html)的Lambda表达式。

如果只想要从Lambda表达式返回，可以通过使用标注限制return
```kotlin
fun foo(){
    ints.forEach lit@{
        if( it == 0 ) return@lit
        print(it)
    }
}
```
上面即可实现只从Lambda表达式中返回，更为方便的是：可以使用隐式标注，标注与调用Lambda表达式的函数名相同。
```kotlin
fun foo(){
    ints.forEach {
      if( it == 0 ) return@forEach
      print(it)
    }

```
这种情况下，可以使用[`匿名函数`](https://kotlinlang.org/docs/reference/lambdas.html#anonymous-functions)来替换Lambda表达式。匿名函数中的返回语句只返回匿名函数本身。
```kotlin
fun foo(){
  ints.forEach(fun(value: Int){
      if( value == 0 ) return value
      print(value)
  })
}
```
有返回值时，解析器优先处理限制返回语句，如：
```kotlin
return@a 1

```
表示在标注`a`处返回`1`而不是返回标注的表达式`@a 1`

# 条件循环
## if 表达式
在Kotlin中，if为表达式，如返回具体值。不支持三目运算（condition ? then : else ），因为普通的if表达式即能完成三目的目的。
```kotlin
// 经典方式
var max = a
if( a < b ) max = b

// 与else
var max: Int
if(a > b){
    max = a
} else {
    max = b
}

// 表达式
val max = if( a > b ) a else b
```
如果条件分支是代码块，则最后一个表达式为代码块的值
```kotlin
val max = if(a > b){
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}

```
如果使用if作为表达式非条件语句时（如：返回if表达式的值或将if赋值给变量），则须有else条件分支。详见[`if语法`](https://kotlinlang.org/docs/reference/grammar.html#if)

## When表达式
`when`替代了`switch`操作符。when的一个简单形式：
```kotlin
when(x){
  1 -> print(" x == 1")
  2 -> print(" x == 2")
  else -> {  // Note the block
    print("x is neither 1 nor 2")
  }
}
```
when按照顺便比较参数与分支条件直到满足条件。when可以用作表达式或语句。用作表达式时，满足条件的分支对应值则是整个表达式的值。用作语句时，分支的值都会被忽略（如`if`相似，每个分支都可以是代码块，分支的值是代码块中最后一个表达式的值。）

如果没有满足条件的分支，则执行else分支。when用作表达式时，必须要有else分支，除非编译器可以证明所有分支条件都被覆盖到。

如果多个分支处理方式相同，则可以使用逗号组合这些分支：
```kotlin
when(x){
  0 , 1 -> print("x == 0 or x == 1")
  else -> print("otherwise")
}
```
可以使用任意（不仅仅是常量）表达式作为分支条件
```kotlin
when(x){
  parseInt(s) -> print("s encodes x")
  else -> print("s does not encode x")
 
}
```
分支条件可以检查集合或[`range`](https://kotlinlang.org/docs/reference/ranges.html)是否存在指定的参数值
```kotlin
when(x){
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outsize the range")
    else -> print("none of the above")
}
```
也可以用来检查是否是指定的类型参数，因为[`智能转换`](https://kotlinlang.org/docs/reference/typecasts.html#smart-casts)，不使用类型检查就可以访问类型的属性和方法。
```kotlin 
fun hasPrefix(x: Any) = when(x){
    is String -> x.startWith("prefix")
    else -> false
}
```
`when`也可以替代`if-else`if控制链。如果没有任何参数，分支条件则是简单boolean表达式，表达式值为true时则执行对应分支。
```kotlin
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```
跟多关于When,查看[`when语法`](https://kotlinlang.org/docs/reference/grammar.html#when)

## for循环
for循环能够迭代提供迭代器的任意对象。语法：
```kotlin
for(item in collection) print(item)
```
循环体可以是代码块
```kotlin
for(item in collection){
  // ...
}
```
for循环能够迭代任何提供迭代器的对象
- 有成员或扩展函数`iterator()`，并且返回类型为：
    - 有成员或扩展函数`next()` 且
    - 有成员或扩展函数`hasNex()`，返回类型为Boolean

这三个函数都被标记为`operator`

迭代数组的`for`循环被会编译成基于index的循环，而无须创建一个迭代器对象。

使用index来迭代数组或列表
```kotlin
for( i in array.indices){
  print(array[i])
}
```
> `iteration through a range`会被编译优化，无须创建额外对象。

可用`withIndex`库函数实现index迭代
```kotlin
for((index, value) in array.withIndex()){
    println("the element at $index is $ value")
}
```
查看[`for循环语法`](https://kotlinlang.org/docs/reference/grammar.html#for)

## While循环
while 和 do..while
```kotlin
while(x > 0){
    x--
}

do{
  val y = retrieveData()
} while ( y != null)  // a little different
```
查看[`while语法`](https://kotlinlang.org/docs/reference/grammar.html#while)

## break与continue
Kotlin支持循环break和continue操作，参考`跳转`一节中的文档。
