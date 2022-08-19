---
title: 06-broadcast
mathjax: false
date: 2022-08-19 15:47:24
summary: Android 中的广播机制
categories: Android Dev.
tags:
  - Android
---
# Broadcast Mechanism

每个应用程序都可以对感兴趣的广播进行注册，这样该程序就只会收到自己所关心的广播内容
- 源可能来自于系统，也可能是来自于其他应用程序

System Broadcast: 通过监听 System Broadcast 获得各种各样的 System Information(系统完成开机\电池电量变化\系统事件变化\)

# Broadcast Type

Normal Broadcast, 标准广播, 异步执行的广播
- 广播发出之后，所有的 BroadcastReceiver 几乎会在同一时刻收到这条广播消息，因此它们之间没有任何先后顺序可言
- 这种广播的效率会比较高，但同时也意味着它是无法被截断的

Ordered Broadcast, 有序广播, 同步执行的广播
- 广播发出之后，同一时刻只会有一个 BroadcastReceiver 能够收到这条广播消息，当这个 BroadcastReceiver 中的逻辑执行完毕后，广播才会继续传递
- 优先级高的 BroadcastReceiver 就可以先收到广播消息，并且前面的 BroadcastReceiver 还可以截断正在传递的广播

# Registration of BroadcastReceiver

系统中的广播列表: `D:\experiments\AndroidSdk\platforms\android-32\data\broadcast_actions.txt`

## 动态注册

在代码文件中注册：新建一个类，让它继承自 BroadcastReceiver，并重写父类的 onReceive() 方法. 当有广播到来时, onReceive() 方法就会得到执行

优势: 自由的控制广播的注册与注销
劣势: 程序启动后才能接收广播

- 通过内部类的形式动态注册 TimeChangeReceiver()

```kotlin
inner class TimeChangeReceiver: BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
        val mainLayout = findViewById<LinearLayout>(R.id.main_layout)
        val ranColor = Color.argb(255, (0 until 256).random(), (0 until 256).random(), (0 until 256).random())
        mainLayout.setBackgroundColor(ranColor)
    }
}
```

- 在 Activity 的 onCreate 生命周期中完成动态注册的绑定

```kotlin
lateinit var timeChangeReceiver: TimeChangeReceiver

override fun onCreate(savedInstanceState: Bundle?) {
    val intentFilter = IntentFilter()
    intentFilter.addAction("android.intent.action.TIME_TICK")
    timeChangeReceiver = TimeChangeReceiver()
    registerReceiver(timeChangeReceiver, intentFilter)
}
```

- 在 Activity 的 onDestroy 生命周期中完成动态注册的注销

```kotlin
override fun onDestroy() {
    super.onDestroy()
    unregisterReceiver(timeChangeReceiver)
}
```

## 静态注册

能够实现应用启动前的广播监听
> 由于恶意应用程序利用静态注册机制在程序未启动的情况下监听系统广播，从而使任何应用都可以频繁地从后台被唤醒，严重影响了用户手机的电量和
性能，因此 Android 系统几乎每个版本都在削减静态注册 BroadcastReceiver 的功能
> 在 Android 8.0 之后，所有隐式广播都不允许使用静态注册的方式来接收了，但是[少数特殊的系统广播](https://developer.android.com/guide/components/broadcast-exceptions)目前仍然允许使用静态注册的方式来接收
> 隐式广播指没有具体指定发送给哪个应用程序的广播，大多数系统广播属于隐式广播

1. 建立 BroadcastReceive 子类, 重写 onReceive 方法

```kotlin
class BootCompleteReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        // This method is called when the BroadcastReceiver is receiving an Intent broadcast.
        Toast.makeText(context, "JCAPP INFOS: Boot Complete", Toast.LENGTH_SHORT).show()
    }
}
```

2. 在 AndroidManifest.xml 中对 receiver 进行静态注册：Exported 属性表示是否允许 BroadcastReceiver 接收本程序以外的广播；Enabled 属性表示是否启用这个 BroadcastReceiver

```xml
<receiver
    android:name=".BootCompleteReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent-filter>
</receiver>
```

# Customized Broadcast

## Normal Broadcast

标准广播, 异步执行的广播, 发送后所有注册的 Receiver 都会在同一时间进行处理

1. 编写自定义广播 BroadcastReceiver 类, 实现广播响应处理

```kotlin
class JCNormalBroadcast : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        // This method is called when the BroadcastReceiver is receiving an Intent broadcast.
        Toast.makeText(context, "received in JCNormalBroadcast", Toast.LENGTH_SHORT).show()
    }
}
```

2. 在 AndroidManifest 中实现静态注册

```xml
<receiver
    android:name=".JCNormalBroadcast"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="com.example.a09broadcasttest.JC_BROADCAST" />
    </intent-filter>
</receiver>
```

3. 通过按钮点击发送自定义广播

> 8.0 之后，静态注册的 BroadcastReceiver 是无法接收隐式广播的
> 而默认情况下我们发出的自定义广播都是隐式广播
> 因此这里一定要调用 setPackage() 方法，指定这条广播是发送给哪个应用程序的，从而让它变成一条显式广播
> 否则静态注册的 BroadcastReceiver 将无法接收到这条广播

```kotlin
override fun onClick(view: View?) {
    when(view?.id) {
        R.id.main_jc_nbc -> {
            val intent = Intent("com.example.a09broadcasttest.JC_BROADCAST")
            intent.setPackage(packageName)
            sendBroadcast(intent)
        }
    }
}
```

## Ordered Broadcast

有序广播, 同步执行的广播, 发送后各个绑定的 BroadcastReceiver 会按照优先级先后处理

1. 编写有序广播的 BroadReceiver 处理类：`abortBroadcast()` 可以阶段有序广播的向下床底

```kotlin
class JCOrderedBroadcastReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        val toast = Toast.makeText(context, "received in JCOrderedBroadcastReceiver", Toast.LENGTH_SHORT)
        toast.setGravity(Gravity.TOP, 0, 0)
        toast.show()

        abortBroadcast()
    }
}
```

2. 在 AndroidManifest 中实现静态注册：`<intent-filter android:priority="100">` 设置有序广播响应的优先级

默认优先级为 0, 优先级设定范围为 `-1000 ~ 10000`

```xml
<receiver
    android:name=".JCOrderedBroadcastReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter android:priority="100">
        <action android:name="com.example.a09broadcasttest.JC_BROADCAST" />
    </intent-filter>
</receiver>
```

3. 通过按钮点击发送 Ordered Broadcast

```kotlin
R.id.main_jc_obc -> {
    val intent = Intent("com.example.a09broadcasttest.JC_BROADCAST")
    intent.setPackage(packageName)
    sendOrderedBroadcast(intent, null) // Ordered Broadcast
}
```

# BestPractice

登陆与强制下线功能

0. 涉及到下线功能, 因此使用 ActivityCollector 与 BaseActivity 管理所有 Activity

```kotlin
// 单例类
object ActivityCollector {

    private val activities = ArrayList<Activity>()

    fun addActivity(activity: Activity) {
        activities.add(activity)
    }

    fun removeActivity(activity: Activity) {
        activities.remove(activity)
    }

    fun finishAll() {
        for (activity in activities) {
            if (!activity.isFinishing) {
                activity.finish()
            }
        }
        activities.clear()
    }

}
// 可被其它 Activity 继承
open class BaseActivity: AppCompatActivity() {

    lateinit var receiver: ForceOfflineReceiver

    override fun onCreate(savedInstanceState: Bundle?, persistentState: PersistableBundle?) {
        super.onCreate(savedInstanceState, persistentState)
        ActivityCollector.addActivity(this)
    }

    override fun onDestroy() {
        super.onDestroy()
        ActivityCollector.removeActivity(this)
    }
}
```

2. 编写 LoginActivity 界面

```kotlin

class LoginActivity : BaseActivity(), View.OnClickListener {

    lateinit var accountEdit: EditText
    lateinit var passwordEdit: EditText

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_login)

        val button_login = findViewById<Button>(R.id.login)
        button_login.setOnClickListener(this)

        accountEdit = findViewById(R.id.accountEdit)
        passwordEdit = findViewById(R.id.passwordEdit)

    }

    override fun onClick(v: View?) {
        when (v?.id) {
            R.id.login -> {
                val account = accountEdit.text.toString()
                val password = passwordEdit.text.toString()
                if (account == "admin" && password == "123456") {
                    val intent = Intent(this, MainActivity::class.java)
                    startActivity(intent)
                    finish()
                } else {
                    Toast.makeText(this, "Account or Password is invalid", Toast.LENGTH_SHORT).show()
                }
            }
        }
    }
}
```

3. 在登陆后的页面中设置触发强制下线的广播 Trigger

```kotlin
val button_force_offline = findViewById<Button>(R.id.forceOffline)
button_force_offline.setOnClickListener {
    val intent = Intent("com.example.a10broadcastpractice.FORCE_OFFLINE")
    sendBroadcast(intent)
}
```

4. 需要使得当前处于栈顶的 Activity 能够接收到广播并处理, 因此在 BaseActivity 中利用生命周期函数 `onResume()` 与 `onPause()` 实现 Receiver 的动态注册与撤销

```kotlin
// 需要保证只有处于栈顶的 Activity 才能接收到这条强制下线广播
// 非栈顶的 Activity 不应该也没必要接收这条广播，所以写在 onResume() 和 onPause() 方法里就可以很好地解决这个问题
// 当一个 Activity 失去栈顶位置时就会自动取消 BroadcastReceiver 的注册
override fun onResume() {
    super.onResume()
    val intentFilter = IntentFilter()
    intentFilter.addAction("com.example.a10broadcastpractice.FORCE_OFFLINE")
    receiver = ForceOfflineReceiver()
    registerReceiver(receiver, intentFilter)
}

override fun onPause() {
    super.onPause()
    unregisterReceiver(receiver)
}

inner class ForceOfflineReceiver: BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        AlertDialog.Builder(context).apply {
            setTitle("Warning")
            setMessage("You are forced to be offline. Please try to login again.")
            setCancelable(false)
            setPositiveButton("OK") { _, _ ->
                ActivityCollector.finishAll()
                val intent = Intent(context, LoginActivity::class.java)
                context.startActivity(intent)
            }
            show()
        }
    }
}
```