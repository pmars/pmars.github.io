---
layout:     post
title:      "Celery快速入门"
subtitle:   "Celery操作简介，Celery快速编程"
date:       2015-12-17 01:52:54
author:     "xiaoh"
header-img: "img/post-bg-celery-summary.jpg"
tags:
    - Celery
    - Python
---

### 目录

0. [简介](#summary)
    0. [消息中间件](#xiaoxi)
    0. [任务执行单元](#task)
    0. [任务结果存储](#save)
    0. [并发](#bingfa)
    0. [序列化](#xuliehua)
0. [安装](#install)
0. [源码分享](#code)

---

> 写这个博客准备了不少的东西，甚至绕道写了一篇 [Redis](#http://www.xiaoh.me/2015/12/16/redis-summary/) 的博客，不过这一篇也仅仅是简单的介绍和代码的分享，让我们能够最快的写出 Celery 的程序而已。

---

### [简介](#summary)

Celery 是 Python 开发的分布式任务调度模块，Celery本身不含消息服务，它使用第三方消息服务来传递任务，目前，Celery支持的消息服务有RabbitMQ、Redis甚至是数据库，当然Redis应该是最佳选择（主要因为它太简单了）

Celery 的架构由三部分组成，消息中间件（message broker），任务执行单元（worker）和任务执行结果存储（task result store）组成。

##### [消息中间件](#xiaoxi)

Celery本身不提供消息服务，但是可以方便的和第三方提供的消息中间件集成。包括，RabbitMQ, Redis, MongoDB (experimental), Amazon SQS (experimental),CouchDB (experimental), SQLAlchemy (experimental),Django ORM (experimental), IronMQ

##### [任务执行单元](#task)

Worker是Celery提供的任务执行的单元，worker并发的运行在分布式的系统节点中。

##### [任务结果存储](#save)

Task result store用来存储Worker执行的任务的结果，Celery支持以不同方式存储任务的结果，包括AMQP, Redis，memcached, MongoDB，SQLAlchemy, Django ORM，Apache Cassandra, IronCache

另外， Celery还支持不同的并发和序列化的手段

##### [并发](#bingfa)

Prefork, Eventlet, gevent, threads/single threaded

##### [序列化](#xuliehua)

pickle, json, yaml, msgpack. zlib, bzip2 compression， Cryptographic message signing 等等

---

### [安装](#install)

    $ sudo pip install celery
    $ sudo pip install celery-with-redis

    $ celery -A tasks worker --loglevel=info

tasks 是你的 python 文件的名称

### [源码分享](#code)

话不多说，先上代码

```
#!/home/xingming/pyvirt/bin/python
#-*- coding:utf-8 -*-

#############################################
# File Name: main.py
# Author: xiaoh
# Mail: p.mars@163.com·
# Created Time:  2015-12-15 20:50:46
#############################################

from flask import Flask
from tasks import *
import uuid, redis

app = Flask(__name__)
r = redis.Redis()

@app.route('/<msg>')
def message(msg):
    hello.delay(msg)
    return msg

@app.route('/task/')
def doTask():
    id = str(uuid.uuid1()).replace('-', '')[:8]
    changeValue.delay(id)
    return id

@app.route('/result/<id>')
def result(id):
    re = r.get(id)
    return re

if __name__ == "__main__":
    app.debug = True
    app.run()
```

```
#!/home/xingming/pyvirt/bin/python
#-*- coding:utf-8 -*-

#############################################
# File Name: tasks.py
# Author: xiaoh
# Mail: p.mars@163.com·
# Created Time:  2015-12-16 00:09:44
#############################################

from celery import Celery
import time, redis

celery = Celery('tasks', broker='redis://localhost:6379')
r = redis.Redis(host='127.0.0.1')

@celery.task
def hello(name):
    for i in range(3):
        print 'hello,%s' % name
        time.sleep(1)

@celery.task
def changeValue(id):
    for i in range(100):
        r.set(id, i)
        print 'id:%s' % i
        time.sleep(1)
```

上面其实是一个 Flask 的 Web 服务，调用了 Celery 去执行相关的异步操作。

代码可以分为两层：

第一种为 Flask 的 Message 函数，调用的 Celery 的 Hello 函数，这个函数没有返回，这种模式适合做批处理之类的工作，比如发个邮件，做个统计什么的

第二种则是 changeValue 这个函数，他执行了一些列的工作，之后将结果写入到了数据库中，这个数据库仍然可以称作中间件，因为他记录了中间的结果，在 Flask 那边需要再启动一个接口来获取任务执行的结果，这样就可以实现了异步接口的调用。

---

> 不多说了，看代码是最好的选择，基本上使用方式也和这个代码类似的

---

### END


