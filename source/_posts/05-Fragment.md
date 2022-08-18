---
title: 05-Fragment
mathjax: false
date: 2022-08-18 21:01:01
summary: 碎片化 Activity, Fragment
categories: Android Dev.
tags:
  - Android
---

# Fragment 

一种可以嵌入在 Activity 当中的 UI 片段，能让程序更加合理和充分地利用大屏幕的空间，因而在平板上应用得非常广泛
- 与 Activity 类似, 能包含布局, 拥有自己的声明周期
- 但 Activity 中可以包含多个 Fragment 实现分页效果

![](https://raw.githubusercontent.com/Coming98/pictures/main/202208072008439.png)

## Quick Start

1. 为 Fragment 设置好布局

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <Button
        android:id="@+id/left_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:text="Button"
        />
</LinearLayout>
```

2. 新建 Fragment 类, 使其继承 `androidx.fragment.app.Fragment` 实现对 Fragment 布局的加载

```kotlin
class LeftFragment: Fragment() {

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // 通过 LayoutInflater 的 inflate() 方法将刚才定义的 left_fragment 布局动态加载进来
        return inflater.inflate(R.layout.left_fragment, container, false)
    }

}
```

3. 在主 Activity 中的布局中引入 Fragment

```kotlin
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <fragment
        android:id="@+id/leftFrag"
        android:name="com.example.a07fragment_1.LeftFragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1" />

    <fragment
        android:id="@+id/rightFrag"
        android:name="com.example.a07fragment_1.RightFragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1" />


</LinearLayout>
```

## 动态添加 Fragment

1. 在主 Activity 中为动态替换的 Fragment 留有布局

```xml
<FrameLayout
    android:layout_width="0dp"
    android:layout_height="match_parent"
    android:layout_weight="1"
    android:id="@+id/rightLayout"
    />
```

2. 通过点击或其他事件触发 fragment 的替换加载

```kotlin
// 将待添加 Fragment 的实例作为参数传入
private fun replaceFragment(fragment: Fragment) {
    // 通过 getSupportFragmentManager() 获取 fragment 操作对象
    val fragmentManager = supportFragmentManager
    // 开启一个事务，通过调用 beginTransaction() 方法开启
    val transaction = fragmentManager.beginTransaction()
    // 使用 replace()方法向容器内替换 Fragment，需要传入容器的id和待添加的 Fragment 实例
    transaction.replace(R.id.rightLayout, fragment)
    // 提交事务
    transaction.commit()
}
```

## 与主 Activity 的交互

类似于父组件管理多个子组件, 主要通过 `supportFragmentManager` 实现

- 获取 fragment View 实例: `supportFragmentManager.findFragmentById(R.id.leftFrag) as LeftFragment`
- 获取主 Activity 实例: `val mainActivity = getActivity() as MainActivity`


# 生命周期

与 Activity 类似, 拥有四种状态: 运行状态, 暂停状态, 停止状态, 销毁状态
- 运行状态: 当一个 Fragment 所关联的 Activity 正处于运行状态时，该 Fragment 也处于运行状态
- 暂停状态: 同上
- 停止状态: 同上 或 通过 FragmentTransaction 的 remove() replace() 方法将 Fragment 从 Activity 中移除并且调用了 addToBackStack() 也会进入停止状态
- 销毁状态: 当 Activity 被销毁时，与它相关联的 Fragment 就会进入销毁状态 或者 同上但没有调用 addToBackStack() 将会进入销毁状态

特有的状态回调方法:
- `onAttach()`: Fragment 与 Activity 建立关联
- `onCreateView()`: 为 Fragment 创建视图(加载布局) 时调用
- `onActivityCreated()`: 确保与 Fragment 相关联的 Activity 已经创建完毕时调用
- `onDestroyView()`: 当与 Fragment 关联的视图被移除时
- `onDetach()`: 当 Fragment 和 Activity 解除关联时调用

![](https://raw.githubusercontent.com/Coming98/pictures/main/202208092152914.png)

# 动态布局加载技巧

旨在能更灵活的根据分辨率, 屏幕大小等在运行时决定加载哪个布局

## 限定符 qualifier

准备好单页模式的布局 `layout/activity_main.xml` 与双页模式的布局 `layout-large/activity_main.xml` 这里面的 large 就是一个内置的限定符

实际过程中你可以按照双页模式进行布局的绑定初始化, 但是两种布局会根据设备的大小进行展示

因此如果想要程序有更多的适配, 我们就要为各类限定符作好布局, 常见的限定符有

大小类限定符
- small: 小屏幕
- normal: 中等屏幕
- large: 大屏幕
- xlarge: 超大平步
- smallest-width qualifier: 最小宽度限定符, 允许对屏幕的宽度指定一个最小值(单位 dp) 屏幕宽度大于这个值的设备就加载这个布局, 小于这个值的就加载另一个布局, 具体后缀可缩写为 `-sw600dp`

分辨率类限定符:
- ldpi: 120 dpi 以下
- mdpi: 120 - 160 dpi
- hdpi: 160 - 240 dpi
- xhdpi: 240 - 320 dpi
- xxhdpi: 320 - 480 dpi

方向:
- land: 横屏
- port: 竖屏

# Project

编写兼容手机和平板的应用程序

1. MainActivity 中根据 layout 进行动态布局加载, 选择性加载单页布局 `newsTitleFrag` 和双页布局 `newsContentFrag`
2. 针对单页模式设置其 Fragment `NewsTitleFragment`: 在其中加载标题列表所对应的布局 `news_title_frag`; 在生命周期 `onViewCreated` 中加载 `RecyclerView` 的数据 `NewsAdapter`; `NewsAdapter` 中通过 `ViewHolder` 实例为每个数据项添加点击处理事件 `holder.itemView.setOnClickListener`, 点击处理事件中通过判断当前 Activity 中是否是双页布局来选择性定义点击处理方式 `isTwoPane = (activity?.findViewById<View>(R.id.newsContentLayout)) != null`:
- 如果是双页布局, 则定位到目标 fragment 对象后刷新内容即可
- 如果是单页布局, 则需要启动一个新的 Activity 用于展示目标新闻的详细信息

```kotlin
holder.itemView.setOnClickListener {
    val news = newsList[holder.adapterPosition]
    if (isTwoPane) {
        val fragment = activity?.supportFragmentManager?.findFragmentById(R.id.newsContentFrag) as NewsContentFragment
        fragment.refresh(news.title, news.contennt)
    } else {
        NewsContentActivity.actionStart(parent.context, news.title, news.contennt)
    }
}
```

3. 针对双页模式设置其 Fragment `NewsContentFragment`: 在其中加载新闻内容所对应的布局, 并定义布局内容刷新的接口函数 `refresh` 供 MainActivity 调用

```kotlin
fun refresh(title: String, content: String) {
    contentLayout.visibility = View.VISIBLE
    newsTitle.text = title
    newsContent.text = content
}
```

1. 针对单页模式的点击事件, 定义新的 Activity 展示新闻内容 `NewsContentActivity` 通过 `companion object` 定义静态方法 `actionStart` 以向外部提供快速启动该 Activity 的接口; 启动后则根据传递来的数据渲染界面即可 

```kotlin
fun actionStart(context: Context, title: String, content: String) {
    val intent = Intent(context, NewsContentActivity::class.java).apply {
        putExtra("news_title", title)
        putExtra("news_content", content)
    }
    context.startActivity(intent)
}

val title = intent.getStringExtra("news_title")
val content = intent.getStringExtra("news_content")
if (title != null && content != null) {
    val fragment = supportFragmentManager.findFragmentById(R.id.newsContentFrag) as NewsContentFragment
    fragment.refresh(title, content)
}
```