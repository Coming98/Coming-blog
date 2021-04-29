---
title: github pages 个人博客搭建部署记录
date: 2021-04-29 14:48:12
summary: 整理参考路线，常用维护命令以及一些个性化修改的记录
categories: Github
tags:
  - git
  - github
---
基础版的 Github Pages + Hexo + travis ci 实现个人博客配置

即使是在众多大佬博客的帮助下，整体配置过程也有些艰辛，让我最头疼的就是网络问题，感觉给 git 配了代理，改了 hosts 也没有什么用

# Reference

**Tips**：注意主线路教程中使用的是 travis-ci.org, 这个网站马上就要关闭了，其功能迁移到了 [travis-ci.com](https://travis-ci.com/)

**主线路**

【Hexo】使用Hexo+github pages+travis ci 实现自动化部署 - [HERE](https://www.cnblogs.com/mfrank/p/12829882.html)

【Hexo】自定义 Hexo 配置文件 - [HERE](https://www.cnblogs.com/mfrank/p/12830094.html)

【Hexo】Hexo 主题 Matery 配置 - [HERE](https://www.cnblogs.com/mfrank/p/12830097.html)



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
git push -u origin master # 初次提交到目标库
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


