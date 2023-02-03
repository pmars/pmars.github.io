---
layout:     post
title:      "代理服务器的搭建方法"
subtitle:   "自己搭建翻墙工具，再也不用担心被墙了"
date:       2016-01-08 17:56:36
author:     "xiaoh"
header-img: "img/post-bg-proxy-setup.jpg"
tags:
    - Proxy
    - 代理
    - 服务器
---

> 最近搞了一个代理服务器，把方法分享一下

---

### 目录

0. [squid](#squidsquid)
0. [autossh+privoxy](#autosshprivoxyautosshprivoxy)
0. [代理方法](#proxy)
0. [反向代理](#re-proxy)

---

既然是服务器，你必须有一台吧，否则玩啥。

如果是想翻墙，你的服务器还得在墙外，否则玩不起来。

所以，我起了一台 Azure 上的服务器来做代理服务器。

下面介绍一下做代理服务器的方法

---

### [squid](#squid)

前言：这个组件我安装了，用来上 google 还是可以的，但如果上 facebook 这种网站就上不去了，而且会让代理停上一段时间，我估摸着应该是绕不过墙，被封了得缘故。

不过，既然用过也将方法写在这里。

###### 安装

    $ sudo apt-get install squid3
    $ which squid3
    /usr/sbin/squid3

###### 配置

简单解释下配置里面的内容：

    acl SSL_ports port 443          # 将端口添加到 SSL_ports 变量中
    http_access deny !Safe_ports    # 阻止 SSL_ports 以外的端口进行连接（默认都可以连接？）
    http_access allow localhost     # 允许本机进行连接（所以需要修改配置，否则别人用不了）
    http_access deny all            # 阻止除已经允许的IP进行访问
    http_port 3128                  # 服务端口

其他的就没什么可以解释的了，知道这些，配置一下就可以使用了。

既然默认配置无法提供代理，得修改一下 `sudo vi /etc/squid3/squid.conf` 需要修改以下几点：

    acl my_network src 192.168.1.111 192.168.1.112
    http_access allow my_network

按照上面的解释就是，给你需要用代理的 IP 添加到 `my_network` 里面，之后设置成允许就可以了。

###### 重启服务

    $ sudo service squid3 restart

###### 测试

    你可以自己去测试一下，方式我会在下面也说一下。

---

### [autossh+privoxy](#autosshprivoxy)

这个方法是我现在使用的，实际上就是建立了一个 ssh 隧道，先看一下 `ssh隧道`:

#### SSH隧道

Internet上去主动连接一台内网是不可能的，一般的解决方案分两种，一种是端口映射（Port Forwarding），将内网主机的某个端口Open出防火墙，相当于两个外网主机通信；另一种是内网主机主动连接到外网主机，又被称作反向连接（Reverse Connection），这样NAT路由/防火墙就会在内网主机和外网主机之间建立映射，自然可以相互通信了。但是，这种映射是NAT路由自动维持的，不会持续下去，如果连接断开或者网络不稳定都会导致通信失败，这时内网主机需要再次主动连接到外网主机，建立连接。

###### 建立隧道

有两台主机，相关信息如下：

    A，外网，IP：1.1.1.1，ssh端口为1111
    B，内网，ssh端口为2222

两台主机都需要运行 `ssh daemon`

在 B 主机上运行

    $ ssh -NfR 3333:localhost:2222 user@1.1.1.1 -p 1111

这句话的意思是，将 A 主机的 3333 端口和 B 主机的 2222 端口进行绑定，相当于远程端口映射。

在连接的过程中需要输入 A 主机用户 user 的密码，可以通过设置 ssh key 来解决（将 B 主机的 public sshkey 放到 A 主机的 `authorized_keys` 文件夹中），不再多说。

这时在 A 主机上会 listen 端口 3333，你可以自己试一下。

只要上面执行成功，之后去连接 A 主机的 3333 端口，就可以操作 B 主机了。这就是所谓的 SSH隧道

###### AutoSSH

然而上面的方式并不稳定，这就需要 autossh 出场了，这个家伙可以在断开之后重连，完美解决断开问题。

#### 代理方案

那么有了这个 SSH 隧道，怎么来做代理呢，其实理解一下就可以了，既然是隧道，也就是说，从 B 主机的数据可依传到 A 主机进行执行，那么这个方式可以作为代理的。

在你的主机上可以执行以下命令:

    autossh -CND 0.0.0.0:9050 -p 1111 user@1.1.1.1

这样可以将本地的 9050 端口绑定到 A 主机上，这个就可以做代理了，可以通过下面要降到的代理方法来测试一下。

#### HTTP代理

上面这个代理方式为 socks5 类型，对于经常在控制台上活动的人来说不太方便，这就需要 privoxy 来进行转换一下。

privoxy 是一个 HTTP 代理，但这个可以将 HTTP 的代理转换为 Socks5 的。

安装 privoxy，`sudo apt-get install privoxy`，之后需要修改配置文件，`sudo vi /etc/privoxy/config` 添加

    forward-socks5  /  127.0.0.1:9050   .

这句话就是将 HTTP(端口为8118) 的数据转换为 Socks5, 之后传给 127.0.0.1:9050 ，这时，SSH 的服务正在监听 9050 这个端口，就直接发到了 A 主机，A 主机是可以连接外网的，就实现了代理的功能。

#### 测试

    $ export http_proxy=127.0.0.1:8118
    $ curl http://www.google.com | wc -l
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                       Dload  Upload   Total   Spent    Left  Speed
        100  7991  100  7991    0     0   858k      0 --:--:-- --:--:-- --:--:--  975k
        226

数据太多就直接 wc 显示了。

#### 服务执行

上面的命令会阻塞在控制台，所以最好是做成服务的形式来提供服务，具体执行方式：

待我弄好再写。。。TODO

---

### [代理方法](#proxy)

最近我在 Chrome 里面使用的代理叫：SwitchyOmega，他可以设置多个代理，之后选择自动切换代理模式，可以处理不同的网站通过不同的代理来进行连接。

---

### [反向代理](#re-proxy)

    ssh -fCNR 1122:127.0.0.1:22 bs.xiaoh.me
    ssh bs.xiaoh.me
    ssh -p 1122 127.0.0.1

---

### END


