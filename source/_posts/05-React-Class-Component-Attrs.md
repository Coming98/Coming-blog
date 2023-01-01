---
title: 05-React-Class Component Attrs
mathjax: false
date: 2023-01-01 21:02:11
summary: Ref, State, props
categories: React
tags:
  - react
---

# Ref

Motivation: 通过原生的 event 我们只能获取到触发事件的按钮等, 但我们想要的是 input 中的内容, 因此还需要引入额外的属性来实现

## React.createRef()

1. 使用 React.createRef() 维护引用变量: `mytext = React.createRef()`
2. 将引用变量赋值给组件的 ref 属性: `ref={this.mytext}`

```jsx
export default class App extends Component {

    mytext = React.createRef()

    render() {
        return (
            <div>
                <div>
                    <input type="text" ref={this.mytext}></input>
                    <button onClick={ this.handleClick }> ADD </button>
                </div>

            </div>
        )
    }

    handleClick = () => {
        console.log("`this.mytext.current` 获取目标标签对象")
        console.log(this.mytext.current)
        console.log("`this.mytext.current.value` 获取目标内容")
        console.log(this.mytext.current.value)
    }

}
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202301012048910.png)

## this.refs.xxx

不在推荐使用了, 字符串由人为定义, 编译器无法检查, 因此存在潜在的风险
1. 在目标组件上重写 ref 属性, 传入组件名称 `refName`
2. 在事件处理函数中可以通过 `this.refs.refName` 获取目标组件对象

```jsx
export default class App extends Component {

    render() {
        return (
            <div>
                <input type="text" ref="mytext"></input>
                <button onClick={ this.handleClick }> ADD </button>
            </div>
        )
    }

    handleClick = () => {
        console.log("`this.refs.mytext` 获取目标标签对象")
        console.log(this.refs.mytext)
        console.log("`this.refs.mytext.value` 获取目标内容")
        console.log(this.refs.mytext.value)
    }
}
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202301012044177.png)

# State

## state 初始化

### 类中直接初始化

- 类组件中直接定义: `state = {...props}`, state 名称是固定的, 不能人为修改
- 事件处理中更新: `this.setState({...props})`, state 不能像修改普通变量一样直接修改其值
- 更新后的事件处理: `this.setState({}, () => {})` 返回函数是在状态更新后的回调处理函数, 比如更新 betterscroll

```jsx
export default class App extends Component {
    
    state = {
        collectionState: true
    }

    render() {
        const {collectionState} = this.state
        return (
            <div>
                <h1>收藏状态的改变</h1>
                <button onClick={() => {
                    this.setState({
                        collectionState : !collectionState
                    })
                }}> {collectionState ? "🖤 收藏" : "❤️ 取消"} </button>
            </div>
        )
    }
}
```

### 构造器中初始化

```jsx
export default class App extends Component {

    constructor() {
        super() // 继承父类的内容
        this.state = {
            collectionState: true
        }
    }
    ...
    // 后续内容同上 
}
```

## state 渲染

### 列表的循环渲染

使用原生 js 的 map 方法: React 认为如无必要, 勿增实体
- 列表渲染到 template 中要指定 key 值, 通过 key 值能够快速 diff js 对象的变化, 因此 key 值要独一无二的表示目标对象, 不能是索引 (如果存在中间插入或删除的操作)
- template 中不要涉及到复杂逻辑, 复杂逻辑在后台代码/外部逻辑代码中处理完毕

```jsx
export default class App extends Component {

    state = {
        items : ["A", "B", "C", "D", "E"]
    }

    render() {
        // key 值用于 React 的 diff 算法, 需要独一无二的表示目标, 这里暂时的使用内容本身
        const items_info = this.state.items.map(item => <li key={item}>{item}</li>)
        return (
            <div>
                <h1>08-循环渲染.js</h1>
                <ul>
                    {
                        // 通常在外部处理完毕后, 让返回的内容中尽量不包含处理逻辑
                        // Django 视频中的分页处理也是类似的思想, template 中尽量避免复杂逻辑, 在后台代码中处理完毕
                        // this.state.items.map(item => <li>{item}</li>)
                        items_info
                    }
                </ul>
            </div>
        )
    }
}
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202301012055860.png)

### 合并渲染问题

React 18 之前同步时，setState 会进行合并的异步的渲染, 异步时不会合并并且同步顺序渲染

React 18 之后不论同步异步 setState 都会合并渲染，同步异步并不保证

# props

- `<Component props={value}>`: 组件定义属性接收的接口
- 自身的属性自身不能更改，只能通过父组件更改

## 属性类型验证

为了防止封装的组件被错误传入属性, 因此做好属性类型验证十分有必要

- 针对类组件, 通过类属性定义 `propTypes` 即可

Tips: `import prop from 'prop-types'` 为我们封装好了常见的属性验证方法

```jsx
// 类内: 类属性
static propTypes = {
    title: prop.string,
    leftshow: prop.bool
}

// 类外: 类属性
Navbar.propTypes = {
    title: prop.string,
    leftshow: prop.bool
}
```

## 属性默认值

针对类组件, 定义其类属性 `defaultProps`

```jsx
Navbar.defaultProps = {
    title: "Title", 
    leftshow: true,
}
```