---
title: 09-django-ModelForm
mathjax: false
date: 2022-12-30 18:21:59
summary: 使用 Form 和 ModelForm 快速交互
categories: Django
tags:
  - python
---

# ModelForm

借助 Model 的定义快速实现 CRUD 的逻辑任务

## Quick Start

1. 创建目标 Model 的 ModelForm 类

```python
class UserModelForm(forms.ModelForm):
    class Meta:
        model = UserInfo
        fields = ['name', 'password', 'age', 'account', 'create_time', 'department', 'gender']
        # fields = "__all__" # 表示选择所有字段
        # exclude = ['create_time', ] # 黑名单
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        for name, field in self.fields.items():
            field.widget.attrs = {"class": "form-control"}
```

2. 将其示例对象传到 Template 中, 并快速实现 Form 的生成

```python
form = UserModelForm()
return render(request, 'OA/user_add.html', {'form': form})
```

- 通过 label 获取 model 中 verbose_name 属性
- 直接渲染将会渲染出等价的 Form
- 通过 errors 可以获取错误信息

```html
{% for field in form %}
    <div class="form-group">
        <label for="depart_title"> {{ field.label }} </label>
        {{ field }}
        <span style="color: red;"> {{ field.errors.0 }} </span>
    </div>
{% endfor %}
```

3. POST 传递后可以直接通过 ModelForm 类进行实例化, 并进自动行有效性检查, 根据检查结果进行处理

```python
form = UserModelForm(data=request.POST)
if form.is_valid():
    form.save()
    return redirect("OA:user_list")
else:
    # 自带错误信息
    return render(request, 'OA/user_add.html', {'form': form})
```

## 静态数据

添加新数据时有些数据并非用户交互的, 例如创建时间, 初始工龄等, 这些都可以在后台设置默认值来进行填充

```python
form = UserModelForm(data=request.POST)
if form.is_valid():
    form.instance.create_time = datetime.now()
    form.save()
    return redirect("OA:user_list")
```

在前台可以通过 ModelForm 重新定义, 指定 disabled 属性为 True 使得其不可更改, 或直接在 fields 域中排除

```python
mobile = forms.CharField(disalbed=True, label="手机号")
```

## 数据校验

如果需要详细的数据校验控制, 可以在 ModelForm 中重新定义字段的实例

```python
from django.core.validators import RegexValidator
mobile = forms.CharField(
    label="手机号",
    validators=[
        RegexValidator(r'^1[3-9]\d{9}$', '手机号格式错误...')
    ]
)
```

或者通过 Hook 方法, 重写 ModelForm 的内部校验过程: `mobile` 为域名
- 常用于复杂验证, 如数据不重复等
- 验证不通过返回错误提示
- 验证通过, 返回用户输入信息

```python
def clean_mobile(self):
    mobile = self.cleaned_data['mobile']

    # 编辑时要排除自身
    exists_flag = PrettyNum.objects.filter(mobile=mobile).exclude(id=self.instance.pk).exists()
    if exists_flag:
        raise ValidationError("手机号已存在")

    return mobile
```

## 数据编辑

1. 根据数据库对象进行实例化

```python
form = UserModelForm(instance=UserInfo.objects.filter(id=user_id).first())
return render(request, 'OA/user_edit.html', {'form': form})
```

2. 接收 form 数据时需要指定要更新的数据库实例, 才能使用 `save` 方法进行**更新操作**而非添加操作

```python
form = UserModelForm(data=request.POST, instance=UserInfo.objects.filter(id=user_id).first())
if(form.is_valid()):
    form.save()
    return redirect("OA:user_list")
else:
    return render(request, 'OA/user_edit.html', {'form': form})
```
