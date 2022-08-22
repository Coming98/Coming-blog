---
title: 02-Kotlin-Advanced
mathjax: false
date: 2022-07-18 16:16:36
summary: Kotlin 高级应用
categories: Android Dev.
tags:
  - Android
---
# 标准函数

## let


let 函数提供了函数式 API 的编程接口，并将原始调用对象作为参数传递到 Lambda 表达式中, 配合进行非空判断十分方便:
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

## with

接收两个参数: 任意类型的对象 + Lambda 表达式, with 函数在 Lambda 表达式中提供第一个参数对象的上下文, 并使用 Lambda 表达式中的最后一行代码作为返回值返回

- 可以在连续调用同一个对象的多个方法时让代码变得更加精简

```kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape")
val builder = StringBuilder()
builder.append("Start eating fruits.\n")
for (fruit in list) {
    builder.append(fruit).append("\n")
}
builder.append("Ate all fruits.")
val result = builder.toString()
println(result)
// 可以考虑使用 with 共享 StringBuilder 的上下文, 简化代码
val result2 = with(StringBuilder()) {
    append("Start eating fruits.\n")
    for (fruit in list) {
        append(fruit).append("\n")
    }
    append("Ate all fruits.")
    toString()
}
println(result2)
```

## run

相当于另一种形式的 with: 在某个对象基础上进行调用, 接收 Lambda 参数, 并将对象作为 Lambda 表达式的上下文, 以 Lambda 表达式的最后一行作为返回值返回

```kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape")
val result2 = StringBuilder().run {
    append("Start eating fruits.\n")
    for (fruit in list) {
        append(fruit).append("\n")
    }
    append("Ate all fruits.")
    toString()
}
println(result2)
```

## apply

在某个对象上调用, 接收一个 Lambda 参数, 将对象作为 Lambda 函数的上下文, 并将调用对象本身(处理后)返回

```kotlin
val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape")
val stringBuilder = StringBuilder().apply { 
    append("Start eating fruits.\n")
    for (fruit in list) {
        append(fruit).append("\n")
    }
    append("Ate all fruits.")
    }
println(stringBuilder.toString())
```

# 静态方法

又叫作类方法，指的就是那种不需要创建实例就能调用的方法; Kotlin 中将由静态方法组成的工具类用单例类实现(`object` 关键词进行修饰)

但是如果在非工具类中定义静态方法, 这时候就需要 `companion object` 块进行定义了
> 但是其机理是会在类的内部产生一个伴生类, 伴生类是一个单例类, 具有块中定义的方法

如果要显示的定义其为静态方法:
1. 添加 @JvmStatic 注解：只能加在单例类或 companion object 中的方法上
2. 顶层方法: 没有定义在任何类中的方法( package ); 就比如新建一个 tools.kt, 直接定义方法, 这些方法都会被编译为静态方法; 并且所有的顶层方法都可以在任何位置被直接调用，不用管包名路径，也不用创建实例
> 其原理还是以顶层方法的文件名创建一个 Java 类, 将其中的方法写为 Java 的静态方法

# 延迟初始化

比如一个全局变量需要到访问某一个 Activity 时才能被初始化, 如果提前定义在类中, 就需要定义为 null, 后续的操作全都要考虑判空处理, 十分不方便, 因此引入了 lateinit 关键字进行全局初始化; 但相反, 所有的判空逻辑检查实际上就交给代码编写者了, 如果粗心还是容易出现初始化前进行使用的错误~

```kotlin
class MainActivity : AppCompatActivity(), View.OnClickListener {
    private lateinit var adapter: MsgAdapter
    override fun onCreate(savedInstanceState: Bundle?) {
        ...
        adapter = MsgAdapter(msgList)
        ...
    }
    override fun onClick(v: View?) {
        ...
        adapter.notifyItemInserted(msgList.size - 1)
        ...
    }
}
```

如果初始化的操作能够重复触发也是一种重复消耗, 因此可以使用 `::ValueName.isInitialized` 来判断变量 `ValueName` 是否完成初始化工作了

```kotlin
...
if (!::adapter.isInitialized) {
    adapter = MsgAdapter(msgList)
}
...
```

# 密封类

当在 when 语句中传入一个密封类变量作为条件时，Kotlin 编译器会自动检查该密封类有哪些子类，并强制要求你将每一个子类所对应
的条件全部处理。这样就可以保证，即使没有编写 else 条件，也不可能会出现漏写条件分支的情况。


# 扩展函数

扩展函数表示即使在不修改某个类的源码的情况下，仍然可以打开这个类，向该类添加新的函数

- 定义扩展函数的语法结构

```kotlin
fun ClassName.methodName(param1: Int, param2: Int): Int {
    return 0
}
```

- 建议向哪个类中添加扩展函数，就定义一个同名的 Kotlin 文件; 或定义在任何一个现有类当中的
- 扩展函数自动拥有目标类的实例上下文, 通过 this 关键字进行访问

```kotlin
fun String.lettersCount(): Int {
    var count = 0
    for (char in this) {
        if (char.isLetter()) {
            count++
        }
    }
    return count
}
```

# 运算符重载

Kotlin 的运算符重载使用 `operator` 关键字允许我们让任意两个对象进行运算操作:

## Quick Start

在指定函数的前面加上 operator 关键字，就可以实现运算符重载的功能了
- 指定函数: plus(), minus(), 

```kotlin
class A {
    operator fun plus(b: B): C {
        // 处理相加的逻辑
        return this + b
    }
}

class Money(val value: Int) {
    operator fun plus(money: Money): Money {
        val sum = value + money.value
        return Money(sum)
    }
    operator fun plus(newValue: Int): Money {
        val sum = value + newValue
        return Money(sum)
    }
}
```

## 调用对照表

![](https://raw.githubusercontent.com/Coming98/pictures/main/202208181927672.png)

# 高阶函数

如果一个函数接收另一个函数作为参数，或者返回值的类型是另一个函数，那么该函数就称为高阶函数
- 函数类型: `(String, Int) -> Unit`, 左侧为参数类型的声明, 右侧为返回值类型的声明, Unit 表示没有返回值
- 传递函数参数时采用函数引用的方式进行传递: `::func` 或者直接传递 lambda 表达式

```kotlin
val result1 = num1AndNum2(num1, num2) { n1, n2 ->
    n1 + n2
}
val result2 = num1AndNum2(num1, num2) { n1, n2 ->
    n1 - n2
}
```

## 高阶函数实现上下文共享

类似于内置的 apply 函数
- 在函数类型的前面加上 ClassName. 表示这个函数类型是定义在哪个类当中的, 调用 build 时传入的 lambda 表达式将会自动拥有 StringBuilder 的上下文

```kotlin
fun StringBuilder.build(block: StringBuilder.() -> Unit): StringBuilder {
    block()
    return this
}
```

## 高阶函数: Kotlin - Java 

Kotlin 的高阶函数编译为 Java 字节码文件后, 实际上使用的是 Function 接口中的 invoke() 函数实现的, 而传入的 lambda 表达式都会由一个匿名类实现, 这样就会导致较大的系统开销

```java
public static int num1AndNum2(int num1, int num2, Function operation) {
    int result = (int) operation.invoke(num1, num2);
    return result;
}
public static void main() {
    int num1 = 100;
    int num2 = 80;
    int result = num1AndNum2(num1, num2, new Function() {
        @Override
        public Integer invoke(Integer n1, Integer n2) {
            return n1 + n2;
        }
    });
}
```

# 内联函数

针对高阶函数底层的开销较大, Kotlin 引入了内联函数, 使用 `inline` 关键字进行修饰, 在编译时 Kotlin 编译器会将内联函数中的代码在编译的时候自动替换到调用它的地方，这样也就不存在运行时的开销了

![](https://raw.githubusercontent.com/Coming98/pictures/main/202208191254781.png)

针对多个函数参数时, 如果不希望某个函数参数被 inline 则可在前面加上 `noinline` 进行修饰

```kotlin
inline fun inlineTest(block1: () -> Unit, noinline block2: () -> Unit) { }
```

## 内联的局限[略]

内联函数类型的参数在编译的时候会被进行代码替换，因此它没有真正的参数属性，只允许传递给另外一个内联函数，这也是它最大的局限性

非内联的函数类型参数可以自由地传递给其他任何函数，因为它就是一个真实的参数

内联函数实现了替换, 因此可以实现局部的返回与终止; 而非内联函数则重是在局部中止自身的函数体, 并不能实现高阶函数的局部返回与终止

![](https://raw.githubusercontent.com/Coming98/pictures/main/202208191541532.png)

![](https://raw.githubusercontent.com/Coming98/pictures/main/202208191541886.png)

# 委托

委托是一种设计模式, 指操作对象自己不会去处理某段逻辑，而是会把工作委托给另外一个辅助对象去处理
- 意义所在: 让大部分的方法实现调用辅助对象中的方法，少部分的方法实现由自己来重写，甚至加入一些自己独有的方法

![](https://raw.githubusercontent.com/Coming98/pictures/main/202208220907140.png)

## 类委托

通过委托创建新类时使用 `by` 关键字指定委托对象, 就会自动将元对象的方法继承过来, 从而实现对目标对象的继承与封装

```kotlin
class MySet<T>(val helperSet: HashSet<T>) : Set<T> by helperSet {
    fun helloWorld() = println("Hello World")
    override fun isEmpty() = false
}
```

## 属性委托

同类委托类似, 将一个属性（字段）的具体实现委托给另一个类去完成, 依旧使用 `by` 关键字指定委托实例
- 当使用属性 p 的时候会自动调用实例的 getValue() 方法, 赋值属性 p 则调用 setValue() 方法
- 因此类中需要提前准备好接口实现 value, getValue, setValue
- getValue 与 setValue 中的第一个参数是接收使用属性委托的类的实例, 第二个参数 KProperty<*> 是 Kotlin 中的一个属性操作类, 用于获取各种属性相关值

```kotlin
class MyClass {
    var p by Delegate()
}

class Delegate {
    var propValue: Any? = null
    operator fun getValue(myClass: MyClass, prop: KProperty<*>): Any? {
        return propValue
    }
    operator fun setValue(myClass: MyClass, prop: KProperty<*>, value: Any?) {
        propValue = value
    }
}
```

# lazy 函数

懒加载技术使得变量的初始化代码在变量被首次调用时 `by lazy{}` 代码块中的代码才会执行, 本质就是属性委托, lazy 是 Kotlin 中的高阶函数, 用于快速创建并返回一个 Delegate 对象

```kotlin
val p by lazy { ... }
```

动手实现高阶 lazy 高阶函数

```kotlin
class Later<T>(val block: () -> T) {
    var value: Any? = null
    operator fun getValue(any: Any?, prop: KProperty<*>): T {
        if (value == null) {
            value = block()
        }
        return value as T
    }
}
// 通过 Later.kt 中定义的顶层函数进行使用
fun <T> later(block: () -> T) = Later(block)
```
