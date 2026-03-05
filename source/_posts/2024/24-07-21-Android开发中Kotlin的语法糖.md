---
title: Android开发中Kotlin的语法糖
date: 2024-07-21 14:30:45
comments: true
tags:
- Kotlin
- Android
- 语法糖
- 高阶函数
- 扩展函数
- 运算符重载
- 延迟初始化
- 闭包
- 递归
- 成员引用
categories:
- 软件开发
- 小程序和APP
---

Kotlin是Android开发中最常用的语言之一，它提供了简洁、安全和高效的编程方式。
在Kotlin中，我们可以使用多种方法来实现相同的功能。

基础语法可参考 [https://www.runoob.com/kotlin/kotlin-tutorial.html](https://www.runoob.com/kotlin/kotlin-tutorial.html)

我们这里再提取一些更为基础的知识。

## 基本的Kotlin语法

下面是一些常用的方法：

1. 函数
函数是Kotlin中最基本的代码块，它可以接受参数并返回一个值。在Kotlin中，我们可以使用fun关键字来定义函数，例如：

```kotlin
fun add(a: Int, b: Int): Int {
    return a + b
}
```

在上面的例子中，我们定义了一个名为add的函数，它接受两个Int类型的参数a和b，并返回它们的和。

调用方法：

```kotlin
val result = add(1, 2)
println(result) // 输出3
```

2. 表达式

在Kotlin中，我们可以使用表达式来定义函数，例如：

```kotlin
fun add(a: Int, b: Int) = a + b
```

在上面的例子中，我们定义了一个名为add的函数，它接受两个Int类型的参数a和b，并返回它们的和。与函数不同的是，表达式不需要使用return关键字来返回值。

调用方法跟普通的函数调用相同：

```kotlin
val result = add(1, 2)
println(result) // 输出3
```

3. 高阶函数

高阶函数是指接受一个或多个函数作为参数的函数。在Kotlin中，我们可以使用高阶函数来实现更加灵活的代码。例如，我们可以使用高阶函数来实现一个函数，它接受一个函数作为参数，并对列表中的每个元素应用该函数，例如：

```kotlin
fun map001(list: List<Int>, f: (Int) -> Int): List<Int> {
    // 定义一个可变列表，用于存储结果
    val result = mutableListOf<Int>()

    // 遍历列表中的每个元素，并将每个元素应用函数f，然后将结果添加到结果列表中
    for (element in list) {
        result.add(f(element))
    }

    // 返回结果列表
    return result

}
```

在上面的例子中，我们定义了一个名为map的函数，它接受一个List<Int>类型的参数list和一个函数f作为参数，并返回一个新的List<Int>类型的结果。在函数体中，我们使用for循环遍历列表中的每个元素，并将每个元素应用函数f，然后将结果添加到结果列表中。

调用方法：

```kotlin
val list = listOf(1, 2, 3)

// 第二个参数是定义一个函数，它接受一个Int类型的参数，并返回该参数的两倍
// 其中的 it 表示列表中的每个元素，相当于Java中的 element
val result = map(list, { it * 2 })
println(result) // 输出[2, 4, 6]
```

4. 闭包

闭包是指一个函数可以访问其外部作用域中的变量。在Kotlin中，我们可以使用闭包来实现更加灵活的代码。例如，我们可以使用闭包来实现一个函数，它返回一个函数，该函数可以访问其外部作用域中的变量，例如：

```kotlin
fun makeCounter(): () -> Int {
    var count = 0
    return { count++ } 
}
```

在上面的例子中，我们定义了一个名为makeCounter的函数，它返回一个函数。在函数体中，我们定义了一个名为count的变量，并将其初始化为0。然后，我们返回一个函数，该函数可以访问count变量，并将其递增。

调用方法：

```kotlin

// 调用makeCounter函数，它会返回一个函数，该函数可以访问count变量，并将其递增
val counter = makeCounter()

// 调用counter函数，它会返回count变量的值，并将count变量递增
println(counter()) // 注意：输出0，如果是 ++count，则输出1
```

5. 递归

递归是指一个函数可以调用自身。在Kotlin中，我们可以使用递归来实现更加灵活的代码。例如，我们可以使用递归来实现一个函数，它可以计算一个整数的阶乘，例如：

```kotlin
fun factorial(n: Int): Int {
    if (n == 0) {
        return 1
    } else {
        return n * factorial(n - 1)
    }
}
```
在上面的例子中，我们定义了一个名为factorial的函数，它接受一个Int类型的参数n，并返回一个Int类型的结果。在函数体中，我们使用if语句来判断n是否等于0。如果n等于0，我们返回1。否则，我们返回n乘以factorial(n - 1)的结果。

调用方法：
```kotlin
val result = factorial(5)
println(result) // 输出120
```

我们来看一下递归的过程：
```kotlin
factorial(5) = 5 * factorial(4) = 5 * 4 * factorial(3) = 5 * 4 * 3 * factorial(2) = 5 * 4 * 3 * 2 * factorial(1) = 5 * 4 * 3 * 2 * 1 = 120
```

6. 扩展函数

扩展函数是指可以在一个类的外部定义一个函数，该函数可以访问该类的私有成员。

在Kotlin中，我们可以使用扩展函数来实现更加灵活的代码。例如，我们可以使用扩展函数来实现一个函数，它可以将一个字符串转换为大写，例如：

```kotlin
fun String.toUpperCase(): String {
    return this.toUpperCase()
}
```

在上面的例子中，我们定义了一个名为toUpperCase的扩展函数，它接受一个String类型的参数，并返回一个String类型的结果。在函数体中，我们使用this关键字来访问该字符串，并将其转换为大写。

调用方法：
```kotlin
val str = "hello"
val result = str.toUpperCase()
println(result) // 输出HELLO
```

7. 运算符重载

运算符重载是指可以在一个类的外部定义一个运算符的行为。
在Kotlin中，我们可以使用运算符重载来实现更加灵活的代码。例如，我们可以使用运算符重载来实现一个函数，它可以将两个字符串连接起来，例如：

```kotlin
operator fun String.plus(other: String): String {
    return this + other
}
```

在上面的例子中，我们定义了一个名为plus的运算符重载函数，它接受一个String类型的参数，并返回一个String类型的结果。在函数体中，我们使用this关键字来访问该字符串，并将其与另一个字符串连接起来。

> 在Kotlin中，运算符重载是通过特定的函数名来实现的。当你定义一个名为plus的函数并加上operator修饰符时，它就自动与+运算符关联起来了。这是Kotlin语言设计的一个语法糖。
> 当你使用operator fun plus定义函数时，就是在告诉Kotlin："当有人对这个类型的对象使用+运算符时，请调用我这个函数"

其他常见运算符对应的函数名：
```plaintext

- → minus
* → times
/ → div
== → equals
> → compareTo

```

调用方法：
```kotlin
val str1 = "hello"
val str2 = "world"
val result = str1 + str2
println(result) // 输出helloworld
```

8. 延迟初始化

延迟初始化是指在需要时才初始化一个变量。例如，我们可以使用延迟初始化来实现一个函数，它可以在需要时才初始化一个变量，例如：

```kotlin
lateinit var str: String
fun initStr() {
    str = "hello"
}
```

在上面的例子中，我们定义了一个名为str的变量，并使用lateinit修饰符来表示该变量是延迟初始化的。然后，我们定义了一个名为initStr的函数，它可以在需要时才初始化str变量。

调用方法：

```kotlin
initStr()
println(str) // 输出hello
```

Kotlin中的lateinit延迟初始化确实有重要的实际意义，它与普通的"先不赋值"有本质区别。Kotlin强制要求非空类型必须立即初始化，否则编译会报错；而使用lateinit可以推迟初始化的时机，直到真正需要使用时才进行初始化。


| 特性 | lateinit var | 普通var不赋值 | 
|--------------------|-------------------------|---------------------| 
| 类型限制 | 必须是非空类型(不能是Int等基本类型) | 可以是可空类型(如String?) | 
| 初始值检查 | 有明确的未初始化状态检查 | 默认为null | 
| 线程安全 | 非线程安全 | 线程安全(因为初始为null) | 
| 使用前必须初始化 | 否则抛UninitializedPropertyAccessException | 可以直接使用(null值) |


9. 成员引用运算符

成员引用运算符是指可以在一个类的外部定义一个函数，该函数可以访问该类的成员。

我们可以使用成员引用运算符来实现一个函数，它可以访问一个类的成员，例如：

```kotlin
class Person(val name: String, val age: Int) {
    fun sayHello() {
        println("Hello, my name is $name and I am $age years old.")
    }
}
val person = Person("Alice", 25)

// 定义一个函数变量 sayHello，它可以访问Person类的成员函数sayHello
val sayHello = Person::sayHello

// 调用函数变量sayHello，它会调用Person类的成员函数sayHello
sayHello(person) // 输出Hello, my name is Alice and I am 25 years old.
```

在上面的例子中，我们定义了一个名为Person的类，它有两个成员：name和age。然后，我们定义了一个名为sayHello的函数，它可以访问Person类的成员。最后，我们定义了一个名为sayHello的变量，并将其赋值为Person类的sayHello函数。


成员引用运算符可以调用变量的KProperty对象，例如：
```kotlin
val name = Person::name
println(name.name) // 输出name
println(name.get(person)) // 输出Alice
```

我们再看一段代码：
```kotlin
if(::str.isInitialized) {
    // 安全使用
}
```

在Kotlin中，::str.isInitialized 这种双冒号(::)语法叫做成员引用运算符(Member Reference)，它用于获取对类成员(属性或函数)的引用。

具体到::str.isInitialized这个表达式：

- ::str - 获取对属性str的引用(KProperty对象)
- .isInitialized - 访问该属性的初始化状态检查方法

在Kotlin中，KProperty和普通属性str有本质区别。KProperty是Kotlin反射API中的一个类，代表对属性的引用。当你使用::str时，得到的就是一个KProperty对象，它包含了该属性的元信息（如名称、类型等）和扩展功能（如isInitialized）。

isInitialized是Kotlin为KProperty特别添加的扩展属性，专用于检查lateinit变量的初始化状态。

我们再看一个例子：

```kotlin
lateinit var str: String

// 普通属性访问（直接操作值）
str = "hello"  // 设置值
println(str)    // 获取值

// KProperty访问（操作属性引用）
val prop = ::str
println(prop.name) // 输出"str"（属性名）
println(prop.isInitialized) // 检查初始化状态
```





## Kotlin语法糖实例

在总结语法糖之前，我们先来看一下Kotlin的一些语法糖实例。

```kotlin
class Person(val name: String, val age: Int) {
    //获取Android主线程（UI线程）的Looper对象
    private val handler = Handler(Looper.getMainLooper())

    fun sayHello() {
        println("Hello, my name is $name and I am $age years old.")

        // 连接成功后，立即通过主线程发送 "hello001" 消息
        // handler.post(Runnable) 的作用是
        // 将一个 Runnable 任务添加到 handler 所绑定线程的消息队列中，
        // 由该线程的 Looper 循环取出并执行。
        handler.post {
            sendMessage("hello001".toByteArray())  // 调用 sendMessage 发送字节数据
        }
    }
}

```

在 Kotlin 中，这种写法是函数最后一个参数为 Lambda 表达式时的语法糖，本质是将 Lambda 作为参数传递给 handler.post。

Handler.post 的 Java 定义是：

```java
public final boolean post(@NonNull Runnable r) {
    return sendMessageDelayed(getPostMessage(r), 0);
}
```

可以看到，Handler.post 的参数是一个 Runnable 对象，而我们在 Kotlin 中传递的是一个 Lambda 表达式。

Kotlin 支持 SAM（Single Abstract Method）转换：对于只有一个抽象方法的接口（如 Runnable），可以直接用 Lambda 表达式代替接口实例。

因此，Java 中需要显式创建 Runnable 的代码：

```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        // 在这里编写要执行的代码
    }
};
```

这个示例中，

```java
handler.post(new Runnable() {
    @Override
    public void run() {
        sendMessage("hello001".getBytes());
    }
});
```

在 Kotlin 中可以简化为：

```kotlin
handler.post(Runnable { sendMessage("hello001".toByteArray()) })
```

进一步，由于 Runnable 是 post 函数的最后一个参数，Kotlin 允许将 Lambda 表达式移到圆括号外，甚至省略圆括号：

```kotlin
handler.post { 
    sendMessage("hello001".toByteArray()) 
}
```

以上代码等价于：

```kotlin
handler.post(
    Runnable 
    { sendMessage("hello001".toByteArray()) }
)

```

花括号 { ... } 内的内容本质是 Runnable.run() 方法的实现。
Kotlin 编译器会自动将 Lambda 转换为 Runnable 实例，传递给 post 函数。


