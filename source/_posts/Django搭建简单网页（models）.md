---
title: Django搭建简单网页（models）
tags:
  - Python
  - Django
categories:
  - 原创
  - Python
copyright: true
abbrlink: 1f3cc00
date: 2015-09-22 11:25:41
---
## 一、创建一个django工程

1. 选择一个工作目录，然后用下面命令行创建一个project
```
django-admin startproject mysite
```
    创建后的目录如下所示：
    ```
    mysite/
       manage.py
       mysite/
           __init__.py
           settings.py
           urls.py
           wsgi.py
    ```
2. 可以去mysite/settings.py中设置数据库，默认为sqlite3
3. 使用数据库之前，得先在数据库中创建表。用如下命令：
```
python manage.py migrate
```
4. 启动django服务程序： 
```
python manage.py runserver
```
5. 在浏览器中输入地址http://127.0.0.1:8000/ 就可以访问初始界面。

## 二、创建models
1. 在项目目录下创建自己的app应用程序，app名为polls，使用如下命令：
```
python manage.py startapp polls
```
    则app的目录结构如下：
    ```
    polls/
        __init__.py
       admin.py
       migrations/
           __init__.py
       models.py
       tests.py
       views.py
    ```
2. 在polls的models.py文件里编写代码如下：
```
from django.dbimport models

classQuestion(models.Model):

    question_text =models.CharField(max_length=200)

    pub_date = models.DateTimeField('datepublished')

classChoice(models.Model):

    question = models.ForeignKey(Question)

    choice_text =models.CharField(max_length=200)

    votes = models.IntegerField(default=0)
```
    每个类对应数据库里的一个表，类中的成员变量对应表中的字段，即列项。
3. 激活models，在mysite/settings.py中添加应用的名字polls，如下：
```
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'polls',
)
```
4. Django 知道包含了我们自己的应用polls，再用以下命令行告诉django你对models做了改变，一开始是添加了model，以后每次改变了models都要使用这条命令：
```
Python manage.py makemigrations polls
```
5. 再次使用 python manage.py migrate为所有的models在数据库中创建表
6. 以后每次改变models的时候，更新数据库直接使用如下步骤：
   （1）在models.py改变models
   （2）运行命令： python manage.py makemigrations 为改变创建migration
   （3）运行命令：python manage.py migrate 在数据库中改变表

7. 使用django数据库的api，打开python的shell：
```    
python manage.py shell
```
    在shell中可以使用django的database api对models进行操作。
    比如：
    ```
    >>> from polls.models import Question, Choice  

    >>> Question.objects.all()

    []

    >>> from django.utils import timezone

    >>> q = Question(question_text="What's new?",pub_date=timezone.now())

    # Save the objectinto the database. You have to call save() explicitly.

    >>> q.save()

    >>> q.id

    1

    # Access modelfield values via Python attributes.

    >>> q.question_text

    "What'snew?"

    >>> q.pub_date

    datetime.datetime(2012,2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

    # Change values bychanging the attributes, then calling save().

    >>> q.question_text = "What's up?"

    >>> q.save()

    # objects.all()displays all the questions in the database.

    >>> Question.objects.all()

    [<Question:Question object>]
    ```

---