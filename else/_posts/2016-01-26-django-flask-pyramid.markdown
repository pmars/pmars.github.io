---
layout:     post
title:      "选择一个 Python Web 框架：Django vs Flask vs Pyramid"
subtitle:   "Django vs Flask vs Pyramid: Choosing a Python Web Framework"
date:       2016-01-26 17:21:10
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - Python
    - Django
    - Flask
    - Pyramid
---

> 此篇文章为转载，文章出自 [中文翻译](http://www.oschina.net/translate/django-flask-pyramid>)，或 [英文原版](https://www.airpair.com/python/posts/django-flask-pyramid)。
> 这篇文章让我更深入的理解了三个架构，并且横向的对比更让我立体的认识了它们，所以好文章就转载一下。
> 文章很长，但如果认真看的话，真的能学到东西。
> 由于模板限制，文章中 '{'+'%' 都替换为 `{<removeME>%`，'{'+'{' 都转换为 `{ {`

---

### 目录

0. [简介](#introduction)
0. [关于框架](#Architecture)
0. [社区](#group)
0. [Bootstrapping](#bootstrappingboot)
0. [模板](#templates)
0. [利用框架行动起来](#usingTemplates)
0. [总结](#summary)

---

> [Pyramid](https://github.com/Pylons/pyramid), [Django](https://github.com/django/django), 和 [Flask](https://github.com/mitsuhiko/flask) 都是优秀的框架，为项目选择其中的哪一个都是伤脑筋的事。我们将会用三种框架实现相同功能的应用来更容易的对比三者。也可以直接跳到框架实战（Frameworks in Action）章节 [查看代码](https://github.com/ryansb/wut4lunch_demos)。

---

### [简介](#introduction)


世界上可选的基于Python的web框架有很多。Django, Flask, Pyramid, Tornado, Bottle, Diesel, Pecan, Falcon等等，都在争取开发者支持。作为一开发者从一堆选择中筛选出一个来完成项目将会成为下一个大工程。我们今天专注于Flask, Pyramid, 和 Django。它们涵盖了从小微项目到企业级的web服务。

为了更容易在三者中作出选择（至少更了解它们），我们将用每一个框架构建同样的应用并比较它们的代码，对于每一个方法我们会高亮显示它的优点和缺点。如果你只想要代码，直接跳到框架实战章节（Frameworks in Action），或者查看其在 [Github](https://github.com/ryansb/wut4lunch_demos) 上的代码。

Flask是一个面向简单需求小型应用的“微框架（microframework）”。Pyramid和Django都是面向大型应用的，但是有不同的拓展性和灵活性。Pyramid的目的是更灵活，能够让开发者为项目选择合适的工具。这意味着开发者能够选择数据库、URL结构、模板类型等等。Django目的是囊括web应用的所有内容，所以开发者只需要打开箱子开始工作，将Django的模块拉进箱子中。

Django包括一个开箱即用的 [ORM](http://en.wikipedia.org/wiki/Object-relational_mapping) ，而Pyramid和 Flask让开发者自己选择如何或者是否存储他们的数据。到目前为止对于非Django的web应用来说最流行的ORM是 [SQLAlchemy](http://www.sqlalchemy.org/)，同时还有多种其他选择，从 [DynamoDB](http://aws.amazon.com/dynamodb/) 和 [MongoDB](http://www.mongodb.org/) 到简单本地存储的 [LevelDB](https://github.com/google/leveldb) 或朴实的 [SQLite](http://www.sqlite.org/)。Pyramid被设计为可使用任何数据持久层，甚至是还没有开发出来的。

---

### [关于框架](#Architecture)


Django的"batteries included" 特性让开发者不需要提前为他们的应用程序基础设施做决定，因为他们知道Python已经深入到了web应用当中。Django已经内建了模板、表单、路由、认证、基本数据库管理等等。比较起来，Pyramid包括路由和认证，但是模板和数据库管理需要额外的库。

前面为 Flask和Pyramid apps选择组件的额外工作给那些使用案例不适用标准ORM的开发者提供了更多的灵活性，同样也给使用不同工作流和模版化系统的开发者们带来了灵活性。

Flask，作为三个框架里面最稚气的一个，开始于2010年年中。Pyramid框架是从 [Pylons](http://www.pylonsproject.org/about/history) 项目开始的，在2010年底获得 Pyramid这个名字，虽然在2005年就已经发布了第一个版本。Django 2006年发布了第一个版本，就在Pylons项目（最后叫Pyramid）开始之后。Pyramid和Django都是非常成熟的框架，积累了众多插件和扩展以满足难以置信的巨大需求。

虽然Flask历史相对更短，但它能够学习之前出现的框架并且把注意力放在了微小项目上。它大多数情况被使用在一些只有一两个功能的小型项目上。例如 [httpbin](http://httpbin.org/)，一个简单的（但很强大的）调试和测试HTTP库的项目。

---

### [社区](#group)


最具活力的社区当属Django，其有111518个StackOverflow问题和一系列来自开发者和优秀用户的良好的博客。Flask和Pyramid社区并没有那么大，但它们的社区在邮件列表和IRC上相当活跃。StackOverflow上仅有10723个相关的标签，Flask比Django小了11倍。在Github上，它们的star近乎相当，Django有17721个，Flask有18118个。

三个框架都使用的是BSD衍生的协议。Flask和Django的协议是BSD 3条款，Pyramid的 [Repoze Public License RPL](http://repoze.org/license.html) 是BSD协议 4条款的衍生。

---

### [Bootstrapping](#boot)


Django和Pyramid都内建bootstrapping工具。Flask没有包含类似的工具，因为Flask的目标用户不是那种试图构建大型 [MVC](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) 应用的人。

##### Flask

Flask的hello world应用非常的简单，仅仅单个Python文件的7行代码就够了。

    # from http://flask.pocoo.org/ tutorial
    from flask import Flask
    app = Flask(__name__)
    
    @app.route("/") # take note of this decorator syntax, it's a common pattern
    def hello():
        return "Hello World!"
    
    if __name__ == "__main__":
        app.run()

这是Flask没有bootstrapping工具的原因：没有它们的需求。从Flask主页上的Hello World特性看，没有构建Python web应用经验的开发者可以立即开始hacking。

对于各部分需要更多分离的项目，Flask有blueprints。例如，你可以将所有用户相关的函数放在users.py中，将销售相关的函数放在ecommerce.py中，然后在site.py中添加引用它们来结构化你的Flask应用。我们不会深入这个功能，因为它超出了我们展示demo应用的需求。

##### Pyramid

Pyramid 的 bootstrapping工具叫 pcreate，是Pyramid的组成部分. 之前的  [Paste](http://pythonpaste.org/) 工具套装提供了 bootstrapping ，但是从那之后被 Pyramid专用工具链替代了。

    $ pcreate -s starter hello_pyramid # Just make a Pyramid project

Pyramid 比 Flask 适用于更大更复杂的应用程序. 因为这一点,它的 bootstrapping工具创建更大的项目骨架. Pyramid 同样加入了基本的配置文件，一个例子模版和用于将程序打包上传到 [Python Package Index](https://pypi.python.org/) 的所有文件。

    hello_pyramid
    ├── CHANGES.txt
    ├── development.ini
    ├── MANIFEST.in
    ├── production.ini
    ├── hello_pyramid
    │   ├── __init__.py
    │   ├── static
    │   │   ├── pyramid-16x16.png
    │   │   ├── pyramid.png
    │   │   ├── theme.css
    │   │   └── theme.min.css
    │   ├── templates
    │   │   └── mytemplate.pt
    │   ├── tests.py
    │   └── views.py
    ├── README.txt
    └── setup.py

作为最后描述的框架，Pyramid的bootstrapper非常灵活. 不局限于一个默认的程序;pcreate 可以使用任意数量的项目模版. 包括我们上面用到的pcreate里面的"starter"的模版, 还有 SQLAlchemy- ，[ZODB](http://www.zodb.org/en/latest/) -支持scaffold项目. 在 PyPi可以发现已经为 [Google App Engine](https://pypi.python.org/pypi/pyramid_appengine/), [jQuery Mobile](https://github.com/Pylons/pyramid_jqm), [Jinja2 templating](https://pypi.python.org/pypi/jinja2-alchemy-starter), [modern frontend frameworks](https://pypi.python.org/pypi/pyramid_modern) 做好的scaffolds， 还有更多~

##### Django

Django 也有自己的 bootstrap 工具, 内置在 django-admin 中.

    django-admin startproject hello_django
    django-admin startapp howdy # make an application within our project

Django 跟 Pyramid 区别在于: Django 由多个应用程序组成一个项目, 而 Pyramid 以及 Flask 项目是包含 View 和 Model 单一应用程序 . 理论上,  Flask 和 Pyramid 的项目允许存在多个 project/app, 不过在默认配置中只能有一个.

    hello_django
    ├── hello_django
    │   ├── __init__.py
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    ├── howdy
    │   ├── admin.py
    │   ├── __init__.py
    │   ├── migrations
    │   │   └── __init__.py
    │   ├── models.py
    │   ├── tests.py
    │   └── views.py
    └── manage.py

Django 默认只在项目中创建空白的 model 和模板文件, 供新手参考的示范代码不多. 此外, 开发者在发布应用程序的时候, 还要自己配置, 这也是个麻烦.

bootstrap 工具的缺点是没有指导开发者如何打包应用.  对于那些没有经验的新手来说, 第一次部署应用将是个很头疼的问题. 像 [django-oscar](https://github.com/tangentlabs/django-oscar) 这样的大社区, 项目都是打包好了, 放在 PyPi 上供大家安装. 但是 Github 上面的小项目缺少统一的打包方式.

---

### [模板](#templates)


一个Python应用能够响应HTTP请求将是一个伟大的开端，但是有可能你的大多数用户是没有兴趣使用curl与你的web应用交互的。幸运的是，这三个竞争者提供了使用自定义信息填充HTML的方法，以便让大伙们能够享受时髦的 [Bootstrap](http://getbootstrap.com/) 前端。

模板让你能够直接向页面注入动态信息，而不是采用AJAX。你只需要一次请求就可以获取整个页面以及所有的动态数据，这对用户体验来说是很好的。这对于手机网站来说尤其重要，因为一次请求花费的时间会更长。

所有的模板选项依赖于“上下文环境（context）”，其为模板转换为HTML提供了动态信息。模板的最简单的例子是填充已登录用户的名字以正确的迎接他们。也可以用AJAX获取这种动态信息，但是用一整个调用来填写用户的名字有点过头了，而同时模板又是这么的简单。

##### Django

我们使用的例子正如写的那么简单，假设我们有一个包含了用户名的funllname属性的user对象。在Python中我们这样向模板中传递当前用户：

    def a_view(request):
        # get the logged in user
        # ... do more things
        return render_to_response(
            "view.html",
            {"user": cur_user}
        )

拥有这个模板的上下文很简单，传入一个Python对象的字典和模板使用的数据结构。现在我们需要在页面上渲染他们的名字，以防页面忘了他们是谁。

    <!-- view.html -->
    <div class="top-bar row">
      <div class="col-md-10">
      <!-- more top bar things go here -->
      </div>
      {<removeME>% if user %}
      <div class="col-md-2 whoami">
        You are logged in as { { user.fullname }}
      </div>
      {<removeME>% endif %}
    </div>

首先，你会注意到这个  {<removeME>% if user %} 概念。在Django模板中， {<removeME>% 用来控制循环和条件的声明。这里的if user声明是为了防止那些不是用户的情况。匿名用户不应该在页面头部看到“你已经登录”的字样。

在if块内，你可以看到，包含名字非常的简单，只要用{ {}}包含着我们要插入的属性就可以了。{ {是用来向模板插入真实值的，如{ { user.fullname }}。

模板的另一个常用情况是展示一组物品，如一个电子商务网站的存货清单页面。

    def browse_shop(request):
        # get items
        return render_to_response(
            "browse.html",
            {"inventory": all_items}
        )

在模板中，我们使用同样的{<removeME>%来循环清单中的所有条目，并填入它们各自的页面地址。

    {<removeME>% for widget in inventory %}
        <li><a href="/widget/{ { widget.slug }}/">{ { widget.displayname }}</a></li>
    {<removeME>% endfor %}

为了做大部分常见的模板任务，Django可以仅仅使用很少的结构来完成目标，因此很容易上手。

##### Flask

Flask默认使用受Django启发的Jinja2模板语言，但也可以配置来使用另一门语言。不应该抱怨一个仓促的程序员分不清Django和Jinja模板。事实是，上面的Django例子在Jinja2也有效。为了不去重复相同的例子，我们来看下Jinja2比Django模板更具表现力的地方。

Jinja和Django模板都提够了过滤的特性，即传入的列表会在展示前通过一个函数。一个拥有博文类别属性的博客，可以利用过滤特性，在一个用逗号分割的列表中展示博文的类别。

    <!-- Django -->
    <div class="categories">Categories: { { post.categories|join:", " }}</div>
     
    <!-- now in Jinja -->
    <div class="categories">Categories: { { post.categories|join(", ") }}</div>

在Jinja模板语言中，可以向过滤器传入任意数量的参数，因为Jinja把它看成是 使用括号包含参数的Python函数的一个调用。Django使用冒号来分割过滤器的名字和过滤参数，这限制了参数的数目只能为一。

Jinjia和Django的for循环有点类似。我们来看看他们的不同。在Jinjia2中，for-else-endfor结构能遍历一个列表，同时也处理了没有项的情况。

    {<removeME>% for item in inventory %}
    <div class="display-item">{ { item.render() }}</div>
    {<removeME>% else %}
    <div class="display-warn">
    <h3>No items found</h3>
    <p>Try another search, maybe?</p>
    </div>
    {<removeME>% endfor %}

Django版的这个功能是一样的，但是是用for-empty-endfor而不是for-else-endfor。

    {<removeME>% for item in inventory %}
    <div class="display-item">{ { item.render }}</div>
    {<removeME>% empty %}
    <div class="display-warn">
    <h3>No items found</h3>
    <p>Try another search, maybe?</p>
    </div>
    {<removeME>% endfor %}

除了语法上的不同，Jinja2通过执行环境和高级特性提供了更多的控制。例如，它可以关闭危险的特性以安全的执行不受信任的模板，或者提前编译模板以确保它们的合法性。

##### Pyramid

与Flask类似，Pyramid支持多种模板语言（包括Jinja2和Mako），但是默认只附带一个。Pyramid使用 [Chameleon](http://docs.pylonsproject.org/projects/pyramid-chameleon/en/latest/),一个 [ZPT](http://wiki.zope.org/ZPT/FrontPage) (Zope Page Template) 模板语言的实现。我们来回头看看第一个例子，添加用户的名字到网站的顶栏。Python代码除了明确调用了 `render_template` 函数外其他看起来都差不多。

    @view_config(renderer='templates/home.pt')
    def my_view(request):
        # do stuff...
        return {'user': user}

但是我们的模板看起来有些不同。ZPT是一个基于XML得模板标准，所以我们使用了类XSLT语句来操作数据。

    <div class="top-bar row">
      <div class="col-md-10">
      <!-- more top bar things go here -->
      </div>
      <div tal:condition="user"
           tal:content="string:You are logged in as ${user.fullname}"
           class="col-md-2 whoami">
      </div>
    </div>

Chameleon对于模板操作有三种不同的命名空间。TAL（模板属性语言）提供了基本的条件语句，字符串的格式化，以及填充标签内容。上面的例子只用了TAL来完成相关工作。对于更多高级任务，就需要TALES和METAL。TALES（ 模板属性表达式语法的语言）提供了像高级字符串格式化，Python表达式评估，以及导入表达式和模板的表达式。

METAL（宏扩展模板属性语言）是Chameleon模板最强大的（和复杂的）一部分。宏是可扩展的，并能被定义为带有槽且当宏被调用时可以被填充。

---

### [利用框架行动起来](#usingTemplates)

对于各个框架，我们将通过制作一个叫做wut4lunch的应用来了解，这个应用是告诉整个互联网你午饭吃了什么的社交网络。很自由的一个起始想法，完全可以随意改变。应用将有一个简单的接口，允许用户提交他们午饭的内容，并看到其他用户吃的什么的列表。主页完成后将看起来像这样。

![](http://www.xiaoh.me/img/post-django-flask-pyramid-using.png)

##### 使用Flask的Demo应用

最短的实现用了34行Python代码和一个22行的Jinja模板。首先，我们有些管理类的任务要做，比如初始化我们的应用并拉近我们的ORM。

    from flask import Flask
     
    # For this example we'll use SQLAlchemy, a popular ORM that supports a
    # variety of backends including SQLite, MySQL, and PostgreSQL
    from flask.ext.sqlalchemy import SQLAlchemy
     
    app = Flask(__name__)
    # We'll just use SQLite here so we don't need an external database
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///test.db'
     
    db = SQLAlchemy(app)

现在我们看下我们的模型，这将和另两个样例基本一样。

    class Lunch(db.Model):
        """A single lunch"""
        id = db.Column(db.Integer, primary_key=True)
        submitter = db.Column(db.String(63))
        food = db.Column(db.String(255))

哇，相当简单。最难的部分是找到合适的 [SQLAlchemy数据类型](http://docs.sqlalchemy.org/en/rel_0_9/core/types.html)，选择数据库中String域的长度。使用我们的模型也超级简单，这在于我们将要看到 [SQLAlchemy查询语法](http://docs.sqlalchemy.org/en/latest/orm/query.html)。

构建我们的提交表单也很简单。在引入 [Flask-WTForms](https://flask-wtf.readthedocs.org/en/latest/) 和正确的域类型后，你可以看到表单看起来有点像我们的模型。主要的区别在于新的提交按钮和食物与提交者姓名域的提示。

应用中的 `SECRET_KEY` 域是被WTForms用来创建 [CSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery) 符号的。它也被 [itsdangerous](http://pythonhosted.org/itsdangerous/)（Flask内包含）用来设置cookies和其他数据。

    from flask.ext.wtf import Form
    from wtforms.fields import StringField, SubmitField
     
    app.config['SECRET_KEY'] = 'please, tell nobody'
     
    class LunchForm(Form):
        submitter = StringField(u'Hi, my name is')
        food = StringField(u'and I ate')
        # submit button will read "share my lunch!"
        submit = SubmitField(u'share my lunch!')

让表单在浏览器中显示意味着模板要有它。我们像下面那样传递进去。

    from flask import render_template
     
    @app.route("/")
    def root():
        lunches = Lunch.query.all()
        form = LunchForm()
        return render_template('index.html', form=form, lunches=lunches)

好了，发生了什么？我们得到已经用Lunch.query.all()提交的午餐列表，并实例化一个表单，让用户提交他们自己的美食之旅。为了简化，变量使用相同的名字出入模板，但这不是必须的。

    <html>
    <title>Wut 4 Lunch</title>
    <b>What are people eating?</b>
     
    <p>Wut4Lunch is the latest social network where you can tell all your friends
    about your noontime repast!</p>

这就是模板的真实情况，我们在已经吃过的午餐中循环，并在<ul>中展示他们。这几乎与我们前面看到的循环例子一样。

    <ul>
    {<removeME>% for lunch in lunches %}
    <li><strong>{ { lunch.submitter|safe }}</strong> just ate <strong>{ { lunch.food|safe }}</strong>
    {<removeME>% else %}
    <li><em>Nobody has eaten lunch, you must all be starving!</em></li>
    {<removeME>% endfor %}
    </ul>
     
    <b>What are YOU eating?</b>
     
    <form method="POST" action="/new">
        { { form.hidden_tag() }}
        { { form.submitter.label }} { { form.submitter(size=40) }}
        <br/>
        { { form.food.label }} { { form.food(size=50) }}
        <br/>
        { { form.submit }}
    </form>
    </html>

模板的<form>部分仅仅渲染我们在root()视图中传入模板的WTForm对象的表单标签和输入。当表单提交时，它将向/new提交一个POST请求，这个请求会被下面的函数处理。

    from flask import url_for, redirect
     
    @app.route(u'/new', methods=[u'POST'])
    def newlunch():
        form = LunchForm()
        if form.validate_on_submit():
            lunch = Lunch()
            form.populate_obj(lunch)
            db.session.add(lunch)
            db.session.commit()
        return redirect(url_for('root'))

在验证了表单数据后，我们把内容放入我们Model对象中，并提交到数据库。一旦我们在数据库中存了午餐，它将在人们吃过的午餐列表中出现。

    if __name__ == "__main__":
        db.create_all()  # make our sqlalchemy tables
        app.run()

最后，我们只需做（非常）少量的工作来让应用运行起来。使用SQLAlchemy，我们可以创建存储午餐的表，然后开始运行我们写的路径管理就行了。

##### 测试Django版APP

Django版wut4lunch 和Flask版有点像，但是在Django项目中被分到了好几个文件中。首先，我们看看最相似的部分：数据库模型。它和SQLAlchemy版本的唯一不同之处是声明保存文本的数据库字段有轻微的语法区别。

    # from wut4lunch/models.py
    from django.db import models
     
    class Lunch(models.Model):
        submitter = models.CharField(max_length=63)
        food = models.CharField(max_length=255)

在表单系统上。不像Flask，我们可以用Django内建的表单系统。它看起来非常像我们在Flask中使用的WTFroms模块，只是语法有点不同。

    from django import forms
    from django.http import HttpResponse
    from django.shortcuts import render, redirect
     
    from .models import Lunch
     
    # Create your views here.
     
    class LunchForm(forms.Form):
        """Form object. Looks a lot like the WTForms Flask example"""
        submitter = forms.CharField(label='Your name')
        food = forms.CharField(label='What did you eat?')

现在我们只需要构造一个LunchForm实例传递到我们的模板。

    lunch_form = LunchForm(auto_id=False)
     
    def index(request):
        lunches = Lunch.objects.all()
        return render(
            request,
            'wut4lunch/index.html',
            {
                'lunches': lunches,
                'form': lunch_form,
            }
        )

render函数是Django shortcut，以接受请求、模板路径和一个上下文的dict。与Flask的render_template类似，它也接受接入请求。

    def newlunch(request):
        l = Lunch()
        l.submitter = request.POST['submitter']
        l.food = request.POST['food']
        l.save()
        return redirect('home')

保存表单应答到数据库是不一样的，Django调用模型的 .save()方法以及处理会话管理而不是用全局数据库会话。干净利落！

Django提供了一些优雅的特性，让我们管理用户提交的午餐，因此我们可以删除那些不合适的午餐信息。Flask和Pyramid没有自动提供这些功能，而在创建一个Django应用时不需要写另一个管理页面当然也是其一个特性。开发者的时间可不免费啊！我们所要做的就是告诉Django-admin我们的模型，是在wut5lunch/admin.py中添加两行。

    from wut4lunch.models import Lunch
    admin.site.register(Lunch)

Bam。现在我们可以添加删除一些条目，而无需额外的工作。
最后，让我们看下主页模板的不同之处。

    <ul>
    {<removeME>% for lunch in lunches %}
    <li><strong>{ { lunch.submitter }}</strong> just ate <strong>{ { lunch.food }}</strong></li>
    {<removeME>% empty %}
    <em>Nobody has eaten lunch, you must all be starving!</em>
    {<removeME>% endfor %}
    </ul>

Django拥有方便的快捷方式，在你的页面中引用其他的视图。url标签可以使你重建应用中的URLs，而不需破坏视图。这个是因为url标签会主动查询视图中的URL。

    <form action="{<removeME>% url 'newlunch' %}" method="post">
      {<removeME>% csrf_token %}
      { { form.as_ul }}
      <input type="submit" value="I ate this!" />
    </form>

表单被不同的语法渲染，我们需要人工在表单主体中添加CSRF token，但这些区别更多的是装饰

##### 测试Pyramid版App

最后，我们看看用Pyramid实现的同样的程序。与Django和Flask的最大不同是模板。只需要对Jinja2做很小的改动就足以解决我们在Django中的问题。这次不是这样的，Pyramid的Chameleon模板的语法更容易让人联想到 [XSLT](http://en.wikipedia.org/wiki/XSLT) 而不是别的。

    <!-- pyramid_wut4lunch/templates/index.pt -->
    <div tal:condition="lunches">
      <ul>
        <div tal:repeat="lunch lunches" tal:omit-tag="">
          <li tal:content="string:${lunch.submitter} just ate ${lunch.food}"/>
        </div>
      </ul>
    </div>
    <div tal:condition="not:lunches">
      <em>Nobody has eaten lunch, you must all be starving!</em>
    </div>

与Django模板类似，缺少for-else-endfor结构使得逻辑稍微的更清晰了。这种情况下，我们以if-for 和 if-not-for 语句块结尾以提供同样的功能。使用{ {或{<removeME>%来控制结构和条件的Django以及AngularJS类型的模板让使用XHTML标签的模板显得很外行。

Chameleon模板类型的一大好处是你所选择的编辑器可以正确的使语法高亮，因为模板是有些得XHTML。对于Django和Flask模板来说，你的编辑器需要能够正确的支持这些模板语言高亮显示。

    <b>What are YOU eating?</b>
     
    <form method="POST" action="/newlunch">
      Name: ${form.text("submitter", size=40)}
      <br/>
      What did you eat? ${form.text("food", size=40)}
      <br/>
      <input type="submit" value="I ate this!" />
    </form>
    </html>

Pyramid中表单得转换稍微更细致些，因为pytamid_simpleform不像Django表单的form.as_ul函数那样可以自动转换所有的表单字段。

现在我们看看什么返回给应用。首先，定义我们需要得表单并呈现我们的主页。

    # pyramid_wut4lunch/views.py
    class LunchSchema(Schema):
        submitter = validators.UnicodeString()
        food = validators.UnicodeString()
     
    @view_config(route_name='home',
                 renderer='templates/index.pt')
    def home(request):
        lunches = DBSession.query(Lunch).all()
        form = Form(request, schema=LunchSchema())
        return {'lunches': lunches, 'form': FormRenderer(form)}

获取午餐的查询语法和Flask的很相似，这是因为这两个demo应用使用了流行的 [SQLAlchemy ORM](http://www.sqlalchemy.org/) 来提供持久存储。在Pyramid中，允许你直接返回模板上下文的字典，而不是要调用特殊的render函数。`@view_config` 装饰器自动将返回的上下文传入要渲染的模板。避免调用render方法使得Pyramid写的函数更加容易测试，因为它们返回的数据没有被模板渲染对象掩盖。

    @view_config(route_name='newlunch',
                 renderer='templates/index.pt',
                 request_method='POST')
    def newlunch(request):
        l = Lunch(
            submitter=request.POST.get('submitter', 'nobody'),
            food=request.POST.get('food', 'nothing'),
        )
     
        with transaction.manager:
            DBSession.add(l)
     
        raise exc.HTTPSeeOther('/')

从Pyramid的请求对象中更加容易得到表单数据，因为在我们获取时会自动将表单POST数据解析成dict。为了阻止同一时间多并发的请求数据库，ZopeTransactions模块提供了上下文管理器，对写入逻辑事物的数据库进行分组，并阻止应用的线程在各个改变时互相影响，这在你的视图共享一个全局session并接收到大量通信的情况下将会是个问题。

---

### [总结](#summary)

Pyramid是三个中最灵活的。它可以用于小的应用，正如我们所见，但它也支撑着有名的网站如Dropbox。开源社区如 [Fedora](https://fedoraproject.org/) 选择它开发应用，如他们社区中的 [徽章系统](https://fedoraproject.org/)，从项目工具中接受事件的信息，并向用户奖励成就类型的徽章。对于Pyramid的一个最常见的抱怨是，它提供了这么多的选项，以至于用它开始一个新项目很吓人。

目前最流行的框架是Django，使用它的网站列表也令人印象深刻。Bitbucket，Pinterest，Instagram，以及Onion完全或部分使用Django。对于有常见需求的网站，Django是非常理智的选择，也因此它成为中大型网站应用的流行选择。

Flask对于那些开发小项目、需要快速制作一个简单的Python支撑的网站的开发者很有用。它提供小型的统一工具，或者在已有的API上构建的简单网络接口。可以快速开发需要简单web接口并不怎么配置的后端项目使用Flask将会在前端获益，如 [jitviewer](https://bitbucket.org/pypy/jitviewer) 提供了一个web接口来检测PyPy just-in-time的编译日志。

这三个框架都能解决我们简单的需求，我们已经看到了它们的不同。这些区别不仅仅是装饰性的，它们将会改变你设计产品的方法，以及添加新特性和修复的速度。因为我们的例子很小，我们看到Flask的闪光点，以及Django在小规模应用上的笨重。Pyramid的灵活并未体现出来，因为我们的要求是一样的，但在真实场景中，新的需求会常常出现。

---

### END



