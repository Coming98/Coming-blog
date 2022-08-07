---
title: 03-Android-Activity
mathjax: false
date: 2022-07-18 16:16:45
summary: Android Activity 基础使用
categories: Android Dev.
tags:
  - Android
---
# Activity

Activity：一种可以包含用户界面的组件，主要用于和用户进行交互

## 创建 Activity

在我们的项目包下右击创建:
- Generate Layout File: 表示自动为 Activity 创建对应的布局文件
- Launcher Activity: 表示自动将 Activity 设置为当前项目的主 Activity

```kotlin
package com.example.helloandroid

class FirstActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.first_layout)
        Log.d("MainActivity", "onCreate execute")
    }
}
```

## 创建布局

每一个 Activity 都应对应一个布局（逻辑和视图分离），在 `app/src/main/res` 目录中的 `layout` 文件夹中创建 `Layout resource file` 即可

- 在布局中进行布局的配置以及布局中元素的配置

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <Button
        android:id="@+id/button1"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/main_title"
        tools:ignore="MissingConstraints" />

</LinearLayout>
```

## 加载布局

将布局加载到 Activity 中: `setContentView(R.layout.first_layout)` 方法中传入的是目标布局文件的 id
> 项目中添加的任何资源都会在 R 文件中生成一个相应的资源 id

## 注册 Activity

所有的 Activity 都要在 AndroidManifest.xml 中进行注册才能生效, 注册声明放在 `<application>` 标签内
- 在这里配置 activity 的基本属性: 名称(`name`), 是否可被其它 APP 使用(`exported`), 标题名(`label`)
- 配置是否为主 Activity `intent-filter`

Tips: 如果你的应用程序中没有声明任何一个 Activity 作为主 Activity，这个程序仍然是可以正常安装，只是无法在启动器中看到或者打开这个程序 (这种程序一般是作为第三方服务供其他应用在内部进行调用的)

```xml
<activity
    android:name=".FirstActivity"
    android:exported="true"
    android:label="This is FirstActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"></action>
        <category android:name="android.intent.category.LAUNCHER"></category>
    </intent-filter>
</activity>
```

## 销毁 Activity

通过 Back 键或者 `finish()` 方法

# Intent

Intent 是 Android 程序中各组件之间进行交互的一种重要方式，它不仅可以指明当前组件想要执行的动作，还可以在不同组件之间传递数据
- Application: 启动 Activity、启动 Service 以及发送广播

## 显示 Intent

1. 使用构造函数, 定义我们的意图: `Intent(Context packageContext, Class<?> cls)`
- Context: 启动 Activity 的上下文
- Class: 指定想要启动的目标 Activity

2. 使用 `startActivity()` 执行我们的意图: `startActivity(intent)`

```kotlin
val button_main2first = findViewById<Button>(R.id.button_main2first)
button_main2first.setOnClickListener {
    // 在FirstActivit 的环境中打开 SecondActivity
    // (kotlin) FirstActivity::class.java === FirstActivity.class (Java)
    val intent = Intent(this, FirstActivity::class.java)
    startActivity(intent)
}
```

## 隐式 Intent

并不明确指出想要启动哪一个 Activity，而是指定了一系列更为抽象的 action 和 category 等信息，然后交由系统去分析这个 Intent，并帮我们找出合适的 Activity 去启动
- 在 `AndroidManifest.xml` 中的 `activity` 标签中的 `intent-filter` 中可以指定当前 Activity 能够响应的 action 和 category


只有 `<action>` 和 `<category>` 中的内容同时匹配 Intent 中指定的 action 和 category 时，这个 Activity 才能响应该 Intent: 
- 可以指定多个 category, 但是必须要指定能够响应 `android.intent.category.DEFAULT` 这个 category
```xml
<activity
    android:name=".SecondActivity"
    android:exported="true"
    android:label="@string/second_label">
    <intent-filter>
        <!-- 可以响应 com.example.activitytest.ACTION_START -->
        <action android:name="com.comingpro.intent.INFO_SHOW" />
        <!-- 指明了当前 Activity 能够响应的 Intent 中还可能带有的 category -->
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="com.comingpro.category.SECOND" />
    </intent-filter>
</activity>
```

隐式调用: 每个 Intent 中只能指定一个 action，但能指定多个 category
- 使用 `intent.addCategory()` 方法添加自定义的 category
```kotlin
val button_main2second = findViewById<Button>(R.id.button_main2second)
button_main2second.setOnClickListener {
    val intent = Intent("com.comingpro.intent.INFO_SHOW")
    intent.addCategory("com.comingpro.category.SECOND")
    // android.intent.category.DEFAULT 是一种默认的 category
    // 在调用 startActivity() 方法的时候会自动将这个 category 添加到 Intent 中
    startActivity(intent)
}
```

## 跨程序意图

隐式 Intent 可以启动其它程序内的 Activity

### setData()

`setData()` 方法接收一个 `Uri` 对象, 用于指定当前 Intent 正在操作的数据, 这些数据通常是以字符串形式传入 `Uri.parse()` 方法中解析产生的

更详细的配置 Data 格式可以在 `<intent-filter>` 中配置一个 `<data>` 标签, 表明当前 Activity 支持处理的 Data 类型: 只有当 data 标签中指定的内容和 Intent 中携带的 Data 完全一致时，当前 Activity 才能够响应该 Intent
- `a:scheme`: 指定数据的协议部分(https, ...)
- `a:host`: 指定数据的主机名部分(www.baidu.com, ...)
- `a:port`: 指定数据的端口部分
- `a:path`: 指定主机名和端口之后的部分
- `a:mimeType`: 指定可以处理的数据类型，允许使用通配符的方式进行指定

### 启动浏览器

使用本地注册好的浏览器打开百度:
```kotlin
val button_baidu = findViewById<Button>(R.id.button_baidu)
button_baidu.setOnClickListener {
    // Android 系统内置动作，其常量值为 android.intent.action.VIEW
    val intent = Intent(Intent.ACTION_VIEW)
    // 通过 Uri.parse() 方法将一个网址字符串解析成一个 Uri 对象
    // 再调用 Intent 的 setData() 方法将这个 Uri 对象传递进去
    intent.data = Uri.parse("https://www.baidu.com")
    startActivity(intent)
}
```

使用本程序创建的子 Activity 匹配 Intent:
```xml
<activity
    android:name=".FakeBaiduActivity"
    android:exported="true"
    android:label="@string/fake_baidu_label">
    <intent-filter tools:ignore="AppLinkUrlError">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:scheme="https" />
    </intent-filter>
</activity>
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202207162046144.png)

### 拨打电话

action 为内置的 `Intent.ACTION_DIAL` 接收的数据格式为 `tel: 12345`

```kotlin
val button_call12345 = findViewById<Button>(R.id.button_call12345)
button_call12345.setOnClickListener { 
    val intent = Intent(Intent.ACTION_DIAL)
    intent.data = Uri.parse("tel: 12345")
    startActivity(intent)
}
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202207162054621.png)

## 活动间传递数据

重载 `putExtra()` 方法, 将想要传递的数据暂存在 Intent 中进行传递即可, 通过 `key, value` 的形式传递与获取

发送数据:
```kotlin
val button_greet = findViewById<Button>(R.id.button_greet)
button_greet.setOnClickListener {
    val intent = Intent(this, HelloActivity::class.java)
    intent.putExtra("name", "Coming")
    startActivity(intent)
}
```

接收数据:
```kotlin
val text_hello = findViewById<TextView>(R.id.text_hello)
val extra_data_name = intent.getStringExtra("name")
text_hello.text = "Hello $extra_data_name"
```

## 活动间返回数据

目标 Activity 销毁后能给上级 Activity 响应消息

1. 在启动目标 Activity 前使用 `registerForActivityResult` 注册绑定一个回调, 对目标 Activity 销毁后传递的信息进行处理

```kotlin
private val greet_advanced_activity = registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result ->
    if (result.resultCode == Activity.RESULT_OK) {
        val data: Intent? = result.data
        val name = data?.getStringExtra("name") ?: "What?"
        val button_greet_advanced = findViewById<Button>(R.id.button_greet_advanced)
        button_greet_advanced.text = "Say ${name} Success!"
    }
}
```

2. 使用这个回调注册的 `launch()` 方法启动目标 Activity

```kotlin
val button_greet_advanced = findViewById<Button>(R.id.button_greet_advanced)
button_greet_advanced.setOnClickListener {
    val intent = Intent(this, HelloActivity::class.java)
    intent.putExtra("name", "CJC")
    greet_advanced_activity.launch(intent)
}
```

3. 目标 Activity 中编写销毁后返传的消息

```kotlin
val button_back2main = findViewById<Button>(R.id.button_info2main)
button_back2main.setOnClickListener {
    val intent = Intent()
    intent.putExtra("name", "CJC")
    setResult(RESULT_OK, intent)
    finish()
}
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202207162141011.png)

# 生命周期

Android 中的 Activity 是可以层叠的：每启动一个新的 Activity，就会覆盖在原 Activity 之上，然后点击 Back 键会销毁最上面的 Activity，下面的一个 Activity 就会重新显示出来

Android 使用任务（Task）来管理活动的：一个任务就是一组存放在栈里的 Activity 的集合，这个栈也被称作返回栈（Back Stack）

![](https://raw.githubusercontent.com/Coming98/pictures/main/202207170842696.png)

## Activity 状态

1. 运行状态：活动处于栈顶
2. 暂停状态：不再处于栈顶，但仍然可见（并不是每一个活动都占满整个屏幕：对话框）
3. 停止状态：非栈顶 + 完全不可见（系统仍然会为这种 Activity 保存相应的状态和成员变量）
4. 销毁状态：从返回栈中移除后

Tips: 系统回收的优先级, 即当内存不足时会优先回收(销毁状态, 停止状态); 运行状态与暂停状态时可见的, 回收会影响用户体验。

## 生命周期的回调

![](https://raw.githubusercontent.com/Coming98/pictures/main/202207170846189.png)

Activity 类中定义了 7 个回调方法，覆盖了 Activity 生命周期的每一个环节:
- `onCreate()`：活动第一次被创建时调用（初始化操作：加载布局，绑定事件）
- `onStart()`：活动由不可见变为可见时调用
- `onResume()`：活动（栈顶+运行态）准备好和用户进行交互的时候调用
- `onPause()`：系统准备去启动或恢复另一个活动的时候调用
  - 通常会在这个方法中将一些消耗 CPU 的资源释放掉，以及保存一些关键数据
  - 但这个方法的执行速度一定要快，不然会影响到新的栈顶 Activity 的使用
- `onStop()`：在活动完全不可见的时候调用
  - 如果启动的新 Activity 是一个对话框式的 Activity，那么 onPause() 方法会得到执行，而 onStop() 方法并不会执行
- `onDestroy()`：活动被销毁前执行
- `onRestart()`：活动由停止状态变为运行状态之前调用

以上 7 个方法中除了 onRestart() 方法，其他都是两两相对的，从而又可以将 Activity 分为以下 3 种生存期: 
- 完整生存期：onCreate() 方法和 onDestroy() 方法之间经历的
  - onCreate() 完成各种初始化, onDestroy() 完成各种释放工作
- 可见生存期：onStart() 方法 和 onStop() 方法之间（Activity 总是可见的）
  - onStart() 对资源进行加载, onStop() 对资源进行释放(保证处于停止状态的 Activity 不会占用过多内存)
- 前台生存期：onResume() 方法 和 onPause() 方法之间，活动总是处于运行状态, 可以与用户进行交互, 最常见

### 测试生命周期执行过程

1. MainActivity 刚刚启动, 先后执行三个生命周期: `onCreate(), onStart(), onResume()`

2. MainActivity 打开 NormalActivity 时会先后执行两个生命周期: `onPause(), onStop()`

3. 通过 Back 键从 NormalActivity 中返回到 MainActivity 时, 先后执行三个生命周期: `onRestart(), onStart(), onResume()`

4. MainActivity 打开 DialogActivity 时会执行: `onPause()` （只 Pause 未 Stop）

5. 通过 Back 键从 DialogActivity 中返回到 MainActivity 时, 执行: `onResume()`

6. 通过 Back 键退出程序, 先后执行: `onPause(), onStop(), onDestroy()`

### 活动被意外销毁的处理办法

如果因为内存原因将停止状态的活动销毁了, 通过 Back 键返回该活动, 会执行 `onCreate()` 重新创建该活动, 但是活动中的状态数据都丢失了, 因此 Android 提供了 `onSaveInstanceState()` 方法, 其在 Activity 被回收前调用, 对临时数据进行保存

onSaveInstanceState() 方法：活动被回收之前调用，携带一个 Bundle 类型的参数，Bundle 提供了一系列方法保存数据`putString putInt ...`

```kotlin
override fun onSaveInstanceState(outState: Bundle, outPersistentState: PersistableBundle) {
    super.onSaveInstanceState(outState, outPersistentState)
    val userName = "Coming"
    outState.putString("userName", userName)
}
```

`onCreate()` 会接收一个 saveInstanceState 参数, 默认为 null, 不为空时表示要进行恢复处理:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    if (savedInstanceState != null) {
        val userName = savedInstanceState.getString("userName")
        Log.d(tag, "Name is $userName")
    }
}
```

# 启动模式

启动模式会定义目标活动的启动方式，如新建一个目标活动的实例或者用未销毁的旧实例等等，在交互方面发挥重要作用
- 启动模式一共有 4 种，分别是 standard、singleTop、singleTask 和 singleInstance
- 在 AndroidManifest.xml 中通过给 <activity> 标签指定 android:launchMode 属性来选择启动模式

## standard

默认的启动模式，每当启动一个新的活动，都会创建该活动的一个新的实例，入栈，并处于栈顶位置
- 系统不会在乎这个 Activity 是否已经在返回栈中存在，每次启动都会创建一个该 Activity 的新实例

## singleTop

启动活动时，如果发现返回栈的栈顶已经是该活动，则认为可以直接使用它
- 目标活动不处于栈顶时，仍会创建新的活动实例。

![](https://raw.githubusercontent.com/Coming98/pictures/main/202207171009445.png)

## singleTask

启动目标活动(A)时会在返回栈中检查是否已存在该活动的实例(a)，如果发现已经存在则直接使用该实例(a)，并把(a)之上的所有活动实例出栈

![](https://raw.githubusercontent.com/Coming98/pictures/main/202207171016399.png)

## singleInstance

启动活动时会启用一个新的返回栈来管理这个活动

Application: 其他程序和我们的程序可以共享这个 Activity 的实例时, 使用 singleInstance 模式会有一个单独的返回栈来管理这个 Activity，也就解决了共享Activity实例的问题

![](https://raw.githubusercontent.com/Coming98/pictures/main/202207171020987.png)

## 活动常用方法

### 当前可见活动名称

1. 创建一个 BaseActivity 类

```kotlin
open class BaseActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // javaClass == getClass()
        Log.d("BaseActivity", javaClass.simpleName)
    }
}
```

### 统一活动管理

当我们的活动栈中存在三个 Activity 时, 如何直接退出程序呢? 这就需要一个数据结构 `ActivityCollector()` 去维护所有 Activity 的状态, 并统一控制其行为

1. 新建 ActivityCollector 类

```kotlin
class ActivityCollector {

    private val activityCollector = ArrayList<Activity>()

    fun addActivity(activity: Activity) {
        activityCollector.add(activity)
    }

    fun removeActivity(activity: Activity) {
        activityCollector.remove(activity)
    }

    fun finishAll() {
        for ( activity in activityCollector ) {
            if (!activity.isFinishing) {
                activity.finish()
            }
        }
        activityCollector.clear()
        // 销毁当前进程
        android.os.Process.killProcess(android.os.Process.myPid())
    }

}
```

2. 在 BaseActivity 基类中进行封装维护

```kotlin
open class BaseActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // javaClass == getClass()
        Log.d("BaseActivity", javaClass.simpleName)
        ActivityCollector.addActivity(this)
    }

    override fun onDestroy() {
        super.onDestroy()
        ActivityCollector.removeActivity(this)
    }

}
```

3. 使用 finishAll() 方法实现退出即可


### 活动启动接口

两个活动由不同的开发者开发, 那么相互调用时可能因为不熟悉参数命名导致 intent 无法传参, 因此建议开发活动时, 在活动中声明创建该活动的意图的接口
- 这属于活动类的方法, 因此用静态修饰
- Kotlin 提供了 `companion object` 语法结构, 其中的方法都可以使用类似于 java 静态方法的形式调用