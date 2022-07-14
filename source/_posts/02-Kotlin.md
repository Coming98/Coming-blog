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

```kotlin
val a: Int = 10
```

## 定义区间

- `val range = 0..10`: 定义 `[0, 10]` 的区间
- `val range = 0 until 10`: 定义 `[0, 10)` 的区间
- `val range = 10 downTo 0`: 定义 `[10, 0]` 的区间

# 函数

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
