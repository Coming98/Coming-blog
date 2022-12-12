---
title: 03-django-testing
mathjax: false
date: 2022-12-12 18:42:54
summary: automated testing
categories: Django
tags:
  - python
---

# Automated Test

Why I new tests:
- Without tests, the purpose or intended behavior of an application might be rather opaque.
- Code without tests is broken by design.
- [For more reasons](https://docs.djangoproject.com/en/4.1/intro/tutorial05/)

Django 的测试流程:
1. 根据命令会寻找目标 APP 中的 test.py 测试代码
2. 寻找 `django.test.TestCase` 的子类
3. 初始化一个特殊的数据库供测试使用
4. 寻找类中以 `test` 开头的方法
5. 执行并测试该方法

测试准则:
- 对于每个模型和视图都建立单独的 TestClass
- 每个测试方法只测试一个功能
- 给每个测试方法起个能描述其功能的名字

## Quick Start

1. 在 APP 的 tests.py 中编写测试类

```python
import datetime
from django.test import TestCase
from django.utils import timezone

class QuestionModelTests(TestCase):
    pass
```

2. 测试类中编写相关的测试方法验证程序执行的正确性

```python
# was_published_recently() returns False for questions whose pub_date is in the future.
def test_was_published_recently_with_future_question(self):
    time = timezone.now() + datetime.timedelta(days=30)
    future_question = Question(pub_date=time)
    self.assertIs(future_question.was_published_recently(), False)
```

3. 执行测试, 查看测试结果: `python manage.py test app_name`

![](https://raw.githubusercontent.com/Coming98/pictures/main/202212121716700.png)

# Refs

- [Bilibili](https://www.bilibili.com/video/BV1S44y1K7Hd/?vd_source=b982c5b9804c7552564e69b7b5d8a2e0)