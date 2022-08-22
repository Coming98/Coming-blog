---
title: 08-Content Provider
mathjax: false
date: 2022-08-22 11:32:55
summary: 跨程序实现数据共享
categories: Android Dev.
tags:
  - Android
---
# ContentProvider

实现跨程序数据共享, 它提供了一套完整的机制，允许一个程序访问另一个程序中的数据，同时还能保证被访问数据的安全性

ContentProvider 可以选择只对哪一部分数据进行共享

# 运行时权限

防止安装应用时一次性授权全部所需权限导致的风险, 用户可以在软件的使用过程中对某一项权限申请进行授权

普通权限: 不会直接威胁到用户的安全和隐私的权限, 对于这部分权限申请，系统会自动帮我们进行授权，不需要用户手动操作
危险权限: 表示那些可能会触及用户隐私或者对设备安全性造成影响的权限，如获取设备联系人信息、定位设备的地理位置等，对于这部分权限申请，必须由用户手动授权才可以，否则程序就无法使用相应的功能

危险权限共 11 组 30 个权限:
- CALENDAR, 日历: READ_CALENDAR, WRITE_CALENDAR
- CALL_LOG, : READ_CALL_LOG, WRITE_CALL_LOG, PROCESS_OUTGOING_CALLS
- CAMERA, 相机: CAMERA
- CONTACTS, 联系人: READ_CONTACTS, WRITE_CONTACTS, GET_ACCOUNTS
- LOCATION, 位置: ACCESS_FINE_LOCATION, ACCESS_COARSE_LOCATION
- MICROPHONE, 麦克风: RECORD_AUDIO
- PHONE, 手机: READ_PHONE_STATE, CALL_PHONE, READ_CALL_LOG, WRITE_CALL_LOG, ADD_VOICEMAIL, USE_SIP, PROCESS_OUTGOING_CALLS
- SENSORS, 传感器: BODY_SENSORS
- ACTIVITY_RECOGNITION, : ACTIVITY_RECOGNITION
- SMS, 短信: SEND_SMS, RECEIVE_SMS, READ_SMI, RECEIVE_WAP_PUSH, RECEIVE_MMS
- STORAGE, 存储卡: READ_EXTERNAL_STORAGE, WRITE_EXTERNAL_STORAGE, ACCESS_MEDIA_LOCATION

## 运行时申请

1. 在 AndroidManifest 中声明想要获取的权限

```xml
<!--    仅仅是声明, 还需要运行时申请 -->
<uses-permission android:name="android.permission.CALL_PHONE" />
```

2. 在行为执行前进行权限检查 `ContextCompat.checkSelfPermission`, 如果没有相关权限则动态申请 `ActivityCompat.requestPermissions`

```kotlin
R.id.button_make_call -> {
    if (ContextCompat.checkSelfPermission(this, Manifest.permission.CALL_PHONE) != PackageManager.PERMISSION_GRANTED) {
        ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.CALL_PHONE), 1)
    } else {
        call("10086")
    }
}
```

3. 动态获得权限后执行相关行为

```kotlin
override fun onRequestPermissionsResult(
    requestCode: Int,
    permissions: Array<out String>,
    grantResults: IntArray
) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults)
    when(requestCode) {
        1 -> {
            if(grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                call("10086")
            } else {
                Toast.makeText(this, "Permission Denied", Toast.LENGTH_SHORT).show()
            }
        }
    }
}

private fun call(number: String) {
    try {
        // Intent.ACTION_DIAL 只是打开拨号界面, 不需要声明权限
        // Intent.ACTION_CALL 则是拨打电话
        val intnet = Intent(Intent.ACTION_CALL)
        intent.data = Uri.parse("tel:$number")
        startActivity(intent)
    } catch (e: SecurityException) {
        e.printStackTrace()
    }
}
```

# Native ContentProvider

如果想要访问 ContentProvider 中的数据需要借助 ContentResolver 类实现对数据的增删改查操作(insert, update, delete, query)

通过内容 URI 为 ContentProvider 中的数据建立唯一标识符, 由 authority 和 path 组成, 最终加上内容 URI 的协议前缀名 `content://` 即可; 例如 `content://com.example.app.provider/table1` 就可以很明确的表达想要访问哪个程序中哪张表里的数据
- authority: 用于对不同的应用程序做区分, 采用 `包名.provider` 防止冲突
- path: 用于对同一应用程序中不同的表做区分 采用 `authority/tablename` 的形式命名

```kotlin
private fun readContacts() {
    // Uri 封装好了
    contentResolver.query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI, null, null, null, null)?.apply {
        while (moveToNext()) {
            val name = getString(getColumnIndexOrThrow(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME))
            val number = getString(getColumnIndexOrThrow(ContactsContract.CommonDataKinds.Phone.NUMBER))
            phoneList.add(Phone(name, number))
        }
        adapter.notifyDataSetChanged()
        close()
    }
}
```

## insert

与 SQLite 类似

```kotlin
val values = contentValuesOf("column1" to "text", "column2" to 1)
contentResolver.insert(uri, values)
```

## update

```kotlin
val values = contentValuesOf("column1" to "")
contentResolver.update(uri, values, "column1 = ? and column2 = ?", arrayOf("text", "1"))
```

## query

用法与 SQLite 基本一致, 参数有所变化

```kotlin
val cursor = contentResolver.query(
    uri, // 指内容 URI 需要使用 Uri.parse(URI) 进行解析
    projection, // 列名
    selection, // where 约束
    selectionArgs, // 占位符值
    sortOrder // 排序方式
)
while (cursor.moveToNext()) {
    val column1 = cursor.getString(cursor.getColumnIndex("column1"))
    val column2 = cursor.getInt(cursor.getColumnIndex("column2"))
}
cursor.close()
```

## delete

```kotlin
contentResolver.delete(uri, "column2 = ?", arrayOf("1"))
```

# Customized ContentProvider

需要继承 ContentProvider 并重写其六个函数: `onCreate(), query(), insert(), update(), delete(), getType()`
- onCreate: 初始化 contentProvider, 完成对数据库的创建和升级操作
- getType: 根据传入的内容 URI 返回相应的 MIME 类型

## MIME 类型

内容 URI `content://com.example.app.provider/table1/1` 的 MIME 类型为 `vnd.android.cursor.item/vnd.com.example.app.provider.table1`
- 以 `vnd` 开头
- 如果 URI 指示的是路径则添加 `android.cursor.dir/`; 如果 URI 芝士的是 id 则添加 `android.cursor.item/`
- 以 `vnd.<authority>.<path>` 结尾

## 自定义 ContentProvider 类

1. 首先明确可能访问的表与表的条目有哪些, 作为私有属性列出: `bookDir, bookItem`, 并绑定 MIME 类型

```kotlin
class BookContentProvider : ContentProvider() {

    private val bookDir = 0
    private val bookItem = 1
    private val authority = "com.example.a13databasetest.provider"
    private var dbHelper: JCDatabaseHelper? = null

    override fun getType(uri: Uri) = when(uriMatcher.match(uri)) {
        bookDir -> "vnd.android.cursor.dir/vnd.$authority.book"
        bookItem -> "vnd.android.cursor.item/vnd.$authority.book"
        else -> null
    }

    ...
}
```

2. 然后使用 UriMatcher 为 URI 制作白名单

```kotlin
private val uriMatcher by lazy {
    val matcher = UriMatcher(UriMatcher.NO_MATCH)
    matcher.addURI(authority, "book", bookDir)
    matcher.addURI(authority, "book/#", bookItem)
    matcher
}
```

3. 声明周期中 onCreate 时完成 SQLite 的申请

```kotlin
override fun onCreate() = context?.let {
    dbHelper = JCDatabaseHelper(it, "BookStore.db", 2)
    true
} ?: false
```

4. 完成 query, insert, update, delete 处理逻辑

```kotlin
override fun query(
    uri: Uri, projection: Array<String>?, selection: String?,
    selectionArgs: Array<String>?, sortOrder: String?
) = dbHelper?.let {
    val db = it.readableDatabase
    val cursor = when(uriMatcher.match(uri)) {
        bookDir -> db.query("Book", projection, selection, selectionArgs, null, null, sortOrder)
        bookItem -> {
            val book_id = uri.pathSegments[1]
            db.query("Book", projection, "id = ?", arrayOf(book_id), null, null, sortOrder)
        }
        else -> null
    }
    cursor
}

override fun delete(uri: Uri, selection: String?, selectionArgs: Array<String>?) = dbHelper?.let {
    val db = it.writableDatabase
    val deletedRows = when(uriMatcher.match(uri)) {
        bookDir -> db.delete("Book", selection, selectionArgs)
        bookItem -> {
            val book_id = uri.pathSegments[1]
            db.delete("Book", "id = ?", arrayOf(book_id))
        }
        else -> 0
    }
    deletedRows
} ?: 0

override fun insert(uri: Uri, values: ContentValues?) = dbHelper?.let {
    val db = it.writableDatabase
    val uriReturn = when (uriMatcher.match(uri)) {
        bookDir, bookItem -> {
            val book_id = db.insert("Book", null, values)
            Uri.parse("content://$authority/book/$book_id")
        }
        else -> null
    }
    uriReturn
}

override fun update(
    uri: Uri, values: ContentValues?, selection: String?,
    selectionArgs: Array<String>?
) = dbHelper?.let {
    val db = it.writableDatabase
    val updatedRows = when(uriMatcher.match(uri)) {
        bookDir -> db.update("Book", values, selection, selectionArgs)
        bookItem -> {
            val book_id = uri.pathSegments[1]
            db.update("Book", values, "id = ?", arrayOf(book_id))
        }
        else -> 0
    }
    updatedRows
} ?: 0
```

5. 记得检查 AndroidManifest 中是否声明了 Provider

## 外部操作共享数据区域

指定好 URI 通过 contentResolver 进行操作即可

```kotlin
R.id.addData -> {
    val uri = Uri.parse("content://com.example.a13databasetest.provider/book")
    val values = contentValuesOf("name" to "The day", "author" to "ZH", "pages" to 999, "price" to "9.98")
    val newUri = contentResolver.insert(uri, values)
    book_id = newUri?.pathSegments?.get(1)
}
R.id.queryData -> {
    val uri = Uri.parse("content://com.example.a13databasetest.provider/book")
    contentResolver.query(uri, null, null, null, null)?.apply {
        while(moveToNext()) {
            val name = getString(getColumnIndexOrThrow("name"))
            val author = getString(getColumnIndexOrThrow("author"))
            val pages = getInt(getColumnIndexOrThrow("pages"))
            val price = getDouble(getColumnIndexOrThrow("price"))
        }
        close()
    }
}
R.id.updateData -> {
    book_id?.let {
        val uri = Uri.parse("content://com.example.a13databasetest.provider/book/$it")
        val values = contentValuesOf("name" to "A Storm of Swords",
            "pages" to 1216, "price" to 24.05)
        contentResolver.update(uri, values, null, null)
    }
}
R.id.deleteData -> {
    book_id?.let {
        val uri = Uri.parse("content://com.example.a13databasetest.provider/book/$it")
        contentResolver.delete(uri, null, null)
    }
}
```