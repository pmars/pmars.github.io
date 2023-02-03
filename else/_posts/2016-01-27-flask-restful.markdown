---
layout:     post
title:      "Flask-Restful 快速入门"
subtitle:   "快速用Flask搭建Restfull API"
date:       2016-01-27 15:09:53
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - Flask
    - 服务器
---

> Flask-Restful 是一个基于 Flask 的简单框架，他用来创建 Restful API
> 文章简单介绍使用方法，具体细节参考 [官方文档](http://flask-restful-cn.readthedocs.org/en/0.3.5/index.html)

---

### 目录

0. [一个最小化的API](#apiminiapi)
0. [灵活的路由](#route)
0. [端点](#endpoint)
0. [参数解析](#argparse)
0. [日期格式化](#dataformat)
0. [完整的例子](#example)

---

### [一个最小化的API](#miniapi)


一个最小化的 Flask-RESTful API 看起来是这样的:

    from flask import Flask
    from flask_restful import Resource, Api

    app = Flask(__name__)
    api = Api(app)

    class HelloWorld(Resource):
        def get(self):
            return {'hello': 'world'}

    api.add_resource(HelloWorld, '/')

    if __name__ == '__main__':
        app.run(debug=True)

保存为 api.py 并且 用你的Python解释器把它运行起来. 注意我们 开启 [Flask debugging](http://flask.pocoo.org/docs/quickstart/#debug-mode) 模式去提供代码自动重载功能和更好的错误提示.

    $ python api.py
     * Running on http://127.0.0.1:5000/
     * Restarting with reloader

###### 警告

> Debug 模式千万不能在生产环境中使用!

现在开一个终端用curl去测试一下这个API的输出

    $ curl http://127.0.0.1:5000/
    {"hello": "world"}

---

### [灵活的路由](#route)


通过 Flask-RESTful 提供的主要组成部分是资源. 资源是建立在 [Flask pluggable views](http://flask.pocoo.org/docs/views/) 的基础上的, 让你非常方便的用多种HTTP方法去访问刚才创建的资源。 一个 todo 应用基本的增删改查 (当然) 看起来是这样的:

    from flask import Flask, request
    from flask_restful import Resource, Api

    app = Flask(__name__)
    api = Api(app)

    todos = {}

    class TodoSimple(Resource):
        def get(self, todo_id):
            return {todo_id: todos[todo_id]}

        def put(self, todo_id):
            todos[todo_id] = request.form['data']
            return {todo_id: todos[todo_id]}

    api.add_resource(TodoSimple, '/<string:todo_id>')

    if __name__ == '__main__':
        app.run(debug=True)

你可以这样试一下:

    $ curl http://localhost:5000/todo1 -d "data=Remember the milk" -X PUT
    {"todo1": "Remember the milk"}
    $ curl http://localhost:5000/todo1
    {"todo1": "Remember the milk"}
    $ curl http://localhost:5000/todo2 -d "data=Change my brakepads" -X PUT
    {"todo2": "Change my brakepads"}
    $ curl http://localhost:5000/todo2
    {"todo2": "Change my brakepads"}

或者你使用Python的 requests 这个库:

    >>> from requests import put, get
    >>> put('http://localhost:5000/todo1', data={'data': 'Remember the milk'}).json()
    {u'todo1': u'Remember the milk'}
    >>> get('http://localhost:5000/todo1').json()
    {u'todo1': u'Remember the milk'}
    >>> put('http://localhost:5000/todo2', data={'data': 'Change my brakepads'}).json()
    {u'todo2': u'Change my brakepads'}
    >>> get('http://localhost:5000/todo2').json()
    {u'todo2': u'Change my brakepads'}

Flask-RESTful 理解多种视图方法的返回值。 与 Flask 相似, 你可以返回任何 iterable 并且 它都会被转换成一个响应, 包括原生的Flask响应对象. Flask-RESTful 也支持 设置一个响应代码并且使用多个返回值的响应头, 如下所示:

    class Todo1(Resource):
        def get(self):
            # Default to 200 OK
            return {'task': 'Hello world'}

    class Todo2(Resource):
        def get(self):
            # Set the response code to 201
            return {'task': 'Hello world'}, 201

    class Todo3(Resource):
        def get(self):
            # Set the response code to 201 and return custom headers
            return {'task': 'Hello world'}, 201, {'Etag': 'some-opaque-string'}

---

### [端点](#endpoint)


很多情况下, 你的资源将会有很多的 URLs. 你可以通过指定 多个 URLs 给 Api 对象的 add_resource() 方法。 每一个都将指向你的 资源

    api.add_resource(HelloWorld,
        '/',
        '/hello')

你也可以把资源路径的一部分作为变量匹配在你的资源方法上。

    api.add_resource(Todo,
        '/todo/<int:todo_id>', endpoint='todo_ep')

---

### [参数解析](#argparse)


虽然 Flask 能够很方便的提供数据访问 (例如查询字符串和POST数据), 但是它仍然是非常痛苦的来验证表单. Flask-RESTful 已经内置了一种类似与库的 argparse <http://docs.python.org/dev/library/argparse.html> \_来验证数据

    from flask_restful import reqparse

    parser = reqparse.RequestParser()
    parser.add_argument('rate', type=int, help='Rate to charge for this resource')
    args = parser.parse_args()

###### 注解

> 不同与 argparse 模块, `reqparse.RequestParser.parse_args()` 返回Python字典而不是自定义数据结构。

用 [reqparse](http://flask-restful-cn.readthedocs.org/zh/latest/api.html#module-reqparse) 模块还轻松的给出了完整的错误信息。

> 如果一个数据验证失败, Flask-RESTful 会返回一个400错误和一个高亮的具体错误信息。

    $ curl -d 'rate=foo' http://127.0.0.1:5000/todos
    {'status': 400, 'message': 'foo cannot be converted to int'}

这个 inputs 模块提供许多常见的转换功能如

    inputs.date() 和 inputs.url()。

调用 `parse_args with strict=True` ，如果该请求的参数没有意义确保会抛出一个错误。

    args = parser.parse_args(strict=True)

---

### [日期格式化](#dataformat)


默认情况下, 你返回的所有字段将会原样呈现。 虽然在你处理Python数据结构的时候它 能工作得很好, 当处理对象时它会变得非常差劲。为了解决这个为题, Flask-RESTful 提供 fields 模块和 `marshal_with()` 装饰器。 类似与 Django ORM 和 WTForm, 你可以用 fields 模块去结构化你的响应数据.

    from collections import OrderedDict
    from flask_restful import fields, marshal_with

    resource_fields = {
        'task':   fields.String,
        'uri':    fields.Url('todo_ep')
    }

    class TodoDao(object):
        def __init__(self, todo_id, task):
            self.todo_id = todo_id
            self.task = task

            # This field will not be sent in the response
            self.status = 'active'

    class Todo(Resource):
        @marshal_with(resource_fields)
        def get(self, **kwargs):
            return TodoDao(todo_id='my_todo', task='Remember the milk')

上面的例子接收了一个对象，并准备将其序列化。 这个 `marshal_with()` 装饰器通过 `resource_fields` 将适用与所描述的变换。 从对象中提取的唯一字段是 task。这个 fields.Url 字段是一个特殊的字段，它接收一个端点名并且在响应端点生成URL。 许多你需要的字段类型已经包括在里面了。 点击这个 fields 向导查看一个完整的列表。

---

### [完整的例子](#example)


保存这个例子为 api.py

    from flask import Flask
    from flask_restful import reqparse, abort, Api, Resource
    
    app = Flask(__name__)
    api = Api(app)
    
    TODOS = {
        'todo1': {'task': 'build an API'},
        'todo2': {'task': '?????'},
        'todo3': {'task': 'profit!'},
    }
    
    
    def abort_if_todo_doesnt_exist(todo_id):
        if todo_id not in TODOS:
            abort(404, message="Todo {} doesn't exist".format(todo_id))
    
    parser = reqparse.RequestParser()
    parser.add_argument('task')
    
    
    # Todo
    # shows a single todo item and lets you delete a todo item
    class Todo(Resource):
        def get(self, todo_id):
            abort_if_todo_doesnt_exist(todo_id)
            return TODOS[todo_id]
    
        def delete(self, todo_id):
            abort_if_todo_doesnt_exist(todo_id)
            del TODOS[todo_id]
            return '', 204
    
        def put(self, todo_id):
            args = parser.parse_args()
            task = {'task': args['task']}
            TODOS[todo_id] = task
            return task, 201
    
    
    # TodoList
    # shows a list of all todos, and lets you POST to add new tasks
    class TodoList(Resource):
        def get(self):
            return TODOS
    
        def post(self):
            args = parser.parse_args()
            todo_id = int(max(TODOS.keys()).lstrip('todo')) + 1
            todo_id = 'todo%i' % todo_id
            TODOS[todo_id] = {'task': args['task']}
            return TODOS[todo_id], 201
    
    ##
    ## Actually setup the Api resource routing here
    ##
    api.add_resource(TodoList, '/todos')
    api.add_resource(Todo, '/todos/<todo_id>')
    
    
    if __name__ == '__main__':
        app.run(debug=True)

例子的用法

    $ python api.py
     * Running on http://127.0.0.1:5000/
     * Restarting with reloader

获取(GET)这个列表

    $ curl http://localhost:5000/todos
    {"todo1": {"task": "build an API"}, "todo3": {"task": "profit!"}, "todo2": {"task": "?????"}}

获取(GET)单一的任务

    $ curl http://localhost:5000/todos/todo3
    {"task": "profit!"}

删除(DELETE)一个任务

    $ curl http://localhost:5000/todos/todo2 -X DELETE -v
    
    > DELETE /todos/todo2 HTTP/1.1
    > User-Agent: curl/7.19.7 (universal-apple-darwin10.0) libcurl/7.19.7 OpenSSL/0.9.8l zlib/1.2.3
    > Host: localhost:5000
    > Accept: */*
    >
    * HTTP 1.0, assume close after body
    < HTTP/1.0 204 NO CONTENT
    < Content-Type: application/json
    < Content-Length: 0
    < Server: Werkzeug/0.8.3 Python/2.7.2
    < Date: Mon, 01 Oct 2012 22:10:32 GMT

添加(Add) 一个新的任务

    $ curl http://localhost:5000/todos -d "task=something new" -X POST -v
    
    > POST /todos HTTP/1.1
    > User-Agent: curl/7.19.7 (universal-apple-darwin10.0) libcurl/7.19.7 OpenSSL/0.9.8l zlib/1.2.3
    > Host: localhost:5000
    > Accept: */*
    > Content-Length: 18
    > Content-Type: application/x-www-form-urlencoded
    >
    * HTTP 1.0, assume close after body
    < HTTP/1.0 201 CREATED
    < Content-Type: application/json
    < Content-Length: 25
    < Server: Werkzeug/0.8.3 Python/2.7.2
    < Date: Mon, 01 Oct 2012 22:12:58 GMT
    <
    * Closing connection #0
    {"task": "something new"}

更新(Update)一个任务

    $ curl http://localhost:5000/todos/todo3 -d "task=something different" -X PUT -v
    
    > PUT /todos/todo3 HTTP/1.1
    > Host: localhost:5000
    > Accept: */*
    > Content-Length: 20
    > Content-Type: application/x-www-form-urlencoded
    >
    * HTTP 1.0, assume close after body
    < HTTP/1.0 201 CREATED
    < Content-Type: application/json
    < Content-Length: 27
    < Server: Werkzeug/0.8.3 Python/2.7.3
    < Date: Mon, 01 Oct 2012 22:13:00 GMT
    <
    * Closing connection #0
    {"task": "something different"}

---

### END


