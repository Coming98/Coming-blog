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

又叫作类方法，指的就是那种不需要创建实例就能调用的方法; Korlin 中将由静态方法组成的工具类用单例类实现(`object` 关键词进行修饰)

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