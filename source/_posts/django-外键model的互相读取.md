---
title: django 外键model的互相读取
tags:
  - Python
  - Django
categories:
  - 原创
  - Python
copyright: true
abbrlink: d507d673
date: 2015-12-15 10:07:27
---
先设定一个关系模型如下:
``` python
from django.db import models
class Blog(models.Model):
   name = models.CharField(max_length=100)
   tagline = models.TextField()
   def __str__(self):            
       return self.name
 
class Author(models.Model):
   name = models.CharField(max_length=50)
   email = models.EmailField()
   def __str__(self):           
       return self.name
 
class Entry(models.Model):
   blog = models.ForeignKey(Blog)
   headline = models.CharField(max_length=255)
   body_text = models.TextField()
   authors = models.ManyToManyField(Author)
   def __str__(self):            
      return self.headline

```
上面的数据关系很明晰,Entry中有Blog和Author的外键,如果要在Entry中读取blog和author的数据很容易:
``` python
entry = Entry.objects.all()
for e in entry:
    blog = e.blog
    author = e.authors
```
要在Blog和Author中读取Entry也可以：
``` python
blog = Blog.objects.all()
entry = blog.entry_set.all()
 
author = Author.objects.all()
entry = author.entry_set.all()
```

下面通过entry使blog和author互相读取，比如要知道一个blog的Author只需如下:
``` python
blogs = Blog.objects.all()
for blog in blogs:
    if blog.name== “我们想要查询的博客的name”
        author = blog. entry_set.authors
```
要查询一个author的所有blog如下：
``` python
authors = Author.objects.all()
blogs = []
 
for author in authors:
    if author.name== “我们想要查询的Author的name”
        for entry in author.entry_set.all():
            blogs.append(entry. blog)
```
-----