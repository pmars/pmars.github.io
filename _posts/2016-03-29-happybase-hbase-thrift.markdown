---
layout:     post
title:      "使用HappyBase处理HBase数据"
subtitle:   "Happybase通过HBase的Thrift服务处理数据"
date:       2016-03-29 00:25:24
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - HappyBase
    - Python
    - HBase
    - Thrift
---

### 目录

0. [HBase的表结构](#hbasehbasetablestructure)
    0. [Row Key](#row-keyrowkey)
    0. [列族](#columnfamily)
    0. [时间戳](#timestamp)
0. [thrift简介](#thriftthriftsummary)
0. [为HBase开启Thrift服务](#hbasethriftstartthrift)
0. [HappyBase相关介绍](#happybasehappybase)

---

> 最近有需求需要在Python下处理HBase数据，查了一下，Happybase这个库还不错，就简单的学习了一下，这篇文章主要介绍了Happybase的使用方法

---

### [HBase的表结构](#hbasetablestructure)

详细的HBase文档参见 [官方文档](https://hbase.apache.org/book.html)

##### [Row Key](#rowkey)

Row key行键 (Row key)可以是任意字符串(最大长度是 64KB，实际应用中长度一般为 10-100bytes)，在hbase内部，row key保存为字节数组。

##### [列族](#columnfamily)

hbase表中的每个列，都归属与某个列族。列族是表的chema的一部分(而列不是)，必须在使用表之前定义。列名都以列族作为前缀。例如courses:history ， courses:math 都属于 courses 这个列族。

##### [时间戳](#timestamp)

HBase中通过row和columns确定的为一个存贮单元称为cell。每个 cell都保存着同一份数据的多个版本。版本通过时间戳来索引。时间戳的类型是 64位整型。时间戳可以由hbase(在数据写入时自动 )赋值，此时时间戳是精确到毫秒的当前系统时间。时间戳也可以由客户显式赋值。如果应用程序要避免数据版本冲突，就必须自己生成具有唯一性的时间戳。每个 cell中，不同版本的数据按照时间倒序排序，即最新的数据排在最前面。

为了避免数据存在过多版本造成的的管理 (包括存贮和索引)负担，hbase提供了两种数据版本回收方式。一是保存数据的最后n个版本，二是保存最近一段时间内的版本（比如最近七天）。用户可以针对每个列族进行设置。

**对Hbase而言，表结构设计会对系统的性能以及开销上造成很大的区别**

---

### [thrift简介](#thriftsummary)

一个软件框架，用来进行可扩展且跨语言的服务的开发。最初由Facebook开发，作为Hadoop的一个工具，提供跨语言服务开发；

* 资料介绍：<http://www.xiaoh.me/2016/03/28/thrift-summary/>
* 官方使用手册：<https://thrift.apache.org/>

我们项目里客户端是用python开发，因此需要Thrift提供server端，经过thrift对Hbase进行数据读写操作，性能非常不错，并且可以在Hadoop集群上做并行拓展，稳定性高，Facebook内部通信也是采用thrift来做

---

### [为HBase开启Thrift服务](#startthrift)

```Shell
./bin/start-hbase.sh
./bin/hbase-daemon.sh start thrift
```

thrift默认的监听端口是9090，可以用 `netstat -nl | grep 9090` 看看该端口是否有服务。

* 关于Thrift2和Thrift的对比可以参看这篇文章: <http://blog.csdn.net/guxch/article/details/12163047>

这里使用了thrift的服务，其实hbase还有一个thrift2的服务，而且，thrift会渐渐的被取消支持转到thrift2上面。奈何，今天我要用的是happybase，这个库是基于thrift来编写的。所以在这里使用了thrift的服务而没有选择thrift2

---

### [HappyBase相关介绍](#happybase)

相关文档可以参考：<https://happybase.readthedocs.org/en/latest/index.html>

其实文档中讲的相当详细了，所以我这里也不多说废话，简单写了一些例子，来使用happybase，简单直观的解释一下使用happybase的方法

使用pip来安装即可 `pip install happybase`

使用起来相当简单：

```Python
#!/usr/bin/python
#-*- coding:utf-8 -*-

#############################################
# File Name: happybase_examply.py
# Author: xiaoh
# Mail: xiaoh@about.me
# Created Time:  2016-03-28 10:51:20
#############################################

import happybase

hostname = '192.168.1.81'
table_name = 'xiaoh_test'
column_family = 'info'
row_key = 'row_test'

conn = happybase.Connection(hostname)

def pretty_print(msg):
    print msg.ljust(80, '*')

def show_rows(table, row_keys=None):
    if row_keys:
        pretty_print('show value of row named %s' % row_keys)
        if len(row_keys) == 1:
            print table.row(row_keys[0])
        else:
            print table.rows(row_keys)
    else:
        pretty_print('show all row values of table named %s' % table.name)
        for key, value in table.scan():
            print key, value

def put_row(table, column_family, row_key, value):
    pretty_print('insert one row to hbase')
    table.put(row_key, {'%s:name' % column_family:'name_%s' % value})

def put_rows(table, column_family, row_lines=3):
    pretty_print('insert rows to hbase now')
    for i in range(row_lines):
        put_row(table, column_family, 'row_%s' % i, i)

def delete_row(table, row_key, column_family=None, keys=None):
    if keys:
        pretty_print('delete keys:%s from row_key:%s' % (keys, row_key))
        key_list = ['%s:%s' % (column_family, key) for key in keys]
        table.delete(row_key, key_list)
    else:
        pretty_print('delete row(column_family:) from hbase')
        table.delete(row_key)

def create_table(table_name, column_family):
    pretty_print('create table %s' % table_name)
    conn.create_table(table_name, {column_family:dict()})

def show_tables():
    pretty_print('show all tables now')
    print conn.tables()

def delete_table(table_name):
    pretty_print('delete table %s now.' % table_name)
    conn.delete_table(table_name, True)

def counter(table, row_key, column_family):
    pretty_print('test counter now.')
····
    print table.counter_inc(row_key, '%s:counter' % column_family)  # prints 1
    print table.counter_inc(row_key, '%s:counter' % column_family)  # prints 2
    print table.counter_inc(row_key, '%s:counter' % column_family)  # prints 3
    print table.counter_dec(row_key, '%s:counter' % column_family)  # prints 2
    print table.counter_inc(row_key, '%s:counter' % column_family, value=3)  # prints 5
    print table.counter_get(row_key, '%s:counter' % column_family)  # prints 5
    table.counter_set(row_key, '%s:counter' % column_family, 12)
    print table.counter_get(row_key, '%s:counter' % column_family)  # prints 5

def pool():
    pretty_print('test pool connection now.')
    pool = happybase.ConnectionPool(size=3, host=hostname)
    with pool.connection() as connection:
        print connection.tables()

def main():
    show_tables()
    create_table(table_name, column_family)
    show_tables()

    table = conn.table(table_name)
    show_rows(table)
    put_rows(table, column_family)
    show_rows(table)

    put_row(table, column_family, row_key, 'xiaoh.me')
    show_rows(table, [row_key])

    delete_row(table, row_key)
    show_rows(table, [row_key])

    delete_row(table, row_key, column_family, ['name'])
    show_rows(table, [row_key])

    counter(table, row_key, column_family)

    delete_table(table_name)
    show_tables()

    pool()

    pretty_print('all works have done.')

if __name__ == "__main__":
    main()
```

输出如下:

```Shell
show all tables now*************************************************************
['SYSTEM.CATALOG', 'SYSTEM.FUNCTION', 'SYSTEM.SEQUENCE', 'SYSTEM.STATS']
create table xiaoh_test*********************************************************
show all tables now*************************************************************
['SYSTEM.CATALOG', 'SYSTEM.FUNCTION', 'SYSTEM.SEQUENCE', 'SYSTEM.STATS', 'xiaoh_test']
show all row values of table named xiaoh_test***********************************
insert rows to hbase now********************************************************
insert one row to hbase*********************************************************
insert one row to hbase*********************************************************
insert one row to hbase*********************************************************
show all row values of table named xiaoh_test***********************************
row_0 {'info:name': 'name_0'}
row_1 {'info:name': 'name_1'}
row_2 {'info:name': 'name_2'}
insert one row to hbase*********************************************************
show value of row named ['row_test']********************************************
{'info:name': 'name_xiaoh.me'}
delete row(column_family:) from hbase*******************************************
show value of row named ['row_test']********************************************
{'info:name': 'name_xiaoh.me', 'info:': ''}
delete keys:['name'] from row_key:row_test**************************************
show value of row named ['row_test']********************************************
{'info:name': '', 'info:': ''}
test counter now.***************************************************************
1
2
3
2
5
5
12
delete table xiaoh_test now.****************************************************
show all tables now*************************************************************
['SYSTEM.CATALOG', 'SYSTEM.FUNCTION', 'SYSTEM.SEQUENCE', 'SYSTEM.STATS']
test pool connection now.*******************************************************
['SYSTEM.CATALOG', 'SYSTEM.FUNCTION', 'SYSTEM.SEQUENCE', 'SYSTEM.STATS']
all works have done.************************************************************
```

---

### END


