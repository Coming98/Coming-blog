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

```jsx
// 1、定义一个函数式组件
function Demo() {
    console.log(this) // undefined: 经过 babel 翻译开起来严格模式，从而禁止了自定义 this 指向 window，所以指向 undefined
    return <h2>函数式组件</h2>
}
// 2、渲染组件到页面
// <Demo/> - 组件标签，首先解析组件标签，寻找 Demo 组件的位置；直接调用 Demo 函数，将返回的虚拟 DOM 渲染到页面
ReactDOM.render(<Demo/>, document.getElementById('test'))
```

## 类定义组件

```jsx
// 1、定义一个类式组件
class Demo extends React.Component {
    render() {
        return <h2>类式组件</h2>
    }
}
// 2、渲染到页面
ReactDOM.render(<Demo/>, document.getElementById('test'))
/*
    1、React 解析组件标签，寻找目标组件的定义位置
    2、类定义组件：React 创建实例对象，调用 render() 方法
*/
```

# 组件核心属性

## state

### 企业版本

```jsx
// 忽略构造器，直接初始化 state 属性
state = {isHot: true}

// 添加工具函数
// 使用箭头函数默认没有 this, 使用 this 时自动寻找上层的 this
// 组件类中程序员定义的事件回调，必须写成赋值语句 + 箭头函数
changeWeather = () => {
    // state 不可以直接修改，要使用 API 间接修改 - setState
    const isHot = !this.state.isHot
    this.setState({isHot:isHot})
}
render() {
    return <h2 onClick={this.changeWeather}>今天天气很{this.state.isHot ? '炎热' : '凉爽'}！</h2>
}
```

### 繁琐版本

在构造器中设置 `state` 属性

```jsx
constructor(props) {
    super(props)
    this.state = { isHot : false}
    // 添加实例方法，使得后续执行中 this 指向没问题（如果不使用箭头函数的话）
    this.changeWeather = this.changeWeather.bind(this)
}
```

`state` 不可以直接修改，要使用 API 间接修改 - `setState`

```jsx
render() {
    return <h2 onClick={this.changeWeather}>今天天气很{this.state.isHot ? '炎热' : '凉爽'}！</h2>
}

changeWeather() {
    // state 不可以直接修改，要使用 API 间接修改 - setState
    const isHot = !this.state.isHot
    this.setState({isHot:isHot})
}
```

## props

结构化赋值

```jsx
const {name, sex, age} = this.props
```

### 类组件中使用

在渲染时传入参数由 `props` 接收

```jsx
ReactDOM.render(<Person name="cjc" sex="男" age={16} />, document.getElementById('person1'))
ReactDOM.render(<Person {...data.person1} />, document.getElementById('person1'))
```

通过添加类属性 `propTypes` 与 `defaultProps` 对属性进行限制

```jsx
static propTypes = {
    name: PropTypes.string.isRequired,
    age: PropTypes.number,
    sex: PropTypes.string
}

static defaultProps = {
    age: 18
}
```

### 函数组件中使用

定义函数，传参为 `props`

```jsx
function Person(props) {
    const {name, sex, age} = props
    return (
        <ul>
            <li>姓名:{name}</li>
            <li>性别:{sex}</li>
            <li>年龄:{age}</li>    
        </ul>
    )
}
```

给函数定义属性，限制 `props`

```jsx
Person.propTypes = {
    name: PropTypes.string.isRequired,
    age: PropTypes.number,
    sex: PropTypes.string
}

Person.defaultProps = {
    age: 18
}
```

## ref

### 字符串形式 ref

<font color='red'>不再推荐</font>

执行回调函数时通过 ref 获取目标

```jsx
<input type="text" ref="clickInput"/>&nbsp;
<button onClick={this.clickShow}>点击提示</button>&nbsp;
<input type="text" onBlur={this.blurShow} ref="blurInput"/>
```

通过结构化赋值，语义性良好

```jsx
clickShow = () => {
    // 结构化赋值
    // const {refs:{clickInput:{value}}} = this // 语义性不好
    const {clickInput} = this.refs
    console.log(clickInput.value)
}

blurShow = () => {
    const {blurInput} = this.refs
    console.log(blurInput.value)
}
```

### 回调形式 ref

<font color='red'>不再推荐</font>

React 调用该函数（箭头函数），默认传参为目标节点

箭头函数的 this 向外找，即可找到 类组件，即可记录该目标节点到类组件上

```jsx
<input type="text" ref={(item)=> {this.clickInput = item}}/>&nbsp;
<button onClick={this.clickShow}>点击提示</button>&nbsp;
<input type="text" onBlur={this.blurShow} ref={(item)=> {this.blurInput = item}} placeholder="失去焦点提示"/>         
```

函数内使用

```jsx
clickShow = () => {
    const {clickInput} = this
    console.log(clickInput.value)
}

blurShow = () => {
    const {blurInput} = this
    console.log(blurInput.value)
}
```

### create 形式

1、类中 cerate ref 的容器 container

```jsx
clickContainer = React.createRef()
```

2、调用时 ref 指定容器

```jsx
<input type="text" ref={this.clickContainer}/>&nbsp;
```

3、函数内通过容器获取目标组件

```jsx
clickShow = () => {
    const item = this.clickContainer.current
    console.log(item.value)
}
```

# 组件生命周期

## 常用声明周期

![image-20210928220114057](https://gitee.com/Butterflier/pictures/raw/master/image-20210928220114057.png)

**componentDidMount**: 只执行一次，开启定时器，ajax 请求，消息订阅等初始化的事务

**componentDidUpdate**: 完成更新后执行，用户控制页面显示等操作

**componentWillUnmount**: 关闭定时器，取消订阅等收尾事务
```javascript
// 需要由 ReactDOM 调用
unloadomponent = () => {
    ReactDOM.unmountComponentAtNode(document.getElementById('demo'))
}
componentWillUnmount() {
    console.log('componentWillUnmount')
}
```
## 新生命周期（全）

![image-20210928215958508](https://gitee.com/Butterflier/pictures/raw/master/image-20210928215958508.png)

**getDerivedStateFromProps**: 不常见，有 bug，不推荐

```jsx
static getDerivedStateFromProps(props, state)
```

通过 props 完成 state 的初始化，但是 setState 后也会执行他，因此当 state 完全由 初始化参数决定时才会使用这个 hook

**getSnapshotBeforeUpdate**: 不常用，必须与 componentDidUpdate 同时使用，必须返回一个 snapshotValue 值 或 null

应用场景：用户在阅读/观看聊天记录时，有新内容来时，滚动条位置不变，需要在更新前拿到老的页面高度 以得到新内容的高度

```javascript
getSnapshotBeforeUpdate() {
    return this.refs.container.scrollHeight // old scroll height
}

componentDidUpdate(prevProps, prevState, snapshotValue) {
    const {container} = this.refs
    container.scrollTop += container.scrollHeight - snapshotValue
}
```

## 旧生命周期

![image-20210927203306313](https://gitee.com/Butterflier/pictures/raw/master/image-20210927203306313.png)

左侧线路表示初次挂载：constructor  componentWillMount  render  componentDidMount

右侧线路：

1、更新组件信息：shouldComponentUpdate() componentWillUpdate()  Render()  componentDidUpdate()

> setState() 是导火索，回调函数是 shouldComponentUpdate() 
>
> shouldComponentUpdate 是阀门，如果返回 false 则后面不会执行  

2、强制更新组件：forceUpdate()  componentWillUpdate()  componentDidUpdate()

> 绕过了阀门 valve

3、父子组件传递（非首次）：componentWillReceiveProps()  


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
