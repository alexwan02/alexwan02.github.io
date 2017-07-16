---
title: Kotlin基础之数据类、密封类、嵌套类和枚举类
date: 2017-07-03 15:49:27
thumbnailImage: https://pic1.zhimg.com/v2-30253c279faba2e77120862dd54d49d4_r.jpg
tags: kotlin
---
# 数据类
我们经常创建只用来存储数据的类。在这样的类中，一些标准功能常由数据来驱动。这些类在Kotlin中使用`data`修饰符标记为***数据类***
```kotlin
data class User(val name: String, val age: Int)
```
编译器自动驱动在私有构造中声明的所有属性，生成为成员。
- `equals()`和`hashCode()`函数对
- `User(name=John, age=42)`形式的`toString()`方法
- 相关声明顺序的属性，对应的[`组件函数`](https://kotlinlang.org/docs/reference/multi-declarations.html)
- `copy()`函数
如果类中或基类中已显式定义对应的函数，则不会额生成。

为了保证生成代码的一致性和意义性，数据类需要满足以下条件：
- 首要构造器至少要有一个参数
- 所有首要构造器参数都要标记为`val`或`var`
- 数据类不能是抽象，open，密封或内部类型。
- （在1.1之前）数据类只能实现接口

从1.1开始，数据类可以继承其他类（参考[`密封类`](https://kotlinlang.org/docs/reference/sealed-classes.html)）
在JVM上，如果生成的类需要无参构造，则要为参数指定默认值（参考[`构造器`](https://kotlinlang.org/docs/reference/classes.html#constructors)）

```kotlin
data class User(val name: String = "", val age: Int = 0)
```
## 类的拷贝
我们常常拷贝一个对象，修改一部分属性且保留剩余的属性。`copy`函数就是为了这一目的创建。比如上面的`User`类，它的`copy`函数可能就是：
```kotlin
fun copy(name: String = this.name, age: Int = this.age) = User(name, age)    
```
然后就可以这样写：

```kotlin
val jack = User(name = "Jack", age = 1)
val olderJack = jack.copy(age = 2)
```
## 数据类与解构声明
数据类生成的组件函数可以能够用在[`解构`](https://kotlinlang.org/docs/reference/multi-declarations.html)声明中。
```kotlin
val jane = User("Jane", 35) 
val (name, age) = jane
println("$name, $age years of age") // prints "Jane, 35 years of age"
```

## 标准数据类
标准库提供了`Pair`和`Triple`。在大多数情况中，命名的数据类是更好的选择，因为具有意义的属性名，他们可以让代码更具有可读性。

# 密封类
当值是只是限定类型组中的一种时，可使用密封类表示受限的类层次。从某种意义上来说，密封类是枚举类的扩展类型：枚举类型值的限定集合，但枚举类型只能单独实例，而密封类的子类可有包含状态的多个实例。

声明密封类，在类名前加上`sealed`修饰符。密封类可以有子类，但所有子类都必须在密封类对应的文件中声明。（在Kotlin1.1之前，规则更加严格：类必须在密封类中声明）。
```kotlin
sealed class Expr
data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()
```
Kotlin 1.1新特性：数据类可继承其他类包括密封类。

> 继承密封类子类的类可以位于任何地方，不仅仅可以在同文件中。

密封类的主要好处就是在[`when表达式`](https://kotlinlang.org/docs/reference/control-flow.html#when-expression)中使用。如果能够覆盖所有条件，而不用在条件语句中添加`else`代码块。

```kotlin
fun eval(expr: Expr): Double = when(expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
    // 无须 `else` 字句，因为已经覆盖所有条件
}
```

# 嵌套类

类可以嵌套在其他类中
```kotlin
class Outer {
    private val bar: Int = 1
    class Nested {
        fun foo() = 2
    }
}

val demo = Outer.Nested().foo() // == 2
```

## 内部类
嵌套类可使用`inner`标记，访问外部类成员。内部类持有外部类引用
```kotlin
class Outer {
    private val bar: Int = 1
    inner class Inner {
        fun foo() = bar
    }
}

val demo = Outer().Inner().foo() // == 1
```

查看[`this表达式`](https://kotlinlang.org/docs/reference/this-expressions.html)解释inner类中的this问题。

## 匿名内部类

匿名内部类是使用[`对象表达式`](https://kotlinlang.org/docs/reference/object-declarations.html#object-expressions)创建的实例

```kotlin
window.addMouseListener(object: MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }
              
    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
})
```
如果对象是函数式Java接口实例（如：只有一个抽象方法的Java接口），可使用前缀接口类型的Lambda表达式。
```kotlin
val listener = ActionListener { println("clicked")}
```

# 枚举类
枚举类最基本的用法就是实现类型安全的枚举
```koltin
enum class Direction {
    NORTH, SOUTH, WEST, EAST
}
```
每个枚举常量都是一个对象，枚举常量使用逗号分隔。

## 初始化
因为每个枚举都是枚举类的实例，都可初始化。
```kotlin
enum class Color(val rgb: Int) {
        RED(0xFF0000),
        GREEN(0x00FF00),
        BLUE(0x0000FF)
}
```

## 匿名枚举类
枚举类可声明自己的匿名类
```kotlin
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },

    TALKING {
        override fun signal() = WAITING
    };

    abstract fun signal(): ProtocolState
}
```
对应的方法，与复写基本方法一致。
> 枚举类只要定义任意成员，都要使用分号区分定义枚举常量与成员。

## 枚举常量的使用

与Java相似，Kotlin的枚举常量也有综合方法来列取枚举常量，通过常量名来获取常量值。这些方法的签名如下所示（假设枚举类名为`EnumClass`）：
```kotlin
EnumClass.valueOf(value: String): EnumClass
EnumClass.values(): Array<EnumClass>
```
如果给定名称不满足枚举类中的任一枚举常量，会抛出`IllegalArgumentException`异常。

从Kotlin 1.1起，可以通过一般方法来访问枚举中常量：`enumValues<T>()`与`enumValueOf<T>()`函数
```kotlin
enum class RGB { RED, GREEN, BLUE }

inline fun <reified T : Enum<T>> printAllValues() {
    print(enumValues<T>().joinToString { it.name })
}

printAllValues<RGB>() // prints RED, GREEN, BLUE
```
每个枚举常量都有属性来获取枚举名称和在枚举类中声明的位置。

```kotlin
val name: String
val ordinal: Int
```
枚举常量实现了[`Comparable`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-comparable/index.html)接口，按照在枚举类中定的顺序自然排序。