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

## 扩展函数优化 Toast

对 Sting 与 Int 进行扩展, 快速 Toast 提示

```kotlin
fun String.showToast(context: Context, duration: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(context, this, duration).show()
}
fun Int.showToast(context: Context) {
    Toast.makeText(context, this, duration).show()
}
// Exec
"Success!".showToast(this)
// 甚至可以将提示信息资源话
R.string.process_success_info.showToast(this, Toast.LENGTH_LONG)
```

## 扩展函数优化 Snackbar

Snackbar 是扩展版的 Toast, 能够根据指定的 View 向上查询 Context 并给出可交互的提示信息, 因此选择对 View 进行扩展, 优化 Snackbar 的调用

```kotlin
// block 表示其类型可以是函数也可以是 null, 默认值为 null
fun View.showSnackbar(text: String, actionText: String? = null, 
                    duration: Int = Snackbar.LENGTH_SHORT, block: (() -> Unit)? = null) {
    val snackbar = Snackbar.make(this, text, duration)
    if (actionText != null && block != null) {
        snackbar.setAction(actionText) {
            block()
        }
    }
    snackbar.show()
}
// Exec
view.showSnackbar("Delete Success!", "UNDO") {
    "Undo Success!".showToast(this)
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

# infix

使用 infix 关键字能够创建类似 `contentProviderOf("name" to "Coming")` 这样的 A to B 的用法, 实质为 `A.to(B)` 的语法糖

约束条件: infix 函数必须为某个类的成员函数, 不支持顶层函数; infix 函数必须接收且只能接收一个参数

```kotlin
// 使用 infix 对 String 建立扩展函数 beginsWith
infix fun String.beginsWith(prefix: String) = startsWith(prefix)
// infix 函数额外支持下方调用方式
if ("Hello Kotlin" beginsWith "Hello") {}
// 等价于
if ("Hello Kotlin".startsWith("Hello")) {}
```

Example2: 判断集合中是否有某个元素

```kotlin
infix fun <T> Collection<T>.has(element: T) = contains(element)

val list = listOf("Apple", "Banana", "Orange", "Pear", "Grape")
if (list has "Banana") {}
```

Example3: infix to 函数的源码

```kotlin
public infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
```

# Advanced Generic

## 泛型实化

类型擦除机制: 泛型对于类型的约束只在编译时期存在，运行的时候 JVM 是识别不出来我们在代码中指定的泛型类型的; 即运行时类型已经被擦除了
- 例如，我们创建了一个 `List<String>` 集合，虽然在编译时期只能向集合中添加字符串类型的元素，但是在运行时期 JVM 并不能知道它本来只打算包含哪种类型的元素，只能识别出来它是个List

泛型实化: 借助 Kotlin 的内联函数替换的功能, 从而保留了函数体内的泛型, 甚至支持在函数体中操作泛型

![](https://raw.githubusercontent.com/Coming98/pictures/main/202208231403755.png)

```kotlin
// 内联函数 + 声明泛型的地方加上 reified 关键字表示该泛型要进行实化
inline fun <reified T> getGenericType() {}

// 函数体内甚至可以通过泛型获取其在执行时的真正类型
inline fun <reified T> getGenericType() = T::class.java
```

### Application

优化 startActivity:

```kotlin
inline fun <reified T> startActivity(context: Context, block: Intent.() -> Unit) {
    val intent = Intent(context, T::class.java)
    // 添加参数等信息
    intent.block()
    context.startActivity(intent)
}
// Application
startActivity<TestActivity>(context) {
    putExtra("param1", "data")
    putExtra("param2", 123)
}
```

## 泛型的协变[HARD, TODO]

假如定义了一个 `MyClass<T>` 的泛型类，其中 A 是 B 的子类型，同时 `MyClass<A>`又是 `MyClass<B>` 的子类型，那么我们就可以称 MyClass 在 T 这个泛型上是协变的
- 前提: 一个泛型类在其泛型类型的数据上是只读的话. 要实现这一点，则需要让 MyClass<T> 类中的所有方法都不能接收 T 类型的参数。换句话说，T 只能出现在 out 位置上，而不能出现在 in 位置上

![](https://raw.githubusercontent.com/Coming98/pictures/main/202208231508585.png)

```kotlin
class SimpleData<out T>(val data: T?) {
    fun get(): T? {
        return data
    }
}
```

# 协程

协程可以理解为轻量级的线程, 普通的线程需要依靠操作系统的调度实现线程间的切换, 使用协程可以仅在编程语言的层面就能实现不同协程之间的切换，从而大大提升了并发编程的运行效率
- 在单线程模式下模拟多线程编程的效果，代码执行时的挂起与恢复完全是由编程语言来控制的，和操作系统无关

## Quick Start

1. 引入依赖

```shell
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.1.1"
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:1.1.1"
```

2. 使用 `GlobalScope.launch` 创建顶层协程作用域: 这种协程当应用程序运行结束时也会跟着一起结束

```kotlin
val job = GlobalScope.launch {
    println("codes run in coroutine scope")
    for(i in 0..100 step 2) {
        println("index = $i")
    }
}
job.cancel() // 手动取消协程
```

## 阻塞

常见的又 `Thread.sleep(millis)` 与 `delay(millis)`
- delay: 是一个非阻塞式的挂起函数，它只会挂起当前协程，并不会影响其他协程的运行 (只能在协程的作用域或其他挂起函数中调)
- sleep: 会阻塞当前的线程，这样运行在该线程下的所有协程都会被阻塞

## runBlocking 协程

GlobalScope.launch 创建的协程会随着应用程序的结束而结束, 这在离线测试时十分不友好, 因此封装了 runBlocking.launch 其可以保证在协程作用域内的所有代码和子协程没有全部执行完之前一直阻塞当前线程

Tips: runBlocking 函数通常只应该在测试环境下使用，在正式环境中使用容易产生一些性能上的问题

### 创建多个子协程

如果外层作用域的协程结束了，该作用域下的所有子协程也会一同结束

```kotlin
runBlocking {
    launch {
        println("launch1")
        delay(1000)
        println("launch1 finished")
    }
    launch {
        println("launch2")
        delay(1000)
        println("launch2 finished")
    }
}
```

## 对协程操作的封装

如果想要将协程中执行的任务封装为函数, 但是如何拥有其协程作用域呢？可以使用 `suspend` 关键字, 将任意函数声明为挂起函数, 并且挂起函数之间是可以相互调用的, 这就支持了如 delay 等挂起函数的调用

```kotlin
suspend fun printDot() {
    repeat(100) {
        print(".")
        delay(100)
    }
}
```

但是如何真正的实现协程作用域的共享呢, 需要借助 `coroutineScope` 函数进行显示的声明
- coroutineScopre() 也是一个挂起函数, 可以在任何挂起函数中或协程作用域中调用
- 其可以继承外部的协程的作用域并创建一个子协程，借助这个特性，我们就可以给任意挂起函数提供协程作用域了
- 其可以保证其作用域内的所有代码和子协程在全部执行完之前，外部的协程会一直被挂起
- 和 runBlocking 不同的是 coroutineScope 函数只会阻塞当前协程，既不影响其他协程，也不影响任何线程，因此是不会造成任何性能上的问题的: 涉及的是子协程, 不用关注外部线程
- 而 runBlocking 函数由于会挂起外部线程，如果你恰好又在主线程中当中调用它的话，那么就有可能会导致界面卡死的情况，所以不太推荐在实际项目中使
用: 涉及的是顶层协程, 需要阻塞线程以保存活

```kotlin
suspend fun printDot() = coroutineScope {
    launch {
        println(".")
        delay(1000)
    }
}
```

## 推荐的协程实践

主要就是方便多个子协程的维护

```kotlin
val job = Job()
val scope = CoroutineScope(job)
scope.launch {
    for (i in 1..100 step 2) {
        delay(50)
        println("index = $i")
    }
}
scope.launch {
    for (i in 0..100 step 2) {
        delay(50)
        println("index = $i")
    }
}
// 销毁的时候调用
// job.cancel() // 调用一次 cancel 方法即可取消目标 scope 中的所有协程, 维护起来更加方便
```

## 协程的回调

### async/await

使用 `async` 关键字获取目标协程的返回值, 该关键字必须在协程作用域中使用, 其原理是创建一个新的子协程, 并返回一个 Deferred 对象, 通过执行该对象的 `await()` 方法获取执行结果
- 调用了 async 函数之后，代码块中的代码就会立刻开始执行
- 当调用 await() 方法时，如果代码块中的代码还没执行完，那么 await() 方法会将当前协程阻塞住，直到可以获得 async 函数的执行结果
- 因此选择何时调用 await 的时机很重要

```kotlin
GlobalScope.launch {
    getCoroutineRet()
}

//--------------------------------------//

suspend fun getCoroutineRet() {
    val job = Job()
    val scope = CoroutineScope(job)
    val ret = scope.async {
        var ret = 0
        for(i in 0..100 step 2) {
            ret += i
            delay(50)
        }
        ret
    }.await()
    println("$ret")
}
```

使用 runBlocking 时更加简便, 因为有一个全局的协程作用域

```kotlin
runBlocking {
    val result = async {
        5 + 5
    }.await()
    println(result)
}
```

### withContext

可以理解成 async 函数的一种简化版写法
- 调用 withContext() 函数之后，会立即执行代码块中的代码，同时将外部协程挂起
- 当代码块中的代码全部执行完之后，会将最后一行的执行结果作为 withContext() 函数的返回值返回
- Dispatchers.Default 是 withContext 强制要求的线程参数, 数给协程指定一个具体的运行线程
  - Dispatchers.Default 表示会使用一种默认低并发的线程策略，当你要执行的代码属于计算密集型任务时，开启过高的并发反而可能会影响任务的运行效率，此时就可以使用 Dispatchers.Default
  - Dispatchers.IO 表示会使用一种较高并发的线程策略，当你要执行的代码大多数时间是在阻塞和等待中，比如说执行网络请求时，为了能够支持更高的并发数量，此时就可以使用 Dispatchers.IO
  - Dispatchers.Main 则表示不会开启子线程，而是在 Android 主线程中执行代码，但是这个值只能在 Android 项目中使用，纯 Kotlin 程序使用这种类型的线程参数会出现错误


```kotlin
runBlocking {
    val result = withContext(Dispatchers.Default) {
        5 + 5
    }
    println(result)
}
```

## 使用协程优化传统回调

借助 suspendCoroutine 函数就能将传统回调机制的写法大幅简化, 该函数必须在协程作用域或挂起函数中才能调用，它接收一个 Lambda 表达式参数，主要作用是将当前协程立即挂起，然后在一个普通的线程中执行 Lambda 表达式中的代码

Lambda 表达式的参数列表上会传入一个 Continuation 参数，调用它的 resume() 方法或 resumeWithException() 可以让协程恢复执行

# DSL

DSL 是领域特定语言（Domain Specific Language）通过它我们可以编写出一些看似脱离其原始语法结构的代码，从而构建出一种专有的语法结构

例如构造一个能够生成 table, tr, td 标签的语法结构

1. 首先定义底层单元结构 td

```kotlin
class Td {
    var content = ""
    fun html() = "\n\t\t<td>$content</td>"
}
```

2. 逐层向上, 定义 tr 的结构, 实现对子 td 的封装维护

```kotlin
class Tr {
    private val children = ArrayList<Td>()
    fun td(block: Td.() -> String) {
        val td = Td()
        td.content = td.block()
        children.add(td)
    }
    fun html(): String {
        val builder = StringBuilder()
        builder.append("\n\t<tr>")
        for (childTag in children) {
            builder.append(childTag.html())
        }
        builder.append("\n\t</tr>")
        return builder.toString()
    }
}
```

3. 继续向上, 定义 table 的结构, 实现对子 tr 的封装维护

```kotlin
class Table {
    private val children = ArrayList<Tr>()
    fun tr(block: Tr.() -> Unit) {
        val tr = Tr()
        tr.block()
        children.add(tr)
    }
    fun html(): String {
        val builder = StringBuilder()
        builder.append("<table>")
        for (childTag in children) {
            builder.append(childTag.html())
        }
        builder.append("\n</table>")
        return builder.toString()
    }
}
```

4. 实现顶层结构的封装

```kotlin
fun table(block: Table.() -> Unit): String {
    val table = Table()
    table.block()
    return table.html()
}
```

5. 构建 table

```kotlin
fun main() {
    val html = table {
        tr {
            td { "Apple" }
            td { "Grape" }
            td { "Orange" }
        }
        tr {
            td { "Pear" }
            td { "Banana" }
            td { "Watermelon" }
        }
    }
    println(html)
}
```
![](https://raw.githubusercontent.com/Coming98/pictures/main/202209111418343.png)

# Java2Kotlin

- 借助 Android Studio: 将 Java 代码粘贴到 Android Studio 的 Kotlin 文件中, 会自动识别并提示转换
- 导航栏中的 Code → Convert Java File to Kotlin File

# Kotlin2Java

Kotlin 拥有许多 Java 中并不存在的特性，因此很难执行一键转换

但是可以先将 Kotlin 代码转换成 Kotlin 字节码，然后通过反编译的方式将它还原成 Java 代码；这种反编译出来的代码可能无法像正常编写的 Java 代码那样直接运行，但是非常有利于帮助我们理解诸多 Kotlin 特性背后的实现原理
- 导航栏中的 Tools→Kotlin→Show Kotlin Bytecode 
- 点击窗口左上角的 Decompile 按钮，就可以将这些 Kotlin 字节码反编译成 Java 代码