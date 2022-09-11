---
title: 01-Android Basic
mathjax: false
date: 2022-07-14 10:46:17
summary: Android 开发基础知识
categories: Android Dev.
tags:
  - Android
---

# Android 系统架构

四层架构：
- Linux 内核层：为 Android 设备的各种硬件提供了底层的驱动，如驱动、电源管理等
- 硬件抽象层: Hardware Abstraction Layer, HAL, 提供标准界面，向更高级别的 Java API 框架显示设备硬件功能
  - HAL 包含多个库模块，其中每个模块都为特定类型的硬件组件实现一个界面，例如相机或蓝牙模块
  - 当框架 API 要求访问设备硬件时，Android 系统将为该硬件组件加载库模块
- Android Runtime
  - Android 5.0（API 级别 21）或更高版本的设备，每个应用都在其自己的进程中运行，并且有其自己的 Android Runtime (ART) 实例
- Native Layer: C/C++ Libraries, 
  - 许多核心 Android 系统组件和服务（例如 ART 和 HAL）构建自原生代码，需要以 C 和 C++ 编写的原生库
  - Android 平台提供 Java 框架 API 以向应用显示其中部分原生库的功能
- Java layer:
  - 通过以 Java 语言编写的 API 使用 Android OS 的整个功能集
- 应用层: 手机上的应用程序

![](https://raw.githubusercontent.com/Coming98/pictures/main/202208050902709.png)

# Android 系统四大组件

- Activity: 程序的界面
- Service: 后台服务, 逻辑处理
- BroadcastReceiver: 接收/发出广播消息
- ContentProvider: 应用程序间数据共享

# SQLite 数据库

自带轻量级、运算速度极快的嵌入式关系型数据库：
- 支持标准的 SQL 语法
- 通过 Android 封装好的 API 进行操作，让存储和读取数据变得非常方便

# 开发环境

- JDK: Java 语言的软件开发包, 包含了 Java 的运行环境、工具集合、基础类库等
- Android SDK: Google 提供的 Android 开发工具包，在开发 Android 程序时，我们需要通过引入该工具包来使用 Android 相关的 API
- Android Studio: [Android 程序开发](https://developer.android.google.cn/studio)

## 注意事项

- Android 系统就是通过包名来区分不同应用程序
- Android 程序的涉及讲究逻辑和视图分离

# 项目结构

- `app`: 项目中的代码、资源等内容都放置在这个目录下
- `.gradle & .idea`：存放自动生成的文件，不必关心
- `build`: 编译时自动生成的文件，不必关心
- `gradle`: 包含了 gradle wrapper 的配置文件
- `.gitignore`: 将指定的目录或文件排除在版本控制之外
- `build.gradle`: 全局的 gradle 构建脚本，通常不需要修改
- `gradle.properties`: 全局的 gradle 配置文件，在这里配置的属性会影响到项目中所有的 gradle 编译脚本
- `gradlew & gradlew.bat`: 在命令行界面中执行 gradle 命令
- `*.iml`: Android Studio 基于 IntelliJ IDEA 开发，每个项目都会自动生成一些 iml 文件
- `local.properties`: 指定本机中 Android SDK 路径
- `settings.gradle`: 指定项目中所有引入的模块（app模块）自动完成引入
- 
## app

- `java`: Java 代码
- `build`: 与外层 build 类似，包含了一些在编译时自动生成的文件
- `libs`: 第三方 jar 包
- `androidTest`: 编写 AndroidTest 测试用例，自动化测试
- `res`: 项目中使用到的所有图片、布局、字符串等资源
- `AndroidManifest.xml`: Android 项目配置文件，其余组件需要在此进行注册/添加权限声明
- `test`: 编写 Unit Test 测试用例，自动化测试的另一种方法
- `build.gradle`: app 模块的 gradle 构建脚本，指定项目构建相关配置
- `proguard-rules.pro`: 指定项目代码的混淆规则, 保护代码

## AndroidManifest.xml

规则:
- 没有在 AndroidManifest.xml 里注册的 Activity 是不能使用的

对 Activity 的声明 `<activity/>`: 
- 声明项目的主 Activity `<intent-filter/>
- 指定活动的标题栏内容 `android:label`

```xml
<activity android:name=".MainActivity"
          android:label="This is FirstActivity">  // 指定标题栏的内容
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

## Res

- `drawable-*`：存放图片
* `mipmap-*`：存放图标
* `values-*`：存放字符串、样式、颜色等配置
* `layout-*`：存放布局文件

### strings.xml

在 `res/values/strings.xml` 中对静态字符串进行定义
- 在代码中通过 `R.string.app_name` 引用
- 在 XML 中通过 `@string/app_name` 进行引用

Tips: drawable 与 mipmap 同理, `R.drawable.app_name`

## build.gradle

Gradle 是项目构建工具，基于 Groovy 的领域特定语言（DSL）来声明项目设置，摒弃了传统基于 XML（如Ant和Maven）的各种繁琐配置

### 外层 build.gradle

- jcenter：代码托管仓库，Android 开源项目都会将代码托管到 jcenter 上。声明后可以在项目中轻松引用任何 jcenter 上的开源项目
- gradle 插件：用于构建 Android 项目

### 内部 build.gradle

- 应用程序模块：com.android.applicatioin、可以直接运行
- 库模块：com.android.library、作为代码库依附于别的应用程序模块来运行
- compileSdkVersion：项目的编译版本
- buildToolsVersion：指定项目构建工具的版本
- applicationId：指定项目的包名
- minSdkVersion：指定项目最低兼容的 Android 系统版本
- targetSdkVersion：目标上线版本
- versionCode：指定项目的版本号
- versionName：指定项目的版本名
- buildType：指定生成安装文件的相关配置
- debug：指定生成测试版安装文件的配置
- release：指定生成正式版安装文件的配置
- minifyEnabled：指定是否对项目进行混淆
- proguardFiles：指定混淆时使用的规则
- proguard-android.txt：在Android SDK目录下，包含所有项目通用的混淆规则
- proguard-rules.pro：当前项目的混淆规则
- dependencies：指定当前项目所有的依赖关系
  * 本地依赖：对本地jar包或目录添加依赖关系
  * 库依赖：对项目中的库模块添加依赖关系
  * 远程依赖：对jcenter库上的开源项目添加依赖关系

# 日志工具

- `Log.v()`: verbose, 最为琐碎的、意义最小的日志信息
- `Log.d()`: debug, 调试信息
- `Log.o()`: info, 比较重要的数据，这些数据应该是你非常想看到的、可以帮你分析用户行为的数据
- `Log.w()`: warn, 警告信息, 提示程序在这个地方可能会有潜在的风险，最好去修复一下这些出现警告的地方
- `Log.e()`: error, 错误信息

参数一般两个: tag + msg
- tag: 一般传入当前的类名就好，主要用于对打印信息进行过滤
- msg: 具体内容

## 定制自己的日志工具

Targets
- 当程序处于开发阶段时就让日志打印出来，当程序上线之后就把日志屏蔽掉

```kotlin
object LogUtil {
    private const val VERBOSE = 1
    private const val DEBUG = 2
    private const val INFO = 3
    private const val WARN = 4
    private const val ERROR = 5
    // 开发阶段将 level 手动指定成 VERBOSE
    // 项目正式上线的时候将 level 指定成 ERROR 就可以了
    private var level = VERBOSE

    fun v(tag: String, msg: String) {
        if (level <= VERBOSE) {
            Log.v(tag, msg)
        }
    }
    fun d(tag: String, msg: String) {
        if (level <= DEBUG) {
            Log.d(tag, msg)
        }
    }
    fun i(tag: String, msg: String) {
        if (level <= INFO) {
            Log.i(tag, msg)
        }
    }
    fun w(tag: String, msg: String) {
        if (level <= WARN) {
            Log.w(tag, msg)
        }
    }
    fun e(tag: String, msg: String) {
        if (level <= ERROR) {
            Log.e(tag, msg)
        }
    }
}
```