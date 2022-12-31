---
title: 03-React-Component Basic
mathjax: false
date: 2022-12-31 18:57:46
summary: 类组件与函数式组件的介绍
categories: React
tags:
  - react
---
# 类组件

## 最基本的类组件

可以使用 rcc ( in vscode ) 快速构建

```jsx

import React from 'react'

class App extends React.Component {
    render() {
        return <div><h1>Hello App from React.Component </h1></div>
    }
}

export default App
```

引入创建的类组件后, 通过 ReactDom 进行渲染

```jsx
// 这里的 App 是别名, 不一定与目标文件导出对象的名称一致
import APP from './01-base/01-1-class组件初识.js'

ReactDOM.render(<div>Test</div>, document.getElementById('root'))
```

Tips: React 18 版本不在推荐使用 ReactDOM.render() 的渲染方式
```jsx
const container = document.getElementById('root');
const root = createRoot(container);
root.render(<APP />);
```

## 根据属性构造简单的动态类组件

```jsx
class App extends React.Component {
    render() {
        return <div><h1>Hello App from React.Component @ {this.props.name} </h1></div>
    }
}

// .render 直接实例化, 渲染时直接传递该对象即可
const app = new App({
    name: 'ComingPro'
}).render()

export default app
```

此时已经实例化啦, 因此直接渲染目标对象即可

```jsx
const container = document.getElementById('root');
const root = createRoot(container);
root.render(App);
```

# 函数式组件

无状态组件, 十分简单, 但自身能承担的功能也有限
- 组件中  `this` 为 `undefined`，是因为经过 `babel` 翻译开起严格模式，从而禁止了自定义 `this` 指向 `window`

## 基本的函数式组建

```jsx
function App() {
    return (
        <div>
            <h1> 02-函数式组件.js </h1>
            <h2> Hello ComingPro </h2>
            <h2> You are the Best </h2>
        </div>
    )
}
ReactDOM.render(<App/>, document.getElementById('test'))
```

# 组件的嵌套

```jsx
import React, {Component} from 'react'

class Navbar extends Component {
    render() {
        return(
            <h2>Navbar</h2>
        )
    }
}

function Swiper() {
    return (
        <h2>Swiper</h2>
    )
}

const Tabbar = () => <h2>Tabbar</h2>

export default class App extends Component {
    render() {
        return (
            <div>
                <h1>03-组件的嵌套.js</h1>
                <Navbar></Navbar>
                <Swiper></Swiper>
                <Tabbar></Tabbar>
            </div>
        )
    }
}
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212311851361.png)

# 组件的样式

推荐使用行内样式, 方便组件复用
- 如果导入外部 CSS 文件, 整合的 webpack 会将其转为内部 `style` 从而将样式应用
- 推荐每一个组件建立一个文件夹，文件夹中 `index.js` 写组件设计代码；`index.css` 写组件样式代码

```jsx
import React, {Component} from 'react'
import './css/01-index.css' // webpack 的支持

export default class App extends Component {
    render() {
        var username = "ComingPro"
        var style1 = {
            background: "yellow"
        }
        return (
            <div>
                <ul>
                    <li>10 + 20 = {10 + 20}</li>
                    <li> Username: {username}</li>
                    <li> 10 &gt; 20 : {10 > 20 ? "Yes" : "No"}</li>
                    <li> 10 &lt; 20 : {10 < 20 ? "Yes" : "No"}</li>
                </ul>
                <div style={style1}> 推荐使用这两种行内样式, 方便组件的管理 </div>
                <div style={{backgroundColor: "blue", color: "white", fontSize: "22px", padding: "8px"}}> 命名变为驼峰 </div>
                <div className='active'> 引入的外部 CSS </div>
                <div>
                    <label htmlFor="username">Username: </label>
                    <input id="username" type="text" placeholder='for should be htmlFor'></input>
                </div>
            </div>
        )
    }
}
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212311853206.png)


# 组件的事件绑定

所有事件命名都是 `on` + `事件的驼峰命名` 如 `onClick, onMouseOver`
- React 事件绑定原理: React 并没有真正的绑定事件到每一个具体的标签上，而是采用事件代理的模式，再根标签上进行事件的监听（冒泡）

1、普通函数调用：`onClick={this.function}` 这里的 `function` 指向的是非匿名函数
> 普通函数一般 this 应指向调用者，但是事件并不是直接绑定到标签上的，而是通过事件代理的机制实现的，因此 this 指向了 `undefined`；可使用 bind 修正
> `.call(this)` 会改变 this 指向但是会制动执行函数
> `.apply(this)` 和 call 类似改变 this 指向的同时会执行函数
> `.bind(this)` 仅仅改变 this

2、显式匿名函数调用：`onClick={()=>{do sth...}}`
> 箭头函数中的 this 与外界保持一致

3、隐式匿名函数调用：`onClick={this.function}` 这里的 `function` 指向的是匿名函数

4、匿名函数+普通函数：`onClick={()=>{this.function()}}` 

比较推荐第四种方式，既能修正 `this` 指向还有较好的封装并且参数传递也容易实现


# 组件二次封装

针对组件中有固有属性时且较多时，我们可以自定义一个一般组件实现封装

如 NavLink 中 className 属性固定，因此可以进行二次封装
```jsx
<NavLink className="list-group-item" to="/about">About</NavLink>
```

二次封装我们向上要符合用户的编程习惯，即使用标签体传递表明名称，属性依旧通过 props 传递；向下要对接好初始组件
```jsx
<MyNavLink to="/about">About</MyNavLink>
<MyNavLink to="/home">Home</MyNavLink>
```

方便向下对接的是，标签体中的内容将使用 this.props.children 接收，因此可以直接传递给 NavLink 的child属性，完成二次封装的优秀对接
```jsx
render() {
    return (
        <NavLink className="list-group-item" {...this.props} />
    )
}
```