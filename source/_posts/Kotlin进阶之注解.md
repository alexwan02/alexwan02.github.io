---
title: Kotlin进阶之注解
date: 2017-07-17 09:38:51
tags: kotlin
---
# 注解
## 注解声明
注解就是向代码中添加元数据。在类名前添加`annotation`，来声明注解。
```kotlin
annotation class Fancy
```
使用`元注解`来注解注解类，来指定注解的额外信息

- [@Target]() 
    
    表示可以注解的元素类型（类、函数、属性、表达式等）
- [@Retention ]()

    表示注解是否存储在编译后的类文件中，是否在运行时可见。（默认值都为true）
- [@Repeatable ]()

    单个元素是否可以多次使用注解
- [@MustBeDocumented ]()

    表示注解是公共API的一部分。生成的API文档是否显示类或方法包含的注解
    

```kotlin
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION,
        AnnotationTarget.VALUE_PARAMETER, AnnotationTarget.EXPRESSION)
@Retention(AnnotationRetention.SOURCE)
@MustBeDocumented
annotation class Fancy
```

## 使用

```kotlin
@Fancy class Foo {
    @Fancy fun baz(@Fancy foo: Int): Int {
        return (@Fancy 1)
    }
}

```

如果要注解类的首要构造器，则要使用`constructor`关键字，并在其前面添加注解

```kotlin

class Foo @Inject constructor(dependency: MyDependency) {
    // ...
}
```
也可以注解属性访问器
```kotlin
class Foo{
    var x: MyDependency? = null 
        @Inject set
}
```

## 构造器

注解类可以声明有参构造
```kotlin
annotation class Special(val why: String)

@Special("example") class Foo {}
```
可使用的参数类型有:
- Java的基本类型对应的Kotlin类型
- 字符串
- 枚举
- 其他注解
- 上面类型的数组

注解参数不能为空类型，因为`JVM`不支持存储`null`作为注解属性值

如果注解为其他注解的参数，则注解名没有`@`前缀
```kotlin
annotation class ReplaceWith(val expression: String)

annotation class Deprecated(
        val message: String,
        val replaceWith: ReplaceWith = ReplaceWith(""))

@Deprecated("This function is deprecated, use === instead", ReplaceWith("this === other"))
```
如果要给注解指定class类型参数，使用Koltin的[KClass](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html)。Kotlin编译器会将它自动转为Java类，这样Java代码就可以正常访问注解和参数
```kotlin
import kotlin.reflect.KClass

annotation class Ann(val arg1: KClass<*>, val arg2: KClass<out Any?>)

@Ann(String::class, Int::class) class MyClass
```


## Lambdas
Lambdas表达式也可以使用注解。编译会在Lambda表达式中生成`invoke()`方法，对像[ Quasar](http://www.paralleluniverse.co/quasar/)并发控制等框架有用
```kotlin
annotation class Suspendable

val f = @Suspendable { Fiber.sleep(10) }
```
## 使用处目标注解
在注解属性或首要构造器的参数时，会生成对应Kotlin元素的多个Java元素，因此注解有多个位置来生成Java字节码。使用下面语法规定要生成的具体注解
```kotlin
class Example(@field: Ann val foo  , // java字段注解
              @get:Ann val bar ,   // java getter注解
              @param:Ann val quux)  // java 构造器参数注解
```

还可以使用相同语法注解整个文件。在文件顶部，在包声明或所有默认包导入的文件之前，使用`@file`

```kotlin
@file:JvmName("Foo")

package org.jetbrains.demo
```

如果相同目标有多个注解，在目标之后使用方括号来添加所有注解
```kotlin
class Example {
    @set:[Inject VisibleForTesting]
    var collaborator: Collaborator
}
```
支持的使用处目标
- file
- property（Java不可见）
- field
- get（属性getter）
- set（属性setter）
- receive（扩展函数参数接收者或属性）
- param（构造参数）
- setParam（属性setter参数）
- delegate（存储委托属性的委托实例）

注解扩展函数接收者参数语法
```kotlin
fun @receiver:Fancy String.myExtension() { }
```
如未指定使用处目标，则选择使用`@Target`注解的目标。如果有多个可用目标，则使用以下中一个可用目标
- param
- property
- field

## Java注解
Java注解完全适配Kotlin
```kotlin
import org.junit.Test
import org.junit.Assert.*
import org.junit.Rule
import org.junit.rules.*

class Tests{
    // @Rule注解
    @get:Rule val tempFolder = TemporaryFolder()
    
    @Test fun simple() {
        val f = tempFolder.newFile()
        assertEquals(42, getTheAnswer())
    }
}
```

因为Java未定义注解参数顺序，因此无法使用传递参数的常规函数调用，使用命名参数替代
```java
// Java
public @interface Ann {
    int intValue();
    String stringValue();
}
```

```kotlin
// Kotlin
@AnnWithValue("abc") class C
```

如果Java中`value`值为数组类型，则在Kotlin中变为`vararg`类型
```java
// Java
public @interface AnnWithArrayValue {
    String[] value();
}
```

```kotlin
// Koltin
@AnnWithArrayValue("abc" , "foo" , "bar") class C
```
对于其他数组类型的参数，则要显式使用`arrayOf`
```java
// Java
public @interface AnnWithArrayMethod {
    String[] names();
}
```

```kotlin
// Kotlin
@AnnWithArrayMethod(names = arrayOf("abc", "foo", "bar")) class C
```
Java注解实例的值作为属性暴露给Kotlin代码

```java
// Java
public @interface Ann {
    int value();
}
```

```kotlin
// Kotlin
fun foo(ann: Ann) {
    val i = ann.value
}
```