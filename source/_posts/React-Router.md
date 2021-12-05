---
title: React Router
mathjax: false
date: 2021-12-04 20:22:16
summary: React 组件相关知识点
tags:
  - react
  - react components
---
# React 路由

## SPA 

SPA，single page web application，单个页面 Web 应用，整个应用只有一个完整的页面（单页面多组件），点击页面中的连接不会刷新页面，只会做页面的局部更新
> 数据都需要通过 ajax 请求获取，并在前端异步展现

## 路由

路由就是一种映射关系，实现 url 中路径与组件/处理函数的映射：

1、点击链接，变换url；

2、检测到 url 变化提取 path；

3、根据 path 更新页面；

### 前端路由

实现 path 与 组件的映射，直接加载目标组件更新页面

工作过程：当 path 变化为 `/demo` 时，点前路由组件就会变为 `<Demo/>` 组件

### 后端路由

实现的是 path 与 函数的映射，用于处理客户端提交的请求

注册路由方式：`router.get(path, function(req, res))`

工作过程：当 node 接收到一个请求时，根据请求路径找到匹配的路由，调用路由中的处理函数返回响应数据

## 浏览器地址操作

浏览器的历史记录是一种栈结构，BOM 中的 histroy 对象对其维护，使用 history.js 可以更加方便的操作 BOM 中的 histroy

1、push：`history.push('/test')`，向栈中推入一个路径并更改 url

2、replace：`history.repalce('/test')`，替换栈顶的路径并更改 url
> 注意替换操作将不能回退比如我从 demo1 推入了 demo2 随后执行替换 到了 demo3，那么执行回退路径时会直接到 demo1，demo2 的记录丢失了

3、监听路径变化：
```javescript
history.listen((location) => {
    console.log('请求路由路径变化了', location)
})
```

另一种方法是锚点跳转的思想：以 `index.html` 为首，进行锚点跳转会形成：`index.html#demo`

# react-router-dom

用于实现一个 SPA 项目，基于 React 的项目基本都会用到此库

```shell
yarn add react-router-dom
```

1、明确页面中的导航区与展示区

2、导航区的 a 标签改为 Link 标签，to 属性指路由变化
```jsx
{/* <a className="list-group-item" href="./about.html">About</a> */}
<Link className={'list-group-item ' + (this.state.activeName === 'About' ? 'active' : '') } to="/about" onClick={this.setActive('About')}>About</Link>
```

3、展示区与路由匹配
```jsx
{/* 注册路由 */}
<Routes>
    <Route path="/about" element={<About/>}/>
    <Route path="/home" element={<Home/>}/>
    <Route path="/" element={<Welcome/>}/>
    <Route path="/welcome" element={<Welcome/>}/>
</Routes>
```

4、<App> 最外侧包裹 <BrowserRouter> - 基于浏览器路由 或 <HashRouter> - 基于锚点路由
```jsx
ReactDOM.render(<BrowserRouter><App/></BrowserRouter>, document.getElementById('root'))
```

# 常用组件

```jsx
import {Routes, Route, Redirect, Link, NavLink} from 'react-router-dom'
```

## Link

Link 组件用于导航区，实现路由的发起
```jsx
import {Link} from 'react-router-dom'
<Link className={'list-group-item ' + (this.state.activeName === 'About' ? 'active' : '') } to="/about" onClick={this.setActive('About')}>About</Link>
```

## NavLink

通常用到的就是当前导航项的高亮，可以通过 state 与 onClick 进行控制动态添加 active 这个 class 属性

但是更加方便的是直接使用 NavLink 这个组件，默认情况下它自动给当前点击的 Link 添加 active 属性（应该是通过路由匹配结果来高亮组件），也可以通过 `activeClassName` 参数的传递指定自动添加的属性名称

```jsx
<NavLink className="list-group-item" to="/about" >About</NavLink>
<NavLink activeClassName="redActive" className="list-group-item" to="/about" >About</NavLink>
```

### 二次封装
更进一步的可以利用组件二次封装中所学，将 NavLink 中的固定字段进行二次封装处理

```jsx
render() {
    // this.props 中使用 children 接收 标签体内容，因此可以直接传递给 NavLink 的child属性，完成二次封装的优秀对接
    return (
        <NavLink className="list-group-item" {...this.props} />
    )
}
```

## Routes
用于包裹众多路由组件 `<Route>` 实现多 Route 的快速查找映射

**caseSensitive 属性实现路由大小写区分**：默认不区分大小写
Tips: 但是我测试时并没有成功，母鸡
```jsx
<Route caseSensitive={true} path="/about" element={<About/>}/>
```

### 二级路由

需要明确的是二级路由需要通过一级路由的匹配才能访问，因此以及路由通常设定为 `/pagename/*` 的形式这样才能访问二级路由 `/pagename/pageone` 

v6 中会对二级路由的前缀进行自动拼接，因此写耳机路由时不必加上前缀

Example：
一级路由如下：
```jsx
<Routes>
    <Route path="/about" element={<About/>}/>
    <Route path="/home/*" element={<Home/>}/>
    <Route path="*" element={<Welcome/>}/>
</Routes>
```

耳机 `/home/*` 路由如下：
```jsx
<Routes>
    <Route path="/news" element={<News/>}/>
    <Route path="/message" element={<Message/>}/>
    {/* 使用跳转实现 NavLink 的默认选中功能 */}
    <Route path="*" element={<Navigate to="/home/news"/>}/>
</Routes>
```

### 默认路由

默认路由：在 Routes 中匹配未找到匹配字段时的缺省处理，通常设置 `path="*"` 来实现

在 `<Routes>` 中匹配路由时，如果前面的 `<Route>` 都没有匹配上，最后就会走 带有 index 属性的路由` （如果配置了的话）

这个在我之前自己设置时考虑到了，我想默认让他路由到 <Welcome/> 组件也就是对 `/` 进行路由
> 添加 `/welcome` 路由的原因是，<Welcome/> 也是个页面，用户可定能通过某个交互去 Welcome，因此用户主动去的时候总不能让他的 url 还是 `/` 吧
```jsx
<Routes>
    <Route path="/about" element={<About/>}/>
    <Route path="/home" element={<Home/>}/>
    <Route path="/" element={<Welcome/>}/>
    <Route path="/welcome" element={<Welcome/>}/>
</Routes>
```

但是使用 `path="*"` 会让用户有更好的体验
```jsx
<Routes>
    <Route path="/about" element={<About/>}/>
    <Route path="/home/*" element={<Home/>}/>
    <Route path="*" element={<Welcome/>}/>
</Routes>
```

### 重定向

默认路由也可以和重定向配合使用：当用户访问了某一不存在的资源时都重定向到同一个地方，这将通过 Navigate 组件以及 to 属性实现
> Tips：to 属性的值不受二级路由的影响哦

```jsx
<Routes>
    <Route path="/news" element={<News/>}/>
    <Route path="/message" element={<Message/>}/>
    <Route path="*" element={<Navigate to="/home/news"/>}/>
</Routes>
```

### index 属性

**index 属性实现默认路由：**

在 `<Routes>` 中匹配路由时，如果前面的 `<Route>` 都没有匹配上，最后就会走 带有 index 属性的路由` （如果配置了的话）

这个在我之前自己设置时考虑到了，我想默认让他路由到 <Welcome/> 组件也就是对 `/` 进行路由
> 添加 `/welcome` 路由的原因是，<Welcome/> 也是个页面，用户可定能通过某个交互去 Welcome，因此用户主动去的时候总不能让他的 url 还是 `/` 吧
```jsx
<Routes>
    <Route path="/about" element={<About/>}/>
    <Route path="/home" element={<Home/>}/>
    <Route path="/" element={<Welcome/>}/>
    <Route path="/welcome" element={<Welcome/>}/>
</Routes>
```

引入 index 属性后一切都简单了
```jsx
<Routes>
    <Route caseSensitive path="/about" element={<About/>}/>
    <Route path="/home" element={<Home/>}/>
    <Route index element={<Welcome/>}/>
</Routes>
```
> 然后从安全角度想想：可以通过判断页面是否为 <Welcome/> 来用字典扫描资源奥，其实也无伤大雅，因为这仅仅是对不存在的资源的一个默认处理，还需要从流量角度解决这个爆破问题

## 常见问题

### 静态样式丢失问题

样式引入时不能以当前相对路径来引入，路由机制会导致资源找不到
```html
<link rel="stylesheet" href="./css/bootstrap.css">
```

解决方法有三：

1、`href="%PUBLIC_URL%/css/bootstrap.css"` 使用 `%PUBLIC_URL%` 构造绝对路径引入 public 下资源文件

2、`href="/css/bootstrap.css"` 使用 `/` 以根路径形式访问

3、使用 `<HashRouter>` 锚路由被 `#` 隔离了正常的 url 不会对相对路径产生影响了（不推荐）

## 路由的匹配机制

V6 中没有这个概念了，下面就简单记录下历史吧...

### 模糊匹配

默认情况为模糊匹配，只需要匹配到前缀即可

### 严格匹配

就是完全一致，在 Routes 中从上到小找，匹配到就渲染，匹配不到就空

## 二级路由

![二级路由](https://gitee.com/Butterflier/pictures/raw/master/20211204213135.png)