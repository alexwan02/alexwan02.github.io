---
title: Kotlin进阶之类型检查、This表达式
date: 2017-07-17 09:38:15
tags: kotlin
---
# 类型检查与转型
## is和!is操作符
使用`is`和`!is`在运行时检查对象是否满足给定类型。
```kotlin
if (obj is String) {
    print(obj.length)
}

if (obj !is String) { // same as !(obj is String)
    print("Not a String")
}
else {
    print(obj.length)
}
```

## 智能转换
在很多情况中，Kotlin无须使用显式转换操作，因为编译器跟踪不变值的`is`检查，自动插入（安全地）转换。
```kotlin
fun demo(x: Any){
    if(x is String){
        print(x.length)  // x 自动会转为String
    }
}
```
如果是取反操作导致return，编译器也能进行职能转换
```kotlin
if (x !is String) return
    print(x.length) // x 自动转为String
```
或者在`||`和`&&`右侧
```kotlin
// 在`||` x自动转为String
if (x !is String || x.length == 0) return
// 在`&&` x自动转为String
if (x is String && x.length > 0) {
    print(x.length) // x 自动转为String
}
```
智能转换同样适应于`when表达式`和`while循环`
```kotlin
when(x){
    is Int -> print(x + 1)
    is String -> print(x.length + 1)
    is IntArray -> print(x.sum())
}
```
> 当编译器不能保证变量在检查和使用期间不能变化，则智能转换无效。

更加特殊的，智能转换适应下面规则：
- val 局部变量 - always
- val 属性 - 如果属性为私有、内部、在与声明的相同module检查。智能转换不适应于open属性，自定义getter的属性。
- var 局部变量 - 在检查与使用之间没有被修改，并且修改后没有被Lambda中捕获。
- var 属性 - never

## "非安全"转换操作
通常，如果不能转换，转换操作符会抛出异常。因此称之为非安全。Kotlin使用`as`操作符完成非安全转换（查看[操作符优先级](https://kotlinlang.org/docs/reference/grammar.html#precedence)）

```kotlin
val x: String = y as String
```
`null`无法转为`String`，因为类型是[非空类型](https://kotlinlang.org/docs/reference/null-safety.html)。比如，如果`y`为null，上面的代码则会抛出异常。为了满足Java转换语义，Kotlin需要在转换右侧添加可为空类型
```kotlin
val x: String? = y as String?
```

## "安全"（可为空）转换操作
避免抛出异常，可以使用`as?`安全转换操作，失败返回null
```kotlin
val x: String? = y as? String
```
> 尽管事实上`as?`右侧为非空类型String，转换结果可为空。

# This表达式
Kotlin使用`this表达式`表示当前接收者，
- 在类的成员中，`this`表示类的对象
- 在[扩展函数](https://kotlinlang.org/docs/reference/extensions.html)和[接收者表示的函数字面量](https://kotlinlang.org/docs/reference/lambdas.html#function-literals-with-receiver)，`this`表示点号左侧的接收者参数。

如果`this`没有限定符，指向最内闭合作用域。在其他作用域使用`this`，则需要使用label限定符

## this限定符
使用`this@label`访问作用域（如[class](https://kotlinlang.org/docs/reference/classes.html)或[扩展函数](https://kotlinlang.org/docs/reference/extensions.html)或[接收者表示的函数字面量](https://kotlinlang.org/docs/reference/lambdas.html#function-literals-with-receiver)）外部的this，`@label`表示`this`来源作用域的[label](https://kotlinlang.org/docs/reference/returns.html)
```kotlin
class A { // 隐性 label @A
    inner class B { // 隐性 label @B
        fun Int.foo() { // 隐性 label @foo
            val a = this@A // A's this
            val b = this@B // B's this

            val c = this // foo()的接收者, Int
            val c1 = this@foo // foo()的接收者, Int

            val funLit = lambda@ fun String.() {
                val d = this // funLit的接收者
            }


            val funLit2 = { s: String ->
                // foo()的接收者, 因为闭合Lambda表达式没有任何接收者
                val d1 = this
            }
        }
    }
}
```
