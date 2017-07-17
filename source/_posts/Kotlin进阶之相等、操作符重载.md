---
title: Kotlin进阶之相等、操作符重载
date: 2017-07-17 09:38:32
tags: kotlin
---
# 相等
Koltin有两种相等比较
- 引用相等（指向同一个对象的引用）
- 结构相等（`equals()`）

## 引用相等
使用`===`（`!==`）操作符检查引用相等。如果`a`和`b`指向相同对象，则`a === b`为`true`

## 结构相等
结构相等使用`==`操作符。按照规定，`a == b`会被翻译为：
```kotlin
a?.equals(b) ?: (b === null)
```
如果`a`不为空，则调用`equals()`函数，否则检查`b`引用是否为空。
> 非空检查`a == null`自动转化为`a === null`

# 操作符重载
Kotlin支持为自定义实现预定义操作符。这些操作符有固定的符号表示（如`+`或`*`）和固定的[优先级](https://kotlinlang.org/docs/reference/grammar.html#precedence)。使用固定名称[成员函数](https://kotlinlang.org/docs/reference/functions.html#member-functions)或[扩展函数](https://kotlinlang.org/docs/reference/extensions.html)实现对应类型操作符。

重载操作符的函数要用`operator`修饰符标记

长远起见Koltin定义了重载不同操作的常规操作的规范。

## 一元操作符

表达式 | 转化为
---|---
+a | a.unaryPlus()
-a | a.unaryMinus()
!a | a.not()

表格说明了在编译器处理时，`+a`操作符要执行一下步骤

- 确定`a`的类型，假设为`T`
- 查找`operator`修饰的`unaryPlus()`，调用者为`T`的无参函数。（可以是成员函数或者是扩展函数）
- 如果没有对应函数，或函数有歧义，则编译失败
- 如果找到对应函数，并且返回类型为`R`，则表达式`+a`类型为`R`

> 与其他操作符一样，[基本类型](https://kotlinlang.org/docs/reference/basic-types.html)的这些操作符会自动优化，不会引入额外开销

如下面，重载一元减号操作符
```kotlin
data class Point(val x: Int , val y: Int)

operator fun Point.unaryMinus() = Point(-x, -y)

val point = Point(10, 20)
println(-point)  // 输出"(-10, -20)"
```
## 自加和自减

表达式 | 转化为
---|---
a++ | a.inc()
a-- | a.dec() 

`inc()`和`dec()`函数必须指定返回值，赋值给调用`++`、`--`操作符的变量。不应该改变调用`inc()`和`dec()`函数的对象。

编译器执行以下步骤完成后缀形式（如`a++`）的操作。

- 确定`a`的类型，假设为`T`
- 查找适应`T`类型，`operator`修饰的`inc()`无参函数，
- 检查返回类型是否为`T`的子类

计算表达式的作用

- 存储初始值`a`的临时变量`a0`
- 将`a.inc()`赋值给`a`
- 返回`a0`作为表达式返回值

`a--`步骤同`a++`

对于前缀形式的`++a`和`--a`编译步骤相同， 作用

- 将`a.inc()`赋值给`a`
- 将`a`的新值作为返回值

## 二元操作数
算数操作符

Expression | 转换为
---|---
a + b | a.plus(b)
a - b | a.minus(b)
a * b | a.times(b)
a / b | a.div(b)
a % b | a.rem(b), ~~a.mod(b) (deprecated)~~
a..b | a.rangeTo(b)

编译器值只将表中操作表达式解析为`转换`列中形式。

> `rem`操作符从Kotlin 1.1版本起替换1.1之前的`mod`操作符

## 范例
如下面的Counter类复写`+`操作符，完成对指定值的累加操作
```kotlin
data class Counter(val dayIndex: Int){
    operator fun plus(increment: Int): Counter{
        return Counter(dayIndex + increment)
    }
}
```

## `in`操作符

表达式 | 转换
---|---
a in b | b.contains(a)
a !in b | !b.contains(a)

`in`和`!in`操作符处理步骤相同，但是转换后的参数相反。


## 索引访问操作符

表达式 | 转换
---|---
a[i] | a.get(i)
a[i, j] | a.get(i, j)
a[i_1, ..., i_n] | a.get(i_1, ..., i_n)
a[i] = b | a.set(i, b)
a[i, j] = b | a.set(i, j, b)
a[i_1, ..., i_n] = b | a.set(i_1, ..., i_n, b)

方括号转换为相同参数数量的`get`和`set`函数

## Invoke操作符

表达式 | 转换
---|---
a() | a.invoke()
a(i) | a.invoke(i)
a(i, j) | a.invoke(i, j)
a(i_1, ..., i_n) | a.invoke(i_1, ..., i_n)

括号转换为相同数量参数的`invoke`函数。

## 增强赋值

表达式 | 转换
---|---
a += b | a.plusAssign(b)
a -= b | a.minusAssign(b)
a *= b | a.timesAssign(b)
a /= b | a.divAssign(b)
a %= b | a.modAssign(b)

对于赋值操作符（如`a += b`），编译器执行以下步骤
- 如果提供表中右侧对应的函数
    - 如果有对应的二元函数（如`plusAssign()`对应的`plus()`），则编译出错（歧义）
    - 返回值为`Unit`，否则出错
    - 生成`a.plusAssign(b)`代码
- 否则尝试为`a = a + b`生成代码（包括类型检查：`a + b`的类型必须为`a`的子类）

> 在Kotlin中，赋值**不是**表达式

## 相等与不等操作符

表达式 | 转换
---|---
a == b | a?.equals(b) ?: (b === null)
a != b | !(a?.equals(b) ?: (b === null))

> `===`和`!==`（一致性检查）不可以进行重载，所以不存在规定。

`==`操作符为特殊情况：转换为屏蔽`null`值的复杂表达式。`null == null`值为`true`，且非空x的`x == null`值为`false`，不调用`x.equals()`

## 比较操作符

表达式 | 转换
---|---
a > b | a.compareTo(b) > 0
a < b | a.compareTo(b) < 0
a >= b | a.compareTo(b) >= 0
a <= b | a.compareTo(b) <= 0

所有比较操作符转换为调用`compareTo`，返回类型为`Int`

## 中缀调用命名函数
Koltin支持使用[中缀函数调用](https://kotlinlang.org/docs/reference/functions.html#infix-notation)模仿自定义中缀操作符。
