---
layout:     post
title:      "Hadoop集群的安装使用"
subtitle:   "大数据开发之--Hadoop,Hbase,Zookeeper总结"
date:       2016-03-10 21:21:59
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - Zookeeper
    - Hadoop
    - HBase
    - 大数据
---

### 目录

0. [预配置](#frontconfig)
0. [Zookeeper](#zookeeperzookeeper)
    0. [安装方法](#install)
    0. [启动停止状态](#zookeeperstartstop)
    0. [补充](#zookeeperelse)
    0. [Zookeeper理解](#zookeeperunderstand)
0. [Hadoop](#hadoophadoop)
    0. [安装方法](#install)
    0. [启动停止状态](#hadoopcommand)
    0. [查看状态](#hadoopstatus)
0. [HBase](#hbasehbase)
    0. [安装方法](#installhbase)
    0. [修改配置](#modifyhbaseconfig)
    0. [启动](#starthbase)
    0. [测试](#hbasetest)
    0. [使用](#hbaseusing)

---

> 最近看了好多大数据相关的技术框架，这里简单总结一下一个Hadoop集群的安装过程，部分会有自己的理解。

---

### [预配置](#frontconfig)

在安装Hadoop集群之前需要做一些准备工作

* 无密码访问

我们需要在每个Cluster上面执行 `ssh-keygen -t rsa -P ''`，生成 AccessKey

之后将所有的 public key 集中到文件 `authorized_keys` 中，并将文件分发到所有的机器上

* JDK环境配置

下载JDK，并配置 `JAVA_HOME` 和 `PATH`

* 修改 `/etc/hosts` 文件

将所有机器的IP对应不同的名字（我这起的是clusterN），写入到每个机器的 `/etc/hosts` 文件内

---

### [Zookeeper](#zookeeper)

Zookeeper是其中非常重要的一环，这里介绍Zookeeper的安装和使用方法

关于Zookeeper的所有内容都可以在 [官网](https://zookeeper.apache.org/) 找到，这也是最推荐的学习方法（如果你英语还不错的话）

##### [安装方法](#install)

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

##### [启动停止状态](#zookeeperstartstop)

* 启动Zookeeper: `zkServer.sh start`
* 停止Zookeeper: `zkServer.sh stop`
* Zookeeper状态：`zkServer.sh status`

##### [补充](#zookeeperelse)

独立安装 Zookeeper 可以在一台机器上运行

集群安装的 Zookeeper 必须在基数台机器上运行，并且数量必须大于等于3

在查看Zookeeper状态的时候，必须将集群全部启动，否则会有问题。

Zookeeper的启动和停止都是针对单台服务器，如果想都启动或停止，必须在每台机器上都执行操作。

##### [Zookeeper理解](#understand)

Zookeeper可以看成是一个特殊的文件系统，我们可以在里面存储数据，但这个文件系统没有文件夹这一个说法，可以堪称是一个巨大的树，每个结点可以有自己的数据，可以有多个孩子，就容易理解他了。

具体的操作可以通过 zkCli.sh 来进行，进去之后各种操作都很容易了解。就不多说了。

---

### [Hadoop](#hadoop)

之所以先安装Zookeeper是因为在用Hadoop的时候我们需要配置Zookeeper的路径（因为我没有用Hadoop内含的Zookeeper），具体Hadoop的文档可以参看 [官方文档](http://hadoop.apache.org/)，以下是我安装部署的过程。

##### [安装方法](#install)

###### 程序包下载

和Zookeeper一样，下载Hadoop也有很多源可以选，可以到 [这里](http://hadoop.apache.org/releases.html) 选一个源来下载。下面是我下载的链接和解压的命令

```shell
wget http://apache.fayea.com/hadoop/common/hadoop-2.6.0/hadoop-2.6.0.tar.gz
tar -xvzf hadoop-2.6.0.tar.gz
```

###### 修改配置文件

Hadoop的配置文件有几个需要进行修改：

* hadoop-env.sh 修改 `JAVA_HOME`
* yarn-env.sh 修改 `JAVA_HOME`
* core-site.xml

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>·
        <value>hdfs://cluster1:9000</value>
    </property>
</configuration>
```
* hdfs-site.xml

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/xingming/package/hadoop-2.6.0/data</value>
    </property>
</configuration>
```
* mapred-site.xml

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```
* yarn-site.xml

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>cluster1</value>
    </property>
</configuration>
```
* slaves

```xml
cluster1
cluster2
cluster3
```

我的集群只用到的三台主机，所以以上配置主要就是针对三台主机的配置，如果更多的台数，配置情况会有所不同，这个需要我们注意。

* 配置Hadoop的路径

```shell
export HADOOP_HOME=/path/to/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

##### [启动停止状态](#hadoopcommand)

使用Hadoop之前我们需要先format一下，实则就是清理下数据

`hdfs namenode -format`

Hadoop 的启动可以通过 `start-all.sh` 来启动，其实际上是调用了一下两条命令：

```shell
start-dfs.sh
start-yarn.sh
```

相对应的停止命令就是 `stop-all.sh` （将start变成stop即可）

##### [查看状态](#hadoopstatus)

* http://cluster1:50070 即可查看hdfs的状态
* http://cluster1:8088 查看yarn状态
* http://cluster1:19888 查看JobHistory Server状态

或者通过各种命令来查看集群的运行状态

```shell
hdfs dfsadmin -report
yarn node -list
```

---

### [HBase](#hbase)

HBase 作为Hadoop集群的一员，官方的说法为 *A scalable, distributed database that supports structured data storage for large tables.*，具体的内容可以查看 [官方文档](http://hbase.apache.org/)

* 这里有一份关于HBase的中文文档：<http://abloz.com/hbase/book.html>
* 关于HBase的扩展阅读：<http://www.daxixiong.com/?/article/20>

##### [安装方法](#installhbase)

可以到 [这里](http://www.apache.org/dyn/closer.cgi/hbase/) 选择一个地址，之后进去找到对应的下载链接，下载一个tar包即可。

以下是我用到的下载地址和解压缩命令

```shell
wget http://apache.arvixe.com/hbase/1.1.3/hbase-1.1.3-bin.tar.gz
tar hbase-1.1.3-bin.tar.gz
```

##### [修改配置](#modifyhbaseconfig)

###### hbase-env.sh

```shell
export HBASE_MANAGES_ZK=false 是否使用内置的zookeeper
export JAVA_HOME=/path/to/jdk  配置环境变量
```

###### hbase-site.sh

```xml
<configuration>
    <property>
      <name>hbase.cluster.distributed</name>
      <value>true</value>
    </property>
    <property>
      <name>hbase.rootdir</name>
      <value>hdfs://cluster1:9000/hbase</value>
    </property>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>cluster1,cluster2,cluster3</value>
    </property>
</configuration>
```

###### regionservers

```xml
cluster2
cluster3
```

###### 复制hadoop下面的hdfs-site.xml到配置文件文件夹

###### /etc/profile

```shell
export HBASE_HOME=/path/to/hbase
export PATH=$PATH:$HBASE_HOME/bin
```

##### [启动](#starthbase)

```shell
start-hbase.sh 
```

##### [测试](#hbasetest)

```shell
(pyvirt)xingming@cluster1:~/package/hbase-1.1.3/conf$ hbase shell
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/xingming/package/hbase-1.1.3/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/xingming/package/hadoop-2.6.0/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.1.3, r72bc50f5fafeb105b2139e42bbee3d61ca724989, Sat Jan 16 18:29:00 PST 2016

hbase(main):001:0> list
TABLE
0 row(s) in 0.4020 seconds

=> []
hbase(main):002:0>
```

##### [使用](#hbaseusing)

使用 HBase 很简单的几条命令就可以:

```shell
create 'indexdemo-user', { NAME => 'info', REPLICATION_SCOPE => '1' }
hbase> put 'indexdemo-user', 'row1', 'info:firstname', 'John'
hbase> put 'indexdemo-user', 'row1', 'info:lastname', 'Smith'
```

---

### END


