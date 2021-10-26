---
title: Hexo博客中使用Latex配置方法
date: 2021-07-07 23:28:28
mathjax: false
summary: 在 Hexo 博文中使用 Latex 数学公式
categories: Ungrouped
tags:
  - method
---

本教程使用插件 `hexo-renderer-markdown-it-plus` 完成 Latex 公式渲染

GitHub: https://github.com/CHENXCHEN/hexo-renderer-markdown-it-plus

# 1 清除原有渲染插件

在博客根目录下，右键单击，选择 `Git Bash Here` 后依次输入命令

``` shell
npm uninstall hexo-renderer-marked
npm install hexo-renderer-kramed --save
```

# 2 安装指定插件

在博客根目录下，右键单击，选择 `Git Bash Here` 后输入命令

``` shell
npm i -S hexo-renderer-markdown-it-plus
```

# 3 修改配置文件

1. 在博客根目录下，编辑 `_config.yml` 在底部添加如下配置信息

``` yaml
markdown_it_plus:
  highlight: true
  html: true
  xhtmlOut: true
  breaks: false
  langPrefix:
  linkify: true
  typographer: false
  quotes:
  plugins:
    - plugin:
        name: markdown-it-mark
        enable: false
```

2. 在使用的主题文件根目录下，编辑 `_config.yml` 在 `<head>` 标签底部添加如下引入信息

``` ejs
<head>
    ...
    ...

    <% if (page.mathjax) { %>
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.11.1/dist/katex.min.css" integrity="sha384-zB1R0rpPzHqg7Kpt0Aljp8JPLqbXI3bhnPWROx27a9N0Ll6ZP/+DiW/UqRcLbRjq" crossorigin="anonymous">
        <script defer src="https://cdn.jsdelivr.net/npm/katex@0.11.1/dist/katex.min.js" integrity="sha384-y23I5Q6l+B6vatafAwxRu/0oK/79VlbSz7Q9aiSZUvyWYIYsd+qj+o24G5ZU2zJz" crossorigin="anonymous"></script>
    <% } %>
</head>
```

# 4 指定渲染博文

为了避免无公式博文引入渲染文件的性能浪费，因此执行了选择性引入

在后续书写博文时，在开头博文属性栏添加如下属性，已决定本文是否加载渲染文件

``` yaml
mathjax: false # 不引入
mathjax: true # 引入
```
