---
title: 03-React-Component Basic
mathjax: false
date: 2022-12-31 18:57:46
summary: 类组件与函数式组件的介绍
categories: React
tags:
  - react
---

# 类的基础

1. JS 中继承关键字: `extends`
2. react 引入了 `js` 文件就会默认执行

## 有关类的 js 文件

```jsx

class Test {
    constructor() {
        this.a = 1;
    }

    testa() {
        console.log('a - test')
        console.log(this.a)
    }
}

// 继承
class ChildTest extends Test{
    testb() {
        console.log('b - test')
        console.log(this.a)
    }
}

var obj = new Test()
obj.testa()

var childObj = new ChildTest()
childObj.testb()
```

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
export default App
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

为组件添加样式: 推荐使用行内样式, 方便组件复用
- 如果导入外部 CSS 文件, 整合的 webpack 会将其转为内部 `style` 从而将样式应用

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