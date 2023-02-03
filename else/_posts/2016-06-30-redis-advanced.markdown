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

0. [简介](#summary)
0. [排序](#sort)
0. [事物](#transaction)
0. [管道](#pipeline)
0. [发布/订阅](#subpub)
0. [持久化](#persistence)
0. [主从复制](#master-slave)
0. [虚拟内存](#swapmemory)
0. [文档](#documents)

---

> 关于Redis的简述可以查看以前的博客 <http://www.xiaoh.me/2015/12/17/redis-summary/>


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

### [事物](#transaction)

`redis`对事务的支持目前还比较简单。`redis`只能保证一个`client`发起的事务中的命令可以连续的执行，而中间不会插入其他`client`的命令。由于`redis`是单线程来处理所有`client`的请求的所以做到这点是很容易的。

##### multi

一般情况下`redis`在接受到一个`client`发来的命令后会立即处理并返回处理结果，但是当一个`client`在一个连接中发出`multi`命令，这个连接会进入一个事务上下文，该连接后续的命令并不是立即执行，而是先放到一个队列中。当从此连接受到`exec`命令后，`redis`会顺序的执行队列中的所有命令。并将所有命令的运行结果打包到一起返回给`client`.然后此连接就结束事务上下文.

    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> incr a
    QUEUED
    127.0.0.1:6379> incr a
    QUEUED
    127.0.0.1:6379> incr b
    QUEUED
    127.0.0.1:6379> exec
    1) (integer) 1
    2) (integer) 2
    3) (integer) 1
    127.0.0.1:6379>

例子中已经说明，当执行`incr a` `incr b` 的时候，是放到了队列里面，当`exec`时，执行了队列中的命令。

当事物写到一半，可以用`discard`来取消事物

    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> incr a
    QUEUED
    127.0.0.1:6379> incr b
    QUEUED
    127.0.0.1:6379> discard
    OK
    127.0.0.1:6379> get a
    "2"
    127.0.0.1:6379> get b
    "1"
    127.0.0.1:6379>

可以发现这次`incr a` `incr b`都没被执行。`discard`命令其实就是清空事务的命令队列并退出事务上下文

##### watch

`Watch` 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

    127.0.0.1:6379> watch a
    OK
    127.0.0.1:6379> get a
    "5"
    127.0.0.1:6379> set a 6
    OK
    127.0.0.1:6379> incr a
    (integer) 7
    127.0.0.1:6379> get a
    "7"
    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> set a 2
    QUEUED
    127.0.0.1:6379> exec
    (nil)
    127.0.0.1:6379> get a
    "7"
    127.0.0.1:6379>

`exec` `discard` `unwatch` 会清除 `watch` 的监听

##### 缺点

`redis`的事务实现是如此简单，当然会存在一些问题。第一个问题是`redis`只能保证事务的每个命令连续执行，但是如果事务中的一个命令失败了，并不回滚其他命令，比如使用的命令类型不匹配。

    127.0.0.1:6379> set b 1
    OK
    127.0.0.1:6379> set a e
    OK
    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379> incr a
    QUEUED
    127.0.0.1:6379> incr b
    QUEUED
    127.0.0.1:6379> exec
    1) (error) ERR value is not an integer or out of range
    2) (integer) 2
    127.0.0.1:6379>

可以发现，虽然执行中间有问题，但并没有回滚，其他命令还是执行了。

最后一个十分罕见的问题是，当事务的执行过程中，如果`redis`意外的挂了。很遗憾只有部分命令执行了，后面的也就被丢弃了。当然如果我们使用的`append-only file`方式持久化，`redis`会用单个`write`操作写入整个事务内容。即是是这种方式还是有可能只部分写入了事务到磁盘。发生部分写入事务的情况下，`redis`重启时会检测到这种情况，然后失败退出。可以使用`redis-check-aof`工具进行修复，修复会删除部分写入的事务内容。修复完后就能够重新启动了。

---

### [管道](#pipeline)

`redis`是一个`cs`模式的`tcp server`，使用和`http`类似的请求响应协议。

一个`client`可以通过一个`socket`连接发起多个请求命令。每个请求命令发出后`client`通常会阻塞并等待`redis`服务处理，`redis`处理完后请求命令后会将结果通过响应报文返回给`client`。基本的通信过程如下

    Client: INCR X
    Server: 1
    Client: INCR X
    Server: 2
    Client: INCR X
    Server: 3
    Client: INCR X
    Server: 4

基本上四个命令需要8个`tcp`报文才能完成。由于通信会有网络延迟,假如从`client`和`server`之间的包传输时间需要0.125秒。那么上面的四个命令8个报文至少会需要1秒才能完成。这样即使`redis`每秒能处理100个命令，而我们的`client`也只能一秒钟发出四个命令。这显示没有充分利用`redis`的处理能力。除了可以利用`mget`,`mset`之类的单条命令处理多个`key`的命令外我们还可以利用`pipeline`的方式从`client`打包多条命令一起发出，不需要等待单条命令的响应返回，而`redis`服务端会处理完多条命令后会将多条命令的处理结果打包到一起返回给客户端。通信过程如下

    Client: INCR X
    Client: INCR X
    Client: INCR X
    Client: INCR X
    Server: 1
    Server: 2
    Server: 3
    Server: 4

假设不会因为`tcp`报文过长而被拆分。可能两个`tcp`报文就能完成四条命令,`client`可以将四个`incr`命令放到一个`tcp`报文一起发送，`server`则可以将四条命令的处理结果放到一个`tcp`报文返回。

通过`pipeline`方式当有大批量的操作时候。我们可以节省很多原来浪费在网络延迟的时间。需要注意到是用`pipeline`方式打包命令发送，`redis`必须在处理完所有命令前先缓存起所有命令的处理结果。打包的命令越多，缓存消耗内存也越多。所以并是不是打包的命令越多越好。具体多少合适需要根据具体情况测试。

下面是我测试是否使用pipeline的代码：

    from redis import Redis
    import time

    r = Redis()

    def without_pipeline(times=100000):
        r.set('a', 0)
        for i in range(times):
            r.incr('a')

    def use_pipeline(times=100000):
        r.set('a', 0)
        r.set('a', 0)
        pip = r.pipeline()
        for i in range(times):
            pip.incr('a')
        pip.execute()

    start = time.time()
    without_pipeline()
    end = time.time()
    print 'without pipeline spendtime:%f' % (end-start)

    start = time.time()
    use_pipeline()
    end = time.time()
    print 'use pipeline spendtime:%f' % (end-start)

结果为：

    without pipeline spendtime:4.221240
    use pipeline spendtime:1.572646

看起来对效果提升还是很明显的。

---

### [发布/订阅](#subpub)

发布订阅(`pub/sub`)是一种消息通信模式，主要的目的是解耦消息发布者和消息订阅者之间的耦合，这点和设计模式中的观察者模式比较相似。

`pub/sub`不仅仅解决发布者和订阅者直接代码级别耦合也解决两者在物理部署上的耦合。`redis`作为一个`pub/sub server`，在订阅者和发布者之间起到了消息路由的功能。订阅者可以通过`subscribe`和`psubscribe`命令向`redis server`订阅自己感兴趣的消息类型，`redis`将消息类型称为通道(`channel`)。当发布者通过`publish`命令向`redis server`发送特定类型的消息时。订阅该消息类型的全部`client`都会收到此消息。这里消息的传递是多对多的。一个`client`可以订阅多个`channel`,也可以向多个`channel`发送消息。

`Pub/Sub`功能（`means Publish`, `Subscribe`）即发布及订阅功能。基于事件的系统中，`Pub/Sub`是目前广泛使用的通信模型，它采用事件作为基本的通信机制，提供大规模系统所要求的松散耦合的交互模式：订阅者(如客户端)以事件订阅的方式表达出它有兴趣接收的一个事件或一类事件；发布者(如服务器)可将订阅者感兴趣的事件随时通知相关订阅者。

Pub/Sub是可适用于可扩展要求高、松散耦合系统的分布式交互模型

在抽象层中，它的时间非耦合、空间非耦合和同步非耦合性可允许参与者不依赖另一个而独立操作，具有一定的可扩展性；然而在实现层，可扩展性仍受其他原因的牵制。

* 灵活的订阅要求复杂的过滤和路由算法
* 高可用性开销（事件侦听、日志重传）；
* 消息认可带来的网络流量消耗；
* 庞大的订阅者数据带来的系统开销；

基于事件的Pub/Sub中间件的开发与利用在一定程度上可以提高系统的效率。

以下是我的测试：

    # code
    import time, redis
    
    r = redis.StrictRedis()
    p = r.pubsub()
    channel = 'channel'
    
    def handler(message):
        print 'Receive msg:%s' % message['data']
    
    p.subscribe(**{channel:handler})
    
    thread = p.run_in_thread(sleep_time=0.01)
    
    time.sleep(60)
    thread.stop()
    
    # running script
    python pubsub.py
    
    # ipython
    In [2]: import redis
    In [3]: r = redis.StrictRedis()
    In [5]: r.publish('channel', 'xiaoh.me')
    Out[5]: 1L
    In [6]: r.publish('channel', 'my name is xiaoh')
    Out[6]: 1L
    
    # script output
    Receive msg:xiaoh.me
    Receive msg:my name is xiaoh

以上进行了简单的测试。

---

### [持久化](#persistence)

`Redis`是一个支持持久化的内存数据库，也就是说`redis`需要经常将内存中的数据同步到磁盘来保证持久化。`redis`支持两种持久化方式，一种是`Snapshotting`（快照）也是默认方式，另一种是`Append-only file`（缩写aof）的方式。下面分别介绍

##### Snapshotting

快照是默认的持久化方式。这种方式是就是将内存中数据以快照的方式写入到二进制文件中,默认的文件名为`dump.rdb`。可以通过配置设置自动做快照持久化的方式。我们可以配置`redis`在n秒内如果超过m个key被修改就自动做快照，下面是默认的快照保存配置

    save 900 1  #900秒内如果超过1个key被修改，则发起快照保存
    save 300 10 #300秒内容如超过10个key被修改，则发起快照保存
    save 60 10000

下面介绍详细的快照保存过程

1. `redis`调用`fork`,现在有了子进程和父进程。
2. 父进程继续处理`client`请求，子进程负责将内存内容写入到临时文件。由于os的写时复制机制（`copy on write`)父子进程会共享相同的物理页面，当父进程处理写请求时os会为父进程要修改的页面创建副本，而不是写共享的页面。所以子进程的地址空间内的数据是`fork`时刻整个数据库的一个快照。
3. 当子进程将快照写入临时文件完毕后，用临时文件替换原来的快照文件，然后子进程退出。

`client`也可以使用`save`或者`bgsave`命令通知`redis`做一次快照持久化。

`save`操作是在主线程中保存快照的，由于`redis`是用一个主线程来处理所有 `client`的请求，这种方式会阻塞所有`client`请求。所以不推荐使用。另一点需要注意的是，每次快照持久化都是将内存数据完整写入到磁盘一次，并不是增量的只同步脏数据。如果数据量大的话，而且写操作比较多，必然会引起大量的磁盘io操作，可能会严重影响性能。

另外由于快照方式是在一定间隔时间做一次的，所以如果`redis`意外`down`掉的话，就会丢失最后一次快照后的所有修改。如果应用要求不能丢失任何修改的话，可以采用`aof`持久化方式。

##### Append-only file

`aof`比快照方式有更好的持久化性，是由于在使用`aof`持久化方式时,`redis`会将每一个收到的写命令都通过`write`函数追加到文件中(默认是 `appendonly.aof`)。

当`redis`重启时会通过重新执行文件中保存的写命令来在内存中重建整个数据库的内容。当然由于os会在内核中缓存`write`做的修改，所以可能不是立即写到磁盘上。这样aof方式的持久化也还是有可能会丢失部分修改。不过我们可以通过配置文件告诉`redis`我们想要通过`fsync`函数强制os写入到磁盘的时机。有三种方式如下（默认是：每秒fsync一次）

* appendonly yes           //启用aof持久化方式
* appendfsync always       //每次收到写命令就立即强制写入磁盘，最慢的，但是保证完全的持久化，不推荐使用
* appendfsync everysec     //每秒钟强制写入磁盘一次，在性能和持久化方面做了很好的折中，推荐
* appendfsync no           //完全依赖os，性能最好,持久化没保证

`aof`的方式也同时带来了另一个问题。持久化文件会变的越来越大。例如我们调用`incr test`命令100次，文件中必须保存全部的100条命令，其实有99条都是多余的。因为要恢复数据库的状态其实文件中保存一条`set test 100`就够了。为了压缩aof的持久化文件。`redis`提供了`bgrewriteaof`命令。收到此命令`redis`将使用与快照类似的方式将内存中的数据以命令的方式保存到临时文件中，最后替换原来的文件。具体过程如下

1. `redis`调用`fork` ，现在有父子两个进程
2. 子进程根据内存中的数据库快照，往临时文件中写入重建数据库状态的命令
3. 父进程继续处理`client`请求，除了把写命令写入到原来的`aof`文件中。同时把收到的写命令缓存起来。这样就能保证如果子进程重写失败的话并不会出问题。
4. 当子进程把快照内容写入已命令方式写到临时文件中后，子进程发信号通知父进程。然后父进程把缓存的写命令也写入到临时文件。
5. 现在父进程可以使用临时文件替换老的`aof`文件，并重命名，后面收到的写命令也开始往新的aof文件中追加。

需要注意到是重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件,这点和快照有点类似。

---

### [主从复制](#master-slave)

`Redis`主从复制配置和使用都非常简单。通过主从复制可以允许多个`slave server`拥有和`master server`相同的数据库副本。

关于`redis`主从复制的一些特点

1. `master`可以有多个`slave`
2. 除了多个`slave`连到相同的`master`外，`slave`也可以连接其他`slave`形成图状结构
3. 主从复制不会阻塞`master`。也就是说当一个或多个`slave`与`master`进行初次同步数据时，`master`可以继续处理`client`发来的请求。相反`slave`在初次同步数据时则会阻塞不能处理client的请求。
4. 主从复制可以用来提高系统的可伸缩性,我们可以用多个`slave`专门用于`client`的读请求，比如`sort`操作可以使用`slave`来处理。也可以用来做简单的数据冗余
5. 可以在`master`禁用数据持久化，只需要注释掉`master`配置文件中的所有`save`配置，然后只在`slave`上配置数据持久化。

下面介绍下主从复制的过程

当设置好`slave`服务器后，`slave`会建立和`master`的连接，然后发送`sync`命令。

无论是第一次同步建立的连接还是连接断开后的重新连接，`master`都会启动一个后台进程，将数据库快照保存到文件中，同时`master`主进程会开始收集新的写命令并缓存起来。后台进程完成写文件后，`master`就发送文件给`slave`，`slave`将文件保存到磁盘上，然后加载到内存恢复数据库快照到`slave`上。

接着`master`就会把缓存的命令转发给`slave`。而且后续`master`收到的写命令都会通过开始建立的连接发送给`slave`。

从`master`到`slave`的同步数据的命令和从`client`发送的命令使用相同的协议格式。当`master`和`slave`的连接断开时`slave`可以自动重新建立连接。

如果`master`同时收到多个`slave`发来的同步连接命令，只会使用启动一个进程来写数据库镜像，然后发送给所有`slave`。

配置slave服务器很简单，只需要在配置文件中加入如下配置

    slaveof 192.168.1.1 6379  #指定master的ip和端口

---

### [虚拟内存](#swapmemory)

首先说明下`redis`的虚拟内存与`os`的虚拟内存不是一码事，但是思路和目的都是相同的。就是暂时把不经常访问的数据从内存交换到磁盘中，从而腾出宝贵的内存空间用于其他需要访问的数据。尤其是对于`redis`这样的内存数据库，内存总是不够用的。除了可以将数据分割到多个`redis server`外。另外的能够提高数据库容量的办法就是使用`vm`把那些不经常访问的数据交换的磁盘上。

如果我们的存储的数据总是有少部分数据被经常访问，大部分数据很少被访问，对于网站来说确实总是只有少量用户经常活跃。当少量数据被经常访问时，使用vm不但能提高单台`redis server`数据库的容量，而且也不会对性能造成太多影响。

`redis`没有使用`os`提供的虚拟内存机制而是自己在用户态实现了自己的虚拟内存机制,作者在自己的`blog`专门解释了其中原因。<http://antirez.com/post/redis-virtual-memory-story.html>

##### 主要的理由有两点

1. `os`的虚拟内存是已4k页面为最小单位进行交换的。而`redis`的大多数对象都远小于4k，所以一个os页面上可能有多个`redis`对象。另外`redis`的集合对象类型如`list`,`set`可能存在与多个os页面上。最终可能造成只有10%key被经常访问，但是所有`os`页面都会被`os`认为是活跃的，这样只有内存真正耗尽时os才会交换页面。
2. 相比于os的交换方式。`redis`可以将被交换到磁盘的对象进行压缩,保存到磁盘的对象可以去除指针和对象元数据信息。一般压缩后的对象会比内存中的对象小10倍。这样`redis`的vm会比os vm能少做很多io操作。

##### 下面是vm相关配置

* vm-enabled yes                    #开启vm功能
* vm-swap-file /tmp/redis.swap      #交换出来的value保存的文件路径/tmp/redis.swap
* vm-max-memory 1000000             #redis使用的最大内存上限，超过上限后redis开始交换value到磁盘文件中。
* vm-page-size 32                   #每个页面的大小32个字节
* vm-pages 134217728                #最多使用在文件中使用多少页面,交换文件的大小 = vm-page-size * vm-pages
* vm-max-threads 4                  #用于执行value对象换入换出的工作线程数量。0表示不使用工作线程（后面介绍)

`redis`的`vm`在设计上为了保证`key`的查找速度，只会将`value`交换到`swap`文件中。所以如果是内存问题是由于太多`value`很小的`key`造成的，那么`vm`并不能解决。

和os一样`redis`也是按页面来交换对象的。`redis`规定同一个页面只能保存一个对象。但是一个对象可以保存在多个页面中。 

在`redis`使用的内存没超过`vm-max-memory`之前是不会交换任何`value`的。当超过最大内存限制后，`redis`会选择较老的对象。如果两个对象一样老会优先交换比较大的对象，精确的公式`swappability = age*log(size_in_memory)`。 

对于`vm-page-size`的设置应该根据自己的应用将页面的大小设置为可以容纳大多数对象的大小。太大了会浪费磁盘空间，太小了会造成交换文件出现碎片。

对于交换文件中的每个页面，`redis`会在内存中对应一个`1bit`值来记录页面的空闲状态。所以像上面配置中页面数量(`vm-pages 134217728`)会占用16M内存用来记录页面空闲状态。`

vm-max-threads`表示用做交换任务的线程数量。如果大于0推荐设为服务器的`cpu core`的数量。如果是0则交换过程在主线程进行。

##### 参数配置讨论完后，在来简单介绍下vm是如何工作的，

###### 当`vm-max-threads`设为0时(`Blocking VM`)

** 换出 **

主线程定期检查发现内存超出最大上限后，会直接已阻塞的方式,将选中的对象保存到`swap`文件中，并释放对象占用的内存,此过程会一直重复直到下面条件满足

1. 内存使用降到最大限制以下
2. `swap`文件满了
3. 几乎全部的对象都被交换到磁盘了

** 换入 **

当有`client`请求`value`被换出的`key`时。主线程会以阻塞的方式从文件中加载对应的`value`对象，加载时此时会阻塞所以`client`。然后处理`client`的请求

###### 当`vm-max-threads`大于0(`Threaded VM`)

** 换出 **

当主线程检测到使用内存超过最大上限，会将选中的要交换的对象信息放到一个队列中交由工作线程后台处理，主线程会继续处理`client`请求。

** 换入 **

如果有`client`请求的`key`被换出了，主线程先阻塞发出命令的`client`,然后将加载对象的信息放到一个队列中，让工作线程去加载。加载完毕后工作线程通知主线程。主线程再执行`client`的命令。这种方式只阻塞请求`value`被换出`key`的`client`

总的来说`blocking vm`的方式总的性能会好一些，因为不需要线程同步，创建线程和恢复被阻塞的`client`等开销。但是也相应的牺牲了响应性。`threaded vm`的方式主线程不会阻塞在磁盘io上，所以响应性更好。

如果我们的应用不太经常发生换入换出，而且也不太在意有点延迟的话则推荐使用`blocking vm`的方式。

关于`redis vm`的更详细介绍可以参考下面链接

* <http://antirez.com/post/redis-virtual-memory-story.html>
* <http://redis.io/topics/internals-vm>

---

### [文档](#documents)

* <http://redisdoc.com/index.html>
* <https://github.com/andymccurdy/redis-py>

---

### END


