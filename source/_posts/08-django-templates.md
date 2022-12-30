---
title: 08-django-templates
mathjax: false
date: 2022-12-30 18:21:52
summary: django 模板文件相关操作
categories: Django
tags:
  - python
---

# templates

在所有 app 目录下的 templates 文件夹中寻找
- [For more details](https://docs.djangoproject.com/en/4.1/topics/templates/)
- 项目目录下的 `settings.py` 中 `TEMPLATES` 列表中的 `DIRS` 属性中可以配置搜索根目录: `os.path.join(BASE_DIR, "templates")`
- 如果根目录中没有找到, 则再去各个 APP 中寻找
- 寻找顺序按照 APP 注册的先后, APP 内推荐目录结构为: `app_name/templates/app_name/template.html` 即多建一层以 APP 命名的目录, 以声明 template 文件的命名空间

## 传统渲染方式

```python
from django.template import loader
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]

    template = loader.get_template("polls/index.html")
    context = {
        "latest_question_list": latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```

## 快速渲染方式

使用 `django.shortcuts` 的 `render` 实现模板的快速渲染命令

```python
from django.shortcuts import render

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]

    context = { "latest_question_list": latest_question_list}
    return render(request, "polls/index.html", context)
```

# 模板语法

- 变量值: `{{ var }}`
- 数组索引: `{{ arr.index }}`
- 字典属性: `{{ dict.key }}`
- 日期: `{{ user.create_time | date:"Y-m-d H:i:s" }}`

## 循环

- 数组循环: 

```html
{% for item in arr %}
    <span>{{ item }}</span>
{% endfor %}
```

- 字典循环: 同 python, `dict.keys, dict.values, dict.items`, 就是不用加括号了

```html
{% for key, value in dict.items %}
    <span>{{ item }}</span>
{% endfor %}
```

## 条件

- if [elif] else

```html
{% if name == "admin" %}
    pass
{% elif name == "user" %}
    pass
{% else %}
    pass
{% endif %}
```

## 模板继承

- 根模板中: 配置好占位符

```html
{% block content %}{% endblock %}
```

- 子模板中: 继承根模板+ 填充占位符

```html
{% extends 'OA/layout.html' %}

...
{% block content %}
...
{% endblock %}
```

## 静态文件引入

django 将静态文件放置再 `app` 下的 `static` 目录下统一管理(配置文件中管理), 内部用户再自行创建文件夹进行规整

```html
{% load static %}
<link rel="stylesheet" href="{% static 'plugins/bootstrap-4.3.1-dist/css/bootstrap.css' %}">
<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
<script src="{% static 'plugins/bootstrap-4.3.1-dist/js/bootstrap.js' %}"></script>
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212052248658.png)

## 非硬编码 URL

在 urlpattern 中配置的 name 属性, 就可以用在模板语法中用于索引想要跳转的 url, 而不必再用拼接的形式, 如 `<a href="/polls/{{ question.id }}/">`

模板标签为 `{% url %}` 以空格为分隔符, 第一个参数表示目标 url 的 name, 后续参数为参数列表
```html
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

如果要传递 GET 参数则在模板后面拼接即可

```html
<a href="{% url 'OA:depart_delete' %}?depart_id={{depart.id}}" >删除</a>
```