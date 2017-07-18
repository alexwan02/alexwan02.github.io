---
title: Kotlin进阶之与Java交互
date: 2017-07-18 17:38:17
tags: kotlin
---
# Kotlin调用Java
Kotlin可以自然地调用Java代码，同样Java也可以丝滑般调用Kotlin代码。这节详细描述kotlin调用Java代码。

可以使用几乎所有Java代码。
```kotlin
import java.util.*

fun demo(source: List<Int>) {
    val list = ArrayList<Int>()
    // 'for'-loops work for Java collections:
    for (item in source) {
        list.add(item)
    }
    // Operator conventions work as well:
    for (i in 0..source.size() - 1) {
        list[i] = source[i] // get and set are called
    }
}
```
## Getters和Setters
按照Java约定`getter`和`setter`方法作为Kotlin的属性。`Boolean`访问者方法（getter方法以`is`开头，setter以`set`开头）属性与getter方法名相同。

```kotlin
import java.util.Calendar

fun calendarDemo() {
    val calendar = Calendar.getInstance()
    if (calendar.firstDayOfWeek == Calendar.SUNDAY) {  // call getFirstDayOfWeek()
        calendar.firstDayOfWeek = Calendar.MONDAY       // call setFirstDayOfWeek()
    }
}
```
> 如果Java类只有一个setter方法，则在Kotlin中不能视为属性。因为Kotlin现在不支持只有set的属性。

## 返回void的方法
如果Java方法返回void，则在Kotlin中返回`Unit`。万一使用了返回值，则Kotlin编译器在调用处对其赋值（Unit）。

## 对Kotlin中的关键字的Java标识符的转义
Kotlin中一些关键字（如`in`、`object`、`is`等）是Java的标识符。如果Java库中方法使用Kotlin关键字，人可以使用(==`==)转义：
```kotlin
foo.`is`(bar)
```

## 空安全与平台类型
Java中任何引用都可能为`null`，让Kotlin空安全严格的需求不适应于Java中的对象。Kotlin特殊对待Java声明的类型，称其为`平台类型`。对这些类型，空检查不那么严格，所以它们的安全保证与在Java一样

假设以下方法
```kotlin
val list = ArrayList<String>() // 非空
list.add("Item")
val size = list.size()         // 非空（原始int）
val item = list[0]            //  平台类型推断（原始Java对象）
```

当在调用平台类型变量的方法时，Kotlin不会在编译时报错，但是在运行时可能会出错。因为空指针或Kotlin生成的断言禁止传播空值

```kotlin
items.substring(1) // 允许，如果item == null 时抛出异常
```

平台类型是不可表示的，意味着无法在语言中明确指明。当给Kotlin变量赋值平台时，依赖类型推断，或者选择期望的类型（非空和可空类型都允许）

```kotlin
val nullable: String? = item  // 允许
val notNull: String = item    // 允许，运行时可能出错
```
如果选择非空类型，编译器会在赋值上生成一个断言。保证Kotlin非空类型持有空值，当将平台类型值传给期望非空类型值的函数时，也会生成断言。以上，编译尽可能控制`空值`在程序中传播（尽管因为泛型，不能够完全消除空值）
### 平台类型标记
如上所述，平台类型无法在程序中显示标记，所以语言中没有相关的语法。不过，编译器和IDE有时需要显示这些类型（错误信息，参数信息等等），所以Kotlin提供了一些助记符

- `T!`表示`T` 或`T?`
- `(Mutable)Collection<T>!`表示类型为`T`的Java集合可变或不变，可能为空或不为空
- `Array<(out) T>!`表示类型为`T`或`T的子类`的Java集合，可以为空或不为空

### 可空性注解
有可空性注解的Java类型不会作为平台类型处理，而是实际的可为空或非空的Kotlin类型。编译器支持以下几种可空性注解

- [JetBrains](https://www.jetbrains.com/idea/help/nullable-and-notnull-annotations.html)（`@Nullable`和`@NotNull`）
- Android（`com.android.annotations`和`android.support.annotations`）
- JSR-305（`javax.annotation`）
- FindBugs（`edu.umd.cs.findbugs.annotations`）
- Eclipse（`org.eclipse.jdt.annotation`）
- Lombok（`lombok.NonNull`）

[Kotlin编译器源码](https://github.com/JetBrains/kotlin/blob/master/core/descriptor.loader.java/src/org/jetbrains/kotlin/load/java/JvmAnnotationNames.kt)提供完整列表

## 映射类型
Kotlin对Java一些类型特殊对待，并不照样加载，而是映射成Kotlin对应的类型。映射只在编译时进行，不影响运行时。

Java的基本类型映射为对应的Kotlin类型（牢记[平台类型](https://kotlinlang.org/docs/reference/java-interop.html#null-safety-and-platform-types)）

Java type | Kotlin type
---|---
byte | kotlin.Byte
short | kotlin.Short
int | kotlin.Int
long | kotlin.Long
char | kotlin.Char
float | kotlin.Float
double | kotlin.Double
boolean | kotlin.Boolean

非基本类型的固定类

Java type | Kotlin type
---|---
java.lang.Object | kotlin.Any!
java.lang.Cloneable |	kotlin.Cloneable!
java.lang.Comparable |	kotlin.Comparable!
java.lang.Enum	| kotlin.Enum!
java.lang.Annotation |	kotlin.Annotation!
java.lang.Deprecated |	kotlin.Deprecated!
java.lang.CharSequence |	kotlin.CharSequence!
java.lang.String |	kotlin.String!
java.lang.Number |	kotlin.Number!
java.lang.Throwable	| kotlin.Throwable!

Java装箱的基本类型映射为Kotlin的可空类型

Java type | Kotlin type
---|---
java.lang.Byte	| kotlin.Byte?
java.lang.Short	| kotlin.Short?
java.lang.Integer	| kotlin.Int?
java.lang.Long	| kotlin.Long?
java.lang.Character	| kotlin.Char?
java.lang.Float	| kotlin.Float?
java.lang.Double	| kotlin.Double?
java.lang.Boolean	| kotlin.Boolean?

> 装箱的私有类型可作为映射为平台类型的类型参数：如`List<java.lang.Integer>`在Kotlin中变为`List<Int!>`

集合类型在Kotlin变为`只读`或`可变`，所以Java类型按照下面规则进行映射（表中所有Kotlin类型属于包`kotlin.collections`）

Java type |	Kotlin read-only type |	Kotlin mutable type	 | Loaded platform type
--- | --- | --- | ---
Iterator<T>	| Iterator<T>	| MutableIterator<T>	| (Mutable)Iterator<T>!
Iterable<T>	| Iterable<T>	| MutableIterable<T>	| (Mutable)Iterable<T>!
Collection<T>	| Collection<T>	| MutableCollection<T>	| (Mutable)Collection<T>!
Set<T>	| Set<T>	| MutableSet<T>	| (Mutable)Set<T>!
List<T>	| List<T>	| MutableList<T>	| (Mutable)List<T>!
ListIterator<T>	| ListIterator<T>	| MutableListIterator<T>	| (Mutable)ListIterator<T>!
Map<K, V> | Map<K, V>	| MutableMap<K, V>	| (Mutable)Map<K, V>!
Map.Entry<K, V>	| Map.Entry<K, V>	| MutableMap.MutableEntry<K,V> | 	(Mutable)Map.(Mutable)Entry<K, V>!

Java数组按照下面规则映射

Java type | Kotlin type
---|---
int[]	| kotlin.IntArray!
String[]	| kotlin.Array<(out) String>!

## Kotlin中的Java泛型
Kotlin的泛型与Java稍微不一样（参考[泛型](https://kotlinlang.org/docs/reference/generics.html)）。在Kotlin中导入Java类型时，要进行一些转换

- Java通配符转换成类型投影
    - `Foo<? extends Bar>` 变为`Foo<out Bar!>!`
    - `Foo<? super Bar> ` 变为 `Foo<in Bar!>!`
- Java 原始类型转换为星投影
    - `List`变为`List<*>!`或`List<out Any?>!`

与Java一样，Kotlin运行时不保留泛型（如对象没有传入构造器的类型参数的信息），泛型擦除。`ArrayList<Integer>()`与`ArrayList<Character>()`两者无法区分。所以无法对带泛型类型执行`is`检查，只支持星投影泛型类型的`is`检查
```kotlin
if (a is List<Int>) // 错误：无法检查Int数组

if (a is List<*>)   // 可以：数组内容没有保证
```

## Java数组

Kotlin中数组都是不变的，表示无法将`Array<String>`赋值给`Array<Any>`，防止可能的运行出错。Kotlin同时禁止将子类数组作为超类数组参数传递给方法，但是Java允许（通过`Array<(out) String>!`的[平台类型](platform types)形式）

Java使用基本类型数组避免装箱和拆箱操作消耗，而Kotlin隐藏这些实现细节，所以需要一个变通方案来与Java代码结合：为每个基本类型提供了特殊类（`IntArray`，`DoubleArray`，`CharArray`等），与`Array`没有关系，为了性能考虑最终编译为Java基本类型

假设有接收Int类型数组作为参数的Java方法：
```java
public class JavaArrayExample{
    public void removeIndices(int[] indices){
        // ...
    }
}
```

在Kotlin中调用：
```kotlin
val javaObj = JavaArrayExample()
val array = intArrayOf(0, 1, 2, 3)
javaObj.removeIndices(array)  // 传入int[]
```

当编译为JVM字节码时，编译器将进行优化
```kotlin
val array = arrayOf(1, 2, 3, 4)

array[x] = array[x] * 2  //  

for( x in array){   // 不会创建迭代器
    print(x)
}
```

最后，`in`检查也不会有额外开销
```kotlin
if(i in array.indices){  // 与(i >= 0 && i < array.size)相同
    print(array[i])
}
```
## Java可变参数
Java有时使用可变参数声明方法
```kotlin
public class JavaArrayExample{
    public void removeIndices(int... indices){
        // ...
    }
}
```
在Kotlin中使用`*`扩展操作符传入`IntArray`
```kotlin
val javaObj = JavaArrayExample()
val array = intArrayOf(0, 1, 2, 3)
javaObj.removeIndicesVarArg(*array)
```

目前不支持传入`null`值给参数为可变参数的方法

## 操作符

因为Java没有使用操作符语法标记方法的方式，Kotlin允许使用正确的名称和签名的方法作为操作符重载和转换（如`invoke()`等），不允许使用中缀调用语法调用Java方法。

## 检查型异常
Kotlin不支持检查型异常，意味着编译器不会强制捕获检查型异常。所以当调用声明为检查型异常的方法时，Kotlin不会强制做任何检查：
```kotlin
fun render(list: List<*> , to: Appendable){
    for(item in list){
        to.append(item.toString())   // 在Java中需要捕获 IOException 异常
    }
}
```

## 对象方法
Kotlin导入Java类型时，所有`java.lang.Object`类型转为`Any`。因为`Any`非平台特定，只声明了`toString()`，`hashCode()`，`equals()`成员函数，所有创建其他`java.lang.Object`对应成员函数，Kotlin使用[扩展函数](https://kotlinlang.org/docs/reference/extensions.html)方式来实现

### wait()/notify()
Effective Java[第69条](http://www.oracle.com/technetwork/java/effectivejava-136174.html)建议并发工具优先于`wait()`和`notify()`，所以`Any`类型没有这些方法。如果需要使用，则可以将对象强转为`java.lang.Object`类型

```kotlin
(foo as java.lang.Object).wait()
```
### getClass()
检索对象的Java类型，使用[类引用](https://kotlinlang.org/docs/reference/reflection.html#class-references)的`java`扩展属性
```kotlin
val fooClass = foo::class.java
```
上面代码使用Kotlin 1.1 引入的[约束类引用](https://kotlinlang.org/docs/reference/reflection.html#bound-class-references-since-11)。也可以使用`javaClass`扩展属性

```kotlin
val fooClass = foo.javaClass
```

### clone()
继承`kotlin.Cloneable`复写`clone()`

```kotlin
class Example : Cloneable {
    override fun clone(): Any { ... }
}
```
谨记[Effective Java第11条](http://www.oracle.com/technetwork/java/effectivejava-136174.html)：慎重复写`clone`

### finalize()
以声明`finalize`方式复写`finalize()`，不使用`override`关键字
```kotlin
class C {
    protected fun finalize() {
        // finalization logic
    }
}
```
根据Java规则，`finalize()`不能为`private`

## 继承Java类
Kotlin中类最多只能继承一个Java父类，但可实现多个接口

## 静态成员访问
Java类静态成员为`伴生对象`，不能把`伴生对象`作为值传入，但可以直接显式访问这个成员
```kotlin
if (Character.isLetter(a)) {
    // ...
}
```
## Java反射
Java反射同样对Kotlin类有效，反之亦然。上面提到过可以使用`instance::class.java`、`ClassName::class.java`、`instance.javaClass`获取`java.lang.Class`进行反射

包括获取java的`setter`/`getter`方法、kotlin属性的`backing`字段。`KProperty`表示Java字段，`KFunction`表示Java方法或构造器。

## SAM转换
与Java8一样，Kotlin支持SAM（Single Abstract Method）转换，也就是说Kotlin函数字面量可以自动转换为只有单独一个方法的Java接口的实现，只要接口方法参数类型符合Kotlin函数的参数类型

可以用来创建SAM接口实例
```kotlin
val runnable = Runnable { println("This runs in a runnable") }
```

调用
```kotlin
val executor = ThreadPoolExecutor()
// Java签名：void execute(Runnable command)
executor.execute { println("This runs in a thread pool") }
```
如果Java类中多个获取函数接口的方法， 可以选择其中一个调用的方法，通过一个适配器函数将Lambda转换为指定的SAM类型。编译器也会生成所需的适配器函数

```kotlin
executor.execute(Runnable { println("This runs in a thread pool") })
```
> SAM转换只适应于接口，不能用于抽象类。

> 只对与Java交互有效；因为Kotlin本身有函数类型，Kotlin接口的实现的自动转换没有必要，所以不支持。

## 在Kotlin中使用JNI
Kotlin使用`external`表示函数是native(C/C++)代码实现。
```kotlin
external fun foo(x: Int): Double
```

# Java调用Kotlin
Java也可以轻松调用Kotlin代码
## 属性
kotlin属性编译成Java
- Getter方法，`get`开头的方法
- Setter方法，`set`开头的方法（只有`var`属性有）
- private 字段，字段名与属性名相同（有backing字段的属性）

如`var firstName: String`会被编译为
```kotlin
private String firstName;

public String getFirstName(){
    return firstName;
}
public void setFirstName(String firstName) {
    this.firstName = firstName;
}
```

如果属性名以`is`开头，则使用不同映射规则：
- Getter方法，名称与属性名一致
- Setter方法，`is`替换为`set`

比如`var isOpen: Boolean`
```kotlin
private boolean isOpen

public boolean isOpen(){
    return isOpen;
}

public void setOpen(boolean isOpen){
    this.isOpen = isOpen;
}
```

这个规则适应于任意属性，不仅仅是Boolean

## 包级函数
包`org.foo.bar`中文件`example.kt`声明的所有函数和属性（包括扩展函数）编译成名为`org.foo.bar.ExampleKt`的Java类的静态方法。

```kotlin
// example.kt
package demo

class Foo

fun bar() {
}
```

```kotlin
// Java
new demo.Foo();
demo.ExampleKt.bar();
```

使用`@JvmName`注解指定生成类的名称
```kotlin
@file:JvmName("DemoUtils")

package demo

class Foo

fun bar() {
}
```

```java
// Java
new demo.Foo();
demo.DemoUtils.bar();
```

不允许有多个相同（包相同且类名相同或`@JvmName `注解名相同）Java类名文件，所以编译器只生成单个指定名称的Java外观类，这个类包含所有同名文件中的声明。Kotlin使用`@JvmMultifileClass`注解来生成这个类文件。

```kotlin
// oldutils.kt
@file:JvmName("Utils")
@file:JvmMultifileClass

package demo

fun foo() {
}
```

```kotlin
// newutils.kt
@file:JvmName("Utils")
@file:JvmMultifileClass

package demo

fun bar() {
}
```

Java调用
```kotlin
// Java
demo.Utils.foo();
demo.Utils.bar();
```
## 实例字段
使用`@JvmField`注解，将Kotlin属性作为字段暴露给Java。字段具有与基本属性相同的可见性。

如果属性有`backing`字段、非private、没有`open`、`override`或`const`修饰符、并且不是委托属性，那么也可使用`@JvmField`注解

```kotlin
// Kotlin
class C(id: String){
    @JvmField val ID = id
}
```
Java调用
```java
// Java
class JavaClient {
    public String getID(C c) {
        return c.ID;
    }
}
```
[懒初始化](https://kotlinlang.org/docs/reference/properties.html#late-initialized-properties)属性也可以暴露给字段，字段的可见性与`lateinit`属性setter方法相同。

## 静态字段
声明在具名对象或伴生对象中的属性，则具名对象或包含伴生对象的类中有静态`backing`字段。

这些字段通常为`private`，但也可以通过以下其中一个方式暴露给Java
- `@JvmField`注解
- `lateinit`修饰符
- `const`修饰符

使用`@JvmField`生成一个相同可见性的静态字段
```kotlin
class Key(val value: Int) {
    companion object {
        @JvmField
        val COMPARATOR: Comparator<Key> = compareBy<Key> { it.value }
    }
}
```
Java调用

```java
// Java
Key.COMPARATOR.compare(key1, key2);
// public static final 字段

```

对象或伴生对象中的[懒初始化](https://kotlinlang.org/docs/reference/properties.html#late-initialized-properties)对象有与setter方法相同可见性的静态`backing`字段

```kotlin
object Singleton{
    lateinit var provider: Provider
}
```
Java调用

```java
// Java
Singleton.provider = new Provider();
// public static 非final字段
```

`const`注释的属性（类中或顶层）在Java变为静态字段：
```kotlin
// file example.kt
object obj{
    const val CONST = 1
}

class C {
    companion object {
        const val VERSION = 9
    }
}

const val MAX = 239
```
在Java中
```java
int c = obj.CONST
int d = ExampleKt.MAX
int v = C.VERSION
```
## 静态方法
Kotlin将包级函数作为静态方法处理。使用`@JvmStatic`注解定义在具名对象或伴生对象中的函数，Kotlin编译器则会在对象对应的闭合类中和对象内部生成一个静态方法
```kotlin
class C{
    companion object {
        @JvmStatic fun foo(){}
        fun bar(){}
    }
}
```
在Java中

```java
C.foo();  // OK
C.bar();  // 错误：非静态方法
C.Companion.foo() // 实例方法
C.Companion.bar(); // 使用bar的唯一方式
```
对具名对象同样有效
```kotlin
object Obj{
    @JvmStatic fun foo(){}
    fun bar(){}
}
```
Java调用

```java
Obj.foo(); // 有效
Obj.bar(); // 无效
Obj.INSTANCE.bar(); // 有效，通过单例调用
Obj.INSTANCE.foo(); // 同样有效
```
`@JvmStatic`注解也可以用在对象或伴生对象的属性上，这个属相需要让它的`setter`和`getter`方法成为对象或包含伴生对象的类的静态成员。
## 可见性
Kotlin可见性映射为Java规则
- `private`成员编译为`private`成员
- `private`顶层声明编译为包本地声明
- `protected`仍然是`protected`（Java允许相同包的类访问protected成员，Kotlin则不能，Java扩大了代码的可见性）
- `internal`编译为`public`。依据Kotlin规则：`internal`类成员经过名称重编，防止Java意外调用；允许
重载相同签名的成员但互不可见。

## KClass
有时需要调用参数类型为`KClass`的Kotlin方法，`Class`不会自动转为`KClass`类型，需要手动调用等价的`Class<T>.kotlin`类型

```kotlin
kotlin.jvm.JvmClassMappingKt.getKotlinClass(MainView.class)
```

## @JvmName处理签名冲突
Kotlin中具名函数需要在JVM字节码中具有不同名称。常见*泛型擦除*引起的函数签名冲突问题
```kotlin
fun List<String>.filterValid(): List<String>
fun List<Int>.filterValid(): List<Int>
```

无法同时定义上面两个函数，因为JVM签名相同：`filterValid(Ljava/util/List;)Ljava/util/List;`。使用`@JvmName`注解指定不同函数名
```kotlin
fun List<String>.filterValid(): List<String>

@JvmName("filterValidInt")
fun List<Int>.filterValid(): List<Int>
```

Kotlin使用相同函数名`filterValid`调用函数，Java则是`filterValid`和`filterValidInt`

还可以用于属性名为`x`且有对应函数`getX()`
```kotlin
val x: Int
    @JvmName("getX_prop")
    get() = 15
fun getX() = 10
```

## 生成重载
如果在kotlin声明带有默认值参数的函数，对于Java只能看到带有所有参数的完整签名。使用`@JvmOverloads`注解可以暴露给Java多个重载调用
```kotlin
@JvmOverloads fun f(a: String, b: Int = 0, c: String = "abc") {
    ...
}
```
每多有一个具有默认值的参数，就会额外生成一个重载方法。生成的重载方法移除指定参数右侧所有参数

```java
// Java
void f(String a, int b, String c) { }
void f(String a, int b) { }
void f(String a) { }
```
注解同样适用于构造器、静态方法等。不能用于抽象方法（包括接口中声明的方法）
> 如[次要构造器](https://kotlinlang.org/docs/reference/classes.html#secondary-constructors)描述，如果所有构造器参数都具有默认参数，即使没有指定`@JvmOverloads`注解，也会生成一个无参构造器。

## 检查型异常
Kotlin没有检查型异常，正常情况下Kotlin函数的Java签名没有异常抛出的声明。所以如果Kotlin有以下函数
```kotlin
// example.kt
package demo

fun foo() {
    throw IOException()
}
```

Java调用时，捕获异常
```kotlin
// Java
try {
  demo.Example.foo();
}
catch (IOException e) { // 错误: foo() 没有声明抛出IOException异常
  // ...
}
```

Java编译出错，因为`foo()`没有声明抛出IOException。使用`@Throws`注解解决这个问题：
```kotlin
@Throws(IOException::class)
fun foo() {
    throw IOException()
}
```
## 空安全
在Java调用Kotlin函数时，无法防止将`null`作为非空参数传递给函数。所以Kotlin为所有期望非空参数的public函数生成运行时检查。这样会在Java代码中立即出现`NullPointerException`异常。

## 可变泛型
当Kotlin使用[声明处变量](https://kotlinlang.org/docs/reference/generics.html#declaration-site-variance)时，则在Java中有两种可选的使用方式。假设在Koltin的类声明了下面两个函数：
```kotlin
class Box<out T>(val value: T)

interface Base

class Derived: Base

fun boxDerived(value: Derived): Box<Derived> = Box(value)
fun unboxBase(box: Box<Base>): Base = box.value
```

想当然地转为Java代码
```kotlin
Box<Derived> boxDerived(Derived value) { ... }
Base unboxBase(Box<Base> box) { ... }
```

Kotlin中可以这样使用`unboxBase(boxDerived("s"))`，而Java不可能：因为Java中Box的参数`T`为不可变的，`Box<Derived>`不是`Box<Base>`的子类，Java需要这样定义`unboxBase`函数：
```java
Base unboxBase(Box<? extends Base> box) { ... }
```

利用Java的通配符类型（? extends Base）和`使用处变量`来模拟声明处变量。

为了让Kotlin API能够在Java中使用：生成`Box<Super>`作为`Box<? extends Super>`的协变量，定义`Box`（或`Foo<? super Bar>`作为`Foo`的逆变量）作为参数。当为返回值时，不生成通配符，否则Java需要处理这些通配符（违反Java通用编码方式），因此例子中的函数实际上转换为：

```kotlin
// 返回类型 -- 无通配符
Box<Derived> boxDerived(Derived value) { ... }
// 参数 -- 通配符
Base unboxBase(Box<? extends Base> box) { ... }
```

> 当参数类型为final（不可被继承）时，生成通配符也就毫无意义，所以无论使用什么姿势，`Box<String>`也就一直是`Box<String>`类型。

如果默认无法生成通配符，在需要通配符时，可以使用`@JvmWildcard`注解
```kotlin
fun boxDerived(value: Derived): Box<@JvmWildcard Derived> = Box(value)

// 转换为
// Base unboxBase(Box<Base> box) { ... }
```
如果不需要生成处的通配符时，使用`@JvmSuppressWildcards`注解
```kotlin
fun unboxBase(box: Box<@JvmSuppressWildcards Base>): Base = box.value
// 转换为
// Base unboxBase(Box<Base> box) { ... }
```
> `@JvmSuppressWildcards`不仅能用在单个类型参数处，也可以用于整体声明，如：函数、类，用作整体声明时，则忽略内部所有通配符

### Nothing类型转换

[Nothing](https://kotlinlang.org/docs/reference/exceptions.html#the-nothing-type)是Kotlin的特殊类型，因为Java与之对应的类型。事实上每个Java引用类型（包括`java.lang.Void`）接受`null`作为值，Nothing甚至不接受`null`值，所以Java无法精确表示`Nothing`类型，Kotlin使用`Nothing`类型参数时会生成原始类型。

```kotlin
fun emptyList(): List<Nothing> = listOf()
// 转换为
// List emptyList() { ... }
```