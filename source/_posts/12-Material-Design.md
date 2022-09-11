---
title: 12-Material Design
mathjax: false
date: 2022-09-05 18:35:34
summary: Material UI 基本使用
categories: Android Dev.
tags:
  - Android
---
# Material Design

一套全新的界面设计语言，包含了视觉、运动、互动效果等特性, 其设计理念和工具都封装到了 Material 库中

# Toolbar

Toolbar 不仅继承了 ActionBar 的所有功能, 而且拥有更强的灵活性

## ActionBar

ActionBar 的配置来源于主题文件: `res/values/themes.xml`
- 在这里实现父主题的选取以及各个主题色的配置, 使用 `.NoActionBar` 即为没有 ActionBar 的主题

```xml
<style name="Theme.23MaterialTest" parent="Theme.MaterialComponents.Light.NoActionBar">
    <!-- Primary brand color. -->
<!--        <item name="colorPrimary">@color/purple_500</item>-->
    <item name="colorPrimaryVariant">@color/purple_700</item>
    <item name="colorOnPrimary">@color/white</item>
    <!-- Secondary brand color. -->
    <item name="colorSecondary">@color/teal_200</item>
    <item name="colorSecondaryVariant">@color/teal_700</item>
    <item name="colorOnSecondary">@color/black</item>
    <!-- Status bar color. -->
    <item name="android:statusBarColor" tools:targetApi="l">?attr/colorPrimaryVariant</item>
    <!-- Customize your theme here. -->
    <item name="colorPrimary">@color/colorPrimary</item>
    <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
    <item name="colorAccent">@color/colorAccent</item>
</style>
```

## 主题色的分布

![](https://raw.githubusercontent.com/Coming98/pictures/main/202209011220381.png)

- colorAccent: 不只是用来指定一个按钮的颜色，而更多表达了一种强调的意思，比如一些控件的选中状态也会使用该颜色
- colorPrimary: 标题栏颜色
- colorPrimaryVariant: 状态栏颜色
- windowBackgroud: 
- navigationBarColor:

```xml
<COde>
<!--        标题文字-->
    <item name="android:textColorPrimary">@color/colorAccent</item>
<!--        背景色-->
    <item name="android:windowBackground">@color/teal_700</item>
<!--        底部导航栏-->
    <item name="android:navigationBarColor">@color/purple_200</item>
<!--        标题栏-->
    <item name="colorPrimary">@color/colorPrimary</item>
<!--        状态栏-->
    <item name="colorPrimaryVariant">@color/colorPrimaryDark</item>
</Code>
```

## Quick Start

1. 编写布局文件: 新增 app 命名空间, 使得 Material 能够兼容老系统

Tips: 主程序主题为浅色, 为了和主体颜色区分, 因此 Toolbar 使用深色 (Dark) 主题; 为了避免 Toolbar 中的菜单按钮也继承深色主题导致无法区分, 因此手动指定 Toolbar 中的菜单按钮为浅色 (Light) 主题

```xml
<!--使用 xmlns: app 指定新的命名空间, 增加兼容性-->
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="@color/colorPrimary"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
</FrameLayout>
```

2. MainActivity 中配置 toolbar 实例

```kotlin
toolbar = findViewById(R.id.toolbar)
setSupportActionBar(toolbar)
```

## 属性修改

- 标题栏文字: AndroidManifest 中 activity 中的 label 属性指定; 默认使用 AndroidManifest 中 Application 中的 label

# 滑动菜单

滑动菜单: 将一些菜单选项隐藏起来，而不是放置在主屏幕上，然后可以通过滑动的方式将菜单显示出来

## DrawerLayout

DrawerLayout 布局中允许放入两个直接子控件：
1. 主屏幕中显示的内容
2. 滑动菜单中显示的内容

```xml
<androidx.drawerlayout.widget.DrawerLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/drawerLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.appcompat.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="@color/colorPrimary"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

    </FrameLayout>
<!--        滑动菜单内容必须指定 layout_gravity, 指定是从左侧滑出菜单还是右侧-->
<!--    start 属性表示会根据系统语言进行判断，如果系统语言是从左往右的，比如英语、汉语，滑动菜单就在左边
        如果系统语言是从右往左的，比如阿拉伯语，滑动菜单就在右边 -->
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:background="#FFF"
        android:text="This is menu"
        android:textSize="30sp" />

</androidx.drawerlayout.widget.DrawerLayout>
```

为了更好的触发滑动菜单, 可以在标题栏中添加导航按钮

```kotlin
supportActionBar?.let {
    it.setDisplayHomeAsUpEnabled(true) // 本意是用作 Home 键, 也可以用作菜单导航键
    it.setHomeAsUpIndicator(R.drawable.ic_menu)
}
// 添加点击事件
override fun onOptionsItemSelected(item: MenuItem): Boolean {
    when (item.itemId) {
            android.R.id.home -> drawerLayout.openDrawer(GravityCompat.START)
    }
    return true
}
```

## NavigationView 美化滑动菜单

1. app/build.gradle 中引入相关依赖

```kotlin
implementation 'com.google.android.material:material:1.1.0'
implementation 'de.hdodenhof:circleimageview:3.0.1' // 图片圆化
```

2. 使用 Material 库需要替换默认的 ActionBar 即使用 NoActionBar 主题
3. menu: ret/menu/nav_menu, 在 NavigationView 中显示具体的菜单项: group 表示一个组，checkableBehavior 指定为 single 表示组中的所有菜单项只能单选

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <group android:checkableBehavior="single">
        <item
            android:id="@+id/navCall"
            android:icon="@drawable/nav_call"
            android:title="Call" />
        <item
            android:id="@+id/navFriends"
            android:icon="@drawable/nav_friends"
            android:title="Friends" />
        <item
            android:id="@+id/navLocation"
            android:icon="@drawable/nav_location"
            android:title="Location" />
        <item
            android:id="@+id/navMail"
            android:icon="@drawable/nav_mail"
            android:title="Mail" />
        <item
            android:id="@+id/navTask"
            android:icon="@drawable/nav_task"
            android:title="Tasks" />
    </group>
</menu>
```

4. headerLayout: res/layout/nav_header, 在 NavigationView 中显示头部布局: 显示用户的头像，用户名，邮箱
   
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="180dp"
    android:padding="10dp"
    android:background="@color/teal_200">
    <de.hdodenhof.circleimageview.CircleImageView
        android:id="@+id/iconImage"
        android:layout_width="70dp"
        android:layout_height="70dp"
        android:src="@drawable/nav_icon"
        android:layout_centerInParent="true" />
    <TextView
        android:id="@+id/mailText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:text="tonygreendev@gmail.com"
        android:textColor="#FFF"
        android:textSize="14sp" />
    <TextView
        android:id="@+id/userText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_above="@id/mailText"
        android:text="Tony Green"
        android:textColor="#FFF"
        android:textSize="14sp" />
</RelativeLayout>
```

5. 完善基本的交互

```kotlin
navView.setCheckedItem(R.id.navCall)
navView.setNavigationItemSelectedListener {
    drawerLayout.closeDrawers()
    true
}
```

# 悬浮维度

仿立体面的操作

## FloatingActionButton 悬浮按钮

1. 加入到布局中

Tips: 
- layout_gravity 属性与滑动菜单一致, 按照人们的习惯, 如果语言从左往右, 则 start 表示左, end 表示右
- elevation: 表示悬浮高度, 高度越高投影效果越明显

```xml
<com.google.android.material.floatingactionbutton.FloatingActionButton
    android:id="@+id/fab"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="bottom|end"
    android:layout_margin="16dp"
    android:src="@drawable/ic_done"
    android:elevation="8dp"/>

```

2. 添加点击事件

```kotlin
floatingActionButton.setOnClickListener{
    Toast.makeText(this, "FAB clicked", Toast.LENGTH_SHORT).show()
}
```

## Snackbar 交互性提示

在提示中加入了可交互式的按钮, 用户能与提示进行交互

- 参数需要传入一个 view 对象, 只要是当前界面布局的任意一个 View 都可以，因为 Snackbar 会使用这个 View 自动查找最外层的布局，用于展示提示信息
- setAction 设置动作, 进行交互

```kotlin
floatingActionButton.setOnClickListener{ view ->
    Snackbar.make(view, "Data Updated", Snackbar.LENGTH_SHORT)
        .setAction("Undo") {
            Toast.makeText(this, "Data backed", Toast.LENGTH_LONG).show()
        }
        .show()
}
```

通过对 View 进行扩展, 简化 Snackbar 的使用

```kotlin

```

## CoordinatorLayout 悬浮维度的布局

加强版的 FrameLayout, 可以监听其所有子控件的各种事件，并自动帮助我们做出最为合理的响应
- 例如 Snackbar 提示信息将悬浮按钮遮挡住了，那么 CoordinatorLayout 能监听到 Snackbar的弹出事件，会自动将内部的 FloatingActionButton 向上偏移，从而确保不会被 Snackbar 遮挡
- 但是 Snackbar 所传入的触发 View 必须要是 CoordinatorLayout 的子组件奥

```xml
<androidx.coordinatorlayout.widget.CoordinatorLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="@color/colorPrimary"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="16dp"
        android:src="@drawable/ic_done"
        android:elevation="8dp"/>
</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

# 卡片式布局

## MaterialCardView

基于 FrameLayout, 常用于 RecyclerView 的数据项布局

```xml
<com.google.android.material.card.MaterialCardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="5dp"
    app:cardCornerRadius="4dp"
    >

    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <ImageView
            android:id="@+id/fruitImage"
            android:layout_width="match_parent"
            android:layout_height="100dp"
            android:scaleType="centerCrop"/>

        <TextView
            android:id="@+id/fruitName"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center_horizontal"
            android:layout_margin="5dp"
            android:textSize="16sp"/>
    </LinearLayout>

</com.google.android.material.card.MaterialCardView>
```

使用合适的布局去布局多个卡片

```kotlin
val layoutManager = GridLayoutManager(this, 2)
val recyclerView: RecyclerView = findViewById(R.id.recyclerView)
recyclerView.layoutManager = layoutManager
val adapter = FruitAdapter(this, fruitList)
recyclerView.adapter = adapter
```

## AppBarLayout

解决父布局遮挡 Toolbar 问题

1. 将 Toolbar 嵌入到 AppBarLayout 中
2. 给 RecyclerView 指定一个布局行为 (如果存在父组件, 则布局行为要上移)

```xml
<androidx.recyclerview.widget.RecyclerView
    android:id="@+id/recyclerView"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layout_behavior="@string/appbar_scrolling_view_behavior"
    />
```

这个布局属性能够实现与 AppBar 的交互, 如 RecyclerLayout 滚动时, AppBarLayout 能接收到滚动事件并进行响应, 这时候可以通过对 Toolbar 添加 layout_scrollFlags 属性实现控制, 可以混合使用, 如 `scroll|enterAlways`
- scroll: 当 RecyclerView 向上滚动的时候，Toolbar 会跟着一起向上滚动并实现隐藏
- enterAlways: 表示当 RecyclerView 向下滚动的时候，Toolbar会跟着一起向下滚动并重新显示
- snap: ；表示当 Toolbar 还没有完全隐藏或显示的时候，会根据当前滚动的距离，自动选择是隐藏还是显示

```xml
<androidx.appcompat.widget.Toolbar
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    android:background="@color/colorPrimary"
    android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
    app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
    app:layout_scrollFlags="scroll|enterAlways|snap"
    />
```

# 下拉刷新 SwipeRefreshLayou

1. 把想要实现下拉刷新功能的控件放置到 SwipeRefreshLayout 中，就可以迅速让这个控件支持下拉刷新

```xml
<androidx.swiperefreshlayout.widget.SwipeRefreshLayout
    android:id="@+id/swipeRefresh"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layout_behavior="@string/appbar_scrolling_view_behavior"
    >

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        />
</androidx.swiperefreshlayout.widget.SwipeRefreshLayout>
```

2. 刷新事件的逻辑处理

```kotlin
swipeRefresh = findViewById(R.id.swipeRefresh)
// 设置下拉进度条颜色
swipeRefresh.setColorSchemeResources(R.color.colorPrimary)
swipeRefresh.setOnRefreshListener {
    refreshFruits(adapter)
}
```

# 可折叠式标题栏

CollapsingToolbarLayout 是基于 Toolbar 的布局, 能够让 Toolbar 的效果变得更加丰富, 但是使用其的基础很高, 需要作为 AppBarLayout 的子布局(解决标题栏遮挡问题), 而 AppBarLayout 又是 CoordinateLayout 的子布局(监听子组件实现处理)

## QuickStart

1. 布局: CoordinateLayout + AppBarLayout + CollapsingToolbarLayout + Toolbar 实现标题栏布局

```xml
<androidx.coordinatorlayout.widget.CoordinatorLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.google.android.material.appbar.AppBarLayout
        android:id="@+id/appBar"
        android:layout_width="match_parent"
        android:layout_height="250dp">
<!-- app:contentScrim 属性用于指定 CollapsingToolbarLayout 在趋于折叠状态以及折叠之后的
背景色，其折叠后就是一个普通的 Toolbar-->

<!-- scroll 表示 CollapsingToolbarLayout 会随着水果内容详情的滚动一起滚动
    exitUntilCollapsed 表示当其滚动完成折叠之后就保留在界面上，不再移出屏幕-->
        <com.google.android.material.appbar.CollapsingToolbarLayout
            android:id="@+id/collapsingToolbar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:contentScrim="@color/colorPrimary"
            android:fitsSystemWindows="true"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

<!-- 由普通的标题栏加上图片组合而成的 -->
<!-- app:layout_collapseMode 用于指定当前控件在 CollapsingToolbarLayout 折叠过程中的折叠模式，其中 Toolbar 指定成 pin，表示在折叠的过程中位置始终保持不变，ImageView 指定成 parallax，表示会在折叠的过程中产生一定的错位偏移，这种模式的视觉效果会非常好-->
            <ImageView
                android:id="@+id/fruitImageView"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                app:layout_collapseMode="parallax" />
            <androidx.appcompat.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin" />

        </com.google.android.material.appbar.CollapsingToolbarLayout>
    </com.google.android.material.appbar.AppBarLayout>
    ...
</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

## 配置浮动按钮

可在合适的位置进行配置，并且会随着折叠消失
- 其相对位置通过 layout_anchor 决定

```xml
<com.google.android.material.floatingactionbutton.FloatingActionButton
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_margin="16dp"
    android:src="@drawable/ic_comment"
    app:layout_anchor="@id/appBar"
    app:layout_anchorGravity="bottom|end" />
```