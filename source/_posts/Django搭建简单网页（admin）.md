---
title: Django搭建简单网页（admin）
tags:
  - Python
  - Django
categories:
  - 原创
  - Python
copyright: true
abbrlink: 9a5c839
date: 2015-09-22 22:01:55
toc: true
---
## 一、运行后台管理

1. 创建超级用户以便于登录到后台管理
```python
    python manage.py createsuperuser
```
    接着输入用户名，邮箱，密码完成创建

2. 运行服务程序：
```
python manage.py runserver
```
    然后在浏览器里输入：http://127.0.0.1:8000/admin/, 在登录界面完成登录就可以进入管理界面了。
<!--more-->
3. 为了在后台管理界面中显示我们编写的应用polls，需要在polls/admin.py中添加如下代码：
``` python
from django.contrib import admin
from .models import Question
admin.site.register(Question)  #在管理网页中注册需要显示的Question
```
    这时后台会自动为Question生成一些管理表单。
    为了在后台管理界面中显示我们编写的应用polls，需要在polls/admin.py中添加如下代码：
    ```
    from django.contrib import admin
    from .models import Question
    admin.site.register(Question)  #在管理网页中注册需要显示的Question
    ```
    这时后台会自动为Question生成一些管理表单。

## 二、自定义管理表单

上面只是简单的注册了Question，然后让django自动生成表单。接下来要自定义表单显示的方式。
1. 把publication date放在Question text的前面显示，还是在polls/admin.py中，只是用以下代码替代admin.site.register(Question) ：
``` python
class QuestionAdmin(admin.ModelAdmin):
   fields = ['pub_date', 'question_text']
admin.site.register(Question,QuestionAdmin)
```
    同样，以后也可以自定义其他的管理对象，然后作为register()函数的第二个参数。

2. 当Question字段太多的时候，需要把它们分开显示，这时就可以用下面的代码代替上面的代码：
``` python
class QuestionAdmin(admin.ModelAdmin):
   fieldsets = [
       (None,               {'fields':['question_text']}),
       ('Date information', {'fields': ['pub_date']}),
    ]
admin.site.register(Question,QuestionAdmin)
```
 
3. 一个Question对应多个Choice,所以需要将多个Choice显示在一个Question下，在polls/admin.py中用如下代码实现：
```
from django.contrib import admin
from .models import Choice, Question
class ChoiceInline(admin.StackedInline):
   model = Choice
   extra = 3
class QuestionAdmin(admin.ModelAdmin):
   fieldsets = [
       (None,               {'fields':['question_text']}),
       ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),#日期信息这一栏设置了可隐藏
    ]
   inlines = [ChoiceInline]
admin.site.register(Question,QuestionAdmin)
```
 
4. 当一个问题的投票选项太多的时候，上面那样的Choice每条都比较占空间，不够简洁，如下修改ChoiceInline继承的类为TabularInline：
```
   class ChoiceInline(admin.TabularInline):
       #...
```
    这样每个Question下面每个Choice就分两列罗列出来，简洁明了。

    关于表单的自定义，如各种元素的显示方式，都可以参考官方文档的ModelAdmin ：https://docs.djangoproject.com/en/1.8/ref/contrib/admin/#django.contrib.admin.ModelAdmin

## 三、自定义自己项目的template
之前是django自动生成的template，从而定义的后台管理界面的风格和显示内容，现在要自己自定义自己项目的template，这样就可以在自己项目的template里修改代码，定制界面风格。根据以下步骤可以完成这个任务：

1. 在项目目录下（即与manage.py同层次的目录）创建一个template文件夹，然后在mysite/settings.py中的DIRS选项中添加如下代码：
```
'DIRS':[os.path.join(BASE_DIR, 'templates')],
```

2. 在刚创建的template文件夹下再创建admin文件夹，然后在Django的安装目录django\contrib\admin\templates\admin文件夹下，把相关的html文件复制到刚刚创建的admin下，如把admin/base_site.html复制到刚创建的admin中。

3. 然后打开base_site.html编辑代码如下：
```
{%block branding %}
<h1 id="site-name"><ahref="{% url 'admin:index' %}">PollsAdministration</a></h1>
{% endblock %}
```
    这样标题就会变为PollsAdministration.

4. 同理可以自定义其他的内容格式，一样的从django/admin目录里复制html文件到自己创建的admin中，修改其中的代码。

---