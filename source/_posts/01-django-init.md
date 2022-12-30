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

安装 django: `pip install django`

验证 django 版本: `python -m django --version`

主要关注两个文件夹：
- `Scripts/django-admin.exe`: 脚手架, 自动化创建 django 项目
- `site-packages/django`: 源码文件, 引用使用

# APP

TODO: 后续可以研究多 APP 项目

使用 APP 进行后台各功能的划分, 每个 APP 中可以拥有独立的表结构、函数、HTML 模板、CSS 等
- 创建 APP: `python manage.py startapp appName`

The difference between a project and an app:
- An app is a web application that does something – e.g., a blog system, a database of public records or a small poll app
- A project is a collection of configuration and apps for a particular website. 
- A project can contain multiple apps. An app can be in multiple projects.

# Quick Start

1. 创建项目: `django-admin startproject projectName`; 项目名称后续可以任意更改(you can rename it to anything you like)

```html
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```

- [manage.py](https://docs.djangoproject.com/en/4.1/ref/django-admin/): 项目的管理，启动项目、创建 APP、数据管理
- [settings.py](https://docs.djangoproject.com/en/4.1/topics/settings/): 项目的配置文件, 数据库管理、注册 APP
- [urls.py](https://docs.djangoproject.com/en/4.1/topics/http/urls/): 管理 url 与 func 的对应关系
- [asgi.py](https://docs.djangoproject.com/en/4.1/howto/deployment/asgi/) 与 [wsgi.py](https://docs.djangoproject.com/en/4.1/howto/deployment/wsgi/): 接收网络请求, 不用更改

2. 创建 APP: `python manage.py startapp appName`

```html
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```

- `models.py`: 对数据库的操作
- `view.py`: func 相关的定义

3. 注册 APP: 在主项目中 `settings.py` 中的 `INSTALLED_APPS` 列表中注册; `appName.apps.AppConfigName`

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212071742844.png)

4. 编写 URL 与 视图函数的对应关系：在 app 中 `urls.py`（自行新建） 中的 [urlpatterns](https://docs.djangoproject.com/en/4.1/topics/http/urls/) 列表中编写

```python
from django.urls import path

from . import views

urlpatterns = [
    path("", views.index, name="index")
]
```

5. 编写函数体：在相关 `app` 下的 `views.py` 中

6. 在主项目中引入 app 的路由信息

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("polls/", include("polls.urls"))
]
```

- the path() function is passed four arguments, two required: route and view, and two optional: kwargs, and name
  - route: 待匹配的 URL pattern, 按照注册顺序从上到下匹配
  - view: 匹配成功后调用的函数
  - kwargs: 其它参数, 后续介绍...
  - name: TODO, 还不大清楚其作用, Naming your URL lets you refer to it unambiguously from elsewhere in Django, especially from within templates. This powerful feature allows you to make global changes to the URL patterns of your project while only touching a single file.
- The include() function allows referencing other URLconfs. Whenever Django encounters include(), it chops off whatever part of the URL matched up to that point and sends the remaining string to the included URLconf for further processing.

7. 启动 APP: `python manage.py runserver [port]`

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212032323367.png)

> don’t use this server in anything resembling a production environment. It’s intended only for use while developing. 
> We’re in the business of making web frameworks, not web servers.

# Django Shell

使用 `python manage.py shell` 打开与当前项目的交互, 可以通过 django 内置的 API 进行功能测试

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212071757229.png)

## Database API

1. 引入数据模型: `from polls.models import Choice, Question`
2. 创建数据条目: `q = Question(question_text="What's new?", pub_date=timezone.now())`
3. 将条目保存到数据库中: `q.save()`
4. 将条目删除: `q.delete()`
5. 根据数据模型查看/更改属性值(更改后要 save)

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212071800325.png)

5. 查看表中的所有信息: `Question.objects.all()`; 如果要修改展示效果, 可以重写数据模型中的 __str__ 方法

```python
def __str__(self) -> str:
    return self.question_text
```

重启一下 shell 即可看到效果

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212071809453.png)

6. 表支持过滤与筛选: 
- `Question.objects.filter(id=1)`
- `Question.objects.filter(question_text__startswith='What')`
- `Question.objects.get(pub_date__year=timezone.now().year)`
- 通过主键筛选是较为常用的, 因此简化了改命令: `Question.objects.get(pk=1)`

7. 在 Django 中会自动为外键的对象创建一个集合, 用于存放目标数据条目, 例如 Question_id 是 Choice 的外键, 因此每一个 question 对象都会有一个 choice_set 属性

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212071818369.png)

8. 为这个集合添加数据就能自动作用到 Choice 表中

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212071821430.png)


# Refs

- [Bilibili](https://www.bilibili.com/video/BV1S44y1K7Hd/?vd_source=b982c5b9804c7552564e69b7b5d8a2e0)