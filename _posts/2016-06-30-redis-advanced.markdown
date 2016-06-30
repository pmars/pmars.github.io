---
layout:     post
title:      "Redis进阶教程"
subtitle:   "总结一下Redis的高级用法"
date:       2016-06-30 05:43:38
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - 数据库
    - Redis
---

### 目录

---

> 关于Redis的简述可以查看以前的博客 <http://www.xiaoh.me/2015/12/17/redis-summary/>

> 这篇博客会引用 python的redis库 <https://github.com/andymccurdy/redis-py>

---

### [简介](#summary)

Redis是一个开源的`key-value`数据库。它又经常被认为是一个数据结构服务器。因为它的`value`不仅包括基本的`string`类型还有`list`,`set` ,`sorted set`和`hash`类型。当然这些类型的元素也都是`string`类型。也就是说`list`,`set`这些集合类型也只能包含`string`类型。

你可以在这些类型上做很多原子性的操作。比如对一个字符`value`追加字符串（`APPEND`命令）。加加或者减减一个数字字符串(`INCR`命令，当然是按整数处理的).可以对`list`类型进行`push`,或者`pop`元素操作（可以模拟栈和队列）。对于`set`类型可以进行一些集合相关操作 (`intersection union difference`)。

`memcache`也有类似与`++`,`--`的命令。不过`memcache`的`value`只包括`string`类型。远没有`redis`的`value`类型丰富。和`memcahe`一样为了性能。`redis`的数据通常都是放到内存中的。当然`redis`可以每间隔一定时间将内存中数据写入到磁盘以防止数据丢失。

`redis`也支持主从复制机制（`master-slave replication`）。`redis`的其他特性包括简单的事务支持和发布订阅(`pub/sub`)通道功能,而且`redis`配置管理非常简单。还有各种语言版本的开源客户端类库。

一些简单的使用方法可以参照以前的博客<http://www.xiaoh.me/2015/12/17/redis-summary/>，这里就不再嗷述。

---

### [排序](#sort)

`redis`支持对`list`，`set`和`sorted set`元素的排序。排序命令是`sort` 完整的命令格式如下

    127.0.0.1:6379> help sort
    
      SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC|DESC] [ALPHA] [STORE destination]
      summary: Sort the elements in a list, set or sorted set
      since: 1.0.0
      group: generic
    
    127.0.0.1:6379>

##### sort key

这种是直接对一个数据结构进行排序

    127.0.0.1:6379> rpush l 3 4 1 5 2
    (integer) 5
    127.0.0.1:6379> lrange l 0 5
    1) "3"
    2) "4"
    3) "1"
    4) "5"
    5) "2"
    127.0.0.1:6379> sort l
    1) "1"
    2) "2"
    3) "3"
    4) "4"
    5) "5"
    127.0.0.1:6379>

对set进行添加变量的时候，他会自动排序，所以用不到sort这种简单的功能。

##### [ASC|DESC] [ALPHA]

`sort`默认的排序方式（`asc`）是从小到大排的,当然也可以按照逆序或者按字符顺序排。逆序可以加上`desc`选项，想按字母顺序排可以加`alpha`选项，当然`alpha`可以和`desc`一起用。下面是个按字母顺序排的例子

    127.0.0.1:6379> lpush l xiaoh huo xingming beijing changping
    (integer) 5
    127.0.0.1:6379> lrange l 0 5
    1) "changping"
    2) "beijing"
    3) "xingming"
    4) "huo"
    5) "xiaoh"
    127.0.0.1:6379> sort l alpha
    1) "beijing"
    2) "changping"
    3) "huo"
    4) "xiaoh"
    5) "xingming"
    127.0.0.1:6379> sort l alpha desc
    1) "xingming"
    2) "xiaoh"
    3) "huo"
    4) "changping"
    5) "beijing"
    127.0.0.1:6379>

##### [BY pattern]

Redis 支持将集合元素内容按照给定的`pattern`进行组合成新的KEY, 并按照新的KEY的对应内容进行排序，如下：

    127.0.0.1:6379> lpush l e s g f a
    (integer) 5
    127.0.0.1:6379> lrange l 0 9
    1) "a"
    2) "f"
    3) "g"
    4) "s"
    5) "e"
    127.0.0.1:6379> sort l alpha
    1) "a"
    2) "e"
    3) "f"
    4) "g"
    5) "s"
    127.0.0.1:6379> set namea 4
    OK
    127.0.0.1:6379> set namee 5
    OK
    127.0.0.1:6379> set namef 3
    OK
    127.0.0.1:6379> set nameg 1
    OK
    127.0.0.1:6379> set names 2
    OK
    127.0.0.1:6379> sort l by name*
    1) "g"
    2) "s"
    3) "f"
    4) "a"
    5) "e"
    127.0.0.1:6379> set namea xiaoh
    OK
    127.0.0.1:6379> set namee me
    OK
    127.0.0.1:6379> set namef huo
    OK
    127.0.0.1:6379> set nameg blog
    OK
    127.0.0.1:6379> set names xingming
    OK
    127.0.0.1:6379> sort l by name* alpha
    1) "g"
    2) "f"
    3) "e"
    4) "a"
    5) "s"
    127.0.0.1:6379>

以上实验了数字和字母的对应KEY，可以看出，在生成对应的新kEY之后，按照对应的内容进行了排序，并且返回的是原来的key的排序。

##### [GET PATTERN]

上面的方法返回的是原始的数据内容，如果想获得新的KEY对应的值（也就是进行排序的KEY）可以使用 GET PATTERN

    127.0.0.1:6379> sort l by name* alpha get name*
    1) "blog"
    2) "huo"
    3) "me"
    4) "xiaoh"
    5) "xingming"
    127.0.0.1:6379> sort l by name* alpha get name* get #
     1) "blog"
     2) "g"
     3) "huo"
     4) "f"
     5) "me"
     6) "e"
     7) "xiaoh"
     8) "a"
     9) "xingming"
    10) "s"
    127.0.0.1:6379>

GET 可以多次使用，上面就是一个例子，获取新的对应值的时候，将原始的内容也进行了获取。

也可以根据自己的内容排序，之后获取对应PATTERN值得列表

    127.0.0.1:6379> sort l alpha
    1) "a"
    2) "e"
    3) "f"
    4) "g"
    5) "s"
    127.0.0.1:6379> sort l alpha get name*
    1) "xiaoh"
    2) "me"
    3) "huo"
    4) "blog"
    5) "xingming"
    127.0.0.1:6379>

这里的GET还可以获取对象中的一个字段的值（hash可以作为对象来使用）使用特殊字符`->`

    127.0.0.1:6379> lpush l 5 3 4 2 1
    (integer) 5
    127.0.0.1:6379> hset user1 name this
    (integer) 1
    127.0.0.1:6379> hset user4 name xiaoh
    (integer) 1
    127.0.0.1:6379> hset user2 name blog
    (integer) 1
    127.0.0.1:6379> hset user5 name me
    (integer) 1
    127.0.0.1:6379> hset user3 name from
    (integer) 1
    127.0.0.1:6379> sort l get user*->name
    1) "this"
    2) "blog"
    3) "from"
    4) "xiaoh"
    5) "me"
    127.0.0.1:6379>

如果对应的hash对象不存在，会返回`(nil)`

##### [LIMIT start count]

默认排序结果全部显示，可以使用LIMIT来限制显示的数量，下标start默认从0开始

    127.0.0.1:6379> sort l limit 2 3
    1) "3"
    2) "4"
    3) "5"
    127.0.0.1:6379>

    127.0.0.1:6379> sort l get user*->name limit 1 3
    1) "blog"
    2) "from"
    3) "xiaoh"
    127.0.0.1:6379>

##### [STORE dstkey]

如果对集合经常按照固定的模式去排序，那么把排序结果缓存起来会减少不少`cpu`开销.使用`store`选项可以将排序内容保存到指定`key`中。保存的类型是`list`

    127.0.0.1:6379> sort l get user*->name limit 1 3 store dst
    (integer) 3
    127.0.0.1:6379> lrange dst 0 -1
    1) "blog"
    2) "from"
    3) "xiaoh"
    127.0.0.1:6379>

功能介绍完后，再讨论下关于排序的一些问题。如果我们有多个`redis server`的话，不同的`key`可能存在于不同的`server`上。比如name1 name2 name3 name4 name5，很有可能分别在多个不同的`server`上存贮着。这种情况会对排序性能造成很大的影响。

`redis`作者在他的`blog`上提到了这个问题的解决办法，就是通过`key tag`将需要排序的`key`都放到同一个`server`上 。

由于具体决定哪个`key`存在哪个服务器上一般都是在`client`端`hash`的办法来做的。我们可以通过只对`key`的部分进行`hash`.

举个例子假如我们的`client`如果发现`key`中包含`[]`。那么只对`key`中`[]`包含的内容进行`hash`。我们将四个`name`相关的`key`，都这样命名`[name]1 [name]2 [name]3 [name]4 [name]5`，于是`client`程序就会把他们都放到同一`server`上。

还有一个问题也比较严重。如果要`sort`的集合非常大的话排序就会消耗很长时间。由于`redis`单线程的，所以长时间的排序操作会阻塞其他`client`的请求。解决办法是通过主从复制机制将数据复制到多个`slave`上。然后我们只在`slave`上做排序操作。并进可能的对排序结果缓存。另外就是一个方案是就是采用`sorted set`对需要按某个顺序访问的集合建立索引。

---



