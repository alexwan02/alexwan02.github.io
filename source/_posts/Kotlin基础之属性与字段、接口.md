---
title: Kotlin基础之字段、接口
date: 2017-07-03 15:49:07
thumbnailImage: https://pic1.zhimg.com/v2-30253c279faba2e77120862dd54d49d4_r.jpg
tags: kotlin
---
# 属性与字段
## 属性声明
Kotlin中类拥有属性。用`var`关键字声明为可变属性，或用`val`关键字声明为只读属性。

```kotlin
class Address {
    var name: String = ...
    var street: String = ...
    var city: String = ...
    var state: String? = ...
    var zip: String = ...
}
```
使用属相，只要像Java的字段一样，通过名称来进行引用。
```kotlin
fun copyAddress(address: Address): Address {
    val result = Address() // 初始化对象，无`new`关键字
    result.name = address.name // accessors are called
    result.street = address.street
    // ...
    return result
}
```

## Getter、Setter方法
声明属性的完整语法
```kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```
初始化，getter和setter都为可选项。如果属性类型可以从初始化（或从getter的返回值）中推断出，则可省略属性的`PropertyType`

```kotlin
var allByDefault: Int? // 错误：需要显式初始化。getter和setter默认
var initialized = 1  // Int类型，默认getter和setter
```
只读属性声明的语法与可变属性有两处不同：
1. val
2. 没有setter函数
```kotlin
val simple: Int? // 类型为Int，默认getter方法， 必须在构造器中初始化
val inferredType = 1 // Int类型，默认getter

```
与普通函数相似，在属性声明中，也可以自定义访问方法。如自定义`getter`
```kotlin
val isEmpty: Boolean
    get() = this.size == 0
```
自定义的`setter`
```kotlin
var stringRepresentation: String
    get() = this.toString()
    set(value) {
        setDataFromString(value) // parses the string and assigns values to other properties
    }
```
依照惯例，setter参数名为`value`，也可以选择期望的属性名。

从Kotlin 1.1起，如果可以从getter推断类型，则属性类型可以省略
```kotlin
val isEmpty get() = this.size == 0  // has type Boolean
```
如果需要改变访问器的可见性或注解属性，但不需要改变默认实现，可以不定义访问器的代码块
```kotlin
var setterVisibility: String = "abc"
    private set // setter 为 private且为默认实现

var setterWithAnnotation: Any? = null
    @Inject set // 使用Inject注解setter
```

## Backing字段
Koltin的类没有字段，有时在使用自定义访问器时有个Backing字段很有必要。为此，Kotlin提供了一个自动的backing字段，可以使用`field`修饰来访问
```kotlin
var counter = 0 // 初始化值直接写给backing字段
  set(value){
    if( value > 0 ) field = value
  }
```
`field`修饰符只能用在属性访问器中

如果使用至少一个访问器的默认实现，会为属性生成一个Backing字段，或自定义访问器通过`field`修饰符来引用。

下面情况下，就不会有Backing字段
```kotlin
val isEmpty: Boolean
    get() = this.size == 0 
```

## Backing属性
如果不满足`隐式Backing字段`条件，可以退而求其次，持有Backing属性
```kotlin
private var _table: Map<String, Int>? = null

public val table: Map<String , Int>
    get(){
        if(_table == null){
            _table = HashMap() // Type parameters are inferred
        }
        return _table ?: throw AssertionError("Set to null by another thread")
    }
```
从各个方面来看，这一点与Java相同，因为用默认getter和setter来访问属性已经过优化，不会引入函数调用开销。

## 编译时常量
编译时常量是编译时已知的属性值，使用`const`修饰符来标记为*编译时常量*。此时的属性需要完全满足以下条件
- 对象的顶层或成员
- 通过String类型值或基本类型进行初始化
- 无自定义getter

这样的属性可用于注解。
```kotlin
const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated"
@Deprecated(SUBSYSTEM_DEPRECATED) fun foo() { ... }
```

## 懒初始化属性
正常情况，非空类型的属性声明必须在构造器中声明。但是在少数情况时，这样做并不方便。如：属性可以通过依赖注入或在单元测试的`setup`方法中进行初始化，在这种情况，没办法在构造器中进行非空初始化，但仍希望在类中引用属性时，避免进行非空检查。

此时，可以使用`lateinit`修饰符标记属性
```kotlin
public class MyTest {
    lateinit var subject: TestSubject

    @SetUp fun setup() {
        subject = TestSubject()
    }

    @Test fun test() {
        subject.method()  // dereference directly
    }
}
```
这个修饰符只能用在类体（非首要构造器）中声明为`var`类型的属性，并且属性不能有自定义的`getter`和`setter`方法。属性类型必须为非空并且不能为基本类型。

访问未初始化的`lateinit`属性会抛出`访问未初始化属性`的异常。

## Overriding属性
查看[`Overriding属性`](https://kotlinlang.org/docs/reference/classes.html#overriding-properties)

## 委托属性
最常见属性的种类是从Backing字段读取（或写入）的属性。另一种是有自定义getter和setter，可实现属性任意行为的属性。介于两者之间的有些操作属性的常见模式，如：`lazy values` , `给定指定key返回map对应的值`，`访问数据库`，`通知访问的监听`等等。
这些常见行为模式可以使用[`委托属性`](https://kotlinlang.org/docs/reference/delegated-properties.html)库来实现。

# 接口
 Kotlin的接口与Java 8非常相似，包含一些抽象方法的声明和实现。与抽象类不同的是，接口无法存储状态。接口可以有属性，但是只能是抽象的或提供具体实现的访问器。
 
接口使用`interface`关键字来标识
```kotlin
interface MyInterface{
    fun bar()
    
    fun foo(){
        // optional body
    }
}
```

## 接口实现
类与对象可以实现一个或多个接口
```kotlin
class Child : MyInterface {
    override fun bar() {
        // body
    }
}
```

## 接口属性
可以在接口中声明属性。接口中声明的属性可以为抽象类型或提供具体访问器的实现。接口中声明的属性不能有Backing字段，所以接口中声明的访问器无法引用Backing字段。

```kotlin
interface MyInterface {
    val prop: Int // abstract

    val propertyWithImplementation: String
        get() = "foo"

    fun foo() {
        print(prop)
    }
}

class Child : MyInterface {
    override val prop: Int = 29
}
```

## 解决接口复写冲突
当在超类中声明多种类型时，继承这些接口时可能会出现复写相同方法情况：
```kotlin
interface A {
    fun foo() { print("A") }
    fun bar()
}

interface B {
    fun foo() { print("B") }
    fun bar() { print("bar") }
}

class C : A {
    override fun bar() { print("bar") }
}

class D : A, B {
    override fun foo() {
        super<A>.foo()
        super<B>.foo()
    }

    override fun bar() {
        super<B>.bar()
    }
}
```
接口A和接口B都声明了函数`foo`和`bar`，全部实现了`foo`，但只有B实现了`bar`。C类继承了A，所以需要复写方法`bar`并提供实现。

类D实现了A和B，需要实现A和B中的所有方法并指定`D`如何实现。这个规则适应于单一实现的继承（`bar`），也适应多实现（`foo`）。