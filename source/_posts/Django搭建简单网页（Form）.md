---
title: Django搭建简单网页（Form）
tags:
  - Python
  - Django
categories:
  - 原创
  - Python
copyright: true
abbrlink: 3633b975
date: 2015-10-11 15:58:00
---
## 一、使用表单处理数据

1. 为了实现投票功能，当点击某一选项时，能提交数据到vote里去处理并返回结果，需要使用表单提交数据，在之前的detail.html里写上如下代码：
```js
#polls/templates/polls/detail.html

<h1>{{ question.question_text }}</h1>
{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}" />
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br />
{% endfor %}
<input type="submit" value="Vote" />
</form>
```
2. 为了能找到vote的路径，需要在urls.py中设置路径：
```py
#polls/urls.py
url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
```
3. 然后在views.py的vote函数中编写代码处理数据，实现一个问题的某一选项的票数增加并存储：
```py
#polls/views.py

from django.shortcuts import get_object_or_404, render
from django.http import HttpResponseRedirect, HttpResponse
from django.core.urlresolvers import reverse
from .models import Choice, Question
# ...

def vote(request, question_id):
    p = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = p.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        return render(request, 'polls/detail.html', {
            'question': p,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        return HttpResponseRedirect(reverse('polls:results', args=(p.id,)))
```
4. 点击投票按钮后需要跳转到投票结果页面results.html，显示某个问题的得票情况，也就是显示票数，并提示是否需要继续投票，以下代码修改view.py中的results函数，处理点击投票按钮后的数据，指定返回results.html的页面：
```py
#polls/views.py

from django.shortcuts import get_object_or_404, render

def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```
    在polls的template中创建results.html，在results.html中编写如下代码：
    ```js
    #polls/templates/polls/results.html

    <h1>{{ question.question_text }}</h1>
    <ul>
    {% for choice in question.choice_set.all %}
        <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
    {% endfor %}
    </ul>
    <a href="{% url 'polls:detail' question.id %}">Vote again?</a>
    ```
 
## 二、精简代码，使用generic views

接下来，我们使用如下步骤，来转变我们之前的代码：
1. 改变URL配置
2. 删除一些不需要的，旧的view
3. 引进新的，基于Django的 generic views

***第一步，修改URL配置：***

首先，打开polls/urls.py，作如下修改：
```py
#polls/urls.py

from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'^$', views.IndexView.as_view(), name='index'),
    url(r'^(?P<pk>[0-9]+)/$', views.DetailView.as_view(), name='detail'),
    url(r'^(?P<pk>[0-9]+)/results/$', views.ResultsView.as_view(), name='results'),
    url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
]
```

***第二步，修改view，引进新的generic views：***
删除之前的函数，新建类引进generic view，实现之前函数实现的功能：
```py
#polls/views.py

from django.shortcuts import get_object_or_404, render
from django.http import HttpResponseRedirect
from django.core.urlresolvers import reverse
from django.views import generic
from .models import Choice, Question

class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'
    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]

class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'

class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'
def vote(request, question_id):

    ... # same as above
```
-----