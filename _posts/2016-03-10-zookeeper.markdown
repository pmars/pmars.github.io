---
layout:     post
title:      "Zookeeper的安装使用"
subtitle:   "大数据开发之--Zookeeper总结"
date:       2016-03-10 21:21:59
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - Zookeeper
    - 大数据
---

### 目录

0. [安装方法](#install)

---

> 最近看了好多大数据相关的技术框架，Zookeeper是其中非常重要的一环，这里介绍Zookeeper的安装和使用方法

---

关于Zookeeper的所有内容都可以在 [官网](https://zookeeper.apache.org/) 找到，这也是最推荐的学习方法（如果你英语还不错的话）

### [安装方法](#install)

###### 预配置

在安装Hadoop集群之前需要做一些准备工作

* 无密码访问

我们需要在每个Cluster上面执行 `ssh-keygen -t rsa -P ''`，生成 AccessKey

之后将所有的 public key 集中到文件 `authorized_keys` 中，并将文件分发到所有的机器上

* JDK环境配置

下载JDK，并配置 `JAVA_HOME` 和 `PATH`

* 修改 `/etc/hosts` 文件

将所有机器的IP对应不同的名字（我这起的是clusterN），写入到每个机器的 `/etc/hosts` 文件内

###### 程序包下载

下载Zookeeper的话，可以从 [这个网址](http://www.apache.org/dyn/closer.cgi/zookeeper/) 随意找到一个源，进去之后最好下载一个 stable 的源，这个算是一个稳定版，比如，我写这篇文章的时候，里面只有这个 <http://apache.arvixe.com/zookeeper/stable/zookeeper-3.4.8.tar.gz> 下载地址，点击即可下载。

    wget http://apache.arvixe.com/zookeeper/stable/zookeeper-3.4.8.tar.gz
    tar -xvzf zookeeper-3.4.8.tar.gz

###### 修改配置文件

Zookeeper的配置文件默认为 conf/zoo.cfg，所以我们需要通过一个 sample 的配置文件复制一份，并加以修改。

    cp zoo_sample.cfg zoo.cfg

`vi zoo.cfg`

    dataDir=/opt/zookeeper/data
    dataLogDir=/opt/zookeeper/logs
    clientPort=2181
    tickTime=2000
    initLimit=5
    syncLimit=2
    server.1=cluster1:2888:3888
    server.2=cluster2:2888:3888
    server.3=cluster3:2888:3888

* dataDir：数据目录
* dataLogDir：日志目录
* clientPort：客户端连接端口
* tickTime：Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。
* initLimit：Zookeeper的Leader 接受客户端（Follower）初始化连接时最长能忍受多少个心跳时间间隔数。当已经超过 5个心跳的时间（也就是tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 `5*2000=10` 秒
* syncLimit：表示 Leader 与 Follower 之间发送消息时请求和应答时间长度，最长不能超过多少个tickTime 的时间长度，总的时间长度就是 `2*2000=4` 秒。
* server.A=B:C:D：其中A 是一个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号。

这个里面的 A 需要在 $dataDir/myid 里面写上对应的A，比如cluster1里面写的就是1

配置 Zookeeper 的路径

    export ZOOKEEPER_HOME=/path/to/zookeeper
    export PATH=$PATH:$ZOOKEEPER/bin

###### 启动停止状态

* 启动Zookeeper: `zkServer.sh start`
* 停止Zookeeper: `zkServer.sh stop`
* Zookeeper状态：`zkServer.sh status`

###### 补充

独立安装 Zookeeper 可以在一台机器上运行

集群安装的 Zookeeper 必须在基数台机器上运行，并且数量必须大于等于3

在查看Zookeeper状态的时候，必须将集群全部启动，否则会有问题。

Zookeeper的启动和停止都是针对单台服务器，如果想都启动或停止，必须在每台机器上都执行操作。

---

### END


