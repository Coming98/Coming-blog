---
title: React components
mathjax: false
date: 2021-12-04 14:03:58
summary: React 组件相关知识点
categories: React
tags:
  - react
  - react components
---

# 两类组件

## 函数定义组件

1、定义一个函数式组件：组件中  `this` 为 `undefined`，是因为经过 `babel` 翻译开起严格模式，从而禁止了自定义 `this` 指向 `window`

2、渲染组件到页面：`<Demo/>` 首先解析组件标签，寻找 `Demo` 组件的位置；直接调用 `Demo` 函数，将返回的虚拟 `DOM` 渲染到页面

```jsx
function Demo() {
    console.log(this) // undefined
    return (<h2>函数式组件</h2>)
}

ReactDOM.render(<Demo/>, document.getElementById('test'))
```

## 类定义组件

1、定义一个类式组件：this 问题后续详述

2、渲染到页面

```jsx
// 1、
class Demo extends React.Component {
    render() {
        return <h2>类式组件</h2>
    }
}

ReactDOM.render(<Demo/>, document.getElementById('test'))
```

# 组件的样式

react 推荐组件使用行内样式，因为 react 旨在实现组件的便利服用，如果组件的设计与展示分开，那么复用组件时就较为麻烦；使用行内样式只需要迁移组件代码即可
> 推荐每一个组件建立一个文件夹，文件夹中 `index.js` 写组件设计代码；`index.css` 写组件样式代码

## 特殊的样式属性

1、`class` 属性改为 `className`

2、`for` 属性改为 `htmlFor`

3、样式中带有 `-` 的样式都改为小驼峰的写法，`backgroundColor`, `fontSize`


# 组件的事件绑定

所有事件命名都是 `on` + `事件的驼峰命名` 如 `onClick, onMouseOver`

常见的事件绑定方式有三种：

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

## React 事件绑定原理

React 并没有真正的绑定事件到每一个具体的标签上，而是采用事件代理的模式，再根标签上进行事件的监听（冒泡）

# 一般组件与路由组件

路由组件通常放到 pages/ 下，与一般组件不一样的是，路由组件会被默认传递一些 props 参数

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
