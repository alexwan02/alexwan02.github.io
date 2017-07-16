---
title: Kotlin基础之函数
date: 2017-06-21 12:00:09
thumbnailImage: https://pic1.zhimg.com/v2-30253c279faba2e77120862dd54d49d4_r.jpg
tags: kotlin
---

## 函数声明
```java
fun double(x: Int): Int {
    return 2*x
}
```

## 函数调用

1. 类Java用法
```java
var result = double(2)
```
2. 使用冒号调用
创建Sample对象并调用foo函数
```java
Sample().foo()
```

## 中缀（Infix）表示法
中缀表示法（或中缀记法）是一个通用的算术或逻辑公式表示方法， 操作符是以中缀形式处于操作数的中间（例：3 + 4）

在以下三种情况下，可以使用中缀表示法调用函数
- 成员函数或扩展函数
- 单个参数
- 使用`infix`关键字标记的函数
```java
// 定义Int扩展函数
infix fun Int.sh(x : Int) : Int{
  
}

// 使用中缀表示法调用函数
1 sh 2

// 相同的

1.sh(2)
```

## 参数

函数参数的定义采用Pascal表示法：`name:type`。多个参数之间使用逗号分隔，每个参数必须要指定类型
```java
fun powerOf(number: Int , exponent: Int){
  ...
}
```

## 默认值
函数参数可以设置默认值。用于对应参数缺省的情况下，降低函数重载数量。
```java
fun read(b: Array<Byte> , off: Int = 0 , len: Int = b.size){
  ...
}
```
默认值使用紧跟变量类型后的`=`来定义。
复写方法必须使用与父类相同的默认参数值。当复写带有默认值的方法时，必须要省略默认值。
```java
open class A{
    open fun foo(i: Int = 10){
      ...
    }
}

class B : A(){
  // 不允许提供默认值
  override fun foo(i: Int){
    ... 
  }
  }
}
```
## 具名参数
调用函数时可以提供参数的名称。在函数有大量有默认值参数时会很方便。

假设有下面一个函数

```java
fun reformat(str: String
            normalizeCase: Boolean = true,
            upperCaseFirstLetter: Boolean = true,
            divideByCamelHumps: Boolean = false,
            wordSeparator: Char = ' ')
```
可以这样调用

```java
reformat(str)
```
不使用默认参数则需要这样调用

```java
reformat(str , true , true , false , '_')
```
使用具名参数可以让代码可读性更好

```java
reformat(str,
    normalizeCase = true,
    upperCaseFirstLetter = true,
    divideByCamelHumps = false,
    wordSeparator = '_'
)
```
如果不需要所有参数
```java
reformat(str, wordSeparator = '_')
```
> 具名参数语法不适应于Java函数。因为Java字节码并不一直保存参数名称。

## Unit-returning函数
如果函数不返回值，则函数返回类型为`Unit`。`Unit`是唯一值为`Unit`的类型。Unit返回值不需要显式返回。
```java
fun printHello(name: String?): Unit {
    if (name != null)
        println("Hello ${name}")
    else
        println("Hi there!")
    // `return Unit` 或 `return` 可选
}
```
`Unit`返回类型声明为可选项目。上面代码等同于
```java
fun printHello(name: String?) {
    ...
}
```
## 单表达式函数
当函数返回单个表达式，可省略花括号，可在`=`之后指定对应代码。
```java
fun double(x: Int): Int = x * 2
```
可省略返回值类型，由编译器推测。

```java
fun double(x: Int) = x * 2
```
## 显式返回类型
非Unit返回值且有代码块的函数必须显式指明返回值类型。`Kotlin`不会推断有代码块的函数的返回值类型，因为这些函数可能有复杂的控制流程， 返回类型对于编译器或阅读代码的开发者不是那么显而易见。

## 可变参数
函数参数（一般最后一个）可以使用`vararg`修饰符设置为可变参数。
```java
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts) // ts is an Array
        result.add(t)
    return result
}
```
允许函数使用可变数量的参数

```java
val list = asList(1 , 2, 3)
```
在函数中T类型的可变参数被视为T的一个数组。例如上面例子中的`ts`变量就是`Array<out T>`

函数中只能设置一个标记为`vararg`可变参数. 如果可变参数不是最后一个参数，则之后的参数可以使用具名参数语法传值；
可以通过在括号外以lambda方式传入函数类型的参数。
调用可变参数函数时，可以用一个一个方式传参（如：asList(1, 2, 3)）；或使用扩展操作符（前缀`*`）传入数组参数。

```java
val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)
```
## 函数作用域

在Kotlin中可以在文件顶部声明函数，意味着不必创建Class来持有函数。除此Kotlin函数还可以像成员函数和扩展函数一样在局部声明。

## 局部函数
Kotlin支持局部函数，如在函数内部定义函数

```java
fun dfs(graph: Graph) {
    fun dfs(current: Vertex, visited: Set<Vertex>) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v, visited)
    }

    dfs(graph.vertices[0], HashSet())
}
```
局部函数可以访问外部函数的局部变量

```java
fun dfs(graph: Graph) {
    val visited = HashSet<Vertex>()
    fun dfs(current: Vertex) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v)
    }

    dfs(graph.vertices[0])
}
```

## 成员函数
成员函数：定义在Class或对象内部的函数。
```java
class Sample() {
    fun foo() { print("Foo") }
}
```
成员函数可以使用`.`操作符调用
```java
Sample().foo()
```
更多关于Class和复写成员函数查看[`Classes`](https://kotlinlang.org/docs/reference/classes.html)和[`Inheritance`](https://kotlinlang.org/docs/reference/classes.html#inheritance)

## 泛型函数
函数在函数名前以`<T>`形式使用泛型。

```java
fun <T> singletonList(item: T): List<T> {
    // ...
}
```
跟多关于泛型信息查看[`Generics`](https://kotlinlang.org/docs/reference/generics.html)

## 内联函数
内联函数查看[`Inline Functions`](https://kotlinlang.org/docs/reference/inline-functions.html)
## 扩展函数
扩展函数查看[`Extension Functions`](https://kotlinlang.org/docs/reference/extensions.html)

## 高阶函数和Lambdas
高阶函数和Lambdas查看[`高阶函数和Lambdas`](https://kotlinlang.org/docs/reference/lambdas.html)

## 尾递归函数

`尾调用`是指一个函数里的最后一个动作是一个函数调用的情形：即这个调用的返回值直接被当前函数返回的情形。

Kotlin支持尾递归形式的函数编程。目前尾递归只支持JVM后端。定义合格的尾递归，函数必须要在最后一步执行自身调用。当在递归操作后仍有要执行的代码时，或者在try/catch/finally块中都不适合使用尾递归函数。
当函数使用`tailrec`修饰符，满足必要形式，编译器会优化递归，保留快速、高效的迭代基准版本。
```java
tailrec fun findFixPoint(x: Double = 1.0): Double
        = if (x == Math.cos(x)) x else findFixPoint(Math.cos(x))
```
上面函数用于计算余弦的不动点（f(x) = x）。代码与下面相同
```java
private fun findFixPoint(): Double {
    var x = 1.0
    while (true) {
        val y = Math.cos(x)
        if (x == y) return y
        x = y
    }
}
```



