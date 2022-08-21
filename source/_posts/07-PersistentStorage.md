---
title: 07-PersistentStorage
mathjax: false
date: 2022-08-21 09:56:09
summary: Android 数据持久化技术
categories: Android Dev.
tags:
  - Android
---

# Persistence 

Types: 文件存储, SharedPerferences 存储, 数据库存储

# 文件存储

不对存储的内容进行任何格式化处理，所有数据都是原封不动地保存到文件当中的，因而它比较适合存储一些简单的文本数据或二进制数据

## Save
主要通过 Context 类中的 openFileOutput 方法实现, 返回一个 FileOutputStream 对象
- openFileOutput 第一个参数为指定的文件名, 不可以包含路径, 因为所有的文件都默认存储到 `/data/data/<package name>/files/` 目录下
- openFileOutput 第二个参数是文件的操作模式, MODE_PRIVATE 表示覆盖写; MODE_APPEND 表示追加
- FileOutputStream 对象可以使用 Java 流的方式将数据写入文件中

```kotlin
private fun save(inputText: String) {
    try {
        val output = openFileOutput("jcData", Context.MODE_PRIVATE)
        val writer = BufferedWriter(OutputStreamWriter(output))
        writer.use {
            it.write(inputText)
        }
    } catch (e: IOException) {
        e.printStackTrace()
    }
}
```

## Load

通过 Context 类中提供的 openFileInput 方法, 指定文件名即可读取, 返回 FileInputStream 对象

```kotlin
private fun load(): String {
    val content = StringBuilder()
    try {
        val input = openFileInput("jcData")
        val reader = BufferedReader(InputStreamReader(input))
        reader.use {
            reader.forEachLine {
                content.append(it)
            }
        }
    } catch (e: IOException) {
        e.printStackTrace()
    }
    return content.toString()
}
```

# SharedPreferences

使用键值对的方式来存储数据, 支持多种不同的数据类型存储

## 获取 SharedPreferences 对象

`Context.getSharedPreferences(filename, MODE)`
- 第一个参数为文件名称, 存储路径固定为 `/data/data/<package ame>/shared_prefs/`
- 第二个参数为操作模式, 目前仅支持 MODE_PRIVATE, 表示只有当前的应用程序才可以对这个 SharedPreferences 文件进行读写

`Activity.getPreferences(MODE)`
- 自动将当前 Activity 的类名作为文件名

## Save

1. 获取 Editor 对象: `SharedPreferences.edit()`
2. 向 Editor 对象中添加数据: `putBoolean(), putString(), ...`
3. 调用 apply() 方法实现操作提交

```kotlin
val editor = getSharedPreferences("jcData", Context.MODE_PRIVATE).edit()
editor.putString("name", "Coming")
editor.putBoolean("married", false)
editor.putInt("age", 24)
editor.apply()
```

使用高级函数进行 API 优化

```kotlin
// 扩展函数添加 open 方法 接收函数类型为 SharedPreferences.Editor
fun SharedPreferences.open(block: SharedPreferences.Editor.() -> Unit) {
    // open 函数拥有 SharedPreferences 方法的上下文, 可以直接调用 edit 方法
    val editor = edit()
    editor.block()
    editor.apply()
}

getSharedPreferences("jcData", Context.MODE_PRIVATE).open {
    putString("name", "CJC")
    putBoolean("married", false)
    putInt("age", 24)
}
```

## Load

通过 SharedPreferences 对象的 getXX 方法获取目标值

```kotlin
val prefs = getSharedPreferences("jcData", Context.MODE_PRIVATE)
val name = prefs.getString("name", "")
val married = prefs.getBoolean("married", false)
val age = prefs.getInt("age", 0)
```

# SQLite

Android 系统内置的数据库, 轻量级的关系型数据库，运算速度快，占用资源少，通常只需要几百KB的内存就足够了

数据类型: integer 整型，real 浮点型，text 文本类型，blob 二进制类型

关键字: primary key, autoincrement

## SQLiteOpenHelper

是一个抽象类, 拥有两个抽象方法 `onCreate()` 与 `onUpgrade()` 用于实现创建和升级数据库的逻辑; 拥有两个实例方法 `getReadableDatabase()` 和 `getWritableDatabase()`
- 当数据库不可写入的时候（如磁盘空间已满），getReadableDatabase() 方法返回的对象将以只读的方式打开数据库，而 getWritableDatabase() 方法则将出现异常
- 构造方法参数列表
  - Context, 数据库名, null(允许我们在查询数据的时候返回一个自定义的 Cursor), 当前数据库的版本号(Int)
- 数据库的文件统一存放在 `/data/data/<package name>/databases/`

## 创建和升级数据库

主要完成数据库中表的创建, 后续可能根据业务情况额外增加表, 这时候需要利用升级功能实现增加

1. 建立并维护项目自身的 DatabaseHelper 类：版本变化时会执行 onUpgrade 从而实现热更新, 用 if 判断每一次升级改动, 使得低版本用户支持跨版本升级

```kotlin
class JCDatabaseHelper(val context: Context, name: String, version: Int): 
        SQLiteOpenHelper(context, name, null, version) {

    private val createBook = "create table Book (" +
            " id integer primary key autoincrement," +
            "author text," +
            "price real," +
            "pages integer," +
            "name text)"


    private val createCategory = "create table Category (\n" +
            "    id integer primary key autoincrement,\n" +
            "    category_name text,\n" +
            "    category_code integer\n" +
            ")"

    override fun onCreate(db: SQLiteDatabase) {
        db.execSQL(createBook)
        db.execSQL(createCategory)
        Toast.makeText(context, "Create succeeded", Toast.LENGTH_SHORT).show()
    }

    override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
        Toast.makeText(context, "Upgrade Success: $oldVersion -> $newVersion", Toast.LENGTH_SHORT).show()
        if (oldVersion <= 1) {
            db.execSQL(createCategory)
        }
        if (oldVersion <= 2) {
            db.execSQL("alter table Book add column category_id integer")
        }
    }
}
```

2. 在 Activity 中触发数据库的创建/更新, 生成 DatabaseHelper 的实例对象, 并获取数据库 `writableDatabase/readableDatabase`

```kotlin
class MainActivity : AppCompatActivity(), View.OnClickListener {

    lateinit var dbHelper: JCDatabaseHelper

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        dbHelper = JCDatabaseHelper(this, "BookStore.db", 2)

        val button_createdb = findViewById<Button>(R.id.button_createdb)
        button_createdb.setOnClickListener(this)
    }

    override fun onClick(v: View?) {
        when(v?.id) {
            R.id.button_createdb -> {
                dbHelper.writableDatabase
            }
        }
    }
}
```

## CRUD

SQLite 将 `Create 添加, Retrieve 查询, Update 更新, Delete 删除` 封装好了, 借助 SQLiteDatabase 对象即可轻松完成操作

## Create - insert

1. 通过按钮点击获取输入数据并用 `ContentValues()` 封装

```kotlin
R.id.button_add -> {

    val name = edit_name.checkBlack("书名不能为空") ?: return
    val author = edit_author.checkBlack("作者不能为空") ?: return
    val pages = (edit_pages.checkBlack("页数不能为空") ?: return).toInt()
    val price = (edit_price.checkBlack("") ?: "99999").toFloat()


    val db = dbHelper.writableDatabase
    val values = ContentValues().apply {
        put("name", name)
        put("author", author)
        put("pages", pages)
        put("price", price)
    }
    db.insert("Book", null, values)

    Toast.makeText(this, "Add Success!", Toast.LENGTH_SHORT).show()
    edit_name.text.clear()
    edit_author.text.clear()
    edit_pages.text.clear()
    edit_price.text.clear()
}
```

2. 判空可以使用扩展方法优雅实现

```kotlin
fun EditText.checkBlack(message: String): String? {
    val text = this.text.toString()
    if (text.isBlank()) {
        if(message.isNotEmpty()) {
            showError(message)
        }
        return null
    }
    return text
}

fun showError(message: String) {
    Toast.makeText(this, message, Toast.LENGTH_SHORT).show()
}
```

3. 数据库的相关操作现在推荐放到 DatabaseHelper 类中进行实现

```kotlin
fun addBook(name: String, author: String, pages: Int, price: Float): Boolean {
    val db = this.writableDatabase
    val values = ContentValues().apply {
        put("name", name)
        put("author", author)
        put("pages", pages)
        put("price", price)
    }
    return !db.insert("Book", null, values).equals(-1L)
}
```

## Retrieve - rawQuery

SQLite 对查询的封装较为详尽也复杂, 因此推荐使用原生 SQL 语法, 使用 rawQuery 执行

```kotlin
fun findAllBooks(): ArrayList<Book> {
    val booklist = ArrayList<Book>()

    val query = "select * from Book"
    val db = this.writableDatabase
    val cursor = db.rawQuery(query, null)

    if(cursor.moveToFirst()) {
        do {
            val id = cursor.getString(cursor.getColumnIndexOrThrow("id")).toInt()
            val name = cursor.getString(cursor.getColumnIndexOrThrow("name"))
            val author = cursor.getString(cursor.getColumnIndexOrThrow("author"))
            val pages = cursor.getInt(cursor.getColumnIndexOrThrow("pages")).toInt()
            val price = cursor.getDouble(cursor.getColumnIndexOrThrow("price")).toDouble()
            booklist.add(Book(id, name, author, pages, price))
        } while(cursor.moveToNext())
    }
    cursor.close()
    return booklist
}
```

rawQuery 的第二个参数是用于为查询语句 query 填充数据

```kotlin
fun findBookByID(book_id: Int): Book {

    val db = this.writableDatabase
    val query = "select * from Book where id=?"
    val cursor = db.rawQuery(query, arrayOf(book_id.toString()))
    cursor.moveToFirst()
    val name = cursor.getString(cursor.getColumnIndexOrThrow("name"))
    val author = cursor.getString(cursor.getColumnIndexOrThrow("author"))
    val pages = cursor.getInt(cursor.getColumnIndexOrThrow("pages")).toInt()
    val price = cursor.getDouble(cursor.getColumnIndexOrThrow("price")).toDouble()
    return Book(book_id, name, author, pages, price)
}
```

## Update - update/execSQL

可以使用封装好的 update 方法或直接使用 execSQL 方法执行原生的 SQL 语句

update 方法参数为: 表名, 要更新的键值封装, where 语句的约束信息, 占位符内容

```kotlin
fun updateBookById(book: Book) {
    val db = this.writableDatabase
    val values = ContentValues().apply {
        put("name", book.name)
        put("author", book.author)
        put("pages", book.pages)
        put("price", book.price)
    }

    db.update("Book", values, "id = ?", arrayOf(book.id.toString()))
}
```

### ContentValues 高阶封装

```kotlin
// 接收 Pair 参数; vararg 关键字表示接收的是可变参数列表, 即允许传入 0-n 个 Pair 参数
fun cvOf(vararg pairs: Pair<String, Any?>) = ContentValues().apply {
    for (pair in pairs) {
        val key = pair.first
        val value = pair.second
        when (value) {
            // Smart Cast 功能 如果进入了 Int 分支则会自动转型, 不必手动向下转型了
            is Int -> put(key, value)
            is Long -> put(key, value)
            is Short -> put(key, value)
            is Float -> put(key, value)
            is Double -> put(key, value)
            is Boolean -> put(key, value)
            is String -> put(key, value)
            is Byte -> put(key, value)
            is ByteArray -> put(key, value)
            null -> putNull(key)
        }
    }
}
```

在真正传值时能显著减少代码量

```kotlin
val values = cvOf("name" to book.name, "author" to book.author, "pages" to book.pages, "price" to book.price)
```

## Delete - delete/execSQL

与 update 基本一致, 连方法的使用也基本一致

```kotlin
fun deleteBookById(book_id: Int) {
    val db = this.writableDatabase
    db.delete("Book", "id = ?", arrayOf(book_id.toString()))
}
```

## 事务

```kotlin
db.beginTransaction() // 开启事务
try {
    // SQL 处理
    db.setTransactionSuccessful() // 事务执行完毕且成功
} catch(e: Exception) {
    e.printStackTrace()
} finally {
    db.endTransaction() // 结束事务
}
```