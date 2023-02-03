---
layout:     post
title:      "Python读取Config配置文件"
subtitle:   "讲解ConfigParser的基本操作"
date:       2015-12-16 13:30:15
author:     "xiaoh"
header-img: "img/post-bg-python-config.jpg"
tags:
    - Python
---

> 最近在用 `Docker` 做部署相关的工作，在连接数据库的时候需要从配置文件中读取，但有的地方数据库的端口并不确定，这样就导致每次修改都需要重新 `Build Docker Images`，觉得很麻烦，后来就想通过环境变量来指定端口号。
> 此文先讲述一下 `ConfigParser` 的基本使用方法，之后引申一下环境变量的设置和读取方法

---

### ConfigParser的基本操作

##### 读取配置文件

* `read(filename)` 直接读取配置文件内容
* `readfp(fp)` 读取配置文件，fp 为文件操作类，可以用 `open(filename)` 来填充
* `sections()` 返回所有的 section 列表
* `options(section)` 返回 section 下的所有的 option 列表
* `items(section)` 得到 section 下面的键值对
* `get(section, option)` 得到 section 中的 option 的值，string 类型返回
* `getint(section, option)` 返回类型为 int
* `getboolean(section, option)` 返回类型为 bool
* `getfloat(section, option)` 返回类型为 float

##### 修改配置文件

* `add_section(section)` 添加一个新的 section
* `set(section, option, value)` 对 section 中的 option 进行修改
* `write(fp)` 将内容写入到配置文件，fp 为文件操作类，可以用 `open(filename)` 来填充
* `remove_section(section)` 删除 section
* `remove_option(section, option)` 删除 option

##### 源码分享

直接看代码：

    import ConfigParser
    
    cf = ConfigParser.ConfigParser()
    # cf.read('test.conf')
    cf.readfp(open('test.conf'))
    
    def main():
        # return all section
        secs = cf.sections()
        print 'secs:%s' % secs
    
        # return options of the giving section
        opts = cf.options('db')
        print 'opts:%s' % opts
    
        # return all key-value pairs of the giving section
        its = cf.items('db')
        print 'its:%s' % its
    
        # read values
        host = cf.get('db', 'host')
        port = cf.getint('db', 'port')
        user = cf.get('conn', 'user')
        password = cf.get('conn', 'pass')
        print 'host:%s' % host
        print 'port:%s' % port
        print 'user:%s' % user
        print 'pass:%s' % password
    
        # modify password and write to file
        cf.set('conn', 'pass', password[::-1])  # 对password进行反转，不支持汉字
        cf.write(open('test.conf', 'w'))
    
        # add section and option
        cf.add_section('section')
        cf.set('section', 'ok', 'okstr')
        cf.write(open('test.conf', 'w'))
    
        # remove section and option
        cf.remove_section('conn')
        cf.remove_option('db', 'host')
        cf.write(open('test.conf', 'w'))
    
    if __name__ == "__main__":
        main()

输出就不粘贴了。

---

### ConfigParser高级功能

先看下新的配置文件

    $ cat test.conf
    [db]
    url = http://%(host)s:%(port)s/db
    host = 1.2.3.4
    port = 3423

介绍三种读取配置文件的工具：`RawConfigParser`、`ConfigParser`、`SafeConfigParser` 的异同点

    import ConfigParser
    
    cf = ConfigParser.ConfigParser()
    cf.read('test.conf')
    
    scf = ConfigParser.SafeConfigParser()
    scf.read('test.conf')
    
    rcf = ConfigParser.RawConfigParser()
    rcf.read('test.conf')
    
    def main():
        print 'ConfigParser Read:%s' % cf.get('db', 'url')
        print 'SafeConfigParser Read:%s' % scf.get('db', 'url')
        print 'RawConfigParser Read:%s' % rcf.get('db', 'url')
    
        cf.set('db', 'URL', 'http://%(host)s:%(port)s/URL')
        scf.set('db', 'URL', 'http://%(host)s:%(port)s/URL')
        rcf.set('db', 'URL', 'http://%(host)s:%(port)s/URL')
    
        print 'ConfigParser Read:%s' % cf.get('db', 'URL')
        print 'SafeConfigParser Read:%s' % scf.get('db', 'URL')
        print 'RawConfigParser Read:%s' % rcf.get('db', 'URL')
    
    if __name__ == "__main__":
        main()

输出为：

    ConfigParser Read:http://1.2.3.4:ENV_PORT/db
    SafeConfigParser Read:http://1.2.3.4:ENV_PORT/db
    RawConfigParser Read:http://%(host)s:%(port)s/db
    ConfigParser Read:http://1.2.3.4:ENV_PORT/URL
    SafeConfigParser Read:http://1.2.3.4:ENV_PORT/URL
    RawConfigParser Read:http://%(host)s:%(port)s/URL

---

### 环境变量

`ConfigParser` 貌似没有办法处理环境变量，我只好在读取的时候处理了一下：

    import os
    import ConfigParser
    
    cf = ConfigParser.ConfigParser()
    cf.read('test.conf')
    
    def resolveEnv(con):
        if con.startswith('ENV_'):
            return os.environ.get(con)
        return con
    
    def main():
        host = resolveEnv(cf.get('db', 'host'))
        port = resolveEnv(cf.get('db', 'port'))
    
        print 'host:%s' % host
        print 'port:%s' % port
    
    if __name__ == "__main__":
        main()

查看一下配置文件：

    $ cat test.conf
    [db]
    host = 1.2.3.4
    port = ENV_PORT

设置环境变量：

    $ export ENV_PORT=3333
    $ echo $ENV_PORT
    3333

读取配置文件：

    $ python test.py
    host:1.2.3.4
    port:3333

OK, 现在可以配置环境变量到配置文件了。

---

### END



