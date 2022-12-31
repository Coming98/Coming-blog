---
title: 01-React-Create React App
mathjax: false
date: 2022-12-31 18:57:23
summary: React 脚手架相关介绍
categories: React
tags:
  - react
---

# scaffold

脚手架, 基于 WebPack, 用于快速创建一个模板项目, 包含了需要的配置（语法检查, jsx编译等）, 下载好了相关依赖, 可以直接运行

架构：`react  webpack  es6  eslint`

特点：模块化, 组件化, 工程化

## Quick Start

0、配置淘宝 npm 镜像

```shell
npm config set registry https://registry.npm.taobao.org
```

1、全局安装 `create-react-app`

```shell
npm i -g create-react-app
```

2、切换到项目目录, 创建项目: 不能含有大写字母

```shell
create-react-app 01hello-react
```

主要安装了三样东西:
- react: react 顶级库
- react-dom: web 上的 react 为 react-dom
- react-scripts: 包含运行和打包 react 应用程序的所有脚本和配置

3、启动项目

```shell
cd 01hello-react
npm start
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212311830537.png)

4. Demo 页面

```jsx
import React from 'react'
import ReactDOM from 'react-dom' // 将 React 组建渲染到页面中

// render 方法将组件渲染并构造 DOM 树并插入到页面上某个特定的元素上
ReactDOM.render(
    <h1>Hello World</h1>, // 组件内容, jsx 语法直接支持 HTML 的编写
    document.getElementById('root')
)
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212311832715.png)


# 基本架构

- node_modules: 所有的依赖安装的目录
- public: 静态公共目录
  - `public\index.html`: 主页面
  - `public\robots.txt`: 爬虫协议文件
- src: 开发用的源代码目录
  - `src\index.js`: 入口文件
  - `src\App.js`: 根组件, 后续组件都为其子组件
  - `src\App.test.js` - 测试 App 接口 - 了解即可
- hooks: 定义自定义的函数式组件的 Hook, 实现封装与复用

## package-lock.json 

package.json 则只提供了所用依赖包的大致版本信息, package-lock.json 则提供本项目所用到依赖包的具体版本信息以及依赖包的下载地址

Application：因为一个项目大就大在依赖包（静态资源可以云存储）, 因此项目转移时可以放弃依赖包的转移, 只转移代码等数据文件, 然后使用 `npm i` 这个命令, 通过读取 package-lock.json 实现依赖包的获取

## index.html

`public/index.html`, 主页面, 

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="./css/bootstrap.css">
    <title>Document</title>
</head>
<body>
    <div id="root"></div>
</body>
</html>
```

## 静态资源

public 中还会有 css 文件夹存储一些静态的 css, 通过 link 标签导入即可（因为我们做的是 SPA, 单页面应用）

导入时要注意静态样式丢失问题（常见于路由中）因此导入时不推荐使用相对路径, 而是使用相对于项目的绝对路径

```html
<link rel="stylesheet" href="%PUBLIC_URL%/css/bootstrap.css">
```

## App.jsx

`src/App.jsx`, 根组件, 后续组件都为其子组件

```jsx
// 并非结构赋值, 这是在暴露时选择了 分别暴露的形式
/*
export default React = ....
export const Component = ...
*/
import React, { Component } from 'react'

export default class App extends Component {
    render() {
        return (
            <div>
                
            </div>
        )
    }
}
```

## index.js

`src/index.js`, 入口文件, 将根组件渲染到页面

Tips: 为了保持适用性, 推荐使用严格模式

```javascript
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App'

ReactDOM.render(<React.StrictMode><App/></React.StrictMode>, document.getElementById('root'))
```

## components

`src/components/`, 各类小组件, 由 `index.jsx` 与 `index.css` 组成

导入时：`import componentsName from '.../src/components/componentName'`
