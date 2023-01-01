---
title: 08-React-类组件生命周期
mathjax: false
date: 2023-01-01 22:51:54
summary: 类组件声明周期简单介绍
categories: React
tags:
  - react
---
# 初始化

## componentWillMount

因为该周期可有可无, 不再推荐使用：只会执行一次, 在组件将要渲染前执行, 无法访问到组件节点, 可以进行状态的访问与修改
- 如果依旧要用可以使用 `UNSAFE_componentWillMount` 忽略警告

## render

每次渲染时执行, 不能够进行状态修改

## componentDidMount

只会执行一次, 组件完成渲染后执行, 能访问到组件节点

Application:
1. 开启定时器
2. ajax 请求
3. 消息订阅等初始化的事务
4. 基于 DOM 进行初始化 - BetterScroll

# 运行中

## componentWillReceiveProps

父组件修改属性触发子组件更新时, 子组件将执行本函数
- 接受 `nextProps` 参数用于对比更新前后的父组件传来的属性值, 可以针对属性值做些逻辑处理, 将处理后的结果与子组件的状态进行绑定
- 不能直接修改 `props` 与 `state`

Application:
1. 属性转为状态
2. 根据属性进行逻辑处理

## shouldComponentUpdate

能访问到更新前的状态, 接受两个参数 `nextProps, nextState` 这里是新旧状态的记录, 也是我们不能直接修改原状态的原因
- 返回 `false` 会阻止 `render` 调用
- 能够进行性能优化, 提升性能
- `state` 更新后的值与之前的值相同时可以阻止不必要的 `render` 调用
- 两个对象判断是否相等可以利用 `JSON.stringify(obj)` 来实现

```jsx
shouldComponentUpdate(nextProps, nextState) {
    // 现在并未更新 this.state 是老状态
    if(this.state.username === nextState.username) {
        return false
    }
    return true
}
```

为了更自动的实现不变的状态进行阻塞更新, 引入了 `PureComponent` 它会自动对比新旧 `props` 与 `state` 决定 shouldComponentUpdate 返回 true 或者 false
> 当然 props 和 state 不能经常变, 否则性能并没有优化太多

```jsx
import React, { Component, PureComponent } from 'react'

export default class App extends PureComponent
```

## componentWillUpdate

组件将要更新, 获取的是更新前的 `DOM`, 不能修改属性和状态

不再推荐使用, 它处于调度阶段的第一个阶段, 能够被高优先级任务打断, 因此不够安全

## render

只能访问 `this.props` 和 `this.state` 不允许修改状态和 DOM 输出
  
## componentDidUpdate

可以访问更新后的 DOM 状态, 可以用于 Betterscroll

接收两个参数 `prevProps` 与 `prevState` 表示更新前的状态与属性, 用于筛选更新内容, 防止不必要的渲染
> 比如之前的状态已经有数据了, 那么就不必要初始化 Betterscroll 了

# 销毁

## componentWillUnmount

组件中涉及 window 窗口, 定时器等全局的监听事件需要在组件销毁时一并清理

```javascript
// 需要由 ReactDOM 显式调用
unloadomponent = () => {
    ReactDOM.unmountComponentAtNode(document.getElementById('demo'))
}
componentWillUnmount() {
    console.log('componentWillUnmount')
}
```

# 新生命周期

新老生命周期不能同时存在

## getDerivedStateFromProps

用于替代 `componentWillMount`: 第一次初始化组件以及后续的更新过程中(包括自身状态更新以及父传子), 返回一个对象作为新的 state, 返回 null 则说明不需要在这里更新 state

```jsx
static getDerivedStateFromProps(nextProps, nextState) {
    // 在这里因为丢失了 this 推荐只进行将属性转为状态
    // 并且该声明周期会频繁调用
    return {
        type: nextProps.type
    }
}
```

然后借助 `componentDidUpdate` 完成更新

```jsx
componentDidUpdate(prevProps, prevState) {
    // 这里面如果 setState 又会引起 getDerivedFromProps 更新, 需要注意
    if(this.state.type === prevState.type) {
        return
    }
    
    if(this.state.type == 1) {
        console.log("请求卖座正在热映的数据")
        axios({
            url:"https://m.maizuo.com/gateway?cityId=110100&pageNum=1&pageSize=10&type=1&k=6369301",
            headers:{
                'X-Client-Info': '{"a":"3000","ch":"1002","v":"5.2.0","e":"16395416565231270166529","bc":"110100"}',
                'X-Host': 'mall.film-ticket.film.list'
            }
        }).then(res=>{
            console.log(res.data.data.films)
            this.setState({
                list:res.data.data.films,
            })
        })
    } else {
        console.log("请求卖座即将上映的数据")

        axios({
            url:"https://m.maizuo.com/gateway?cityId=110100&pageNum=1&pageSize=10&type=2&k=8077848",
            headers:{
                'X-Client-Info': '{"a":"3000","ch":"1002","v":"5.2.0","e":"16395416565231270166529","bc":"110100"}',
                'X-Host': 'mall.film-ticket.film.list'
            }
        }).then(res=>{
            console.log(res.data.data.films)
            this.setState({
                list:res.data.data.films,
            })
        })
    }
}
```

## getSnapshotBeforeUpdate

用于代替 `componentWillUpdate` 触发时间为 update 发生的时候, 在 render 之后 dom 渲染之前返回一个值, 作为 componentDidUpdate 的第三个参数