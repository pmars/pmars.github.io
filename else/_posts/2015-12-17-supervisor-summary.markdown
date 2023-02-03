---
layout:     post
title:      "Supervisor简介"
subtitle:   "快速了解Supervisor，使用Supervisor监督进程运行"
date:       2015-12-17 20:58:54
author:     "xiaoh"
header-img: "img/post-bg-supervisor-summary.jpg"
tags:
    - Supervisor
---

### 目录

0. [简介](#summary)
0. [安装](#install)
0. [配置文件](#config)
0. [运行](#run)
0. [多进程](#multi)

---

> 今天同事的服务要做成异步的，使用到了 Flask 和 Celery 两个进程，之后要用 Supervisor 来确保另个进程正常运行，而在处理 Celery 的时候遇到了些问题，所以就研究了一下

---

### [简介](#summary)

`Supervisor` 是一个使用 `Python` 编写的进程管理工具，用途就是有一个进程需要每时每刻不断的跑，但是这个进程又有可能由于各种原因有可能中断。当进程中断的时候我希望能自动重新启动它，此时，我就需要使用到了 ·Supervisor·

这个工具主要就两个命令：

* `supervisord`  : `supervisor` 的服务器端部分，启动 `supervisor` 就是运行这个命令

* `supervisorctl`：启动 `supervisor` 的命令行窗口。

---

### [安装](#install)

    $ sudo apt-get install supervisor
    $ service supervisor status
     is running

---

### [配置文件](#config)

`supervisor` 的配置文件默认位置为 `/etc/supervisor/supervisor.conf`，你也可以在任何地方去编写，只要启动的时候指定 `-c` 参数即可。

    $ sudo vi /etc/supervisor/supervisor.conf
    [program:flask]             ; 前面的program是必须要有的，否则不好使啊，我是弯路狗。。。
    command=/home/xingming/pyvirt/bin/python /home/xingming/help/main.py  ;需要执行的命令
    user = xingming                             ;执行命令的用户，默认 root
    autostart=true                              ;是否和 supervisord 一同启动，默认 true
    autorestart=true                            ;自动重启，默认 unexpected
    startsecs=3                                 ;
    stderr_logfile=/tmp/flask.log               ;错误输出重定向
    stdout_logfile=/tmp/flask.log               ;log输出
    environment=ENV_A=aaa                       ;可以设置环境变量

`;` 是注释的意思，后面都是注释

---

### [运行](#run)

其实 Supervisor 也是一个服务而已，所以启动可以用：

    $ sudo service supervisor start
    $ sudo supervisord

上面两个命令都可以运行 supervisor，你也可以在启动的时候添加参数（例如：`-c`），具体还有其他参数可以查看 `-h`

查看一下进程是否正常运行：

    $ ps -ef | grep main
    xingming 49208     1  0 21:54 ?        00:00:02 /home/xingming/pyvirt/bin/python /home/xingming/help/main.py
    xingming 50115 20529  0 22:06 pts/5    00:00:00 grep --color=auto main

可以查看一下 flask 是否在服务：

    $ curl http://localhost:5000/helloworld
    helloworld

---

### [多进程](#multi)

给 Celery 也加入到进程中去

    [program:celery]
    command=celery -A tasks worker --loglevel=info  ;需要执行的命令wd)
    directory=/home/xingming/help
    user = xingming               
    autostart=true                 
    autorestart=true              
    startsecs=3                  
    stderr_logfile=/tmp/celery.log 
    stdout_logfile=/tmp/celery.log

可以发现，这个只是比上面的 Flask 多了一个 directory 变量而已。

---

### END



