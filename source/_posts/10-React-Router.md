---
title: 10-React-Router
mathjax: false
date: 2023-01-02 18:59:27
summary: React 路由相关介绍
categories: React
tags:
  - react
---

# React 路由

SPA，single page web application，单个页面 Web 应用，整个应用只有一个完整的页面（单页面多组件），点击页面中的连接不会刷新页面，只会做页面的局部更新
- 数据都需要通过 ajax 请求获取，并在前端异步展现

## 路由

路由就是一种映射关系，实现 url 中路径与组件/处理函数的映射：

1. 点击链接，变换url
2. 检测到 url 变化提取 path
3. 根据 path 更新页面

### 前端路由

实现 path 与 组件的映射，直接加载目标组件更新页面

工作过程：当 path 变化为 `/demo` 时，路由组件就会变为 `<Demo/>` 组件

### 后端路由

实现的是 path 与 函数的映射，用于处理客户端提交的请求

注册路由方式：`router.get(path, function(req, res))`

工作过程：当 node 接收到一个请求时，根据请求路径找到匹配的路由，调用路由中的处理函数返回响应数据

## 浏览器地址操作

浏览器的历史记录是一种栈结构，BOM 中的 histroy 对象对其维护，使用 history.js 可以更加方便的操作 BOM 中的 histroy

1、push：`history.push('/test')`，向栈中推入一个路径并更改 url

2、replace：`history.repalce('/test')`，替换栈顶的路径并更改 url
> 注意替换操作将不能回退
> 比如我从 demo1 推入了 demo2 随后执行替换 到了 demo3，那么执行回退路径时会直接到 demo1，demo2 的记录丢失了

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

## Quick Start

1. 明确页面中的导航区与展示区
2. 导航区的 a 标签改为 Link 标签，to 属性指路由变化
```jsx
{/* <a className="list-group-item" href="./about.html">About</a> */}
<Link className={'list-group-item ' + (this.state.activeName === 'About' ? 'active' : '') } to="/about" onClick={this.setActive('About')}>About</Link>
```

3. 展示区与路由匹配
```jsx
{/* 注册路由 */}
<Routes>
    <Route path="/about" element={<About/>}/>
    <Route path="/home" element={<Home/>}/>
    <Route path="/" element={<Welcome/>}/>
    <Route path="/welcome" element={<Welcome/>}/>
</Routes>
```

4. <App> 最外侧包裹 <BrowserRouter> - 基于浏览器路由 或 <HashRouter> - 基于锚点路由
```jsx
ReactDOM.render(<BrowserRouter><App/></BrowserRouter>, document.getElementById('root'))
```

## 重定向

- 使用 Navigate 组件实现重定向

```jsx
<Route path="*" element={<Navigate to="/film"/>}/>
```

- 使用 Redirect 组件实现重定向: Redirect 需要自行设计, 通常配合 useNavigate 使用

```jsx
<Route path="*" element={<Redirect to="/film" />} />
function Redirect(props) {
    const navigate = useNavigate()
    useEffect(() => {
        navigate(props.to, { replace: true })
    })

    return null
}
```

## 默认匹配

可以用默认匹配实现通用 404

```jsx
<Route path="*" element={<NotFound/>} />
```

## 嵌套路由

1. 父组件中要使用 `<Outlet></Outlet>` 设置好路由容器

```jsx
function Film() {
    return (
        <div>
            <h3> Film </h3>
            <div style={{height: "200px", background: "cyan"}}> 大轮播 </div>
            {/* 设置好路由容器, 这样多级路由不必分开, 方便管理 */}
            <Outlet></Outlet>
        </div>
    )
}
```

2. 路由组件中直接嵌套写即可: `index` 用于匹配到只提供了父路径没有提供子路径的情况

```jsx
<Routes>
    <Route path="/film" element={<Film></Film>} >
        {/* 没有匹配到子路径时 */}
        <Route index element={<Navigate to="/film/nowplaying"/>}></Route>
        <Route path="nowplaying" element={<Nowplaying/>}></Route>
        <Route path="comingsoon" element={<Comingsoon/>}></Route>
    </Route>
    ...
</Routes>
```

## 声明式导航

Link 与 NavLink

```jsx
<footer>
    <ul>
        <li> <NavLink to="/film" className={handleJcActive}> 电影 </NavLink> </li>
        <li> <NavLink to="/cinema" className={handleJcActive}> 影院 </NavLink> </li>
        <li> <NavLink to="/center" className={handleJcActive}> 我的 </NavLink> </li>
    </ul>
</footer>
```

默认情况下 NavLink 会给选中的组件添加 `active` 的 className, 但是容易重名, 因此我们可以使用其提供的 className 接口进行更改: 

```jsx
const handleJcActive = (props) => {
    if(props.isActive) {
        return 'jcActive'
    } else {
        return ''
    }
}
```

## 编程式导航

使用 useNavigate 进行编程式控制

```jsx
const navigate = useNavigate()
const hadleChangePage = (id) => {
    navigate(`/detail?id=${id}`)
}
```

## 参数传递

### params

只能传字符串, 传值过多 url 会变得很长, 参数必须在路由上配置
- 刷新页面，参数不丢失

1. 通过 `url` 传递参数

```jsx
<MyNavLink to="/paramsSend/cjc/123456" onClick={this.setActive('paramsSend')}>paramsSend</MyNavLink>
```

2. 动态路由: 匹配时定义好参数模式

```jsx
<Route path="/paramsSend/:user/:pwd" element={<ParamsSend/>}/>
```

3. 引入 `useParams` 获取参数

```jsx
import { useParams } from 'react-router-dom';
const params = useParams()
<p>用户名：{params.user}</p>
<p>密码：{params.pwd}</p>
```

### TODO: search

优点：刷新页面，参数不丢失

缺点：只能传字符串，传值过多url会变得很长，获取参数需要自定义hooks

```jsx
const [searchParams, setSearchParams] = useSearchParams()
if(searchParams.has("username")){
    const username = searchParams.get("username")
}
```

## 路由拦截

未登录的情况下, 多个页面的路由需要拦截并重定向到登陆界面

一种实现方式就是自己写好状态判断函数, 然后在路由匹配后渲染时根据状态选择渲染

```jsx
<Route path="/center" element={isAuth() ? <Center/> : <Redirect to="/login"/>} />
```

但是这样没有和组件绑定, 导致其内容不会改变, 因此要用一个组件封装该逻辑, 在 `AuthComponent` 内部判断是否展示 `this.props.children`

```jsx
<Route path="/center" element={<AuthComponent>
    <Center></Center>
</AuthComponent>} />
```

## TODO: 路由懒加载

路由条目太多, 导致加载拥塞; 路由懒加载将实现按需加载路由

```jsx
const LazyLoad = (path) => {
    const Comp = React.lazy(() => import(`../views/${path}`))
    return (
        <React.Suspense fallback={<> Loading... </>}>
            <Comp/>
        </React.Suspense>
    )
}
```

```jsx
<Route path="/cinema" element={LazyLoad("Cinema")} />
```


# 常用路由组件

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

默认情况下它自动给当前点击的 Link 添加 active 属性（应该是通过路由匹配结果来高亮组件），也可以通过 `activeClassName` 参数的传递指定自动添加的属性名称

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
- caseSensitive 属性: 实现路由大小写区分, 默认不区分大小写

```jsx
<Route caseSensitive={true} path="/about" element={<About/>}/>
```

### 二级路由

需要明确的是二级路由需要通过一级路由的匹配才能访问，因此以及路由通常设定为 `/pagename/*` 的形式这样才能访问二级路由 `/pagename/pageone` 
- v6 中会对二级路由的前缀进行自动拼接，因此写二级路由时不必加上前缀

```jsx
// 一级路由
<Routes>
    <Route path="/about" element={<About/>}/>
    <Route path="/home/*" element={<Home/>}/>
    <Route path="*" element={<Welcome/>}/>
</Routes>

// 二级路由 /home/*
<Routes>
    <Route path="/news" element={<News/>}/>
    <Route path="/message" element={<Message/>}/>
    {/* 使用跳转实现 NavLink 的默认选中功能 */}
    <Route path="*" element={<Navigate to="/home/news"/>}/>
</Routes>
```

### 默认路由与 index 属性

默认路由：在 Routes 中匹配未找到匹配字段时的缺省处理，通常设置 `path="*"` 来实现
`index` 属性: 用于匹配到只提供了父路径没有提供子路径的情况

在 `<Routes>` 中匹配路由时，如果前面的 `<Route>` 都没有匹配上，最后就会走 带有 index 属性的路由` （如果配置了的话）
```jsx
<Routes>
    <Route index element={<Welcome/>}/>
    <Route path="/about" element={<About/>}/>
    <Route path="/home/*" element={<Home/>}/>
    <Route path="*" element={<404/>}/>
</Routes>
```


# 常见问题

## 静态样式丢失问题

样式引入时不能以当前相对路径来引入，路由机制会导致资源找不到
```html
<link rel="stylesheet" href="./css/bootstrap.css">
```

解决方案：
1. `href="%PUBLIC_URL%/css/bootstrap.css"` 使用 `%PUBLIC_URL%` 构造绝对路径引入 public 下资源文件
2. `href="/css/bootstrap.css"` 使用 `/` 以根路径形式访问
3. 使用 `<HashRouter>` 锚路由被 `#` 隔离了正常的 url 不会对相对路径产生影响了（不推荐）

# useRoutes 钩子配置路由

```jsx
export default function JCRouter() {
    const element = useRoutes([
        {
            path: "/film",
            element: LazyLoad("Film"),
            children: [
                {
                    path: "",
                    element: <Redirect to="/film/nowplaying"/>
                },
                {
                    path: "nowplaying",
                    element: LazyLoad("film/Nowplaying"),
                },
                {
                    path: "comingsoon",
                    element: LazyLoad("film/Comingsoon"),
                },
            ]
        },
        {
            path: "/cinema",
            element: LazyLoad("Cinema")
        },
    ])

    return (
        element
    )
}
```