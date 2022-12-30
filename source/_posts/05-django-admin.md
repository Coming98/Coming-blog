---
title: 05-django-admin
mathjax: false
date: 2022-12-12 18:43:13
summary: django 后台管理
categories: Django
tags:
  - python
---
# Admin

Django 自动生成后台的功能十分方便

1. 创建可以访问管理网页的用户: `python manage.py createsuperuser`
2. 启动服务, 进入 admin 界面

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212071825556.png)

3. 默认只存在对用户组和用户的管理, 因此需要引入我们创建的数据模型的管理: 在 app 的 admin.py 中实现引入

```python
from .models import Question
admin.site.register(Question)
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212071829788.png)

# 自定义后台表单

初始的后台可控数据十分少, 可以通过修改 APP 目录下的 admin.py 进行精细化控制

```python
from django.contrib import admin
from .models import Question, Choice

class QuestionAdmin(admin.ModelAdmin):
    # fields = ['pub_date', 'question_text']
    fields = ['question_text', ]

admin.site.register(Question, QuestionAdmin)
```

- 默认使用 `admin.site.register` 注册 Question 时会展示所有的域
- 可以通过继承 `admin.ModelAdmin` 用 `fields` 属性对展示的数据域进行精细化控制

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212121758371.png)

## 其它展示控制

### list_display

控制对象在以列表状态展示时展示的数据域

```python
class QuestionAdmin(admin.ModelAdmin):
    list_display = ('question_text', 'pub_date', 'was_published_recently')
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212121814795.png)

### 数据域控制

想要对某个[数据与进行精细的控制](https://docs.djangoproject.com/zh-hans/4.1/ref/contrib/admin/#django.contrib.admin.ModelAdmin.list_display), 可以在定义数据模型时对目标域添加修饰器
  -  boolean 表示该数据域为布尔类型, 以应用布尔数据的展示方式
  -  ordering 表示该数据域的排序依赖于 pub_date 数据域
  -  description 表示该数据域的列名

```python
@admin.display(
    boolean=True,
    ordering='pub_date',
    description='Published recently?'
)
def was_published_recently(self):
    now = timezone.now()
    return now - datetime.timedelta(days=1) <= self.pub_date <= now
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212121821518.png)

### list_filter

list_filter 列表中的属性将支持更加丰富的过滤规则支持
- Tips: 只支持数据库中存在的域哦, 不支持 was_published_recently 这种“函数域”

```python
list_filter = ['pub_date', 'question_text']
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212121828986.png)

### search_fields

search_fields 支持搜索框的索引: `search_fields = ['question_text']`

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212121830263.png)

## 划分字段集

当 Model 拥有较多字段时, 可以通过 ModelAdmin 中的 filedsets 进行字段集的划分

```python
class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date']}),
    ]

admin.site.register(Question, QuestionAdmin)
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212121758119.png)

## 添加关联对象

常见的就是作为外键, 可以添加其对另一 Model 的引用

1. 配置关联的目标对象: 指定其数据模型, `extra` 属性表示空数据留白, 供创建新的关联对象

```python
# admin.StackedInline: 栈类似的排列展现, 较占空间
# admin.TabularInline: 表格式的单行显示
class ChoiceInline(admin.StackedInline):
    model = Choice
    extra = 1
```

2. 将目标对象在主对象中使用 inlines 属性关联

```python
class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date']}),
    ]
    inlines = [ChoiceInline]
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212121806254.png)

- admin.TabularInline: 表格式的单行显示

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212121809641.png)

# 修改 Admin 模板

## Quick Start

1. 修改根目录 `settings.py` 为 `TEMPLATES` 的 `DIRS` 属性添加本地搜索路径 `"DIRS": [BASE_DIR / 'templates'],`
2. 在根目录创建 `templatex/admin/` 目录
3. 将位于 `django/contrib/admin/templates` 内的原始模板文件 `base_site.html` 复制到这个目录
4. 修改本地的 `base_site.html` 即可

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212121840630.png)

# Refs

- [Bilibili](https://www.bilibili.com/video/BV1S44y1K7Hd/?vd_source=b982c5b9804c7552564e69b7b5d8a2e0)