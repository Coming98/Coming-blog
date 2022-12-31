---
title: 02-React-jsx
mathjax: false
date: 2022-12-31 18:57:34
summary: JSX 语法
categories: React
tags:
  - jsx
---

# jsx

JavaScript XML, 用来简化创建虚拟 DOM，通过 Babel 编译器将 jsx 相关代码转为 React 可识别的组件代码形式
- 表达式：jsx 中只支持表达式; 一个表达式会产生一个值，可以放在任何一个需要值的地方，

## Demo

```jsx
React.createElement(
    type,
    [props],
    [...children]
)
```

```jsx
import React from 'react'
import ReactDOM from 'react-dom'

class App extends React.Component {
    render() {
        return (
            React.createElement(
                "div",
                {
                    className: 'app',
                    id: 'appRoot'
                }, React.createElement(
                    "h1",
                    { className: 'title' },
                    "欢迎进入 React 的世界"
                ),
                React.createElement(
                    "p",
                    null,
                    "React.js 是一个构建页面 UI 的库"
                )
            )
        )
    }
}

ReactDOM.render(
    // <App/>,
    React.createElement(App),
    document.getElementById('root')
)
```


# 语法规则

```jsx
const data = 'Hello React'
const VDOM = (
    <div>
        <h1 className="title" style={ {color: 'white', backgroundColor: 'black', fontSize: '60px'} }>{data}</h1>
        <h2>Hello Peiqi</h2>
        {/*<Peiqi>peiqi is a pig</Peiqi>*/}
    </div>
)
```

Tips：React 组件的样式官方推荐使用行内样式，旨在将组件看作一个整体，复用时只需要迁移一个组件代码即可

1、创建虚拟 DOM 时不要写 引号 

2、标签中混入 js 表达式使用 花括号

3、标签中样式的类名要用 `className` 指定; `for` 关键字要用 `htmlFor`

4、标签中的内联样式要使用 js 对象传入 - 双层花括号，使用小驼峰改写 css 变量名

5、只能有一个根标签

6、单标签需要闭合 `<input type='text' />`

7、标签首字母小写，那么 React 就会去 HTML 中寻找与之同名的 HTML 标签；若首字母大写，则寻找同名组件

8、注释需要使用 花括号 + js 块注释