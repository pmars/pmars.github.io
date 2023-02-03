---
layout:     post
title:      "Django Web应用架构"
subtitle:   "Python 服务器架构之 Django"
date:       2016-01-25 20:27:35
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - Django
    - 服务器
---

> 好久没有写博客了，今天看了一下 `django` 就过来水一篇
> 这篇文章里面写到了好多模板的东西，而我的博客用的就是markdown模板写的，这样在中间转的时候就出现了问题，所以我给文章中的所有 `'{' + '%'(这里写也编译不过，只能字符串拼接了)` 都换成了 `{<removeME>%`，再用的时候需要将 `<removeME>` 删除

---


### 目录

0. [简介](#summary)
0. [安装](#install)
0. [创建项目](#startproject)
0. [创建APP](#appstartapp)
0. [模板](#templates)
0. [数据库](#datebase)

---

### [简介](#summary)


`Django` 是一个开放源代码的 `Web` 应用框架，由 `Python` 写成。采用了 `MVC` 的软件设计模式，即模型M，视图V和控制器C。

`Django` 的主要目标是使得开发复杂的、数据库驱动的网站变得简单。`Django` 注重组件的重用性和“可插拔性”，敏捷开发和 `DRY` 法则（Don't Repeat Yourself）。在 `Django` 中 `Python` 被普遍使用，甚至包括配置文件和数据模型。

`Django` 框架的核心包括：一个 物件導向 的映射器，用作数据模型（以 `Python` 类的形式定义）和關聯性数据库间的媒介；一个基于正则表达式的 `URL` 分发器；一个视图系统，用于处理请求；以及一个模板系统。

核心框架中还包括：

* 一个轻量级的、独立的Web服务器，用于开发和测试。
* 一个表单序列化及验证系统，用于HTML表单和适于数据库存储的数据之间的转换。
* 一个缓存框架，并有几种缓存方式可供选择。
* 中间件支持，允许对请求处理的各个阶段进行干涉。
* 内置的分发系统允许应用程序中的组件采用预定义的信号进行相互间的通信。
* 一个序列化系统，能够生成或读取采用XML或JSON表示的Django模型实例。
* 一个用于扩展模板引擎的能力的系统。

`Django` 包含了很多应用在它的 `contrib` 包中，这些包括：

* 一个可扩展的认证系统
* 动态站点管理页面
* 一组产生RSS和Atom的工具
* 一个灵活的评论系统
* 产生Google站点地图（Google Sitemaps）的工具
* 防止跨站请求伪造（cross-site request forgery）的工具
* 一套支持轻量级标记语言（Textile和Markdown）的模板库
* 一套协助创建地理信息系统（GIS）的基础框架

---

### [安装](#install)

for ubuntu:

    $ pip install django
    $ python -c "import django; print(django.get_version())"

---

### [创建项目](#startproject)

创建项目用 `startproject`

    $ django-admin startproject mypro
    $ tree mypro
    mypro
    ├── manage.py
    └── mypro
        ├── __init__.py
        ├── settings.py
        ├── urls.py
        └── wsgi.py

    1 directory, 5 files

启动项目用 `runserver`

    $ python manage.py runserver 0.0.0.0:8000

这时就可以在浏览器里面查看了（我就不截图了）

---

### [创建APP](#startapp)

    $ cd mypro
    $ python manage.py startapp myapp

`vi myapp/views.py`

    from django.http import HttpResponse
    def index(request):
        return HttpResponse("Hello, world. You're at the polls index.")

`vi myapp/urls.py`

    from django.conf.urls import url
    from . import views
    urlpatterns = [
        url(r'^$', views.index, name='index'),
    ]

`vi mypro/urls.py`

    from django.conf.urls import include, url
    from django.contrib import admin
    urlpatterns = [
        url(r'^myapp/', include('myapp.urls')),
        url(r'^admin/', admin.site.urls),
    ]

修改之后就可以通过浏览器查看 `/myapp/` 了。

    $ curl http://localhost:5000/myapp/
    Hello, world. You're at the polls index.

---

### [模板](#templates)

在外层的 `mypro` 文件夹中创建一个 `templates` 文件夹，用来存放各种模板

    $ mkdir -p templates

`vi templates/hello.html`

    <h1>{ { hello }}</h1>

> 上面代码中{ {之间没有空格的，为了博客显示问题，特意加入的。

可以看出来，这就是 `markdown` 的模板啊

修改配置文件中模板的路径，`vi mypro/settings.py`

    TEMPLATES = [
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            'DIRS': [BASE_DIR+'/templates'],
            'APP_DIRS': True,
            'OPTIONS': {
                'context_processors': [
                    'django.template.context_processors.debug',
                    'django.template.context_processors.request',
                    'django.contrib.auth.context_processors.auth',
                    'django.contrib.messages.context_processors.messages',
                ],
            },
        },
    ]

之后再 `myapp/views.py` 中添加一个函数

    def hello(request):
        context = {}
        context['hello'] = 'Hello World!'
        return render(request, 'hello.html', context)

修改 `urls.py`，添加一条刚才函数接口的配置，`vi myapp/urls.py`

    url(r'hello$', views.hello, name='hello'),

测试一下：

    $ curl http://localhost:5000/myapp/hello
    <h1>Hello World!</h1>

#### django 的模板标签：

###### if/else 标签

基本语法格式如下：

    {<removeME>% if condition %}
         ... display
    {<removeME>% endif %}

或者：

    {<removeME>% if condition1 %}
       ... display 1
    {<removeME>% elif condiiton2 %}
       ... display 2
    {<removeME>% else %}
       ... display 3
    {<removeME>% endif %}

根据条件判断是否输出。`if/else` 支持嵌套。
`{<removeME>% if %}` 标签接受 and ， or 或者 not 关键字来对多个变量做判断 ，或者对变量取反（ not )，例如：

    {<removeME>% if athlete_list and coach_list %}
        athletes 和 coaches 变量都是可用的。
    {<removeME>% endif %}

###### for 标签

`{<removeME>% for %}` 允许我们在一个序列上迭代。

与 `Python` 的 `for` 语句的情形类似，循环语法是 `for X in Y` ，Y是要迭代的序列而X是在每一个特定的循环中使用的变量名称。

每一次循环中，模板系统会渲染在 `{<removeME>% for %}` 和 `{<removeME>% endfor %}` 之间的所有内容。

例如，给定一个运动员列表 `athlete_list` 变量，我们可以使用下面的代码来显示这个列表：

    <ul>
    {<removeME>% for athlete in athlete_list %}
        <li>{{ athlete.name }}</li>
    {<removeME>% endfor %}
    </ul>

给标签增加一个 `reversed` 使得该列表被反向迭代：

    {<removeME>% for athlete in athlete_list reversed %}
    ...
    {<removeME>% endfor %}

可以嵌套使用 `{<removeME>% for %}` 标签：

    {<removeME>% for athlete in athlete_list %}
        <h1>{{ athlete.name }}</h1>
        <ul>
        {<removeME>% for sport in athlete.sports_played %}
            <li>{{ sport }}</li>
        {<removeME>% endfor %}
        </ul>
    {<removeME>% endfor %}

###### ifequal/ifnotequal 标签

`{<removeME>% ifequal %}` 标签比较两个值，当他们相等时，显示在 `{<removeME>% ifequal %}` 和 `{<removeME>% endifequal %}` 之中所有的值。

下面的例子比较两个模板变量 user 和 currentuser :

    {<removeME>% ifequal user currentuser %}
        <h1>Welcome!</h1>
    {<removeME>% endifequal %}

和 `{<removeME>% if %}` 类似， `{<removeME>% ifequal %}` 支持可选的 `{<removeME>% else%}` 标签：8

    {<removeME>% ifequal section 'sitenews' %}
        <h1>Site News</h1>
    {<removeME>% else %}
        <h1>No News Here</h1>
    {<removeME>% endifequal %}

###### 注释标签

Django 注释使用 `{# #}`。

    {# 这是一个注释 #}

###### 过滤器

模板过滤器可以在变量被显示前修改它，过滤器使用管道字符，如下所示：

    {{ name|lower }}

`{{ name }}` 变量被过滤器 lower 处理后，文档大写转换文本为小写。

过滤管道可以被\* 套接\* ，既是说，一个过滤器管道的输出又可以作为下一个管道的输入：

    {{ my_list|first|upper }}

以上实例将第一个元素并将其转化为大写。

有些过滤器有参数。 过滤器的参数跟随冒号之后并且总是以双引号包含。 例如：

    {{ bio|truncatewords:"30" }}

这个将显示变量 bio 的前30个词。

###### 其他过滤器

* addslashes : 添加反斜杠到任何反斜杠、单引号或者双引号前面。
* date : 按指定的格式字符串参数格式化 date 或者 datetime 对象，实例：`{{ pub_date|date:"F j, Y" }}`
* length : 返回变量的长度。
* include 标签 {<removeME>% include %} 标签允许在模板中包含其它的模板的内容。

下面这两个例子都包含了 nav.html 模板：

    {<removeME>% include "nav.html" %}

###### 模板继承

模板可以用继承的方式来实现复用。

接下来我们先创建之前项目的 templates 目录中添加 base.html 文件，代码如下：

    <html>
      <head>
        <title>Hello World!</title>
      </head>

      <body>
        <h1>Hello World!</h1>
        {<removeME>% block mainbody %}
           <p>original</p>
        {<removeME>% endblock %}
      </body>
    </html>

以上代码中，名为mainbody的block标签是可以被继承者们替换掉的部分。

所有的 `{<removeME>% block %}` 标签告诉模板引擎，子模板可以重载这些部分。

hello.html 中继承 base.html，并替换特定block，hello.html修改后的代码如下：

    {<removeME>% extends "base.html" %}
    {<removeME>% block mainbody %}
        <p>继承了 base.html 文件</p>
    {<removeME>% endblock %}

第一行代码说明 `hello.html` 继承了 `base.html` 文件。可以看到，这里相同名字的 `block` 标签用以替换 `base.html` 的相应 `block`。

---

### [数据库](#datebase)

`django` 默认使用的是 `sqlite3`，你可以通过 `sudo apt-get install sqlite3` 来安装，之后进行下面的操作。

使用之前需要先执行一下 `$ python manage.py migrate`

`vi myapp/models.py`

    from django.db import models

    class Question(models.Model):
        question_text = models.CharField(max_length=200)
        pub_date = models.DateTimeField('date published')

    class Choice(models.Model):
        question = models.ForeignKey(Question, on_delete=models.CASCADE)
        choice_text = models.CharField(max_length=200)
        votes = models.IntegerField(default=0)

激活models `vi mypro/settings.py`

    INSTALLED_APPS = [
        'myapp.apps.MyappConfig',
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
    ]

执行下面的语句：

    $ python manage.py makemigrations myapp
    $ python manage.py sqlmigrate myapp 0001
    $ python manage.py migrate

具体参考 <https://docs.djangoproject.com/en/1.9/intro/tutorial02/>

### 最后的目录结构

    $ tree mypro
    mypro
    ├── db.sqlite3
    ├── manage.py
    ├── myapp
    │   ├── admin.py
    │   ├── apps.py
    │   ├── __init__.py
    │   ├── __init__.pyc
    │   ├── migrations
    │   │   └── __init__.py
    │   ├── models.py
    │   ├── tests.py
    │   ├── urls.py
    │   ├── urls.pyc
    │   ├── views.py
    │   └── views.pyc
    ├── mypro
    │   ├── __init__.py
    │   ├── __init__.pyc
    │   ├── settings.py
    │   ├── settings.pyc
    │   ├── urls.py
    │   ├── urls.pyc
    │   ├── wsgi.py
    │   └── wsgi.pyc
    └── templates
        └── hello.html

    4 directories, 22 files

---

### END


