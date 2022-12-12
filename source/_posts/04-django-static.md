---
title: 04-django-static
mathjax: false
date: 2022-12-12 18:43:06
summary: 静态文件管理
categories: Django
tags:
  - python
---

# Static Files

## Quick Start

1. 在 APP 目录下创建 static 目录
2. 为了在引用时区分 APP 在 static 目录下再创建名为 APP 的目录

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212121744138.png)

3. 在 APP 目录中管理该 APP 的静态文件
4. 在模板文件中引入静态文件

```html
{% load static %}

<link rel="stylesheet" href="{% static 'polls/style.css' %}">
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212121746667.png)


# Refs

- [Bilibili](https://www.bilibili.com/video/BV1S44y1K7Hd/?vd_source=b982c5b9804c7552564e69b7b5d8a2e0)