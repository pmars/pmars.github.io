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

NSQ是Golang编写的分布式实时消息平台，Github地址：https://github.com/nsqio/nsq

### NSQ简介

NSQ可用于大规模系统中的实时消息服务，并且每天能够处理数亿级别的消息，其设计目标是为在分布式环境下运行的去中心化服务提供一个强大的基础架构。NSQ具有分布式、去中心化的拓扑结构，该结构具有无单点故障、故障容错、高可用性以及能够保证消息的可靠传递的特征。NSQ非常容易配置和部署，且具有最大的灵活性，支持众多消息协议。

### NSQ组件

+ nsqd （守护进程；接收，接收，缓存，投递消息给客户端）
    + 它可以独立运行，不过通常它是由 nsqlookupd 实例所在集群配置的（它在这能声明 topics 和 channels，以便大家能找到）
+ nsqlookupd （守护进程；为消费者提供运行时发现服务，来查找指定话题（topic）的生产者 nsqd）
+ nsqadmin（提供 Web 页面用来实时的管理你的 NSQ 集群。它通过和 nsqlookupd 实例交流，来确定生产者）

### NSQ工具

+ nsq_pubsub
+ nsq_stat
+ nsq_tail
+ nsq_to_file
+ nsq_to_http
+ nsq_to_ns
+ to_nsq

### NSQ核心概念

+ topic: topic是nsq的消息发布的逻辑关键词。当程序初次发布带topic的消息时,如果topic不存在,则会被创建。
+ channels: 当生产者每次发布消息的时候,消息会采用多播的方式被拷贝到各个channel中,channel起到队列的作用。
+ messages: 数据流的形式。

### NSQ安装（略）

http://nsq.io/deployment/installing.html
网上随便一个地方就能找到安装方式了

### NSQ使用

启动 nsqlookupd: `nsqlookupd`

启动 nsqd: `nsqd --lookupd-tcp-address=127.0.0.1:4160`

启动


---

### END
