---
title: github pages 个人博客搭建部署记录
date: 2021-04-29 14:48:12
summary: 记录参考路线，维护命令以及个性化修改
categories: Github
tags:
  - git
  - github
---

基础版的 Github Pages + Hexo + travis ci 实现个人博客配置

即使是在众多大佬博客的帮助下，整体配置过程也有些艰辛，让我最头疼的就是网络问题，感觉给 git 配了代理，改了 hosts 也没有什么用

回来了 - 2021年7月5日23:53:13

# Reference

**Tips**：注意主路线教程中使用的是 travis-ci.org, 这个网站马上就要关闭了，其功能迁移到了 [travis-ci.com](https://travis-ci.com/)

## 主路线

【Hexo】使用Hexo+github pages+travis ci 实现自动化部署 - [HERE](https://www.cnblogs.com/mfrank/p/12829882.html)

【Hexo】自定义 Hexo 配置文件 - [HERE](https://www.cnblogs.com/mfrank/p/12830094.html)

【Hexo】Hexo 主题 Matery 配置 - [HERE](https://www.cnblogs.com/mfrank/p/12830097.html)

## 辅助路线

主路线使用的主题为 `matery` 其 官方配置地址 (Github Address) - [HERE](https://github.com/blinkfox/hexo-theme-matery/blob/develop/README_CN.md)

比较好看的一个最终话风格 - [HERE](https://matjenin.gitee.io/index.html)

在一些配置文件等遇到的特殊字符转义问题  - [HERE](https://wxnacy.com/2018/01/12/hexo-specific-symbol/)

HEXO 官方主题库 - [HERE](https://hexo.io/themes/)

HEXO 官方ICON库 - [HERE](https://fontawesome.com/icons?d=gallery&p=2)

YAML语言简版教程，便于理解和自定义 `*.yml` [HERE](http://www.ruanyifeng.com/blog/2016/07/yaml.html)

LiveRe 留言评论功能配置 - [HERE](https://starslove.me/2020/07/08/Hexo-comment/)

高水平插件 - [HERE](https://blog.csdn.net/q2158798/article/details/82354154)

# Hexo 项目目录

## Init

实现对目标目录的初始化

``` shell
# blogname - 为博客项目准备的目录名
hexo init blogname
cd blogname
nup install
```

## Catalog

`_config.yml` -

`package.json` - 是应用程序信息，通常不需要关心

`node_moudles` - 用来存放 node 相关的模块，通常不需要关心

`scaffolds/` - 新建文章时的模板文件

`source/` - 资源文件夹，存放用户资源

`theme/` - 主题文件夹，`_config.yml` 将会在此定位使用的主题

# Quick Start

记录日常配置中常用的命令信息

## 首次初始化

~~~shell
git init # 初始化本地库
git remote add origin github仓库地址 # 创建别名
git push -u origin master # 初次提交到目标库 - 记录本分支提交的默认参数
~~~

## 修改提交

~~~shell
# 修改提交到 `github`

cd blogname
git checkout master # 使用master分支
hexo new "new blogname" # 新建一遍博文
git add .
git commit -am "message"
git push origin master
~~~

## 本地预览

注意：配置了 `travis ci` 会自动检测到博客项目的变更，帮助我们重新部署，因此 `修改提交` 中不需要重新生成静态文件

~~~shell
hexo clean # 清除缓存和已生成的静态文件
hexo generate # 生成静态文件
hexo server # 使用静态文件，在本地4000端口进行预览

hexo c & hexo g & hexo s # Magic - 达到上述三条命令的效果
~~~

## 删除博文

也没有查阅到相关命令，直接在本地删除后就可行了

最好执行一次 `hexo c & hexo g & hexo s` 本地先看看，然后在提交仓库确认删除操作

# git push 相关问题

OpenSSL SSL_read: Connection was reset, errno 10054

~~~shell
git config --global http.sslVerify "false"
~~~

Failed to connect to github.com port 443:connection timed out

~~~shell
git config --global http.proxy http://127.0.0.1:1080
git config --global https.proxy http://127.0.0.1:1080
~~~

上述设置后应该就ok了，如果再次出现问题，取消上述代理设置应该就 ok 了

什么原理(⊙o⊙)…

~~~shell
git config --global --unset http.proxy
git config --global --unset https.proxy
~~~

# 很小的问题

1. 在 `_config.yml` 中配置标题，如果想使用 `’` 可以使用双引号括起来，在 `yaml` 语言中 双引号不会对特殊字符转义。
2.
