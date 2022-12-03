---
title: 01-django-init
mathjax: false
date: 2022-12-03 23:25:47
summary: django 的安装与基本使用
categories: Django
tags:
  - python
  - INIT
---

# 安装

```python
pip install django
```

主要关注两个文件夹：
- `Scripts/django-admin.exe`: 脚手架, 自动化创建 django 项目
- `site-packages/django`: 源码文件, 引用使用

# APP

TODO: 后续可以研究多 APP 项目

使用 APP 进行后台各功能的划分, 每个 APP 中可以拥有独立的表结构、函数、HTML 模板、CSS 等
- 创建 APP: `python manage.py startapp appName`

# Quick Start

1. 创建项目: `django-admin startproject projectName`

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212032256125.png)

- `manage.py`: 项目的管理，启动项目、创建 APP、数据管理
- `asgi.py` 与 `wsgi.py`: 接收网络请求, 不用更改
- `urls.py`: 管理 url 与 func 的对应关系
- `settings.py`: 项目的配置文件, 数据库管理、注册 APP

2. 创建 APP: `python manage.py startapp appName`

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212032307366.png)

- `models.py`: 对数据库的操作
- `view.py`: func 相关的定义

3. 注册 APP: 在主项目中 `settings.py` 中的 `INSTALLED_APPS` 列表中注册; `appName.apps.AppConfigName`

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212032314149.png)

4. 编写 URL 与 视图函数的对应关系：在主项目中 `urls.py` 中的 `urlpatterns` 列表中编写; `path('url/', views.index)`

```python
from app import views

urlpatterns = [
    # path("admin/", admin.site.urls),
    path("index/", views.index),
    path("", views.default),
]
```

5. 编写函数体：在相关 `app` 下的 `views.py` 中

6. 启动 APP: `python manage.py runserver [port]`

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212032323367.png)


# Refs

- [Bilibili](https://www.bilibili.com/video/BV1S44y1K7Hd/?vd_source=b982c5b9804c7552564e69b7b5d8a2e0)