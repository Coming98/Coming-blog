---
title: 04-React-CrossDomain
mathjax: false
date: 2022-12-31 22:49:22
summary: React 跨域问题的解决方案
categories: React
tags:
  - method
---


# 跨域问题

简单讲就是当前所处位置与目标位置不同源（域）, 由协议, 主机名和端口号决定

# 配置代理解决

在 package.json 中追加如下配置, 但是不能配置多个代理, 当用 ajax 请求了本地不存在的资源, 将会转发给目标代理

```shell
"proxy":"http://localhost:5000"
```

# 多代理配置

借助中间件 `http-proxy-middleware` 进行配置, 配置完毕后需要重启服务器，因为启用了反向代理，所以在请求 `url` 时不要带有域名，反向代理会自动添加代理的域名

1. 在 src 下创建配置文件：`src/setupProxy.js`
2. 单个代理配置

```js
const createProxyMiddleware = require('http-proxy-middleware')

module.export = function(app) {
    app.use(
    	'/help', // 表示只有 /help 开头的请求才转发给代理服务器 5000
        createProxyMiddleware({
            target: 'http://localhost:5000',
            changeOrigin: true, // 将自己的源改为代理服务器的源, 主要是改 host
            pathRewrite: {'^/help': ''} // 以正则表达式的方式替换前缀为空
        })
    )
}
```

3. 多个代理配置

```javascript
const createProxyMiddleware = require('http-proxy-middleware')

module.export = function(app) {
    app.use(
    	proxy('/help', {
            target: 'http://localhost:5000',
            changeOrigin: true,
            pathRewrite: {'^/help': ''}
        }),
    	proxy('/help2', {
            target: 'http://localhost:6000',
            changeOrigin: true,
            pathRewrite: {'^/help2': ''}
        }),
    )
}
```