---
title: 02-django-views
mathjax: false
date: 2022-12-12 16:45:07
summary: urls 与 views 相关处理
categories: Django
tags:
  - python
---

# views

- [For mor detail](https://docs.djangoproject.com/en/4.1/topics/http/urls/)

在 APP 的 views.py 中书写相关的 view 逻辑, 需要实现：
- url 与 view 的匹配
- view 的渲染返回

> Your view can read records from a database, or not. 
> It can use a template system such as Django’s – or a third-party Python template system – or not. 
> It can generate a PDF file, output XML, create a ZIP file on the fly, anything you want, using whatever Python libraries you want.

# urlpatterns

位于 urls.py 中, 实现 url 与 view 的匹配

## 父域

- 支持 APP 内的维护: 使用 include 在根 urls.py 中导入即可

```python
from django.urls import path, include
urlpatterns = [
    path("admin/", admin.site.urls),
    path("oa/", include("OA.urls")),
]
```

## 子域

- app 内部 urls.py 需要完善: app_name, urlpatterns 属性
- app_name 属性的设置用于外部引用非硬编码 url 时指明命名空间
- name 属性方便外部引用目标 url
- 支持占位符匹配:
```python
from django.urls import path
from . import views

app_name = "OA"
urlpatterns = [
    path("", views.welcome, name="index"),
    path("depart_list", views.depart_list, name="depart_list"),
    path("depart_list/add/", views.depart_add, name="depart_add"),
    path("depart_list/<int:depart_id>/edit/", views.depart_edit, name="depart_edit"),
    path("depart_list/delete/", views.depart_delete, name="depart_delete"),
]
```

在 template 中引用时, 指定命名空间 `app_name`

```html
<a href="{% url 'OA:depart_edit' depart.id %}">
```

# Exception

view 处理时可以显示的 raise an exception, 最常见的就是 404...

```python
from django.http import Http404

def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212112306273.png)

## 404 shortcuts

It’s a very common idiom to use `get()` and raise `Http404` if the object doesn’t exist. 因此 django 也为此提供了更加简便的封装

```python
from django.shortcuts import render, get_object_or_404

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212112309367.png)


# Generic Views

Q: 有些 View 的功能十分普遍, 例如展示一个列表的数据, 或者展示一个 Object 的详细信息, 针对这类 View 其后台操作基本一致: 获取 object 然后渲染模板页面, 因此可以进一步进行封装

A: Generic views abstract common patterns to the point where you don’t even need to write Python code to write an app. [For more details...](https://docs.djangoproject.com/en/4.1/topics/class-based-views/)

本质上就是缩减了后台与数据库交互的代码, 展示的模板代码基本上还是需要靠自己的..., 我原以为也能有一套默认的模板进行 generic 的展示

## Quick Start

1. 将能够优化的 View 封装为 generic 的子类

```python
# ListView: display a list of objects
# 通过 get_queryset 获取要展示的列表
# 通过 context_object_name 配置模板中对应的列表属性名称
class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]

# DetailView: display a detail page for a particular type of object
# 通过 model 属性确定要展示的 object
# 默认的寻找目标对象的条件为主键 pk, 因此需要通过 GET 传入
class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'
```

2. 更改 urlpattern, 注意 pattern 中的参数要去匹配 generic 中的参数配置

```python
urlpatterns = [
    path("", views.IndexView.as_view(), name="index"),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:pk>/vote/', views.vote, name='vote'),
]
```

## ListView

Display a list of objects
- 通过 get_queryset 获取要展示的列表
- 通过 context_object_name 配置模板中对应的列表属性名称

```python
class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]
```

## DetailView

Display a detail page for a particular type of object
- 通过 model 属性确定要展示的 object
- 默认的寻找目标对象的条件为主键 pk, 因此需要通过 GET 传入

```python
class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'
```