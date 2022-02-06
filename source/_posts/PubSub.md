---
title: PubSub
mathjax: false
date: 2022-02-06 19:35:37
summary: React 中消息订阅与发布技术
categories: React 
tags:
  - react
  - react tools
---
# 消息订阅与发布技术

```shell
yarn add pubsub-js
import PubSub from 'pubsub-js'
```

不必将信息一级一级的传递下去，可以实现任一组件之间通信

1、App 作为顶级 壳，不应该过多的涉及管理子组件之间的状态变化与通信问题，因此使用消息订阅与发布技术实现子组件之间的自发通信

Q：实现消息订阅与发布的库很多，你要选择哪个？

A：PubSub，历史悠久，应用广泛

## 何时订阅

A：在组件刚挂载上就要完成所有的订阅，一次性定义完成该组件所接收的所有消息类型。

```jsx
componentDidMount() {
    this.updateHeaderData = PubSub.subscribe('updateHeaderData', (msgName, data) => {
        this.setState(data)
    })
}
```

## 如何发送消息

A：引入 `PubSub` 后调用其 `publish` 方法

```jsx
PubSub.publish('updateHeaderData', {data: activeName})
```

## 如何取消订阅

A：引入 `PubSub` 后调用其 `unsubscribe` 方法，一般再组件卸载时调用

```jsx
componentWillUnmount() {
    // 取消订阅
    PubSub.unsubscribe(this.updateHeaderData)
}
```