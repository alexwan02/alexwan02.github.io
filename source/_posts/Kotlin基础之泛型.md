---
title: Kotlin基础之泛型
date: 2017-07-03 15:49:52
thumbnailImage: https://pic1.zhimg.com/v2-30253c279faba2e77120862dd54d49d4_r.jpg
tags: kotlin
---
# 泛型
与Java一样，Koltin的类也有类型参数
```kotlin
class Box<T>(t: T){
    var value = t
}
```
常规来说，创建这样的类，需要提供具体的类型
```kotlin
val box: Box<Int> = Box<Int>(1)
```
当类型可以从构造参数或其他上下文中推断出时，可以忽略类型参数
```kotlin
val box = Box(1) // 1 has type Int, so the compiler figures out that we are talking about Box<Int>
```
## 变型
Java类型系统中最复杂的其中一个部分就是通配符类型（[`Java泛型FAQ`](http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html)）。而Kotlin没有任何的通配符类型，它使用`声明处变型`和`类型投影`两种方式替代。

通配符 - 使用问号表示的类型参数，表示未知类型的类型约束方法。

首先，先思考为什么Java需要这些难以理解的通配符。[`Effective Java`](http://www.oracle.com/technetwork/java/effectivejava-136174.html)解释了这个问题，第28条：使用受限通配符来增加API灵活性。首先，Java中泛型为不可变类型，意味`List<String>`不是`List<Object>`的子类型。为什么这样？如果List为可变量，`List<>`不会比Java的数组更好，并且下面的代码能够成功编译，但在运行时会引起异常。
```kotlin
// Java
List<String> strs = new ArrayList<String>();
List<Object> objs = strs; // 会引起错误，Java禁止这样使用。
objs.add(1);  // Here we put an Integer into a list of Strings
String s = strs.get(0); // 类转换异常：无法将Integer转换为String

```
所以Java禁止这样做，目的是保证运行时安全。但会有一些影响。比如：`Collection`接口的`addAll()`方法，这个方法的签名是什么？我们直觉上会这样做：
```kotlin
// Java
interface Collection<E> ... {
  void addAll(Collection<E> items);
}
```
考虑到运行时安全，我们无法做到像下面的简单操作。
```kotlin
// Java
void copyAll(Collection<Object> to, Collection<String> from) {
  to.addAll(from); // !!! Would not compile with the naive declaration of addAll:
                   //       Collection<String> is not a subtype of Collection<Object>
}
```
> 在Java中学到的教训，就是[`Effective Java 第25条`](http://www.oracle.com/technetwork/java/effectivejava-136174.html)：Prefer lists to arrays

实际上，`addAll()`的方法签名是：
```kotlin
// Java
interface Collection<E> ... {
  void addAll(Collection<? extends E> items);
}
```
通配符参数`? extends E`表明方法接收类型为`E`的子类集合，而非`E`本身。意味着可以安全读取集合中为`E`的值（集合的元素类型为`E`的子类实例），但无法写入`E`，因为我们不知道对象是否是E未知的子类。作为交换，我们希望得到这些行为：`Collection<String>`为`Collection<? extends Object>`子类型，带有`extends`或`upper`限制的通配符让类型变为[`协变`](https://msdn.microsoft.com/zh-cn/library/dd799517.aspx)。

理解这点很简单：如果只从集合中获取item，可使用`String`的集合来读取`Object`。相反，如果只向集合放入item，可以向`Object`集合中放入`String`类型。Java中使用`List<Object>`表示`List<? super String>`的超类。

后一个被称为协变量，只能调用基于`List<? super String>`，且使用`String`作为参数的方法（如：可以调用`add(String)`或`set(int, String)`）,而如果你调用`List<T>`返回的`T`，只能获取到Object类型，而不是`String`。

`Joshua Bloch`成只能读取的对象为`生产者`，只能写入的对象为`消费者`。他建议:`为了最大化灵活性，在表示消费者和生产者的输入参数上使用通配符类型`，并提供下面的话来方便记忆：

`PECS`：表示Producer-Extends， Consumer-Super。

> 如果使用生产者对象，比如说`List<? extends Foo>`，不能在这个对象上调用`add()`或`set`方法,但并不表示这个对象是不变的：如不会禁止调用`clear()`来清空list，因为`clear()`没有任何参数。通配符（或其他变型的类型）能够保证类型安全，非可变则是完全不同的说法。

## 声明点变型

假设现有一个`Source<T>`泛型接口，没有使用`T`作为参数的方法，只有一个返回`T`的方法
```kotlin
// Java
interface Source<T> {
  T nextT();
}
```
那么使用`Source<Object>`的变型来存储`Source<String>`实例引用是类型安全的（因为没有消费者方法）。但是Java仍会禁止这样做：

```java
// Java
void demo(Source<String> strs) {
  Source<Object> objects = strs; // !!! Not allowed in Java
  // ...
}
```
可以声明`Source<? extends Object>`来解决。

在Kotlin中，使用`声明点变型`(declaration-site variance)向编译器解释。使用`out`修饰符注解类型`T`,确保只返回（生产者）`Source<T>`成员，而从不写入（消费）。
```kotlin
abstract class Source<out T> {
    abstract fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // This is OK, since T is an out-parameter
    // ...
}
```
泛型规则：当类`C`的泛型参数`T`声明为`out`时，表示T只能出现在C成员的输出位置，作为交换，`C<Base>`是`C<Derived>`类型安全的超类。

称类`C`是参数`T`的协变量，或`T`是协变量类型参数。可以认为类`C`是`T`的生产者，而不是`T`的消费者。

`out`修饰符称为变型注解，因为它提供了类型参数声明点，因此称之为声明点类型。

除了`out`，kotlin提供了一个补充的变型注解：`in`。让类型参数变为`逆变量`：只能消费，从不生产。`Comparable`就是`协变量`一个很好的例子。
```kotlin
abstract class Comparable<in T> {
    abstract fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0) // 1.0 has type Double, which is a subtype of Number
    // Thus, we can assign x to a variable of type Comparable<Double>
    val y: Comparable<Double> = x // OK!
}

```
## 类型投影
使用处变型：类型投影
声明类型参数T为*out*很方便，避免在使用处子类型化。但一些类实际时无法限制只返回`T`，`Array`就是一个很好的例子：
```kotlin
class Array<T>(val size: Int){
    fun get(index: Int): T{ /*...*/}
    fun set(index: Int , value: T){/*...*/}
}
```
`Array`类既不是T的协变，也不是T的逆变，导致不够灵活。考虑下面的函数：
```kotlin
fun copy(from: Array<Any> , to: Array<Any>){
    assert(from.size == to.size)
    for(i in from.indices)
        to[i] = from[i]
    
}
```
函数应该是从拷贝数组中数据到另一个数组，下面将函数用在实际中：
```kotlin
val ints: Array<Int> = arrayOf(1, 2, 3)
val any = Array<Any>(3){ ""}
copy(ints , ant) // 错误：expects(Array<Any>, Array<Any>)
```
遇到了相同的问题：`Array<T>`是不变的，`T`类型的数组，所以`Array<Int>`和`Array<Any>`都不是对方的子类。因为copy可能会坏事，可能会进行写操作，比如像`from`写入String，而实际上这里传入的是`Int`数组，运行时就能出现`ClassCastException`异常。

因此，只需要保证`copy`不会做坏事，禁止向`from`写数据，可以这样做：
```kotlin
fun copy(from: Array<out Any> , to: Array<Any>){
    //...
}
```
这样做法被称为`类型投影`(type projection)，也是说`from`不是一个简单数组，而是受限（投影）类型：只能够调用那些返回类型为`T`的方法，在这种情况意味着只能调用`get`，这也是使用`使用出变型`的目的，对应java的`Array<? extends Object>`，但是是通过一个更为简单的方法。

也可以使用`in`来投影类型
```kotlin
fun fill(dest: Array<in String> , value: String){
    // ...
}
```

`Array<in String>`对应Java的`Array<? super String>`。如向`fill`函数传入`CharSequence`或`Object`数组。


## 星号投影
有时不知道类型参数任何信息，但仍希望安全地使用。此时安全地定义投影的泛型，每个泛型的具体实例都是泛型的子类型。
为此，Kotlin提供称为`星号投影`的语法
- 对于`Foo<out T>`，`T`为带有上界`TUpper`的协变量，`Foo<*>`等价于`Foo<out TUpper>`。意味着`T`类型未知时，可以安全地读取`Foo<*>`中`TUpper`的值
- 对于`Foo<in T>`，`T`为逆变类型参数，`Foo<*>`等价于`Foo<in Nothing>`，意味着当`T`类型未知时，无法安全写入`Foo<*>`
- 对于`Foo<T>`，`T`为不可变类型参数，带有上界`TUpper`，`Foo<*>`等价于`Foo<out TUpper>`用于读取和`Foo<in Nothing>`用于写入值。

如果泛型有多个类型参数，则每个都可以独立投影。比如，如果类型声明为`interface Function<in T, out U>`，可以设想下面的星号投影

- `Function<*, String>` 意味 `Function<in Nothing, String>`
- `Function<Int, *> ` 意味 `Function<Int, out Any?>`
- `Function<*, *>` 意味 `Function<in Nothing, out Any?>`

> 星号投影与Java`raw`类型相似，但是安全。

## 泛型函数
不仅类可以有类型参数，函数也可以有。函数的类型参数在函数名之前声明：
```kotlin
fun <T> SingletonList(item: T ): List<T>{
    // ...
}

fun <T> T.basicToString() : String { // 扩展函数
    // ...
}
```
调用泛型函数，在调用的函数名之后指定具体类型参数
```kotlin
val l = SingletonList<Int>(1)
```

## 泛型约束

所有可以被指定类型参数替代的类型，都可以使用泛型约束进行限制。

## 上界
最常见的泛型约束就是上界，对应java的extends关键字

```kotlin
fun <T : Comparable<T>> sort(list: List<T>){
    // ...
}
```
在冒号之后指定的类型就是上界，只有`Comparable<T>`子类才能替换`T`。如：

```kotlin
sort(listOf(1, 2, 3)) // 可以。Int是Comparable<Int>的子类
sort(listOf(HashMap<Int, String>()))  // 错误。HashMap<Int, String>不是Comparable<HashMap<Int, String>>的子类
```
默认上界类型为`Any?`。尖括号中只允许指定一个上界。可使用`where`条件语句指定超过一个的上界

```kotlin
fun <T> cloneWhenGreater(list: List<T> , threshold: T): List<T> 
    where T : Comparable ,
          T : Cloneable {
    return list.filter{it -> threshold }.map { it.clone()}              
}
```

## 参考

[里氏替换原则](https://zh.wikipedia.org/wiki/%E9%87%8C%E6%B0%8F%E6%9B%BF%E6%8D%A2%E5%8E%9F%E5%88%99)

[协变与逆变](https://zh.wikipedia.org/wiki/%E5%8D%8F%E5%8F%98%E4%B8%8E%E9%80%86%E5%8F%98)

[泛型中的协变和逆变](https://msdn.microsoft.com/zh-cn/library/dd799517.aspx)