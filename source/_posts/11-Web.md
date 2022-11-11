---
title: 11-Web
mathjax: false
date: 2022-08-30 14:03:35
summary: Android 网络请求基础知识
categories: Android Dev.
tags:
  - Android
---

# WebView

WebView 空间可以实现应用程序内浏览器的嵌入以及网页的展示


## Quick Start

1. Layout

```xml
<WebView
    android:id="@+id/webView"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

2. loadUrl: 其中 WebViewClient() 的设置是希望当从一个网页跳转到另一个网页时，希望目标网页仍然在当前 WebView 中显示，而不是打开系统浏览器

```kotlin
R.id.buttonLoadBaidu -> {
    webView.settings.javaScriptEnabled = true
    webView.webViewClient = WebViewClient()
    webView.loadUrl("https://www.baidu.com")
}
```

3. Permission

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

# HTTP in Android

## Quick Start

1. 发送 HTTP 请求: 创建一个子线程完成请求的发送与响应的获取

```kotlin
private fun sendRequestWithHttpURLConnection() {
    thread {
        var connection: HttpURLConnection? = null
        try {
            val response = StringBuilder()
            val url = URL("https://www.baidu.com")
            connection = url.openConnection() as HttpURLConnection

            connection.requestMethod = "GET"
            connection.connectTimeout = 8000 //ms
            connection.readTimeout = 8000

            // 响应获取
            val input = connection.inputStream
            val reader = BufferedReader(InputStreamReader(input))
            reader.use {
                reader.forEachLine {
                    response.append(it)
                }
            }
            showResponse(response.toString())
        } catch (e: Exception) {
            e.printStackTrace()
        } finally {
            connection?.disconnect()
        }
    }

    // 子线程与 UI 交互的封装
    private fun showResponse(response: String) {
        runOnUiThread {
            // UI 操作线程
            responseText.setText(response)
        }
    }
}
```

## Post Data

将 connection 对象的请求方法设置为 POST 后即可通过 outputStream 进行数据的发送

```kotlin
connection.requestMethod = "POST" // 可以提交数据
val output = DataOutputStream(connection.outputStream)
output.writeBytes("username=admin&password=123456")
```

## OkHTTP

超越原生 HttpURLConnection 的外部开源库

### Quick Start

1. `app/build.gradle` 中添加依赖

```kotlin
implementation 'com.squareup.okhttp3:okhttp:4.1.0'
```

2. client + request + response + responseData

```kotlin
val client = OkHttpClient()
val request = Request.Builder()
    .url("https://www.baidu.com")
    .build()
val response = client.newCall(request).execute()
val responseData = response.body?.string()
if (responseData != null) {
    showResponse(responseData)
}
```

### Post data

```kotlin
val requestBody = FormBody.Builder()
    .add("username", "admin")
    .add("password", "123456")
    .build()
val requestPost = Request.Builder()
    .url("https://www.baidu.com")
    .post(requestBody)kotlin
    .build()
```

### HTTP

HTTP 明文传输默认不支持, 需要在 `res/xml` 中手动写一个配置文件 `network_config.xml` 并应用

```xml
<network-security-config>
    <base-config cleartextTrafficPermitted="true">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

在 AndroidManifest 中应用

```xml
    <application
        ...
        android:networkSecurityConfig="@xml/network_config"
        />
```

# XML Parser

在网络上传输数据时最常用的格式有两种：XML 和 JSON, 先来了解下 Android 关于 XML 解析的相关知识


## Pull 解析

首先时请求获取 XML 相关数据

```kotlin
private fun parseXMLTest() {
    thread {
        try {
            val client = OkHttpClient()
            val request = Request.Builder()
                .url("http://192.168.0.107/DVWA/get_data.xml")
                .build()
            val response = client.newCall(request).execute()
            val responseData = response.body?.string()
            if (responseData != null) {
                val xmlData = parseXMLWithPull(responseData)
                showResponse(xmlData)
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
}
```

然后对 XML 数据进行解析

```kotlin
private fun parseXMLWithPull(xmlData: String): String {
    val retXmlData = StringBuilder()
    try {
        // 创建工厂类实例, 获取解析对象: xmlPullParse
        val factory = XmlPullParserFactory.newInstance()
        val xmlPullParser = factory.newPullParser()
        // 设置解析内容:
        xmlPullParser.setInput(StringReader(xmlData))
        // 获得初始的解析状态
        var eventType = xmlPullParser.eventType
        var id = ""
        var name = ""
        var version = ""
        // 解析循环判停条件
        while (eventType != XmlPullParser.END_DOCUMENT) {
            // getName 获取当前解析节点的名称
            val nodeName = xmlPullParser.name
            when (eventType) {
                // 根据开始节点的名称进行处理
                XmlPullParser.START_TAG -> {
                    // 简单的根据开始节点名称获取节点中的值
                    when (nodeName) {
                        "id" -> id = xmlPullParser.nextText()
                        "name" -> name = xmlPullParser.nextText()
                        "version" -> version = xmlPullParser.nextText()
                    }
                }
                // 根据结束节点的名称进行处理
                XmlPullParser.END_TAG -> {
                    // 结束节点为 app 时表示一个 app 的值获取完毕, 进行输出封装
                    if ("app" == nodeName) {
                        retXmlData.append("id is $id + name is $name + version is $version\n")
                    }
                }
            }
            // 获取下一个解析事件类型
            eventType = xmlPullParser.next()
        }
    } catch (e: Exception) {
        e.printStackTrace()
    }
    return retXmlData.toString()
}
```

## SAX 解析方式

用法比 Pull 更复杂, 但是语义更清楚

1. 需要继承 DefaultHandler 类并实现相关处理方法

```kotlin
class XMLContentHandler: DefaultHandler() {

    private var nodeName = ""
    private var retXMLData = StringBuilder()
    private lateinit var id: StringBuilder
    private lateinit var name: StringBuilder
    private lateinit var version: StringBuilder

    // 开始解析 XML 前调用
    override fun startDocument() {
        id = StringBuilder()
        name = StringBuilder()
        version = StringBuilder()
    }

    // 针对一个节点的解析
    override fun startElement(
        uri: String?,
        localName: String?, // 节点名称
        qName: String?,
        attributes: Attributes?
    ) {
        if (localName == null) return
        nodeName = localName
        retXMLData.apply {
            append("uri is $uri\n")
            append("localname is $localName\n")
            append("qName is $qName\n")
            append("attributes is $attributes\n")
        }
    }

    // 针对节点中内容的解析
    override fun characters(ch: CharArray?, start: Int, length: Int) {
        // 通过 startElement 中记录的节点名称, 将内容添加到相应容器中
        when (nodeName) {
            "id" -> id.append(ch, start, length)
            "name" -> name.append(ch, start, length)
            "version" -> version.append(ch, start, length)
        }
    }

    // 完成某个节点解析时调用
    override fun endElement(uri: String?, localName: String?, qName: String?) {
        if ("app" == localName) {
            retXMLData.append("id is ${id.toString().trim()} @ name is ${name.toString().trim()} @ version is ${version.toString().trim()}\n")
            id.setLength(0)
            name.setLength(0)
            version.setLength(0)
        }
    }

    // 完成整个 xml 解析时调用
    override fun endDocument() {
        super.endDocument()
    }

    fun getRetXMLData(): String {
        return this.retXMLData.toString()
    }

}
```

2. 在 SAXParser 中应用 contentHandler 进行 parse

```kotlin
private fun parseXMLWithSAX(xmlData: String): String {
    var retXMLData = ""
    try {
        val factory = SAXParserFactory.newInstance()
        val xmlReader = factory.newSAXParser().xmlReader
        val handler = XMLContentHandler()
        xmlReader.contentHandler = handler
        xmlReader.parse(InputSource(StringReader(xmlData)))
        retXMLData = handler.getRetXMLData()
    } catch (e: Exception) {
        e.printStackTrace()
    }
    return retXMLData
}
```

# JSON Parser

JSON 体积更小，在网络上传输的时候更省流量，但语义性较差，不如 XML 直观

## JSONObject 解析

解析的方式十分简单, 将 JSON 字符串转为 JSONArray 对象, 遍历按类型解析即可

```kotlin
val retJsonData = StringBuilder()
try {
    val jsonArray = JSONArray(jsonData)
    for (i in 0 until jsonArray.length()) {
        val jsonObject = jsonArray.getJSONObject(i)
        val id = jsonObject.getString("id")
        val name = jsonObject.getString("name")
        val version = jsonObject.getString("version")
        retJsonData.append("id is $id | name is $name | version is $version\n")
    }
} catch (e: Exception) {
    e.printStackTrace()
}
return retJsonData.toString()
```

## GSON 解析

1. `app\build.gradle` 中添加依赖

```kotlin
implementation 'com.google.code.gson:gson:2.8.5'
```

2. 为 JSON 数据建立一个 Bean 类

```kotlin
class Book(var id: Int, var name: String, var version: String) {}
```

3. 使用 GSON 的实例进行解析

```kotlin
private fun parseJsonWithGSON(jsonData: String): String {
    val retJsonData = StringBuilder()
    val gson = Gson()
    // 解析单个数据
    // val book = gson.fromJson(jsonData, Book::class.java)
    // 解析多个(数组)数据
    // 防止 List<Book> 中 Book 类型被擦除
    // TypeToken 是个类, 我们使用 object: TypeToken<List<Book>>() {} 定义了其匿名类, 然后获取了这个类的 type 属性
    // 相当于用一个子类 SubList extends List<String> 将父类中的泛型给保存
    val typeOf = object: TypeToken<List<Book>>() {}.type
    val books = gson.fromJson<List<Book>>(jsonData, typeOf)
    books.forEach {
        retJsonData.append("id is ${it.id} @ name is ${it.name} @ version is ${it.version}\n")
    }
    return retJsonData.toString()
}
```

## GSON 处理复杂数据

如果是时间等复杂对象的需要手动配置针对目标对象的 Serializer 与 Deseralizer

```kotlin
val todoGson = GsonBuilder()
    .registerTypeAdapter(LocalDateTime::class.java, // 对 LocalDateTime 添加 Serializer
        object : JsonSerializer<LocalDateTime> {
            override fun serialize(
                src: LocalDateTime?,
                typeOfSrc: Type?,
                context: JsonSerializationContext?
            ): JsonElement {
                return JsonPrimitive(src?.toLong()) // toLong 是我对 LocalDateTime 添加的方法, 转为了 millsSecond
            }

        }
    ).registerTypeAdapter(LocalDateTime::class.java, // 对 LocalDateTime 添加 Deserializer
        object : JsonDeserializer<LocalDateTime> {
        override fun deserialize(
            json: JsonElement?,
            typeOfT: Type?,
            context: JsonDeserializationContext?
        ): LocalDateTime {
            return json!!.asJsonPrimitive.asString.toLong().toLocalDateTime() // 获取 json 存储的 long 格式数据，转为 LocalDatetime
        }
    }).serializeNulls().create()

// to Json
val dbJson = todoGson.toJson(dbInJson)
// back to obj
val dbInJson = todoGson.fromJson(dbJson, DBInJson::class.java)
```

# 网络请求的封装

新建一个工具类对网络请求进行封装, 但是因为是多线程任务, 要再事件完成或失败时进行相应的回调处理, 因此需要定义相关的回调接口, 或实现相关库提供的回调接口

相关的回调接口
```kotlin
interface HttpCallbackListener {
    fun onFinish(response: String)
    fun onError(e: Exception)
}
```

整体工具类的实现

```kotlin
object HttpUtil {

    fun sendHttpRequest(address: String, callbackListener: HttpCallbackListener) {
        thread {
            var connection: HttpURLConnection? = null
            try {
                val response = StringBuilder()
                var url = URL(address)
                connection = url.openConnection() as HttpURLConnection
                connection.connectTimeout = 8000
                connection.readTimeout = 8000
                val input = connection.inputStream
                val reader = BufferedReader(InputStreamReader(input))
                reader.use {
                    reader.forEachLine {
                        response.append(it)
                    }
                }
                callbackListener.onFinish(response.toString())
            } catch (e: Exception) {
                e.printStackTrace()
                callbackListener.onError(e)
            } finally {
                connection?.disconnect()
            }
        }

    }

    fun sendOkHttpRequest(address: String, callbackListener: okhttp3.Callback) {
        val client = OkHttpClient()
        val request = Request.Builder()
            .url(address)
            .build()
        // OkHttp 在 enqueue() 方法的内部已经帮我们开好子线程了，然后会在子线程中执行 HTTP 请求，并将最终的请求结果回调到 okhttp3.Callback 当中
        client.newCall(request).enqueue(callbackListener)
    }
}
```

# 网络库 Retrofit

## Quick Start

1. `app/build.gradle` 中添加依赖

```kotlin
implementation 'com.squareup.retrofit2:retrofit:2.6.1'
implementation 'com.squareup.retrofit2:converter-gson:2.6.1'
```

2. 封装好要请求的对象

```kotlin
class Book(val id: String, val name: String, val version: String) {}
```

3. 定义请求接口

```kotlin
interface BookService {

    @GET("/DVWA/get_data.json")
    // 返回值必须声明成 Retrofit 中内置的 Call 类型
    fun getBookData(): Call<List<Book>>
}
```

4. 触发服务器请求: 传入服务器根地址获取 Retrofit 对象 + 创建服务对象 + 调用服务获取数据

```kotlin
buttonGetBookData.setOnClickListener {
    val retrofit = Retrofit.Builder()
        .baseUrl("http://192.168.0.107/")
        // 用于指定 Retrofit 在解析数据时所使用的转换库，这里指定成 GsonConverterFactory - 获取的是 JSON 数据
        .addConverterFactory(GsonConverterFactory.create())
        .build()
    val appService = retrofit.create(BookService::class.java)
    appService.getBookData().enqueue(object : Callback<List<Book>> {

        override fun onResponse(call: Call<List<Book>>, response: Response<List<Book>>) {
            // 自动的多线程操作
            // 当发起请求的时候，Retrofit 会自动在内部开启子线程，当数据回调到 Callback 中之后，Retrofit 又会自动切换回主线
            val list = response.body()
            if (list != null) {
                for (book in list) {
                    Log.d("MainActivity", "id is ${book.id}")
                    Log.d("MainActivity", "name is ${book.name}")
                    Log.d("MainActivity", "version is ${book.version}")
                }
            }
        }

        override fun onFailure(call: Call<List<Book>>, t: Throwable) {
            t.printStackTrace()
        }
    })
}
```

1. 按照之前步骤配置 HTTP 明文请求 + AndroidManifest 中配置网络权限

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## 复杂接口

通过变量匹配多个接口

- Path 中应用变量

```kotlin
@GET("{page}/get_data.json")
fun getData(@Path("page") page: Int): Call<Data>
```

- GET 请求中的参数: `http://example.com/get_data.json?u=<user>&t=<token>`

```kotlin
@GET("get_data.json")
fun getData(@Query("u") user: String, @Query("t") token: String): Call<Data>
```

- POST 请求中的参数

```kotlin
@POST("data/create.json")
fun createData(@Body data: Data): Call<ResonseBody>
```

- 静态配置请求头参数

```kotlin
@Headers("User-Agent: ...", "Cache-Control: ...")
@GET("get_data.json")
fun getData(): Call<Data>
```

- 动态配置请求头参数

```kotlin
@GET("get_data.json")
fun getData(@Header("User-Agent") userAgent: String,
@Header("Cache-Control") cacheControl: String): Call<Data>
```

## 单例类封装

Service 的动态代理对象是能通用的, 因此依旧可以通过创建一个单例类来实现共享使用

```kotlin
object ServiceCreator {
    private const val BASE_URL = "http://10.0.2.2/"
    private val retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(GsonConverterFactory.create())
        .build()
    fun <T> create(serviceClass: Class<T>): T = retrofit.create(serviceClass)

    inline fun <reified T> create(): T = create(T::class.java)
}
```