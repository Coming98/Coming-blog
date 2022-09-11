---
title: 13-Jetpack
mathjax: false
date: 2022-09-11 14:19:44
summary: Jetpack 高效开发
categories: Android Dev.
tags:
  - Android
---
---
# Jetpack

Jetpack 是一个开发组件工具集，它的主要目的是帮助我们编写出更加简洁的代码，并简化我们的开发过程

全家桶: 基础, 架构, 行为, 界面; 这里我们主要介绍 Jetpack 架构

# ViewModel

Motivation: 传统 Activity 既要负责逻辑处理又要负责 UI 展示, 甚至还得处理网络回调

ViewModel 是专门用于存放与界面相关的数据的
- 手机发生横竖屏旋转时, Activity 会被重建, 但 ViewModel 声明周期与 Activity 不同, 因此数据也不会丢失
- 只有调用 `onCleared()` 方法 ViewModel 才会被销毁

## Quick Start

1. 推荐给每一个 Activity 和 Fragment 都创建一个对应的 ViewModel

```kotlin
class MainViewModel: ViewModel() {
    var counter = 0
}
```

2. 在对应的 Activity 或 Fragment 中创建对应的 viewModel 实例

```kotlin
// pattern: ViewModelProvider(<Activity/Fragment实例>).get(<ViewModel>::class.java)
viewModel = ViewModelProvider(this).get(MainViewModel::class.java)
```

3. 逻辑操作

```kotlin
val buttonPlus: Button = findViewById(R.id.plusOneBtn)
buttonPlus.setOnClickListener {
    viewModel.counter++
    refreshCounter()
}
```

## 向 ViewModel 传递参数

比如让 ViewModel 加载的时候, 可以先在 Activity 中尝试读取本地缓存的数据信息, 然后将读取的内容传递给 ViewModel 进行数据维护

1. 需要创建一个 ViewModel 的工厂类, 重写其 create 方法, 使其适配参数传递

```kotlin
class MainViewModelFactory(private val countReserved: Int): ViewModelProvider.Factory {

    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        return MainViewModel(countReserved) as T
    }

}
```

2. 使用工厂类实现 ViewModel 的实例化

```kotlin
viewModel = ViewModelProvider(this, MainViewModelFactory(countReserved)).get(MainViewModel::class.java)
```

# Lifecycles

Motivation: 对 Activity 生命周期情况进行感知, 某个界面中发起了一条网络请求，但是当请求得到响应的时候，界面或许已经关闭了，这个时候就不应该继续对响应的结果进行处理。因此，我们需要能够时刻感知到Activity的生命周期，以便在适当的时候进行相应的逻辑控制。

Lifecycles 可以让任何一个类都能轻松感知到 Activity 的生命周期，同时又不需要在 Activity 中编写大量的逻辑处理

## Quick Start

1. 新建一个自己的 MyObserver 类，并让它实现 LifecycleObserver 接口（主要就是声明一下）

```kotlin
class MyObserver: LifecycleObserver {}
```

2. 接口中根据相关注解定义感知 Activity 生命周期的逻辑处理方法

声明周期事件类型: `ON_CREATE`, `ON_START`, `ON_RESUME`, `ON_PAUSE`, `ON_STOP`, `ON_DESTROY`; 特殊的有一种 `ON_ANY` 表示可以匹配 Activity 的任何生命周期回调

```kotlin
class MyObserver : LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    fun activityStart() {
        Log.d("MyObserver", "activityStart")
    }
    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun activityStop() {
        Log.d("MyObserver", "activityStop")
    }
}
```

3. 获取 Activity 或 Fragment 本身的 LifecycleOwner 实例, 实现对 MyObserver 的通知

```kotlin
lifecycle.addObserver(MyObserver())
```

## 主动状态感知

在 Observer 中主动获知当前的生命周期状态

1. 构造 Observer 时需要将 Lifecycle 对象传递进来

```kotlin
class MyObserver(val lifecycle: Lifecycle) : LifecycleObserver {
    ...
}
```

2. 通过 lifecycler 对象获取当前 Activity 的状态

```kotlin
lifecycle.currentState
// 返回的生命周期状态是一个枚举类型: INITIALIZED, DESTROYED, CREATED, STARTED, RESUMED
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202209051928800.png)

# LiveData

Motivation: 对 ViewModel 进行写后读时, 在单线程模式没问题, 但是如果复杂写, 需要开启额外线程执行, 那么写后读, 读到的大概率还是旧数据

LiveData 是 Jetpack 提供的一种响应式编程组件，它可以包含任何类型的数据，并在数据发生变化的时候通知给观察者
- 常与 ViewModel 结合使用
- 在子线程中给 LiveData 设置数据，要调用 postValue() 方法，而不能再使用 setValue() 方法

## Quick Start

1. 使用相关的 LiveData 去维护相关变量

```kotlin
var counter = MutableLiveData<Int>()
```

2. 初始化变量, 并书写变量相关的处理接口

```kotlin
init {
    counter.value = countReserved
}

fun plusOne() {
    val count = counter.value ?: 0
    counter.value = count + 1
}

fun clear() {
    counter.value = 0
}
```

3. 在 Activity 中书写相关逻辑处理, 并监听回调

observe 方法接收两个参数：第一个参数是 LifecycleOwner 对象(Activity OR Fragment)；第二个参数是一个 Observer 接口，表示当 counter 中包含的数据发生变化时，就会回调到这里

```kotlin
val buttonPlus: Button = findViewById(R.id.plusOneBtn)
buttonPlus.setOnClickListener {
    viewModel.plusOne()
}

val buttonClean: Button = findViewById(R.id.clearBtn)
buttonClean.setOnClickListener {
    viewModel.clear()
}

viewModel.counter.observe(this) {
    count -> textInfo.text = count.toString()
}
```

## 更好的封装性

Quick Start 中将 counter 这个可变的 LiveData 暴露给了外部，这样即使是在 ViewModel 的外面也是可以给 counter 设置数据的，从而破坏了 ViewModel 数据的封装性，同时也可能带来一定的风险

因此永远只暴露不可变的 LiveData 给外部，这样在非 ViewModel 中就只能观察 LiveData 的数据变化，而不能给 LiveData 设置数据

```kotlin

class MainViewModel(countReserved: Int): ViewModel() {

//    var counter = MutableLiveData<Int>()

    // counter 变量, 类型为不可变的 LiveData, 重写 get 方法返回 _counter 变量
    // 当外部调用 counter 变量时，实际上获得的就是 _counter 的实例（转型为 LiveData 类型），但是无法给 counter 设置数据，从而保证了 ViewModel 的数据封装性
    val counter: LiveData<Int>
        get() = _counter

    // private 修饰, 对外部不可见
    private val _counter = MutableLiveData<Int>()
    init {
        _counter.value = countReserved
    }

    fun plusOne() {
        val count = _counter.value ?: 0
        _counter.value = count + 1
    }

    fun clear() {
        _counter.value = 0
    }
}
```

## map

将实际包含数据的 LiveData 和仅用于观察数据的 LiveData 进行转换

Application: ViewModel 保存了一个用户数据类的实例, 但是界面展示时只会涉及到用户的名字, 不会涉及到其年龄, 身高, 因此将整个用户数据实例暴露给外部读取也十分危险, 需要进一步封装转换, 使其只暴露姓名

```kotlin
private val userLiveData = MutableLiveData<User>()
val username: LiveData<String> = Transformations.map(userLiveData) {
    user -> "${user.firstName} ${user.lastName}"
}
```

## switchMap

前面我们所学的所有内容都有一个前提：LiveData 对象的实例都是在 ViewModel 中创建的

然而在实际的项目中，不可能一直是这种理想情况，很有可能 ViewModel 中的某个 LiveData 对象是调用另外的方法获取的

1. 通过外部单例类的方法获取一个 User 对象

```kotlin
fun getUser(userId: String): LiveData<User> {
    val liveData = MutableLiveData<User>()
    liveData.value = User(userId, userId, 0)
    return liveData
}
```

2. 因为是通过 userId 生成 liveData 每次返回的都是一个新的 liveData 实例, 导致 Activity 中无法观察到数据的变化; switchMap 就是应用在此场景, 如果 ViewModel 中的某个 LiveData 对象是调用另外的方法获取的，那么我们就可以借助 switchMap() 方法，将这个 LiveData 对象转换成另外一个可观察的 LiveData 对象

```kotlin
// 维护并用来观察 userId 的数据变化
private val userIdLiveData = MutableLiveData<String>()

val user: LiveData<User> = Transformations.switchMap(userIdLiveData) {
    // 转换函数中返回一个 LiveData 对象
    userId -> Repository.getUser(userId)
}

fun getUser(userId: String) {
    userIdLiveData.value = userId
}
```

整体流程:
1. 外部调用 MainViewModel 的 getUser() 方法来获取用户数据时，并不会发起任何请求或者函数调用，只会将传入的 userId 值设置到 userIdLiveData 当中
2. userIdLiveData 的数据发生变化，那么观察 userIdLiveData的switchMap() 方法就会执行，并且调用我们编写的转换函数。然后在转换函数中调用 Repository.getUser() 方法获取真正的用户数据
3. switchMap() 方法会将 Repository.getUser() 方法返回的 LiveData 对象转换成一个可观察的 LiveData 对象，对于 Activity 而言，只要去观察这个 LiveData 对象就可以了

Tips: 如果外部生成的 LiveData 实例没有相关属性可供监听, 那么只需要监听一个空属性即可

```kotlin
private val refreshLiveData = MutableLiveData<Any?>()
val refreshResult = Transformations.switchMap(refreshLiveData) {
    Repository.refresh() // 假设Repository中已经定义了refresh()方法
}
// LiveData 内部不会判断即将设置的数据和原有数据是否相同，只要调用了 setValue() 或 postValue() 方法，就一定会触发数据变化事件
fun refresh() {
    refreshLiveData.value = refreshLiveData.value
}
```

# ROOM

ROOM 是为 Android 数据库设计的 ORM 框架, 是对 SQLite 的封装; Room provides an abstraction layer over SQLite to allow fluent database access while harnessing the full power of SQLite
- Object Relational Mapping, 将面向对象的语言和面向关系的数据库之间建立一种映射关系

Overall Structure: Entity, Dao, Database
- Entity: 定义封装实际数据的实体类; 每个实体类在数据库中对应一张表，并且表中的列是根据实体类中的字段**自动生成**的
- Dao: Dao 是数据访问对象的意思，在这里对数据库的各项操作进行封装; 实际编程时，逻辑层不需要和底层数据库打交道，直接和 Dao 层进行交互
- Database: 用于定义数据库中的关键信息，包括数据库的版本号、包含哪些实体类以及提供 Dao 层的访问实例

## Strength

- 用面向对象的思维来和数据库进行交互
- Room 支持在编译时动态检查 SQL 语句语法
- Compile-time verification of SQL queries. each @Query and @Entity is checked at the compile time
- 

## Quick CRUD

常用操作: Create, Read, Update, Delete
常用注解: 
- @Entity(foreignKeys, indices, primaryKeys, tableName)
- @PrimaryKey(autoGenerate = true)
- @ColumnInfo(name="column_name"): allows specifying custom information about column
- @Ignore: field will not be persisted by Room
- @Embeded: nested fields can be referenced directly in the SQL queries

1. 引入相关依赖

- kotlin-kapt 插件: 引入 Room 编译时的注解库

```kotlin
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
apply plugin: 'kotlin-kapt'

dependencies {
    implementation "androidx.room:room-runtime:2.1.0"
    kapt "androidx.room:room-compiler:2.1.0"
}
```

2. 编写相关实体类, 使用 ROOM 的注解完成面向对象

```kotlin
// 定义实体类的注解
@Entity
data class User(var firstName: String, var lastName: String, var age: Int) {

    @PrimaryKey(autoGenerate = true)
    var id: Long = 0
}
```

3. 编写相关的数据库操作接口, DAO

```kotlin
@Dao
interface UserDao {

    // 表示会将参数中传入的 User 对象插入数据库中，插入完成后还会将自动生成的主键 id 值返回
    @Insert
    fun insertUser(user: User): Long

    // 会将参数中传入的 User 对象更新到数据库当中
    @Update
    fun updateUser(newUser: User)

    // 会将参数传入的 User 对象从数据库中删除
    @Delete
    fun deleteUser(user: User)

    // 查询或精细化的删除操作, 需要传入相关的 SQL 语言
    @Query("select * from User")
    fun findAllUsers(): List<User>

    @Query("select * from User where age > :age")
    fun findUsersOlderThan(age: Int): List<User>

    @Query("delete from User where lastName = :lastName")
    fun deleteUserByLastName(lastName: String): Int

}
```

4. 定义 Database: 数据库的版本号、包含哪些实体类、提供 Dao 层的访问实例

# WorkManager

WorkManager 是一个处理定时任务的工具，可以保证即使在应用退出甚至手机重启的情况下，之前注册的任务仍然将会得到执行
- 但是为了减少电量消耗，进行了一些优化，因此不能保证准时执行；比如将触发时间临近的几个任务放在一起执行，这样可以大幅度地减少 CPU 被唤醒的次数，从而有效延长电池的使用时间

## Quick Start

1. 添加相关依赖: `implementation "androidx.work:work-runtime:2.7.1"`
2. 定义一个后台任务，并实现具体的任务逻辑

```kotlin
// 不会运行在主线程
override fun doWork(): Result {
    Log.d("SimpleWorker", "do work in jcworker")
    return Result.success() // Result.failure() OR Result.retry()
}
```
3. 配置该后台任务的运行条件和约束信息，并构建后台任务请求

```kotlin
// Example ////////////////////
val request = OneTimeWorkRequest.Builder(SimpleWorker::class.java).build() // basic
val request = PeriodicWorkRequest.Builder(SimpleWorker::class.java, 15, TimeUnit.MINUTES).build() // Periodic
// Basic /////////////////////
//// 后台任务的具体运行时间是由我们所指定的约束以及系统自身的一些优化所决定
//// 由于这里没有指定任何约束，因此后台任务基本上会在点击按钮之后立刻运行
val request = OneTimeWorkRequest.Builder(JCWorker::class.java).build()
WorkManager.getInstance(this).enqueue(request)
```
4. 将该后台任务请求传入 WorkManager 的 enqueue() 方法中，系统会在合适的时间运行

```kotlin
WorkManager.getInstance(context).enqueue(request) // basic
```

## 延时任务

构建后台任务请求时借助 `setInitialalDelay()` 方法让后台任务延时执行
- 单位: TimeUnit.SECONDS, MINUTES, HOURS, DAYS, MICROSECONDS, MILLISECONDS, NANOSECONDS

```kotlin
// 推迟 5s 执行
val request = OneTimeWorkRequest.Builder(JCWorker::class.java)
    .setInitialDelay(5, TimeUnit.SECONDS)
    .build()
```

## 取消后台任务

1. 可以在后台任务构建时指定该任务的标签, 多个任务可以绑定相同标签, 从而进行统一控制

```kotlin
val request = OneTimeWorkRequest.Builder(JCWorker::class.java)
    .setInitialDelay(5, TimeUnit.SECONDS)
    .addTag("JCWork")
    .build()
```

2. 使用 WorkManager 的实例方法进行取消

```kotlin
WorkManager.getInstance(this).cancelAllWorkByTag("JCWork")
// 或者使用 id 取消
WorkManager.getInstance(this).cancelWorkById(request.id)
```

## 任务执行状态的监听

### Result.retry()

如果在复现的 `doWork()` 方法中返回了 `Result.retry()` 那么可以提前通过 `setBackoffCriteria()` 方法配置好任务的重启配置

Tips: 如果任务一直执行失败, 那么一直重试没有意义, 因此应让重试间隔随着重试次数的增加而增加
- BackoffPolicy.LINEAR: 代表下次重试时间以线性的方式延迟
- BackoffPolicy.EXPONENTIAL 代表下次重试时间以指数的方式延迟

```kotlin
// 第一个参数则用于指定如果任务再次执行失败，下次重试的时间应该以什么样的形式延迟
// 第二, 三个参数用于指定在多久之后重新执行任务，时间最短不能少于 10 秒钟
val request = OneTimeWorkRequest.Builder(JCWorker::class.java)
    .setInitialDelay(5, TimeUnit.SECONDS)
    .addTag("JCWork")
    .setBackoffCriteria(BackoffPolicy.LINEAR, 10, TimeUnit.SECONDS)
    .build()
```

### SUCCEEDED OR FAILED

成功或失败的执行状态需要通过对 WorkManager 的配置进行响应

```kotlin
// 返回一个 LiveData 对象, 用这个维护任务的状态
WorkManager.getInstance(this).getWorkInfoByIdLiveData(request.id)
    .observe(this) { workInfo ->
    if (workInfo.state == WorkInfo.State.SUCCEEDED) {
        Log.d("SimpleWorker", "Succeeded")
    } else if (workInfo.state == WorkInfo.State.FAILED) {
        Log.d("SimpleWorker", "FAILED")
    }
}
```

## 链式任务

定义了多个独立的任务且具有先后顺序的情况, 使用链式任务进行先后执行

Tips: 必须在前一个后台任务运行成功之后，下一个后台任务才会运行。也就是说，如果某个后台任务运行失败，或者被取消了，那么接下来的后台任务就都得不到运行了

```kotlin
val sync = ... // 同步数据任务
val compress = ... // 压缩数据任务
val upload = ... // 上传数据任务
WorkManager.getInstance(this)
    .beginWith(sync)
    .then(compress)
    .then(upload)
    .enqueue()
```

## 额外注意

前面所介绍的 WorkManager 的所有功能，在国产手机上都有可能得不到正确的运行

绝大多数的国产手机厂商在进行 Android 系统定制的时候会增加一个一键关闭的功能，允许用户一键杀死所有非白名单的应用程序。而被杀死的应用程序既无法接收广播，也无法运行 WorkManager 的后台任务

因此 WorkManager 可以用，但是千万别依赖它去实现什么核心功能，因为它在国产手机上可能会非常不稳定
