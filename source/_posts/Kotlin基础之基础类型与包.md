---
title: Kotlin基础之基础类型与包
date: 2017-07-03 15:48:18
thumbnailImage: https://pic1.zhimg.com/v2-30253c279faba2e77120862dd54d49d4_r.jpg
tags: kotlin
---
# 基础类型
在Kotlin中，一切皆对象从这种意义上来说，我们可以调用任何变量的属性和方法。有些内嵌类型的实现是经过优化的，但是对于开发者而言看上就是普通的类。这些类型有：数值类型，字符类型，boolean类型，数组类型

## 数值类型
Kotlin使用类似Java的方式处理数值类型，但是与Java不是完全相同。如：不支持数值的隐式扩大转换或在某些情况中字面常量会些许不同。

Kotlin内嵌了以下几种数值类型

Type | Bit width
---|---
Double | 64
Float | 32
Long | 64
Int  | 32
Short| 16
Byte | 18

在Kotlin中，字符常量不属于数值类型

## 字面常量
整型的字面常量
- 十进制：123
    - Long类型后缀L表示：123L
- 十六进制：0x0F
- 二进制：0b00001011

> 不支持八进制

支持浮点型数值的常规字符
- Double：`123.5` ，`123.5e10`
- Float使用`f`或`F`标记：`123.5f`

## 数字常量中的下划线
可以在数字中使用下划线，让数字可读性更好。
```kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```
## 表示法
在Java平台，数字物理存储为JVM的原始类型，除非需要一个可以为空的数值引用或作为泛型引用，这种情况时，数字会被自动装箱。

数值装箱不一定保留同一性
```kotlin
val a: Int = 10000
print(a === a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA === anotherBoxedA) // !!!Prints 'false'!!!
```
但保留相等性
```kotlin
val a: Int = 10000
print(a == a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA == anotherBoxedA) // Prints 'true'
```

## 显式转换
因为不同的表示法，较小类型并非是较大类型数字的子类型。因为如果是的话，则会遇到下面的问题
```kotlin
// 非实际编译，假设性代码
val a: Int? = 1 //  Int装箱
val b: Long? = a // 隐式装换，生成Long类型装箱
print(a == b) // 因为Long只与Long类型比较，打印`false`
```
不仅仅是同一性，甚至相等性也默默丢失。

因此较小的类型无法隐式转换成较大类型。意思是无法将`Byte`类型数值隐式转为`Int`类型
```kotlin
val b: Byte = 1 // OK, literals are checked statically
val i: Int = b // ERROR
```
使用显式转换扩大数值
```kotlin
val i: Int = b.toInt() // OK: explicitly widened
```
所有数值类型都支持以下转换：
- toByte(): Byte
- toShort(): Short
- toInt(): Int
- toLong(): Long
- toFloat(): Float
- toDouble(): Double
- toChar(): Char

缺少隐式转换很少被注意到，因为数值类型可从上下文中推断出，合适转换算术运算符的重载。如
```kotlin
val l = 1L + 3 // Long + Int => Long
```
## 操作符
Kotlin支持标准的数值操作符，这些操作符声明为合适类的成员（编译会优化操作符到对应指令）。查看[`操作符重载`](https://kotlinlang.org/docs/reference/operator-overloading.html)

因为按位操作符没有特定的符号表示，使用中缀表达式来使用对应名称的函数。如
```kotlin
val x = (1 shl 2) and 0x000FF000
```
下面就是按位操作符的完整列表（仅适应于`Int`和`Long`）
- shl(bits) – 有符号左移 (Java's <<)
- shr(bits) – 有符号右移 (Java's >>)
- ushr(bits) – 无符号右移 (Java's >>>)
- and(bits) – 按位与
- or(bits) – 按位或
- xor(bits) – 按位异或
- inv() – 按位反转

## 字符
`Char`表示字符，不能直接作为数值使用。
```kotlin
fun check(c: Char) {
    if (c == 1) { // ERROR: incompatible types
        // ...
    }
}
```
字符常量使用单引号括起来，如`1`。特殊字符串使用反斜杠转义。`\t`、`\b`、`\n`、`\r`、`\'`、`\\`和`\$`等转义字都符能支持。使用Unicode转义字符语法（`'\uFF00'`）来对其他字符进行编码。

可以显式将Char转为Int类型。
```kotlin
fun decimalDigitValue(c: Char): Int {
    if (c !in '0'..'9')
        throw IllegalArgumentException("Out of range")
    return c.toInt() - '0'.toInt() // Explicit conversions to numbers
}
```
当需要可为空的引用时，字符可以像数字一样装箱。装箱操作不会保留同一性。

## Boolean
Boolean类型表示布尔值，有两个值*true*和*false*

当为可空的引用时，Boolean类型进行装箱。

内置的boolean操作包括：
- || 短路或
- && 短路与
- !  逻辑非

## Array数组
Kotlin使用`Array`类表示数组。Array类定义了`set`与`get`函数（按照函数重载定义会转换成`[]`）,`size`属性，和其他几个有用的函数。
```kotlin
class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ...
}
```
使用函数`arrayOf()`来创建数组，传入多个item参数（如arrayOf(1, 2, ,3) 能够创建[1 , 2, 3]的数组）。使用函数`arrayOfNulls()`来创建指定长度但是元素为空的数组。

使用一个工厂函数传入数组长度，返回数组指定index的元素值。
```kotlin
// Creates an Array<String> with values ["0", "1", "4", "9", "16"]
val asc = Array(5, { i -> (i * i).toString() })
```
正如上面所说，`[]`操作符可以替代成员函数`get`和`set`。

> 与Java不同，Kotlin中的数组是不可变的，也就是说Kotlin不能将`Array<String>`赋值给`Array<Any>`，也就预防可能的运行时问题（但可以使用`Array<outAny>`），详见[`类型保护`](https://kotlinlang.org/docs/reference/generics.html#type-projections)

Kotlin有特殊类来表示原始类型数组，而不用向上装箱：`ByteArray`、`ShortArray`、`IntArray`等等。这些类与`Array`类没有继承关系，但是有相同的方法和属性，每个都有响应的工厂方法。
```kotlin
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]
```
## String
String表示String类型数据。String不可变，string中的字符元素可使用`s[i]`方式访问。可使用`for`循环来迭代String。
```kotlin
for (c in str) {
    println(c)
}
```
String常量
Kotlin支持两种string字面常量：转义string与raw string。转义string可能含有转义字符。raw string 可以有换行符和任意字符。

转义string与java很像
```kotlin
val s = "Hello World !\n"
```
转义与常规方式相同，使用转义字符。详情查看[`字符`](https://kotlinlang.org/docs/reference/basic-types.html#characters)。

raw string使用三个引号界定`"""`，不用转义字符，可以含有换行字符和任意其他字符。
```kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```
可以使用[`trimMargin()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/trim-margin.html)函数去除前导空格。
```kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```
默认使用`|`表示边缘前缀，也可以传入其他字符，如`trimMargin(">")`。

## String模板
String可以包含模板表达式，如将代码中的返回结果串联到字符串中。模板表达式由`$`起始符与参数名构成。
```kotlin
val i = 10
val s = "i = $i" // evaluates to "i = 10"
```
或者在`{}`中的任意表达式
```kotlin
val s = "abc"
val str = "$s.length is ${s.length}" // evaluates to "abc.length is 3"

```
String模板在raw string和转义string中都可以使用。如果要在raw string中表示一个$字符常量（不支持反斜杠转义），则可使用下面语法：
```kotlin
val price = """
${'$'}9.99
"""
```
# Package包
源码文件可能需要声明包命
```kotlin
package foo.bar

fun baz() {}

class Goo {}
```
源码文件所有内容（如类与函数）都包含在声明的包中。上面例子中，`baz()`全称为`foo.bar.baz`，`Goo`的全称为`foo.bar.Goo`。

如果未执行包，文件内容属于无名的默认包

## 默认导入

许多包默认导入到Kotlin的每个文件中

```kotlin
kotlin.*
kotlin.annotation.*
kotlin.collections.*
kotlin.comparisons.* (since 1.1)
kotlin.io.*
kotlin.ranges.*
kotlin.sequences.*
kotlin.text.*
```
根据目标平台导入附加包
```kotlin
JVM:
    java.lang.*
    kotlin.jvm.*
JS:
    kotlin.js.*
```

## 导入
处理默认导入，每个文件可以含有自己导入的目录。导入的语法描述参考[`Import`](https://kotlinlang.org/docs/reference/grammar.html#import)

可以导入单独名称
```kotlin
import foo.Bar // Bar is now accessible without qualification
```
或者访问范围内容（包，类，对象等）
```kotlin
import foo.* // everything in 'foo' becomes accessible
```
出现冲突时，使用`as`关键字给冲突包设置别名
```kotlin
import foo.Bar // Bar is accessible
import bar.Bar as bBar // bBar stands for 'bar.Bar'
```
`import`关键字不限于导入类，也可以用来导入其他声明：
- 上层函数与属性
- 在[`对象声明`](https://kotlinlang.org/docs/reference/object-declarations.html#object-declarations)中声明的函数与属性
- [`枚举常量`](https://kotlinlang.org/docs/reference/enum-classes.html)

与Java不同的是，Kotlin没有单独的`import static`语法；所有声明都使用`import`关键字导入。
## 顶层声明访问
如果顶层声明标记为`private`，则它私有属于在声明的文件。详情查看[`访问修饰符`](https://kotlinlang.org/docs/reference/visibility-modifiers.html)