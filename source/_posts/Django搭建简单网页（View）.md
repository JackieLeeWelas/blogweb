---
title: Django搭建简单网页（View）
tags:
  - Python
  - Django
categories:
  - 原创
  - Python
copyright: true
abbrlink: 6ad03acf
date: 2015-09-22 11:46:00
---
## 一、编写前台界面views：
1. 编写前台界面需要显示的内容，打开polls/views.py，编写如下代码：
``` python
#polls/views.py
from django.http importHttpResponse
def index(request):
   return HttpResponse("Hello, world. You're atthe polls index.")
```
<!--more-->
2. 编写自己应用的urls文件，在应用polls下创建urls.py，添加如下代码：
```python
#polls/urls.py
from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'^$', views.index, name='index'),
]
```
3. 在项目的urls文件里指定自己应用的urls文件，打开项目的urls.py，添加如下代码：
```py
#mysite/urls.py

from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
   url(r'^polls/', include('polls.urls')),
   url(r'^admin/', include(admin.site.urls)),
]
```
4. 在浏览器中打开http://127.0.0.1:8000/polls, 就可以看到刚刚写的view，即显示“Hello,world. You're at the polls index.” 

## 二、编写若干个views界面
同上面的原理是一样的，先写界面内容，再去自己应用的urls里面使用正则表达式指定路径。

1. 在polls/views.py中添加如下代码：
```python
#polls/views.py

def detail(request, question_id):
   return HttpResponse("You're looking at question %s." %question_id)

def results(request, question_id):
   response = "You're looking at the results of question %s."
   return HttpResponse(response % question_id)

def vote(request, question_id):
   return HttpResponse("You're voting on question %s." %question_id)
```
2. 在polls/urls.py中指定路径：
```python
#polls/urls.py

from django.conf.urls import url
from . import views

urlpatterns = [
    #ex: /polls/
   url(r'^$', views.index, name='index'),
    #ex: /polls/5/
   url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
    #ex: /polls/5/results/
   url(r'^(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
    #ex: /polls/5/vote/
   url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
]
```
3. 然后就可以在浏览器里输入各种路径，如：
    http://127.0.0.1:8000/polls/3: 显示“You're looking at question 3.”
    http://127.0.0.1:8000/polls/3/results: 显示“You're looking at the results ofquestion 3.”
    http://127.0.0.1:8000/polls/3/vote: 显示“You're voting on question 3.”

## 三、编写用于显示后台数据的前台界面：
上面的界面内容只是静态的显示一些字符串，接下来是实现从后台读取数据显示在前台界面

修改polls/views.py文件，其他的操作是一样的：
```python
#polls/views.py

from django.http import HttpResponse
from .models import Question

def index(request):
    latest_question_list =Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([p.question_text for pin latest_question_list])
   return HttpResponse(output)
```
这里导入了models里的Question，然后读取出Question里的内容，病按日期排序。

## 四、从views.py中分离出template进行界面编写
1. 在应用polls里创建templates文件夹，再在里面创建polls文件夹，在新建的polls里创建index.html文件，打开并编写如下代码：
```js
{% if latest_question_list %}
   <ul>
   {% for question in latest_question_list %}
       <li><a href="/polls/{{ question.id }}/">{{question.question_text }}</a></li>
   {% endfor %}
   </ul>
{% else %}
   <p>No polls are available.</p>
{% endif %}
```
    上面代码是从views.py里分离出来的用来显示最近问题列表的功能，这里分条显示。

2. 然后在polls的views里修改代码如下：
```python
#polls/views.py

from django.http import HttpResponse
from django.template import RequestContext,loader
from .models import Question

def index(request):
    latest_question_list =Question.objects.order_by('-pub_date')[:5]
    template =loader.get_template('polls/index.html')
    context = RequestContext(request, {
        'latest_question_list': latest_question_list,
    })
    returnHttpResponse(template.render(context))
```
    这里用loader装载template：polls/index.html，然后再传递上下文给template进行render。

## 五、用render( )代替HttpResponse，简化代码
代码修改如下：
```python
#polls/views.py

from django.shortcuts import render
from .models import Question

def index(request):
   latest_question_list = Question.objects.order_by('-pub_date')[:5]
   context = {'latest_question_list': latest_question_list}
   return render(request, 'polls/index.html', context)
```
这样就不用导入loader,RequestContext和HttpResponse了， render本身就是返回一个HttpResponse对象，所以直接返回。
也可以这样修改更简洁：
```python
def index(request):
   latest_question_list = Question.objects.order_by('-pub_date')[:5]
   return render(request, 'polls/index.html', {'latest_question_list':latest_question_list})
```

----