---
title: Kotlin进阶之空安全、异常
date: 2017-07-17 11:25:27
tags: kotlin
---
# 空安全

## 可为空类型和非空类型
Koltin力求消除代码中空引用的问题（[价值10亿的错误](http://en.wikipedia.org/wiki/Tony_Hoare#Apologies_and_retractions)）。

多数编程语言（包括Java）中一个常见难题：访问空引用的成员导致的空引用异常。Java中称为`NullPointerException`异常，简称NPE

引起NPE异常的原因有：
- 显性调用` throw NullPointerException()`
- 使用`!!`操作符
- 外部Java代码引起的异常
- 未初始化的数据（如：未初始化的this）

Kolin对可以持有`null`和非空引用进行区分。如：`String`类型变量不能持有`null`引用：
```kotlin
var a: String = "abc"
a = null // 编译出错
```
需要使用`String?:`来声明可为空的变量
```kotlin
var b: String? = "abc"
b = null  // OK
```

那么，可以调用`a`的方法或属性，并保证不会引起NPE异常：
```kotlin
val l = a.length
```
但是对于`b`，编译器则会报错
```kotlin
val l = b.length // 错误：变量b不能为空
```
为了能够使用b则需要进行一些调整。

## null检查的条件语句

### 显式检查`b`是否为空

```kotlin
val l = if( b != null ) b.length else -1
```
编译器跟踪执行检查的信息，允许在if中调用`length`属性，也支持复杂的条件语句
```kotlin
if( b != null && b.length > 0){
    print("String of length ${b.length}")
} else {
    print("Empty string")
}
```
> 只在b为`不可变`时有效（如在检查和使用之间不变的局部变量；或有`backing字段`且不可复写的`val`成员），因为b可能会在非空检查后变为`null`

### 安全调用操作符
第二种可选择是使用`?.`安全调用操作符
```kotlin
b?.length
```
如果`b`不为空，返回`b.length`；否则返回`null`。表达式类型为`Int?`。

安全调用在链式调用中会比较有用。如：假设有个`Bob`员工，对应一个部门（可能没有），部门有个主管，那么为了获取`部门主管`的名字，可以这样做:
```kotlin
bob?.department?.head?.name
```
如果上面任意属性为`null`，则结果都为`null`。

如果只执行非空类型的操作，可用安全调用与[let](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/let.html)关键字共同使用。
```kotlin
cal listWithNulls: List<String?> = listOf("A" , null)
for(item in listWithNulls){
    item?.let{ println(it)} // 输出"A"并忽略空值
}
```
## Elvis操作符
若有个可空引用`r`，则"r非空时使用r，否则使用其他非空值"

```kotlin
val l: Int = if( b != null ) b.length else -1
```
上面完整的`if`表达式可使用`?:`Elvis操作符替代
```kotlin
val l: Int = b?.length ?: -1
```
`?:`表达式左边不为空时，则返回它的值，否则返回表达式左侧的值。只有在Elvis操作符左测为null时，才计算右侧表达式。

> 因为`throw`和`return`在Kotlin中都是表达式，所以也可以在Elvis操作符右侧使用。因此方便用来检差函数参数。

```kotlin
fun foo(node: Node): String? {
    val parent = node.getParent() ?: return null
    val name = node.getName() ?: throw IllegalArgumentException("name expected")
    // ...
}
```
### `!!`操作符
NPE爱好者的第三种可选择操作符：`b!!`。如果b非空时，返回b的值；否则抛出`NPE`异常
```kotlin
val l = b!!.length
```
适用于需要显式判断的NPE时。

### 安全转换
如果对象非目标类型，常规转换会抛出`ClassCastException`类型转换异常，另外选项是如果转换未成功，使用安全转换返回`null`

```kotlin
val aInt: Int? = a as? Int
```
### 可为空类型集合
如果有一个组可为空类型集合且需要过滤非空元素，可以使用`filterNotNull`操作符
```kotlin
val nullableList: List<Int?> = listOf(1 , 2 , null , 4)

val intList: List<Int> = nullableList.filterNotNull()
```
# 异常
## 异常类
Kotlin中任意异常类都是`Throwable`子类。每个异常都具有`异常信息`，`堆栈信息`和`可选原因`。

使用`throw`表达式抛出异常对象
```kotlin
throw MyException("Hi There!")
```
使用`try`表达式捕获异常
```kotlin
try {
    // 代码
}
catch (e: SomeException) {
    // 处理
}
finally {
    // 可选 finally 块
}
```

`catch`块可以为0个或多个，`finally`块可忽略。但要至少要提供`catch`和`finally`其中一个。

## Try是表达式
`try`为表达式（如：可以有返回值）
```kotlin
val a: Int? = try { parseInt(input) } catch (e: NumberFormatException ) { null } 
```
`try`表达式返回值为`try`块或`catch`（出现异常时）块中最后一个表达式的值。`finally`块不影响表达式结果

## 检查型异常
Kotlin不支持检查型异常

如Java中`StringBuilder`类其中一个实现接口`Appendable`

```java
Appendable append(CharSequence csq) throws IOException;
```

每次拼接`String`时，都要`catch`这些`IOExceptions`异常。所以使用时要手动捕获
```java
try {
    log.append(message)
} 
catch (IOException e){
    // Must be safe
}

```
`Effective Java`并不提倡这种方式，参考[Item 65: Don't ignore exceptions](http://www.oracle.com/technetwork/java/effectivejava-136174.html)

Bruce Eckel说"[Java需要检查型异常么？](http://www.mindview.net/Etc/Discussions/CheckedExceptions)"

> Examination of small programs leads to the conclusion that requiring exception specifications could both enhance developer productivity and enhance code quality, but experience with large software projects suggests a different result – decreased productivity and little or no increase in code quality.

其他关于检查型异常的引用
- [Java检查型异常是个失误](http://radio-weblogs.com/0122027/stories/2003/04/01/JavasCheckedExceptionsWereAMistake.html)（Rod Waldhoff）
- [检查型异常的问题](http://www.artima.com/intv/handcuffs.html)（Anders Hejlsberg）

## Nothing类型
`throw`在Kotlin中是表达式，所以可以在Elvis表达式中使用
```kotlin
val s = person.name ?: throw IllegalArgumentException("Name required")
```
`throw`表达式返回类型为`Nothing类型`的特殊类型，这个类型没有值，经常用来标记无法到达的代码位置。可以使用`Nothing`类型无返回类型标记函数
```kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```
当调用这个函数时，编译则会知道函数调用之后的代码不再执行：
```kotlin
val s = person.name ?: fail("Name required")
println(s) // // 's' is known to be initialized at this point
```

## 与Java交互
参考[Java交互小节](https://kotlinlang.org/docs/reference/java-interop.html)




