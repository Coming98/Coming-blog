---
title: 02-django-views
mathjax: false
date: 2022-12-12 16:45:07
summary: view template url-dispatcher
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

## urlpatterns

实现 url 与 view 的匹配:
- app_name 属性的设置用于外部引用时指明命名空间
- name 属性方便外部引用目标 url
- 支持 APP 内的维护: 使用 include 在根 urls.py 中导入即可

```python
from django.urls import path, include
path("polls/", include("polls.urls"))
```

- 支持占位符匹配:

```python
path('<int:question_id>/results/', views.results, name='results'),
```

## Exception

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

### 404 shortcuts

It’s a very common idiom to use `get()` and raise `Http404` if the object doesn’t exist. 因此 django 也为此提供了更加简便的封装

```python
from django.shortcuts import render, get_object_or_404

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212112309367.png)


## Database API

在 Views 中当然也可以使用 Django 自身的 Database API 让数据库的操作更加简单

```python
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    pass
```


## Generic Views

Q: 有些 View 的功能十分普遍, 例如展示一个列表的数据, 或者展示一个 Object 的详细信息, 针对这类 View 其后台操作基本一致: 获取 object 然后渲染模板页面, 因此可以进一步进行封装

A: Generic views abstract common patterns to the point where you don’t even need to write Python code to write an app. [For more details...](https://docs.djangoproject.com/en/4.1/topics/class-based-views/)

本质上就是缩减了后台与数据库交互的代码, 展示的模板代码基本上还是需要靠自己的..., 我原以为也能有一套默认的模板进行 generic 的展示

### Quick Start

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

### ListView

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

### DetailView

Display a detail page for a particular type of object
- 通过 model 属性确定要展示的 object
- 默认的寻找目标对象的条件为主键 pk, 因此需要通过 GET 传入

```python
class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'
```

# templates

响应式响应模板文件: 在所有 app 目录下的 templates 文件夹中寻找
- 寻找顺序按照 app 注册的先后
- [For more details](https://docs.djangoproject.com/en/4.1/topics/templates/)

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

## 快速渲染

使用 `django.shortcuts` 的 `render` 实现模板的快速渲染命令

```python
from django.shortcuts import render

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]

    context = { "latest_question_list": latest_question_list}
    return render(request, "polls/index.html", context)
```

## 根目录统一管理

在项目目录下的 `settings.py` 中 `TEMPLATES` 列表中的 `DIRS` 属性中可以配置搜索目录, 将项目根目录加入其中即可: `os.path.join(BASE_DIR, "templates")`
- 如果根目录中没有找到, 则再去各个 APP 中寻找

## 静态文件引入

django 将静态文件放置再 `app` 下的 `static` 目录下统一管理(配置文件中管理), 内部用户再自行创建文件夹进行规整
- 静态文件: 图片, CSS, js, plugin
- 使用模板语言引入 static

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212052248658.png)

## 避免硬编码 URL

在 urlpattern 中配置的 name 属性, 就可以用在模板语法中用于索引想要跳转的 url, 而不必再用拼接的形式, 如 `<a href="/polls/{{ question.id }}/">`

模板标签为 `{% url %}` 以空格为分隔符, 第一个参数表示目标 url 的 name, 后续参数为参数列表
```html
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

### 指定 namespace

多个 APP 的项目中, url 的 name 可能冲突, 因此就要指定 namespace 为某个 APP 

1. 在 app/urls.py 中添加 `app_name="app_name"` 指明 namespace
2. 在引用目标 url 时添加 namespace

```html
<a href="{% url 'polls:detail' question.id %}">
```

## 模板语法

- 变量值: `{{ var }}`
- 数组索引: `{{ arr.index }}`
- 字典属性: `{{ dict.key }}`

### 循环

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

### 条件

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

# GET & POST

Whenever you create a form that alters data server-side, use method="post". This tip isn’t specific to Django; it’s good web development practice in general
As the Python comment above points out, you should always return an HttpResponseRedirect after successfully dealing with POST data. This tip isn’t specific to Django; it’s good web development practice in general.
- 也就是说 POST 请求通常用于实现对后台数据库数据的更改, 一旦更改完毕, 便通常以一个 redirect 的形式返回到目标页面, 以实现显示的 URL 更新
- 但我觉得异步请求会更好一些..

## Post Request

```html
<form action="{% url 'polls:vote' question.id %}" method="post">
    {% comment %} In short, all POST forms that are targeted at internal URLs should use the {% csrf_token %} template tag. {% endcomment %}
    {% csrf_token %}
    <fieldset>
        <legend><h1>{{ question.question_text }}</h1></legend>
        {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
        {% for choice in question.choice_set.all %}
            <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
            <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
        {% endfor %}
    </fieldset>
    <input type="submit" value="Vote">
</form>
```

通过 Form 实现 Post 请求:
- action 使用非硬编码 url, 方便后续根据需求更改 url
- input 标签中的 name 属性为 POST 的键名, value 属性为 POST 的键值
- label 标签的指向, 更人性化的选取

### csrf 防止跨站请求伪造

POST 请求时会自动传递 `csrfmiddlewaretoken` 这个参数, 生存期仅一次

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212121308095.png)

如果使用其它工具直接重现请求: 如 Postman, 直接失败

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212121310718.png)

## Post Redirect

```python
def vote(request, pk):
    question = get_object_or_404(Question, pk=pk)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # reverse function helps avoid having to hardcode a URL in the view function.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

- views 函数的参数名需要和 urlpattern 中配置的一致
- 注意后台处理中数据库数据的 selected_choice 并没有添加互斥所, 多个用户同时请求更新这个数据时可能会出现错误, [解决方案](https://docs.djangoproject.com/en/4.1/ref/models/expressions/#avoiding-race-conditions-using-f)后续更新...
- reverse 函数解决了目标 url 的硬编码问题



# Refs

- [Bilibili](https://www.bilibili.com/video/BV1S44y1K7Hd/?vd_source=b982c5b9804c7552564e69b7b5d8a2e0)