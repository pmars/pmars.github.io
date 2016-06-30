---
layout:     post
title:      "Redis简介"
subtitle:   "Redis数据库快速入门"
date:       2015-12-16 21:56:08
author:     "xiaoh"
header-img: "img/post-bg-redis-summary.jpg"
tags:
    - Redis
    - 数据库
---

> 最近在看 `celery` 的时候，用到了 `Redis`，就先了解了一下 `Redis`，所以就先写一个Redis的学习笔记

---

### 目录

0. [优势](#advantage)
0. [安装](#install)
    0. [启动](#start)
0. [数据类型](#data)
    0. [字符串](#string)
    0. [哈希](#hash)
    0. [列表](#list)
    0. [集合](#set)
    0. [有序集合](#listset)
0. [事务](#work)
0. [Python调用](#pythonpython-use)

---

`Redis` 是一个开源，先进的 `key-value` 存储，并用于构建高性能，可扩展的 `Web` 应用程序的完美解决方案。

`Redis` 从它的许多竞争继承来的三个主要特点：

* Redis数据库完全在内存中，使用磁盘仅用于持久性。
* 相比许多键值数据存储，Redis拥有一套较为丰富的数据类型。
* Redis可以将数据复制到任意数量的从服务器。

---

### [优势](#advantage)

* 异常快速：Redis的速度非常快，每秒能执行约11万集合，每秒约81000+条记录。
* 支持丰富的数据类型：Redis支持最大多数开发人员已经知道像列表，集合，有序集合，散列数据类型。这使得它非常容易解决各种各样的问题，因为我们知道哪些问题是可以处理通过它的数据类型更好。
* 操作都是原子性：所有Redis操作是原子的，这保证了如果两个客户端同时访问的Redis服务器将获得更新后的值。
* 多功能实用工具：Redis是一个多功能实用的工具，可以在例如缓存，消息，队列使用(Redis原生支持发布/订阅)，任何短暂的数据，应用程序，如Web应用程序会话，网页命中计数等。

---

### [安装](#install)

```
$ sudo apt-get update
$ sudo apt-get install redis-server
```
这样，`redis` 服务就安装在系统上了，如果你想在 `python` 中使用的话，你还需要安装

    $ pip install python-redis
    $ pip install redis

##### [启动](#start)

安装之后 `Redis` 的服务已经在后台运行了，你可以打开 `Redis` 客户端来看:

    $ redis-server

或者

    $ redis-cli

效果就自己去试吧

---

### [数据类型](#data)

##### [字符串](#string)

`Redis` 字符串是字节序列。`Redis` 字符串是二进制安全的，这意味着他们有一个已知的长度没有任何特殊字符终止，所以你可以存储任何东西，512兆为上限。

    127.0.0.1:6379> set name 'xiaoh'
    OK
    127.0.0.1:6379> get name
    "xiaoh"
    127.0.0.1:6379> del name
    (integer) 1
    127.0.0.1:6379> get name
    (nil)

##### [哈希](#hash)

`Redis` 的哈希是键值对的集合。`Redis` 的哈希值是字符串字段和字符串值之间的映射，因此它们被用来表示对象

    127.0.0.1:6379> hmset hh user xiaoh pass xiaoh add "beijing china"
    OK
    127.0.0.1:6379> hgetall hh
    1) "user"
    2) "xiaoh"
    3) "pass"
    4) "xiaoh"
    5) "add"
    6) "beijing china"

在上面的例子中的哈希数据类型，用于存储其中包含的用户的基本信息用户的对象。

这里 `hmset`，`hgetall` 作为命令，`user`,`pass`,`add` 是键。

##### [列表](#list)

`Redis` 的列表是简单的字符串列表，排序插入顺序。您可以添加元素到 `Redis` 的列表的头部或尾部。

    127.0.0.1:6379> lpush l xiaoh
    (integer) 1
    127.0.0.1:6379> lpush l xingming
    (integer) 2
    127.0.0.1:6379> lpush l huo
    (integer) 3
    127.0.0.1:6379> lrange l 0 10
    1) "huo"
    2) "xingming"
    3) "xiaoh"
    127.0.0.1:6379> rpush l me
    (integer) 4
    127.0.0.1:6379> rpush l blogs
    (integer) 5
    127.0.0.1:6379> lrange l 0 10
    1) "huo"
    2) "xingming"
    3) "xiaoh"
    4) "me"
    5) "blogs"

列表的最大长度为 2^32 - 1 元素（4294967295，每个列表中可容纳超过4十亿的元素）。

##### [集合](#set)

`Redis` 的集合是字符串的无序集合。在 `Redis` 您可以添加，删除和测试文件是否存在，操作均在O（1）的时间复杂度。

    127.0.0.1:6379> sadd s xiaoh xingming
    (integer) 2
    127.0.0.1:6379> sadd s huo
    (integer) 1
    127.0.0.1:6379> sadd s xiaoh
    (integer) 0
    127.0.0.1:6379> smembers s
    1) "xingming"
    2) "huo"
    3) "xiaoh"

**在上面的例子中 我们向 s 集合添加 'xiaoh' 两次，但由于集合元素具有唯一属性。**

集合中的元素最大数量为 2^32 - 1 （4294967295，可容纳超过4十亿元素）。

##### [有序集合](#listset)

`Redis` 的有序集合类似于 `Redis` 的集合，字符串不重复的集合。不同的是，一个有序集合的每个成员用分数，以便采取有序 `set` 命令，从最小的到最大的成员分数有关。虽然成员具有唯一性，但分数可能会重复。

    127.0.0.1:6379> zadd t 0 xiaoh
    (integer) 1
    127.0.0.1:6379> zadd t 1 xiaoh
    (integer) 0
    127.0.0.1:6379> zadd t 1 xiaoz
    (integer) 1
    127.0.0.1:6379> zadd t 0 xingming
    (integer) 1
    127.0.0.1:6379> zrangebyscore t 0 4
    1) "xingming"
    2) "xiaoh"
    3) "xiaoz"
    127.0.0.1:6379> zrangebyscore t 0 3 withscores
    1) "xingming"
    2) "0"
    3) "xiaoh"
    4) "1"
    5) "xiaoz"
    6) "1"

---

### [事务](#work)

Redis事务让一组命令在单个步骤执行。事务中有两个属性，说明如下：

在一个事务中的所有命令按顺序执行作为单个隔离操作。通过另一个客户端发出的请求在Redis的事务的过程中执行，这是不可能的。

Redis的事务具有原子性。原子意味着要么所有的命令都执行或都不执行。

Redis的事务由指令多重发起，然后需要传递在事务，而且整个事务是通过执行命令EXEC执行命令列表。

    redis 127.0.0.1:6379> MULTI
    OK
    List of commands here
    redis 127.0.0.1:6379> EXEC

举例说明如下：

    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> set name xiaoh
    QUEUED
    127.0.0.1:6379> get name
    QUEUED
    127.0.0.1:6379> invr name
    (error) ERR unknown command 'invr'
    127.0.0.1:6379> incr name
    QUEUED
    127.0.0.1:6379> del name
    QUEUED
    127.0.0.1:6379> get name
    QUEUED
    127.0.0.1:6379> exec
    (error) EXECABORT Transaction discarded because of previous errors.

中间有一次的实验失败了，结果就都没有执行, 下面一次中间有条命令错误，另一个是正常执行的。

    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> set name xiaoh
    QUEUED
    127.0.0.1:6379> get name
    QUEUED
    127.0.0.1:6379> incr name
    QUEUED
    127.0.0.1:6379> del name
    QUEUED
    127.0.0.1:6379> get name
    QUEUED
    127.0.0.1:6379> exec
    1) OK
    2) "xiaoh"
    3) (error) ERR value is not an integer or out of range
    4) (integer) 1
    5) (nil)

    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> set num 9
    QUEUED
    127.0.0.1:6379> get num
    QUEUED
    127.0.0.1:6379> incr num
    QUEUED
    127.0.0.1:6379> get num
    QUEUED
    127.0.0.1:6379> exec
    1) OK
    2) "9"
    3) (integer) 10
    4) "10"
    127.0.0.1:6379>

---

> 以上就是 `redis` 的简单操作，下面我想介绍一下，怎么在 `python` 里面使用它

---

### [Python调用](#python-use)

在安装部分已经介绍了如何安装 `Python-Redis` 的插件。这里就直接介绍怎么应用

接口方面很好理解，上面所有接受的命令，基本上在脚本里面都可以使用：

下面的内容我是我从**help**里面看到的：

    $ ipython
    $ import redis
    $ r = redis.Redis()
    $ help(r)

基本操作:

###### 列表

* ltrim(list, start, end) 删除不再这个段上的表中的数据
* lset(list, index, value) 重置表里面第 index 个值为 value
* lrange(list, start, end) 截取表
* lpush(list, \*values) 将 values 添加到列表头，如果表不存在，就创建
* lpushx(list, \*values) 和上面的一样，只是没有表的话不会创建
* llen(list) 返回 list 的长度
* lindex(list, index) 返回 list 中第 index 个值
* lrem(list, value, num=0) 从 list 里面删掉第 num 个出现的 value
* rpop(list) 删除并且返回列表中的最后一个值
* rpoplpush(src, dst) 将 src 的最后一个拿出来，放到 dst 的最前面，并返回
* rpush(list, \*values) 从后面向表里面添加数据
* rpushx(list, \*values) 如果表存在，则从后面向表里面添加数据

```
In [157]: r.lpushx('l', 'xiaoh')
Out[157]: 0

In [158]: r.lpush('l', 'xiaoh')
Out[158]: 1L

In [159]: r.rpush('l', 'xingming')
Out[159]: 2L

In [160]: r.lrange('l', 0, 10)
Out[160]: ['xiaoh', 'xingming']

In [162]: r.llen('l')
Out[162]: 2

In [163]: r.lset('l', 1, 'huo')
Out[163]: True

In [164]: r.lrange('l', 0, 10)
Out[164]: ['xiaoh', 'huo']

In [165]: r.lpushx('l', 'xiaoz')
Out[165]: 3

In [166]: r.lrange('l', 0, 10)
Out[166]: ['xiaoz', 'xiaoh', 'huo']

In [167]: r.rpop('l')
Out[167]: 'huo'
```

###### 变量

* set(name, value) 设置变量
* setnx(name, value) 设置变量之前先检查是否存在，存在就不设置了
* get(name) 获取 name 的值
* getrange(name, start, end) 获取 name 的从 start 到 end 的子串
* getset(name, value) 获取 name 的值，如果没有就将 value 赋值给 name，并返回
* setex(name, value, time) 将 value 赋值给 name，time 秒之后自动消失
* append(key, value) 将 value 追加到 key 变量的后面
* delete(\*names) 删除隶属于 names 列表中的值
* exists(name) 查看变量是否存在

```
In [168]: r.set('name', 'xiaoh')
Out[168]: True

In [169]: r.setnx('user', 'xiaoh')
Out[169]: True

In [170]: r.setnx('name', 'xiaoh')
Out[170]: False

In [171]: r.get('name')
Out[171]: 'xiaoh'

In [172]: r.getrange('name', 2, 4)
Out[172]: 'aoh'

In [173]: r.append('name', '.me')
Out[173]: 8L

In [174]: r.exists('name')
Out[174]: True

In [175]: r.get('name')
Out[175]: 'xiaoh.me'
```

###### 有序集合

* zcard(name) 返回集合大小
* zcount(name, min, max) 看比分，返回中间的数量
* zrange(name, start, end) 获得这段中间的集合元素
* zadd(lset, value, score) 将 value 和 score 作为一对数据添加到 lset 有序集合中
* zrem(name, \*values) 删除集合里面的变量

```
In [177]: r.zadd('zs', 'xiaoh', 1)
Out[177]: 1

In [179]: r.zcard('zs')
Out[179]: 1

In [180]: r.zadd('zs', 'xiaoh', 1)
Out[180]: 0

In [181]: r.zadd('zs', 'xiaoz', 1)
Out[181]: 1

In [182]: r.zadd('zs', 'xingming', 1)
Out[182]: 1

In [183]: r.zrange('zs', 0, 3)
Out[183]: ['xiaoh', 'xiaoz', 'xingming']

In [184]: r.zrem('zs', 'xingming')
Out[184]: 1
```

###### 无序集合

* sadd(name, \*values) 向集合中添加数据
* scard(name) 返回集合大小

```
In [187]: r.sadd('ss', 'xingming', 'xiaoh', 'xiaoz')
Out[187]: 3

In [188]: r.scard('ss')
Out[188]: 3
```

###### 脚本

* eval(script, numkeys, \*keys\_add\_args) 执行 lua 脚本

###### 命令

* execute\_command(\*args, \*\*options) 执行命令
* info(section=None) 返回 section 的信息，如果不指定，返回所有
* keys(pattern='\*') 返回库里面所有符合 pattern 的键值（感觉像表名）
* ping() Ping the Redis Server
* randomkey() 随机返回一个key值
* rename(src, dst) 修改key的名字
* renamenx(src, dst) 先检查是否存在dst，之后再进行修改操作
* delete(\*name) 删除键值在names里面的键

###### 哈希

* hdel(name, \*keys) 删除哈希表 name 中 keys 包含的字段
* hget(name, key) 获取哈希表 name 中 key 对应的值
* hexists(name, key) 查看哈希表 name 中是否存在 key
* hgetall(name) 返回哈希表 name 的所有内容
* hkeys(name) 返回哈希表 name 的所有键值
* hlen(name) 返回哈希表 name 的数据量
* hmget(name, \*keys) 返回哈希表 name 中键在 keys 中的值
* hmset(name, mapping) 批量设置键值对
* hset(name, key, value) 单个设置
* hsetnx(name, key, value) 如果表里面没有 key，就设置为 value，设置的话返回1，以前有返回0
* hvals(name) 返回哈希表中所有的值

###### 增量

* incr(name, amount=1) 设置 name = name + amount if name exist else amount
* incrby(name, amount-1) 和 incr 一样的功能
* incrbyfloat(name, amount=1.0) 和上面一样的功能，只不过是浮点数而已

---

### END


