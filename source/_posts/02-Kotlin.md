---
title: 02-Kotlin
mathjax: false
date: 2022-07-14 10:46:25
summary: Android 开发语言 Kotlin
categories: Android Dev.
tags:
  - Android
---

# Kotlin

JetBrains 公司开发设计的: 
- Java -> .class -> JAVA 虚拟机 -> 二进制
- Kotlin -> .class -> JAVA 虚拟机 -> 二进制

开发工具: 
- IntelliJ IDEA
- [在线](https://try.kotlinlang.org/) 
- Android Studio: 在一个 Android 项目中编写一个 Kotlin 的 main() 函数即可独立运行 Kotlin 代码

## 优势

- 语法简洁, 代码量少了
- 语法高级, 开发效率高
- 语言安全
- 支持使用 Java 第三方的开源库


# 变量

## 变量声明

Kotlin 具有出色的类型推导机制, 因此仅有两种声明变量的关键字:
- `val`: value, 声明一个**不可变**的变量, 在初始赋值之后就再也不能重新赋值(对应 Java 中的 final 变量)
- `var`: variable, 声明一个**可变**的变量

## 显示类型声明

当变量需要延迟赋值时, 通过 `: Type` 的形式指定变量的类型
- Kotlin 完全抛弃了 Java 的基本数据类型, 使用对象数据类型
- `Int, Long, Short, Float, Double, Boolean, Char, Byte`
- `Unit` 类型表示函数返回无意义的值, 可以省略

```kotlin
val a: Int = 10
```

## 定义区间

- `val range = 0..10`: 定义 `[0, 10]` 的区间
- `val range = 0 until 10`: 定义 `[0, 10)` 的区间
- `val range = 10 downTo 0`: 定义 `[10, 0]` 的区间

使用 `in` 关键字检测目标是否在区间/集合中:
- `if (-1 !in 0..list.lastIndex) { ... }`
- `if (list.size !in list.indices) { ... }`

## 模板字符串

- `$value` 在字符串中调用变量
- `${s1.replace("is", "was")}` 模板表达式

## 类型检测

使用 `is` 关键字(类比 Java 的 `instanceof`)

```kotlin
if (obj is String) {
    return obj.length
}
// 取反
if (obj !is String) return null
```


# 函数

- 支持默认参数
- 

## if

Kotlin 中的 if 语句存在返回值, 值为条件中最后一行代码的返回值

```kotlin
fun JCMax2(a: Int, b: Int): Int {
    val ret: Int = if ( a > b) {
        a
    } else {
        b
    }
    return ret
}
```

## when

类似于 switch 语句, 但是功能更加强大:
- 存在返回值, 返回值为分支的返回值
- 匹配值支持任意类型的参数

```kotlin
fun getScore2(name: String) =
        when (name) {
            "Tom" -> 86
            "Jim" -> 77
            "Jack" -> 95
            "Lily" -> 100
            else -> 0
        }
```

- 支持类型匹配: `is` (类似 Java 的 instanceof)

```kotlin
fun checkNumber(num: Number) {
    when (num) {
        is Int -> println("number is Int")
        is Double -> println("number is Double")
        else -> println("number not support")
    }
}
```

- 支持无参数的更复杂的匹配


```kotlin
fun getScore3(name: String) =
        when {
            name.startsWith("Tom") -> 86
            name == "Jim" -> 77
            name == "Jack" -> 95
            name == "Lily" -> 100
            else -> 0
        }
```

## for

主要使用 `for-in` 循环: 
- 遍历区间
- 使用 `step` 设置遍历步长

- 遍历区间

```kotlin
fun main() {
    val range1 = 0..10
    for ( i in range1) {
        println(i)
    }
}
```

- 使用 `step` 设置遍历步长

```kotlin
val range2 = 0 until 10
for ( i in range2 step 2) {
    println(i)
}
```

## while

与 Java 的用法一致

```kotlin
fun main() {
    val items = listOf("apple", "banana", "kiwifruit")
    var index = 0
    while (index < items.size) {
        println("item at $index is ${items[index]}")
        index ++
    }
}
```

## repeat

允许传入一个 n 值, 会把 Lambda 表达式中的内容执行 n 遍:

```kotlin
        repeat(2) {
            fruitList.add(Fruit("Apple", R.drawable.apple_pic))
            fruitList.add(Fruit("Banana", R.drawable.banana_pic))
            // 重复加载数据
        }
```

# 类

- 实例化时取消了 `new` 关键字
- Kotlin 中任何一个非抽象类都是不可以被继承的; 如果允许继承, 则在类之前加上 `open` 修饰

```kotlin
class Person {
    var name = ""
    var age = 0
    fun eat() {
        println(name + " is eating. He is " + age + " years old.")
    }
}
fun main() {
    var p = Person()

    p.name = "Coming"
    p.age = 23
    p.eat()

    println(p)
}
```

## 继承

使用 `:` 表示继承, 继承时父类是带有 `()` 的
- 为什么要有 `()`: 子类要先调用父类的构造函数, 但是主构造函数的实现默认没有函数体, 因此使用 `()` 实现父类构造函数的预调用( 当然有参数的构造函数中括号中是要带参数的)

```kotlin
class Student: Person() {
    var grade = "1"
    fun test() {
        println(name + " 正在 " + grade + " 班考试...")
    }
}
```

## 构造函数

主构造函数: 没有函数体, 直接定义在类后面; 如果想在构造函数中执行一些逻辑, 使用 `init` 块实现

```kotlin
class Person2(val name: String, var age: Int) {
    init {
        if (name == "Coming" ) {
            age += 1
        }
    }
    fun run() {
        println(name + "is running..." + "Age = " + age)
    }
}
```

次构造函数, 拥有函数体, 
- 当一个类既有主构造函数又有次构造函数时，所有的次构造函数都必须调用主构造函数（包括间接调用）
- 任何一个类只有一个主构造函数, 但是可以有多个次构造函数
- 次构造函数也能够用于实例化一个类
- 没有主构造函数时继承时就不必为父类加括号了, 而是在次构造中的函数体内通过 `super` 关键字实现

```kotlin
open class Person3(val name: String, val age: Int) {
    fun showPersonInfo() {
        println("name = " + name + " age = " + age)
    }
}

class Student3(val grade: String, val score: Int, name: String, age: Int): Person3(name, age) {
    constructor(name: String, age: Int): this("", 0, name, age) {}
    constructor() : this("", 0) {} // 没有参数时, 先定义默认值后调用第一个次构造函数
    fun showStudentInfo() {
        println("name = " + name + " age = " + age + " grade = " + grade + " score = " + score)
    }
}

class Student4 : Person3 {
    constructor(name: String, age: Int) : super(name, age) { }
}

fun main() {
    val student1 = Student3()
    val student2 = Student3("Jack", 19)
    val student3 = Student3("a123", 5, "Jack", 19)
    student1.showStudentInfo()
    student2.showStudentInfo()
    student3.showStudentInfo()

    val student4 = Student4("name", 10)
    student4.showPersonInfo()
}
```

## 接口

任何一个类只能继承一个父类, 但是可以实现任意多个接口, 可以在接口中定义一系列抽象行为, 然后由具体的类去实现
- 实现接口中的方法时需要使用 `override` 关键字
- kotlin 允许对接口中定义的函数进行默认实现

```kotlin
open class Person5(val name: String) {
    constructor(): this("UNKNOWN")
    fun showPersonInfo() {
        println("Person name = " + name)
    }
}

interface Study {
    fun readBooks()
    fun doHomework() {
        println("Default implement")
    }
}

class Student5(name: String, val age: Int): Person5(name), Study {
    constructor(age: Int): this("UNKNOWN", age) { }
    constructor(name: String): this(name, 0) { }
    constructor(): this(0)
    override fun readBooks() {
        println(name + "(age = " + age + ") is reading")
    }
    override fun doHomework() {
        println(name + "(age = " + age + ") is homeworking")
    }
}

fun main() {
    val s1 = Student5("Coming", 23)
    val s2 = Student5("CJC")
    val s3 = Student5(24)
    val s4 = Student5()

    println(s1.readBooks())
    println(s2.readBooks())
    println(s3.doHomework())
    println(s4.doHomework())
}
```

## 可见性修饰符

- private: 类内可见
- public: 所有类可见, 默认修饰符
- protected: 当前类和子类可见
- internal: 对同一模块中的类可见, 比如我们开发了一个模块给别人使用，但是有一些函数只允许在模块内部调用，不想暴露给外部，就可以将这些函数声明成 internal

![](https://raw.githubusercontent.com/Coming98/pictures/main/202207141024518.png)

## 数据类

数据类用于将服务器端或数据库中的数据映射到内存中，为编程逻辑提供数据模型的支持, Kotlin 中使用 `data` 关键字进行修饰
- MVC、MVP、MVVM 之类的架构模式中的 M 指的就是数据类
- 数据类通常需要重写 `equals()、hashCode()、toString()` 等方法
  - `equals()`: 判断两个数据类是否相等
  - `hashCode()`: `equals()` 的配套方法
  - `toString()`: 用于提供更清晰的输入日志（否则一个数据类默认打印出来的就是一行内存地址）

```kotlin
// 实现 Cellphone 数据类十分简单
// 当一个类中没有任何代码时, 可以将尾部的大括号省略
data class Cellphone(val brand: String, val price: Double)

```

## 单例类

单例模式: 最常用、最基础的设计模式之一，用于避免创建重复的对象

Java 中的单例思想: 
- 首先为了禁止外部创建 Singleton 的实例，需要用 private 关键字将 Singleton 的构造函数私有化
- 然后给外部提供了一个 getInstance() 静态方法用于获取 Singleton 的实例
- 在 getInstance() 方法中，如果当前缓存的 Singleton 实例为 null，就创建一个新的实例
- 否则直接返回缓存的实例即可

```java
public class Singleton
{
    private static Singleton instance;
    private Singleton() {}
    public synchronized static Singleton getInstance()
    {
        if(instance == null)
        {
            instance = new Singleton();
        }
        return instance;
    }
    public void singletonTest()
    {
        System.out.println("singletonTest is called.");
    }
}

Singleton singleton = Singleton.getInstance();
singleton.singletonTest();
```

Kotlin 中的单例类依旧隐藏了固定的重复的逻辑, 将 `class` 改为 `object` 即可创建单例类

```kotlin
object Singoton {
    val name: String = "Coming"
    fun SingotonTest() {
        println("called")
    }
}

fun main() {
    Singoton.SingotonTest() // Kotlin 会先创建 Singoton 的实例然后在调用(并且保证全局只有一个实例)
    println(Singoton.name)
}
```

# Lambda 编程

Lambda 就是一小段可以作为参数传递的代码

语法结构: `{参数名1: 参数类型, 参数名2: 参数类型 -> 函数体}`

## 集合

- `listOf()`: 初始化不可变集合, 只能读取, 不能添加修改或删除
- `mutableListOf()`: 初始化可变集合
- 同理 `setOf()` 于 `mutableSetOf()` 只是 set 中不可以存放重复的元素, 对于重复的元素只会保留一份
- 方法: `.filter`, `map`, `maxBy`, `any`, `all`

```kotlin
fun main() {
    val fruitList = listOf("Apple", "Banana", "Orange", "Pear", "Grape")
    for (fruit in fruitList) {
        println(fruit)
    }

    val friendList = mutableListOf("Coming", "ZH", "ZCY")
    friendList.add("GTY")
    for (friend in friendList) {
        println(friend)
    }
}
```

### 遍历索引值

通过集合的属性 `indices`

```kotlin
val items = listOf("apple", "banana", "kiwifruit")
for (index in items.indices) {
    println("item at $index is ${items[index]}")
}
```

## Map

- 支持 Java 中的 put 与 get, 但是不推荐
- 推荐使用类似于数组下标的语法结构去赋值与获取
- 也支持 `mapOf()` 与 `mutableMapOf()`

```kotlin
fun main() {
    val map1 = HashMap<String, Int>()
    map1.put("Coming", 23)

    println("Coming's age = " + map1.get("Coming"))

    val map2 = HashMap<String, Int>()
    map2["Coming"] = 23
    println("Coming's age = " + map2["Coming"])

    val fruitMap = mapOf("Apple" to 1, "Banana" to 2, "Orange" to 3, "Pear" to 4, "Grape" to 5)
    for ((fruit, id) in fruitMap) {
        println(fruit + " : " + id)
    }
}
```

## Java 函数式 API 的使用

Kotlin 中调用 Java 方法时也可以使用函数式 API, 如果我们在 Kotlin 代码中调用了一个 Java 方法，并且该方法**接收一个 Java 单抽象方法接口参数**，就可以使用函数式API
- 接口中只有一个待实现方法
- Java 函数式 API 的使用都限定于从 Kotlin 中调用 Java 方法，并且单抽象方法接口也必须是用 Java 语言定义的

在 Java 中使用函数式 API: 匿名类, 创建了一个 Runnable 接口的匿名类实例，并将它传给了 Thread 类的构造方法
```java
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Thread is running");
    }
}).start();
```

使用 Kotlin 直观的改写: object 关键字是用于定义单例类, 在这里也直接用于创建了匿名类的实例(因为单例类只有一个实例, 所以类名直接引用是咧)

```kotlin
fun main() {
    Thread(object : Runnable {
        override fun run() {
            println("RUNNING")
        }
    })
}
```

- 简化实现代码:因为只有一个待实现的方法, 所以没必要显示的重写 `run` 方法

```kotlin
Thread(Runnable {
    println("RUNING2")
}).start()
```

- 继续简化实现代码: 如果一个 Java 方法的参数列表中有且仅有一个 Java 单抽象方法接口参数，我们还可以将接口名进行省略

```kotlin
Thread({
    println("RUNING3")
}).start()
```

- 最终简化: 当 Lambda 表达式是方法的最后一个参数时，可以将 Lambda 表达式移到方法括号的外面
- 同时，如果 Lambda 表达式还是方法的唯一一个参数，还可以将方法的括号省略

```kotlin
Thread { println("RUNNING4") }.start()
```

# 空指针检查

Kotlin 默认所有的参数和变量都不可为空, 但可以在类型名后面加上 `?` 来定义可空类型系统：
- `Int` 表示不可为空的整型, `Int?` 表示可为空的整型

但使用可空类型系统时就需要对参数进行判空处理, 常用的符号有 `?.` `!!.` `?:`
- `a?.doSomething()`: a 为 null 时不再执行, 直接返回 null, a 不为 null 时正常执行
- `a ?: b`: 左右两边都接收一个表达式，如果左边表达式的结果不为空就返回左边表达式的结果，否则就返回右边表达式的结果
- `!!.`: 函数内部进行了非空判断, 但是函数返回后 Kotlin 不能知道已经做了非空判断, 因此使用 `!!.` 进行非空断言, 表明这个对象绝对不为空

```kotlin
fun getTextLength(text: String?): Int {
    println(text?.length)
    if (text != null) {
        return text.length
    }
    return 0
}

// text 不为 null 则 text.length 返回值不为空, ?: 操作符返回左边的结果
// text 为 null 则 text.length 返回值为空, ?: 操作符返回右边的结果
fun JCGetTextLength(text: String?) = text?.length ?: 0

fun main() {
    getTextLength("WWW") // text?.length = 3
    getTextLength(null) // text?.length = null
    println(JCGetTextLength(null))
    println(JCGetTextLength("Hello World"))
}
```

## let 函数

这个函数提供了函数式 API 的编程接口，并将原始调用对象作为参数传递到 Lambda 表达式中, 配合进行非空判断十分方便:
- `obj.let { obj_ -> ... }`: `obj_` 与 `obj` 是相同的对象, 只不过为了不重名
- 如下例, 如果 study 为 null 则不会执行 let 函数; 如果 study 不为 null 则执行 let 函数并实现了非空判断
- 与 `if` 不同的是, let 函数是可以处理全局变量的判空问题的: if 中如果存在全局变量, 其值随时都有可能被其他线程所修改，即使做了判空处理，仍然无法保证 if 语句中的 study 变量没有空指针风险

```kotlin
fun doStudy(study: Study?) {
    study?.let { stu ->
        stu.readBooks()
        stu.doHomework()
    }
}
// lambda 只有一个参数时可以简化改参数为 it
fun doStudy(study: Study?) {
    study?.let {
        it.readBooks()
        it.doHomework()
    }
}
```