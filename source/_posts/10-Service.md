---
title: 10-Service
mathjax: false
date: 2022-08-23 15:18:22
summary: Android Service 基本认识
categories: Android Dev.
tags:
  - Android
---
# Service

后台运行的服务, 不需要与用户交互且需要长期执行的任务

Service 并不是运行在一个独立的进程当中的，而是依赖于创建 Service 时所在的应用程序进程，当某个应用程序进程被杀掉时，所有依赖于该进程的 Service 也会停止运行

## Definition

继承 Service 类, onCreate() 方法在 Service 创建时调用; onStartCommand() 在 Service 启动时调用; onDestroy() ...;

```kotlin
class MyService : Service() {

    private val TAG = "MyService"

    override fun onCreate() {
        super.onCreate()
        Log.d(TAG, "onCreate")
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        Log.d(TAG, "onStartCommand")
        return super.onStartCommand(intent, flags, startId)
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d(TAG, "onDestroy")
    }
}
```

## Contorller

主要通过 intent 控制 Service 的启动或关闭

Tips: 从 Android 8.0 开始，应用的后台功能被大幅削减，现在只有当应用保持在前台可见状态的情况下，Service 才能保证稳定运行，一旦应用进入后台之后，Service 随时都有可能被系统回收

```kotlin
R.id.startServiceBtn -> {
    val intent = Intent(this, MyService::class.java)
    startService(intent)
}
R.id.stopServiceBtn -> {
    val intent = Intent(this, MyService::class.java)
    stopService(intent)
}
```

## Communication

Service 被创建用于执行相应的功能, 那就会涉及到执行结果的反馈, 因此引入 Binder 对象实现 Activity 与 Service 的通信

1. 在 Service 中定义内部 Binder 类, 实现对所传递消息的封装, 通过 onBind 方法与 Activity 进行交互

```kotlin
private val mBinder = DownloadBinder()

inner class DownloadBinder: Binder() {
    fun startDownload() {
        Log.d(TAG, "startDownload")
    }
    fun getProgress(): Int {
        Log.d(TAG, "get process")
        return 0
    }
}

override fun onBind(intent: Intent): IBinder {
    return mBinder
}
```

2. Activity 中通过 bindService 与 unbindService 与目标 Service 进行交互: 主要通过 ServiceConnection 匿名类的实现定义交互的动作

```kotlin

lateinit var downloadBinder: MyService.DownloadBinder

private val connection = object: ServiceConnection {
    override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
        // 获取 Service Binder 对象
        downloadBinder = service as MyService.DownloadBinder
        // 发布命令
        downloadBinder.startDownload()
        downloadBinder.getProgress()
    }

    override fun onServiceDisconnected(name: ComponentName?) {
        Log.d("MyService", "Service Disconnected")
    }
}

R.id.bindServiceBtn -> {
    val intent = Intent(this, MyService::class.java)
    // Intent, ServiceConnetion, 标志位: BIND_AUTO_CREATE, 表示在 Activity 和 Service 进行绑定后自动创建 Service
    bindService(intent, connection, Context.BIND_AUTO_CREATE)
}
R.id.unbindServiceBtn -> {
    unbindService(connection)
}
```

## Life Cycle

- Context.startService() 触发 Service 的 onStartCommand(); 如果 Service 并没有被创建, 还会触发 onCreate()
- Context.stopService() 或 stopSelf() 触发 onDestroy()
- Context.bindService() 触发 onBind(); 如果 Service 并没有被创建, 还会触发 onCreate()
- Context.unbindService() 触发 onDestroy()
- 如果先调用了 startService 然后调用了 bindService 那么需要调用 stopService 与 unbindService 才会触发 onDestroy

# Front Desk Service

从 Android 8.0 开始，应用的后台功能被大幅削减，现在只有当应用保持在前台可见状态的情况下，Service 才能保证稳定运行，一旦应用进入后台之后，Service 随时都有可能被系统回收

可以使用前台 Service 使得服务能保持稳定运行状态, 但一直会有一个正在运行的图标在系统的状态栏显示，下拉状态栏后可以看到更加详细的信息（类似于通知的效果）

1. 在 Service 的 onCreate 声明周期中进行定义

```kotlin
override fun onCreate() {
    super.onCreate()
    Log.d(TAG, "onCreate")
    // 定义前台 Service
    val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        val channel = NotificationChannel("jc Service", "前台 Service 通知", NotificationManager.IMPORTANCE_DEFAULT)
        manager.createNotificationChannel(channel)
    }
    val intent = Intent(this, MainActivity::class.java)
    val pi = PendingIntent.getActivity(this, 0, intent, 0)
    val notification = NotificationCompat.Builder(this, "jc Service")
        .setContentTitle("JC Service")
        .setContentText("This is content text")
        .setSmallIcon(R.drawable.small_icon)
        .setLargeIcon(BitmapFactory.decodeResource(resources, R.drawable.large_icon))
        .setContentIntent(pi)
        .build()
    // 注册前台 Service 等待启动
    startForeground(1, notification)
}
```

2. 在 AndroidManifest 中声明前台 Service 权限

```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```

# Multi Thread

创建子线程执行相关任务, 防止主线程堵塞

## Quick Start

1. 定义一个线程任务类, 实现 Runnable 接口

> Tips: 学习 Java 多线程时有提到, 单继承多接口的机制, 导致继承 Thread 耦合度较高, 因此推荐实现 Runnable 接口

```kotlin
class MyThread: Runnable {
    override fun run() {
        // 编写具体的逻辑
    }
}
```

2. 启动线程

```kotlin
val myThread = MyThread()
Thread(myThread).start()
```

熟悉 Kotlin 的话, 可以直接使用 Lambda 实现快速启动

```kotlin
Thread {
    ...
}.start()
```

更简单的 Kotlin 又进行了封装, 使用 Kotlin 内置的顶层函数

```kotlin
thread {

}
```

## Update UI in thread

原则上 UI 的更新都是线程不安全的, 因此直接再线程中更新 UI 会触发异常: `Only the original thread that created a view hierarchy can touch its views`

如果必须要再线程中更新 UI 那么需要使用异步消息处理机制, 类似于 React 中的消息订阅机制(Redux 那一套)

1. 创建消息, 指定消息 ID 并发送

```kotlin
ButtonChangeText.setOnClickListener {
    thread {
        val msg = Message()
        msg.what = UPDATE_TEXT
        handler.sendMessage(msg)
    }
}
```

2. 消息处理

```kotlin
val handler = object: Handler(Looper.getMainLooper()) {
    // 主线程中执行
    override fun handleMessage(msg: Message) {
        when (msg.what) {
            UPDATE_TEXT -> textView.text = "Nice to meet you"
        }
    }
}
```

# 异步消息处理机制

组成: Message, Handler, MessageQueue, Looper
- Message: 线程之间传递的消息, 可以在内部携带少量的信息，用于在不同线程之间传递数据
- Handler: 用于发送和处理消息的
- MessageQueue: 存放所有通过 Handler 发送的消息, 等待被处理, 每个线程中只会有一个 MessageQueue 对象
- Looper: 每个线程中的 MessageQueue 的管家，调用 Looper 的 loop() 方法后，就会进入一个无限循环当中，然后每当发现 MessageQueue 中存在一条消息时，就会将它取出，并传递到 Handler 的 handleMessage() 方法中。每个线程中只会有一个Looper对象

1. 主线程中创建 Handler 对象, 重写 handleMessage() 方法
2. 子线程创建 Message 对象, 通过 Handler 将 Message 发出
3. 发出的 Message 被添加到 MessageQueue 中等待被处理
4. Looper 一直尝试从 MessageQueue 中取出 Message 进行相应处理

![](https://raw.githubusercontent.com/Coming98/pictures/main/202208222155382.png)

## AsyncTask

基于异步消息处理机制的封装, 是一个抽象类, 共**三个泛型参数**
- Params: 执行相关任务所需的参数
- Progress: 在后台任务执行时，如果需要在界面上显示当前的进度，则使用这里指定的泛型作为进度单位
- Result: 当任务执行完毕后，如果需要对结果进行返回，则使用这里指定的泛型作为返回值类型

常需要重写的方法有四个:
- onPreExecute(): 在后台任务开始执行之前调用，用于进行一些界面上的初始化操作，比如显示一个进度条对话框等
- doInBackground(Params...): 在子线程中运行的具体任务, 不可显示进行 UI 操作, 可以通过一些方法如 publishProgress(Progress...) 来完成
- onProgressUpdate(Progress...): 利用参数中的数值就可以对 UI 进行更新
- onPostExecute(Result): 任务执行完毕后调用, 可以利用返回的数据进行一些 UI 操作，比如说提醒任务执行的结果，以及关闭进度条对话框等


```kotlin
class DownloadTask: AsyncTask<Unit, Int, Boolean>() {
    
}
```

# Service + Multi Thread[Deprecated, no good alternatives were found]

Service 中的代码默认运行在主线程当中，如果直接在 Service 里处理一些耗时的逻辑，就很容易出现 ANR（Application Not Responding）的情况

因此推荐 Service 使用多线程, 在 Service 的每个具体方法里开启一个子线程, 处理耗时逻辑, 而 Android 将这种异步且执行完毕后自动停止的 Service 封装为了 IntentService
