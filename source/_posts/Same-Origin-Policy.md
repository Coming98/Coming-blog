---
title: Same Origin Policy
mathjax: false
date: 2021-10-29 16:36:44
summary: 对同源策略的学习与理解
categories: Security
tags:
  - web security
  - course
---

# Same-Origin Policy

refs：https://www.cnblogs.com/gyrgyr/p/10571616.html

**Definition**: 一种安全机制，用于限制不同源的 document 或 script 相互访问

**Target**：目标是限制 scripts：如果不加这个限制，那么恶意网页下的 scripts 将能够获取别的网页下的 Cookie 等，然后在本地网页开启了一个 iframe，登录后进行恶意性操作，但是同源策略的存在，先不说 Cookie 等私密信息获取不到，即使开起了 iframe 也将和恶意 script 不同源

**Example**：恶意钓鱼网站，引入第三方邮件服务后，因为不同源，一次你不能用脚本访问或操作其数据

# Origin

什么是同源的呢，它将由 URL 决定，他的 协议（http, https），端口号（80,8080），主机名（baidu.com, sina.com) 都要一致才能视为同源

与 `http://store.company.com/dir/page.html` 相比

| URL                                       | Accessible | Reason                            |
| ----------------------------------------- | ---------- | --------------------------------- |
| http://store.company.com/dir2/other.html  | True       | 主机名一致 后续的子目录路径可不同 |
| https://store.company.com/page.html       | False      | 协议不同                          |
| http://store.company.com:81/dir/page.html | False      | 端口号不同，http 默认 80          |
| http://news.company.com/dir/page.html     | False      | 主机名不同                        |

Tips：IE 浏览器的同源策略是不考虑端口的

## Cookie Same-Origin Policy

Cookie 只有同源网页才能共享（Protocol，Port，host）

## DOM Same-Origin Policy

DOM，文档对象模型，Document Object Model：在网页上组织页面或文档的对象被组织在一个树形结构中，用来表示文档中对象的标准模型即为 DOM

![image-20210920152526889](https://gitee.com/Butterflier/pictures/raw/master/image-20210920152526889.png)

1、`<script>, <img>, <iframe>, <link>` 的跨域请求，不受同源策略的约束，但不能通过 ajax 来获取

> 就是我们可以引入外部网站的 script, img, iframe, link，我们所限制的是引入的内容的操作

2、引入外部 script 文件后，该文件的源为当前页面（谁引入的 JS 那么 JS 就属于哪个域）

> `<iframe>` 引入后源不变

3、script 不能随意跨域操作其它页面的 DOM

4、script 不能获取 `<script>, <img>, <iframe>, <link>` 跨域请求得到的内容（可以获取一些不涉及用户隐私的公共信息）

>  注意请求可以送达，但是响应将被拦截
>
>  不同源 JavaScript 受限，同源不受限
>
>  ![image-20210920152846907](https://gitee.com/Butterflier/pictures/raw/master/image-20210920152846907.png)

5、`<iframe>` 父子页面（点击进入的下一页面）交互受同源策略约束

## Ajax Same-Origin Policy

Tips：阻止的是跨域资源的获取，而不是阻止跨域的请求；跨域请求可以正常发出，但是浏览器阻止脚本获取返回的内容。

## Web Storage Same-Origin Policy

克服 Cookie 的限制：如果你的数据不需要服务端处理，只需要存储在客户端即可，但是也存在同源访问的限制

**Target**：1、提供一种在 Cookie 之外存储会话数据的途径；2、提供一种存储大量可以跨会话存在的数据的机制

**localStorage**：与站点源（origin）绑定，关闭浏览器后仍然有效

**sessionStorage**：绑定当前浏览器端口，浏览器会话结束后清理

## 脚本型 URL 的同源策略

伪 URL 的源如何判定？

![image-20210920154445387](https://gitee.com/Butterflier/pictures/raw/master/image-20210920154445387.png)

![image-20210920154552671](https://gitee.com/Butterflier/pictures/raw/master/image-20210920154552671.png)

**Tips**：在脚本型 URL 加载的页面里，以父页面的上下文权限执行相应的脚本代码——与父页面同源

> 通过 bank 执行的脚本型 js 代码，即与 bank 同源，即可操作 iframe 内的内容

![image-20211028222503110](https://gitee.com/Butterflier/pictures/raw/master/image-20211028222503110.png)

# Cross Origin

## Server Proxy

同源策略的作用域是**浏览器**，因此可以利用服务器实现跨域通信，即使用第三方服务器获取目标信息在返回给自己（也就是我们常说的代理服务器）

![image-20210920155214194](https://gitee.com/Butterflier/pictures/raw/master/image-20210920155214194.png)

React Scaffolding 中在 package.json 追加如下配置，当用 ajax 请求了本地不存在的资源，将会转发给目标代理（但是不支持配置多个代理）

```shell
"proxy":"http://localhost:5000"
```

## document.domain

script 父子页面中设置 `document.domain` 为共同祖先域即可，如果非当前  url 的祖先域，则不会生效

> 这样做使得主站和子站，以及子站之间可以跨域访问，不适用于跨基础域的站点间共享数据

![image-20211028215714174](https://gitee.com/Butterflier/pictures/raw/master/image-20211028215714174.png)

Tips：Ajax 请求该跨域方法无效；**Ajax跨域必须 Protocol，port，host 一致**

## JSONP

JSON with Padding，利用 `<script>` 标签可以跨域 `get` 引入 js 脚本这一点，需要服务器端配合接口，用脚本动态生成 `js` 将数据封装在返回的 `js` 脚本里，供客户端提前定义好的回调函数使用

**Tips**：仅能应用于 GET 请求

1、get 方式请求数据，返回 JavaScript 脚本

```html
<script type="text/javascript" src="http://example.com/
     ?some-variable=some-data&jsonp=parseResponse">
</script>
```

2、服务端封装数据与 callback 函数

```java
List<Student> studentList = getStudentList();
JSONArray jsonArray = JSONArray.fromObject(studentList);
String result = jsonArray.toString();

//前端传过来的回调函数名称
String callback = request.getParameter("jsonp");
//用回调函数名称包裹返回数据，这样，返回数据就作为回调函数的参数传回去了
result = callback + "(" + result + ")";

response.getWriter().write(result);
```

3、因为是脚本，并且返回到本地已经预定义了回调函数，因此将直接自动执行，读出返回的目标内容

```html
<script type="test/javascript">
	// parseResponse({"variable": "value", "variable2": "value2"})
	function parseResponse(response) {
		do sth
	}
</script>
```

## Window.name

window.name 是全局变量，在窗口（window）的生命周期内，窗口重定向后载入的页面共享 window.name；同理在 iframe 中即使 url 发生了变化 iframe 中的 window.name 也将不会改变

Example：

简单的，A 页面和 B 页面不同源，B 页面希望获取 A 页面的数据，则 A 页面将数据存储到 window.name 后跳转到 B 页面，B页面即可获取A页面设置的值

简单的方案中是 A 页面主动将数据给 B，如何让 B 主动从 A 那里拿数据呢？

1、A 页面时数据页面，我们将格式化好的数据存到 `window.name` 中

```html
ref: https://blog.csdn.net/weixin_33991727/article/details/86248998
<body>
  <h2> A </h2>
  <script>
    var person = {
      name: 'Coming98',
      age: 22,
    }
    window.name = JSON.stringify(person)
  </script>
</body>
</html>
```

2、B 页面想要获取 A 页面的数据，首先创建一个 iframe 获取 window.name 的内容，但是此时 B 页面和 iframe 中的 A 页面不同源（port 不一致），因此让 iframe 重定向到与 B 页面同源的页面，这时 window.name 中的数据也被带过去了，B 页面此时就能够访问 iframe 中的 window.name 的内容了

```html
B 页面 url："http://localhost:8081/B.html"
<iframe src="http://localhost:8088/A.html" frameborder="1"></iframe>
  <script>
    var ifr = document.querySelector('iframe')
    ifr.style.display = 'none'
    var flag = 0;
    ifr.onload = function () {
        if (flag == 1) {
            console.log('跨域获取数据', ifr.contentWindow.name);
            ifr.contentWindow.close();
        } else if (flag == 0) {
            flag = 1;
            ifr.contentWindow.location = 'http://localhost:8081/proxy.html';
        }
    }
  </script>
```

## CORS

HTML5新特性，只需要让服务器端在响应头中明确的授权客户端有权读取其返回的数据即可

```java
header("Access-Control-Allow-Origin: *")
header("Access-Control-Allow-Origin: http://server.net")
```



Tips：存在预检机制，支持浏览器先询问服务器（即将请求的域名、方法和头信息是否许可，如果得到肯定答复，才发出正式请求，否则报错）

![image-20211029115407916](https://gitee.com/Butterflier/pictures/raw/master/image-20211029115407916.png)

## window.postMessage

允许不同源的脚本采用异步方式进行有限的通信

**信息发送**

```javascript
otherWindow.postMessage(message, targetOrigin, [transfer]);
```

otherWindow - 表示其它窗口的一个引用，比如 iframe 的 contentWindow 属性、执行`window.open` 返回的窗口对象

message - 发送的数据信息

targetOrigin - 指定该消息的发送目标窗口，可以是 `URI` 或 `*`

**事件监听**：目标接收方应当添加事件处理函数，接收并处理发送过来的信息

event.origin - 发送信息的 window 所在的源，如果不在白名单里，不予处理

event.source - 发送消息的窗口对象的引用

event.data - 是传过来的 message

```javascript
// 当信息被 postMessage 送来后调用的处理函数
function receiveMessage(event)
{

  if (event.origin !== "http://white.com:8080")
    return;

   // 事件逻辑处理，可以回一条消息过去
  event.source.postMessage("hi", event.origin);
}

window.addEventListener("message", receiveMessage, false);
```

主动获取信息步骤：A 想要获取 B 的信息，A 通过 iframe 或 window.open 嵌入 B，随后 A postMessage(message, B) 告诉 B 想要什么信息，B 检查 origin 后 postMessage(data, A) 将数据返回过去

被动获取信息步骤：A 将数据准备完毕通过 postMessage 将信息传给 B，B 通过事件监听和处理获取数据

![image-20210920162051524](https://gitee.com/Butterflier/pictures/raw/master/image-20210920162051524.png)

接收方：添加相关事件处理，实现跨域的数据传输

![image-20210920162105671](https://gitee.com/Butterflier/pictures/raw/master/image-20210920162105671.png)

# 
