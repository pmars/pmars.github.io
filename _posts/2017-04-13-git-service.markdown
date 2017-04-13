---
layout:     post
title:      "搭建GIT仓库服务器方法"
subtitle:   "自己搭建自己的GIT仓库"
date:       2017-04-13 20:27:20
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - git
---

### 目录

0. [简介](#summary)
    0. [安装GIT](#installgit)
    0. [创建用户](#adduser)
    0. [创建证书登录](#authorized)
    0. [初始化GIT仓库](#initgit)
    0. [禁用shell登录](#loginshell)
    0. [克隆远程仓库](#clone)
0. [参考资料](#links)

---

> 从几年前在使用.NET语言的时候使用正则，到现在，有的时候用的多一些，有的时候用的比较少，不过，都没有离开正则，毕竟工作中很多内容都是在和字符串打交道。

---

### [简介](#summary)

GitHub就是一个免费托管开源代码的远程仓库。但是对于某些视源代码如生命的商业公司来说，既不想公开源代码，又舍不得给GitHub交保护费，那就只能自己搭建一台Git服务器作为私有仓库使用。

搭建Git服务器需要准备一台运行Linux的机器，强烈推荐用Ubuntu或Debian，这样，通过几条简单的apt命令就可以完成安装。

假设你已经有`sudo`权限的用户账号，下面，正式开始安装。

---

##### [安装GIT](#install)

```shell
$ sudo apt-get install git
```

##### [创建用户](#adduser)

创建一个`git`用户,用来运行`git`服务

```shell
$ sudo adduser git
```

##### [创建证书登录](#authorized)

收集所有需要登录的用户的公钥，就是他们自己的`id_rsa.pub`文件，把所有公钥导入到`/home/git/.ssh/authorized_keys`文件里，一行一个。

##### [初始化GIT仓库](#init)

先选定一个目录作为Git仓库，假定是/srv/sample.git，在/srv目录下输入命令:

```shell
$ sudo git init --bare sample.git
```

`Git`就会创建一个裸仓库，裸仓库没有工作区，因为服务器上的`Git`仓库纯粹是为了共享，所以不让用户直接登录到服务器上去改工作区，并且服务器上的`Git`仓库通常都以`.git`结尾。然后，把`owner`改为`git`：

```shell
$ sudo chown -R git:git sample.git
```

##### [禁用shell登录](#login)

出于安全考虑，第二步创建的`git`用户不允许登录`shell`，这可以通过编辑`/etc/passwd`文件完成。找到类似下面的一行：

```shell
git:x:1001:1001:,,,:/home/git:/bin/bash
改为
git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
```

这样，`git`用户可以正常通过`ssh`使用`git`，但无法登录`shell`，因为我们为`git`用户指定的`git-shell`每次一登录就自动退出。

##### [克隆远程仓库](#clone)

现在，可以通过`git clone`命令克隆远程仓库了，在各自的电脑上运行：

```shell
$ git clone git@server:/srv/sample.git
Cloning into 'sample'...
warning: You appear to have cloned an empty repository.
```

剩下的推送就简单了

---


### [参考资料](#links)

<http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137583770360579bc4b458f044ce7afed3df579123eca000>

---

### END


