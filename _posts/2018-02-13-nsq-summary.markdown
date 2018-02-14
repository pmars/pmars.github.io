---
layout:     post
title:      "NSQ消息队列详解"
subtitle:   "NSQ MQ Summary"
date:       2018-02-13 08:42:44
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - NSQ
    - Golang
    - MQ

---

# NSQ-异步消息队列

NSQ是Golang编写的分布式实时消息平台
+ Github地址：https://github.com/nsqio/nsq
+ NSQ官网的文档：http://nsq.io/overview/quick_start.html

### NSQ简介

NSQ可用于大规模系统中的实时消息服务，并且每天能够处理数亿级别的消息，其设计目标是为在分布式环境下运行的去中心化服务提供一个强大的基础架构。

NSQ具有分布式、去中心化的拓扑结构，该结构具有无单点故障、故障容错、高可用性以及能够保证消息的可靠传递的特征。NSQ非常容易配置和部署，且具有最大的灵活性，支持众多消息协议。

---

### NSQ组件

+ nsqd 守护进程；接收，接收，缓存，投递消息给客户端
    + 它可以独立运行，不过通常它是由 nsqlookupd 实例所在集群配置的（它在这能声明 topics 和 channels，以便大家能找到）
    + 服务启动后有两个端口：一个给客户端，另一个是 HTTP API。还能够开启HTTPS。
    + 同一台服务器启动多个nsqd，要注意端口和数据路径必须不同，包括：–lookupd-tcp-address、 -tcp-address、–data-path。
    + 删除topic、channel需要http api调用。
+ nsqlookupd 守护进程；负责管理拓扑信息并提供最终一致性的发现服务
    + 客户端通过查询 nsqlookupd 来发现指定话题（topic）的生产者，并且 nsqd 节点广播话题（topic）和通道（channel）信息
    + 该服务运行后有两个端口：TCP 接口，nsqd 用它来广播；HTTP 接口，客户端用它来发现和管理。
    + 在生产环境中，为了高可用，最好部署三个nsqlookupd服务。
+ nsqadmin（提供 Web 页面用来实时的管理你的 NSQ 集群。它通过和 nsqlookupd 实例交流，来确定生产者）
    + nsqadmin 是一套 WEB UI，用来汇集集群的实时统计，并执行不同的管理任务。
    + 运行后，能够通过4171端口查看并管理topic和channel。
    + 通常只需要运行一个。

---

### NSQ工具

+ nsq_pubsub
+ nsq_stat
+ nsq_tail
+ nsq_to_file
+ nsq_to_http
+ nsq_to_nsq
+ to_nsq

---

### NSQ核心概念

+ topic: topic是nsq的消息发布的逻辑关键词。当程序初次发布带topic的消息时,如果topic不存在,则会被创建。
+ channels: 当生产者每次发布消息的时候,消息会采用多播的方式被拷贝到各个channel中,channel起到队列的作用。
+ messages: 数据流的形式。

![](https://f.cloud.github.com/assets/187441/1700696/f1434dc8-6029-11e3-8a66-18ca4ea10aca.gif)

上图表明：

+ 没有router
    + 对于消息中间件，话题（topic）和通道（channel）是非常基本的，他们是1:N 的关系。
    + 相对于RabitMQ，NSQ没有router这一层，功能也简化了不少，因此运维非常容易上手。
+ 消费者对应Channel
    + 如果channel没有消费，将会保留。如果同一个channel有多个消费者，则会轮训，按序分配给就绪（当前无处理任务）的消费者
+ 存储
    + Topic和Channel缓冲的数据相互独立，防止缓慢消费者造成对其他通道造成积压（同样适用于话题级别）。

---

### NSQ安装（略）

http://nsq.io/deployment/installing.html
网上随便一个地方就能找到安装方式了

---

### NSQ使用

启动 nsqlookupd
> `nsqlookupd`

启动 nsqd

> `nsqd --lookupd-tcp-address=127.0.0.1:4160`

> `nsqd --lookupd-tcp-address=127.0.0.1:4160 --tcp-address=0.0.0.0:4250 --http-address=0.0.0.0:4251 --data-path=/home/steve-3/nsq/data2`

启动 nsqadmin 浏览地址：http://127.0.0.1:4171/
> `nsqadmin --lookupd-http-address=127.0.0.1:4161`

启动 nsq_to_file
> `nsq_to_file --topic=test --output-dir=./ --lookupd-http-address=127.0.0.1:4161`

向客户端写入消息
```
$ curl -d 'hello world 1' 'http://127.0.0.1:4151/put?topic=test'
$ curl -d 'hello world 2' 'http://127.0.0.1:4251/put?topic=test'
```

---

### 消息生命周期

NSQ提倡co-locating的部署方式, 也就是生产者与nsqd尽量在同一个实例环境中。生产者不需要负责发现其他的nsqd实例, 它们只管往自己的nsqd中投放消息。

##### 具体的流程方式为:

0. 生产者往本地的nsqd中发送消息.这个过程会开启一个连接, 并发送一个带有topic和消息体的PUB的命令到nsqd中. 我们假如是发送一个events的topic

0. events topic 会对消息进行copy,并多路发送到各个channel中, 我们假设有三个channel, 那么这个流程会如下图描述所示:

    ![](http://www.xiaoh.me/img/nsq-message1.png)

0. 在channel中的每条消息会被放进队列中, 直到消息被worker所消费掉, 如果队列占用的内存超出限制, 消息会被写进硬盘

0. nsqd节点会首先向nsqlookd节点广播它的位置信息, 一旦这些信息被nsqlookupd注册上, workers就会发现这些nsqd节点,包括这些节点的events topic

0. 相关过程如下图

    ![](http://www.xiaoh.me/img/nsq-lookups.png)

---

### 消息创建与接收

NSQ封了一个包：https://github.com/nsqio/go-nsq

我们可以很方便的使用NSQ

##### 发布者

消息发布，只能面向具体的nsqd服务进行。在API中对应的是nsq.Producer,直接初始化，就可以用了，非常简单：

```golang
// config是连接配置，作为发布者，不用刻意修改，在集群中足够使用
config := nsq.NewConfig()
p, err := nsq.NewProducer("127.0.0.1:4150", config)
if err != nil {
    panic(err)
}
//发布一条消息
p.Publish("test", []byte(time.Now().String()))
```

总结有以下两点：

0. 一个topic的发布者只对应一个具体的NSQD，但可以多个发布者同时向一个NSQD发送消息，他们是N:1的关系。
0. NSQD与topic是1：N的关系。

##### 消费者

消费者的理解要复杂一些，集群中最容易碰到无法接受到多节点消息的问题。结合官方多个文档及踩过的坑，需要注意：

0. consumer要接收消息，是要连接到具体的nsqd服务的。通常我们能通过封装好的方法，基于lookupd服务来获取所有的nsqd服务地址并连接。
0. 一个消费者订阅的topic分布在哪些nsqd服务中，则会直接连接。nsqd之间是绝对不会互传topic的具体数据的。下图描绘了consumer与nsqd的关系：

    ![](http://www.xiaoh.me/img/nsq-consumer.png)

0. 当多个nsqd服务都有相同的topic的时候，consumer要修改默认设置config.MaxInFlight才能连接。
0. consumer与topic没有直接联系，而是通过具体的channel接受数据。如果consumer退出，channel不会自动删除。 如果不再需要，需要通过http端口删除channel，否则很可能会导致磁盘空间不足。

> 当消费者解析数据抛出错误后，channel会requene，但间隔时间将会越来越长。

```golang
config:=nsq.NewConfig()
//最大允许向两台NSQD服务器接受消息，默认是1，要特别注意
config.MaxInFlight=2

c1, err1 := nsq.NewConsumer("test", "test-channel1", nsq.NewConfig()) // 新建一个消费者
if err1 != nil {
    panic(err1)
}

//对消息进行处理的具体方法
receive:=func(msg *nsq.Message)error{
    fmt.Println(string(msg.Body)
    return nil
}

// 添加消息处理的具体实现
c1.AddHandler(nsq.HandlerFunc(receive))

//将消费者连接到具体的NSQD
//if err := c1.ConnectToNSQD("127.0.0.1:4150"); err != nil {
//  panic(err)
//}

//或者，如果启动了Lookupd服务，可通过nsqlookupd再分发给具体的nsqd
if err := c1.ConnectToNSQLookupd("127.0.0.1:4161"); err != nil {
    panic(err)
}
```

```golang
// 简易实现
func main() {
	topicName := "publish"
	wg := &sync.WaitGroup{}
	wg.Add(1)

	config := nsq.NewConfig()
	q, _ := nsq.NewConsumer(topicName, "test_channel", config)

	q.AddHandler(nsq.HandlerFunc(func(message *nsq.Message) error {
		content := string(message.Body)
		fmt.Printf("time:%v, Got a message: %v\n", time.Now().Format("2006-01-02 15:04:05"), content)
		return nil
	}))

	err := q.ConnectToNSQD("127.0.0.1:4150")
	if err != nil {
		fmt.Printf("Could not connect\n")
	}

	wg.Wait()
}
```

##### 集群环境

```golang
package main

import (
    "github.com/nsqio/go-nsq"
    "time"
    "fmt"
    "utils/waitwraper"
)

func main() {
    var wg waitwraper.WaitWraper
    //接受消息
    consume()
    //分别向不同的服务节点发送消息
    wg.Wrap(func(){ produce("node1","localhost:4150")})
    wg.Wrap(func (){produce("node2","localhost:4152")})

    wg.Wait()
}

func produce(tag string,addr string) {
    config := nsq.NewConfig()
    p, err := nsq.NewProducer(addr, config)
    if err != nil {
        panic(err)
    }
    for {
        time.Sleep(time.Second*5)
        p.Publish("test", []byte(tag+":"+time.Now().String()))
    }
}

func consume() {
    config := nsq.NewConfig()
    //注意MaxInFlight的设置，默认只能接受一个节点
    config.MaxInFlight=2
    c, err := nsq.NewConsumer("test", "consum", config)
    if err != nil {
        panic(err)
    }
    hand := func(msg *nsq.Message) error{
        fmt.Println(string(msg.Body))
        return nil
    }
    c.AddHandler(nsq.HandlerFunc(hand))
    if err:= c.ConnectToNSQLookupd("localhost:4161");err!=nil{
        fmt.Println(err)
    }
}
```

##### 这部分有以下几点需要注意：

0. Producer断线后不会重连，需要自己手动重连，Consumer断线后会自动重连
0. Consumer的重连时间配置项有两个功能(这个设计必须吐槽一下，分开配置更好一点)
    1. Consumer检测到与nsqd的连接断开后，每隔x秒向nsqd请求重连
    2. Consumer每隔x秒，向nsqlookud进行http轮询，用来更新自己的nsqd地址目录
    3. Consumer的重连时间默认是60s(...菜都凉了)，我改成了1s
0. Consumer可以同时接收不同nsqd node的同名topic数据，为了避免混淆，就必须在客户端进行处理
0. 在AddConurrentHandlers和 AddHandler中设置的接口回调是在另外的goroutine中执行的
0. Producer不能发布(Publish)空message，否则会导致panic

### 总结

0. nsqd启动时，端口和数据存放要不同
0. 消息发送必须指定具体的某个nsqd；而消费则可以通过lookupd获取再重定向
0. 消费者接受数据时，要设置 config.MaxInFlight
0. channel在消费者退出后并不会删除，需要特别注意。如果紧紧是想利用nsq作为消息广播，不考虑离线数据保存，不妨考虑nats。
0. channel的名字，有很多限制，基本ASSCI字符+数字，以及点号”.”,下划线”_”。中文（其他非英语文字应该也不行）、以及空格、冒号”:”、横线”-“等都不得出现。
0. nsq大部分情况基本能满足我们作为消息队列的要求,而且性能与单点故障处理能力也比较出色
    + 但它不适用的地方主要有:
        1. 有顺序要求的消息
        2. 不支持副本集的集群方式

---

### END
