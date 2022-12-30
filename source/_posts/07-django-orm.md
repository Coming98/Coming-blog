---
title: 07-django-orm
mathjax: false
date: 2022-12-30 18:21:46
summary: django 数据库相关操作
categories: Django
tags:
  - python
---
# ORM

将数据模型的相关操作翻译为目标数据库语句的框架
- 创建、修改、删除数据库中的表
- 操作表中的数据: 增删改查

## 创建表

遍历注册的 APP 针对 APP 的 models.py 进行翻译与执行
1. 实现 APP 的注册
2. `python manage.py makemigrations`
3. `python manage.py migrate`

# Models

继承 `django.db.models.Model` 即可创建数据模型

## Types

```python
from django.db import models

name = models.CharField(max_length=32)
age = models.IntegerField(default=0)
content = models.IntegerField(null=True, blank=True) # 表示该值可以为空
department = models.ForeignKey(to="Department", to_field="id", on_delete=models.CASCADE)
```

- 外键在使用时可以引用其所引用的一行数据, 因此在做展示时可以使用 `user.department.title` 获取其内部属性

## CRUD

- Model.objects.create(**value_dict)
- Model.objects.filter(id=1).first().name
- Model.objects.all().update(password=666)
- Model.objects.all().delete()
- Model.objects.all().order_by('-field_name')
- Model.objects.filter(id__gt=12) # 大于
- Model.objects.filter(id__gte=12) # 大于等于
- Model.objects.filter(id__lt=12) # 小于
- Model.objects.filter(id__lte=12) # 小于等于
- Model.objects.filter(content__startswith="abc") # 以 abc 开头
- Model.objects.filter(content__endswith="abc") # 以 abc 结尾
- Model.objects.filter(content__contains="abc") # 包含 abc 
- 

# Databases

By default, the configuration uses SQLite which is included in Python.
- Want [other database](https://docs.djangoproject.com/en/4.1/topics/install/#database-installation)

其配置信息在 settings.py 中进行配置

## sqlite

- ENGINE: 选择数据库引擎
- NAME: 设定数据库名称(地址)

```python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db.sqlite3",
    }
}
```

## Mysql

1. 安装 mysqlclient: `pip install mysqlclient`
2. settings.py 中配置 mysql 数据库的连接

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'dbname',
        'USER': 'root',
        'PASSWORD': 'pwd',
        'HOST': '127.0.0.1',
        'PORT': 3306
    }
}
```