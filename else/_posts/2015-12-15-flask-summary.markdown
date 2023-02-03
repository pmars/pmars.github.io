---
layout:     post
title:      "Flask学习笔记"
subtitle:   "Flask的使用方法和代码分享"
date:       2015-12-15 22:05:44
author:     "xiaoh"
header-img: "img/post-bg-flask-summary.jpg"
tags:
    - Flask
---

### 目录

0. [最快入手](#fastin)
0. [路由](#route)
0. [HTTP方法](#httphttp)
0. [静态文件](#static)
0. [请求对象](#requests)
0. [文件上传](#upload)
0. [Cookies](#cookiescookies)
0. [重定向和错误](#error)
0. [会话](#sessions)
0. [日志](#log)

---

> 本来想一起把 `flask` 和 `celery` 一起写一下的，后来看 `flask` 还有挺多的内容的，就先罗列一下 `flask` 的使用方法

---

### [最快入手](#fastin)

一个最简单的 `Flask` 的应用：

    from flask import Flask
    app = Flask(__name__)
    
    @app.route('/')
    def hello_world():
        return 'Hello World!'
    
    if __name__ == '__main__':
        app.run()

把上面的代码放到一个文件里面，我起的名字是 `main.py`，这个名字只要不是 `flask.py` 就可以，怕冲突。

    $ python main.py
     * Running on http://127.0.0.1:5000/

之后可以通过 `curl` 来看看这个接口

    $ curl http://127.0.0.1:5000
    Hello World!

具体的就不解释了，可以看下面的博客。

这段代码由于没有指定访问的 `IP`，所以只能本机访问，如果想让服务器被公开访问，需要改一下 `run()`：

    app.run(host='0.0.0.0')

如果你是在调试代码，那么打开调试模式会更好：

    app.debug = True
    app.run()

或者直接传递参数：

    app.run(debug=True)

__调试模式允许执行任意代码，所以绝对不能在生产环境 中使用调试器__

想使用其他调试器？请参阅 [使用调试器](https://dormousehole.readthedocs.org/en/latest/errorhandling.html#working-with-debuggers) 。

---

### [路由](#route)

根目录，无参数，有参数，参数规则等见代码：

    @app.route('/')
    def index():
        return 'Index Page'
    
    @app.route('/hello')
    def hello():
        return 'Hello World'
    
    @app.route('/user/<username>')
    def show_user_profile(username):
        # show the user profile for that user
        return 'User %s' % username
    
    @app.route('/post/<int:post_id>')
    def show_post(post_id):
        # show the post with the given id, the id is an integer
        return 'Post %d' % post_id

参数规则也叫转换器：

| int | float | path |
|-----|-------|--------|
| 接受整数|接受浮点数|缺省设置，也接受斜杠|

Flask 的 URL 规则都是基于 Werkzeug 的路由模块的。其背后的理念是保证漂亮的 外观和唯一的 URL 。这个理念来自于 Apache 和更早期的服务器。

假设有如下两条规则:

    @app.route('/projects/')
    def projects():
        return 'The project page'
    
    @app.route('/about')
    def about():
        return 'The about page'

第一个后面有个斜杠，第二个没有，这样在访问的时候如果 `URL` 一样的话，相安无事。**区别就是**：第一个没有斜杠访问的时候会重定向过去，第二个就直接反回 `404`

之后提到了 `URL` 构建的规则，就是说，通过 `url_for()` 函数可以查出来一个函数的 `URL`，来检查用吧，具体的请参考 [URL构建](#https://dormousehole.readthedocs.org/en/latest/quickstart.html#url)

---

### [HTTP方法](#http)

接口如果不指定访问方式，默认是 `GET`，具体还有多种，看下样例：

    @app.route('/login', methods=['GET', 'POST', 'HEAD', 'PUT', 'DELETE', 'OPTIONS'])
    def login():
        show_the_login_form()

HTTP 方法（通常也被称为“动作”）告诉服务器一个页面请求要 做 什么。以下是常见 的方法：

GET

> 浏览器告诉服务器只要 得到 页面上的信息并发送这些信息。这可能是最常见的 方法。

HEAD

> 浏览器告诉服务器想要得到信息，但是只要得到 信息头 就行了，页面内容不要。 一个应用应该像接受到一个 GET 请求一样运行，但是不传递实际的内容。在 Flask 中，你根本不必理会这个，下层的 Werkzeug 库会为你处理好。

POST

> 浏览器告诉服务器想要向 URL 发表 一些新的信息，服务器必须确保数据被保存好 且只保存了一次。 HTML 表单实际上就是使用这个访求向服务器传送数据的。

PUT

> 与 POST 方法类似，不同的是服务器可能触发多次储存过程而把旧的值覆盖掉。你 可能会问这样做有什么用？这样做是有原因的。假设在传输过程中连接丢失的情况 下，一个处于浏览器和服务器之间的系统可以在不中断的情况下安全地接收第二次 请求。在这种情况下，使用 POST 方法就无法做到了，因为它只被触发一次。

DELETE

> 删除给定位置的信息。

OPTIONS

> 为客户端提供一个查询 URL 支持哪些方法的捷径。从 Flask 0.6 开始，自动为你 实现了这个方法。

---

### [静态文件](#static)

`Flask` 提供了对页面访问的支持，不过我觉得这些我很喜欢用 `Nginx`，所以就不多说了，可以去看 [官方文档](#https://dormousehole.readthedocs.org/en/latest/quickstart.html#id6)

---

### [请求对象](#requests)

    from flask import request

通过使用 method 属性可以操作当前请求方法，通过使用 form 属性处理表单数据。以下是使用两个属性的例子:

    @app.route('/login', methods=['POST', 'GET'])
    def login():
        error = None
        if request.method == 'POST':
            if valid_login(request.form['username'],
                           request.form['password']):
                return log_the_user_in(request.form['username'])
            else:
                error = 'Invalid username/password'
        # 如果请求访求是 GET 或验证未通过就会执行下面的代码
        return render_template('login.html', error=error)

当 form 属性中不存在这个键时会发生什么？会引发一个 KeyError 。如果你不 像捕捉一个标准错误一样捕捉 KeyError ，那么会显示一个 HTTP 400 Bad Request 错误页面。因此，多数情况下你不必处理这个问题。

要操作 URL （如 ?key=value ）中提交的参数可以使用 args 属性:

    searchword = request.args.get('key', '')

用户可能会改变 URL 导致出现一个 400 请求出错页面，这样降低了用户友好度。因此， 我们推荐使用 get 或通过捕捉 KeyError 来访问 URL 参数。

完整的请求对象方法和属性参见 [request](#https://dormousehole.readthedocs.org/en/latest/api.html#flask.request) 文档

---

### [文件上传](#upload)

通过 `Flask` 来处理文件上传的请求，需要在 `HTML` 表单里面设置 `enctype="multipart/form-data"`

    from flask import request
    
    @app.route('/upload', methods=['GET', 'POST'])
    def upload_file():
        if request.method == 'POST':
            f = request.files['the_file']
            f.save('/var/www/uploads/uploaded_file.txt')
如果想要知道文件上传之前其在客户端系统中的名称，可以使用 filename 属性。但是请牢记这个值是 可以伪造的，永远不要信任这个值。如果想要把客户端的文件名作为服务器上的文件名， 可以通过 Werkzeug 提供的 secure_filename() 函数:

    from flask import request
    from werkzeug import secure_filename
    
    @app.route('/upload', methods=['GET', 'POST'])
    def upload_file():
        if request.method == 'POST':
            f = request.files['the_file']
            f.save('/var/www/uploads/' + secure_filename(f.filename))

更好的例子参见 [上传文件](#https://dormousehole.readthedocs.org/en/latest/patterns/fileuploads.html#uploading-files) 方案。

---

### [Cookies](#cookies)

能用 `Sessions` 不要用 `Cookies`

读取 `Cookies`：

```
from flask import request

@app.route('/')
def index():
    username = request.cookies.get('username')
    # 使用 cookies.get(key) 来代替 cookies[key] ，
    # 以避免当 cookie 不存在时引发 KeyError 。
```

储存 cookies:

```
from flask import make_response

@app.route('/')
def index():
    resp = make_response(render_template(...))
    resp.set_cookie('username', 'the username')
    return resp
```

注意， cookies 设置在响应对象上。通常只是从视图函数返回字符串， Flask 会把它们 转换为响应对象。如果你想显式地转换，那么可以使用 make_response() 函数，然后再修改它。

---

### [重定向和错误](#error)

使用 `redirect()` 函数可以重定向。使用 `abort()` 可以更早 退出请求，并返回错误代码:

```
from flask import abort, redirect, url_for

@app.route('/')
def index():
    return redirect(url_for('login'))

@app.route('/login')
def login():
    abort(401)
    this_is_never_executed()
```

缺省情况下每种出错代码都会对应显示一个黑白的出错页面。使用 errorhandler() 装饰器可以定制出错页面:

```
from flask import render_template

@app.errorhandler(404)
def page_not_found(error):
    return render_template('page_not_found.html'), 404
```

注意 render_template() 后面的 404 ，这表示页面对就的出错代码是 404 ，即页面不存在。缺省情况下 200 表示一切正常。

更多关于错误信息反馈的信息，请查看 [关于响应](#https://dormousehole.readthedocs.org/en/latest/quickstart.html#about-responses)

---

### [会话](#sessions)

除了请求对象之外还有一种称为 session 的对象，允许你在不同请求 之间储存信息。这个对象相当于用密钥签名加密的 cookie ，即用户可以查看你的 cookie ，但是如果没有密钥就无法修改它。

使用会话之前你必须设置一个密钥。举例说明:使用会话之前你必须设置一个密钥。举例说明:

```
from flask import Flask, session, redirect, url_for, escape, request

app = Flask(__name__)

@app.route('/')
def index():
    if 'username' in session:
        return 'Logged in as %s' % escape(session['username'])
    return 'You are not logged in'

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        session['username'] = request.form['username']
        return redirect(url_for('index'))
    return '''
        <form action="" method="post">
            <p><input type=text name=username>
            <p><input type=submit value=Login>
        </form>
    '''

@app.route('/logout')
def logout():
    # 如果会话中有用户名就删除它。
    session.pop('username', None)
    return redirect(url_for('index'))

# 设置密钥，复杂一点：
app.secret_key = 'A0Zr98j/3yX R~XHH!jmN]LWX/,?RT'
```

这里用到的 escape() 是用来转义的。如果不使用模板引擎就可以像上例 一样使用这个函数来转义。

密钥的生成方法：

```
>>> import os
>>> os.urandom(24)
```

基于 cookie 的会话的说明： Flask 会把会话对象中的值储存在 cookie 中。在打开 cookie 的情况下，如果你访问会话对象中没有的值，那么会得到模糊的错误信息：请检查 页面 cookie 的大小是否与网络浏览器所支持的大小一致。

---

### [日志](#log)

```
app.logger.debug('A value for debugging')
app.logger.warning('A warning occurred (%d apples)', 42)
app.logger.error('An error occurred')
```

---


### END


