---
layout:     post
title:      "Docker简介"
subtitle:   "总结Docker的安装部署，配置实用"
date:       2015-12-15 02:41:45
author:     "xiaoh"
header-img: "img/post-bg-docker-summary.jpg"
tags:
    - Docker
---

> 最近项目中用到 `docker` 的地方有很多，最近有时间打算总结一下相关的命令

---

### 目录

0. [Docker 简介](#docker-summary)
0. [安装](#install)
    0. [确认版本号](#install-version)
    0. [更新 APT 源](#apt-install-apt)
    0. [安装Docker](#dockerinstall-docker)
    0. [特定版本安装](#install-version)
0. [镜像](#images)
    0. [简介](#images-summary)
    0. [查看镜像](#images-look)
    0. [下载镜像](#images-pull)
    0. [创建镜像](#images-create)
        0. [修改已有镜像](#images-create-hand)
        0. [通过Dockerfile](#dockerfileimages-create-dockerfile)
    0. [镜像标签](#images-tag)
    0. [存储和载入镜像](#images-sl)
        0. [存储镜像](#images-sl-save)
        0. [载入镜像](#images-sl-load)
    0. [移除镜像](#images-rmi)
0. [容器](#container)
    0. [简介](#container-summary)
    0. [启动容器](#container-run)
        0. [一次运行模式](#container-run-one)
        0. [交互运行方式](#container-run-many)
        0. [后台运行方式](#container-run-back)
    0. [进入容器](#container-in)
        0. [attach](#attachcontainer-in-attach)
        0. [exec](#execcontainer-in-exec)
    0. [停止容器](#container-stop)
    0. [删除容器](#container-rm)
    0. [启动容器](#container-start)
0. [仓库](#repository)
    0. [公共仓库](#repository-public)
    0. [私有仓库](#repository-personal)
0. [Dockerfile](#dockerfiledockerfile)
    0. [FROM](#fromdockerfile-from)
    0. [MAINTAINER](#maintainerdockerfile-maintainer)
    0. [RUN](#rundockerfile-run)
    0. [CMD](#cmddockerfile-cmd)
    0. [EXPOSE](#exposedockerfile-expose)
    0. [ENV](#envdockerfile-env)
    0. [ADD](#adddockerfile-add)
    0. [COPY](#copydockerfile-copy)
    0. [ENTRYPOINT](#entrypointdockerfile-entrypoint)
    0. [VOLUME](#volumedockerfile-volume)
    0. [WORKDIR](#workdirdockerfile-workdir)
    0. [ONBUILD](#onbuilddockerfile-onbuild)

---

> 关于 `Docker` 的所有介绍，看 [官方文档](https://docs.docker.com/) 是最好的选择。
> 如果你觉得英语有困难，可以下载这本开源的书籍，<http://pan.baidu.com/s/1pKz9hqF>

## [Docker 简介](#summary)

Docker 是一个开源项目，诞生于 2013 年初，最初是 dotCloud 公司内部的一个业余项目。它基于 Google 公司推出的 Go 语言实现。 项目后来加入了 Linux 基金会，遵从了 Apache 2.0 协议，项目代码在 GitHub 上进行维护。

Docker 自开源后受到广泛的关注和讨论，以至于 dotCloud 公司后来都改名为 Docker Inc。Redhat 已经在其 RHEL6.5 中集中支持 Docker；Google 也在其 PaaS 产品中广泛应用。

Docker 项目的目标是实现轻量级的操作系统虚拟化解决方案。 Docker 的基础是 Linux 容器（LXC）等技术。

在 LXC 的基础上 Docker 进行了进一步的封装，让用户不需要去关心容器的管理，使得操作更为简便。用户操作 Docker 的容器就像操作一个快速轻量级的虚拟机一样简单。

下面的图片比较了 Docker 和传统虚拟化方式的不同之处，可见容器是在操作系统层面上实现虚拟化，直接复用本地主机的操作系统，而传统方式则是在硬件层面实现。

![](http://www.xiaoh.me/img/post-docker-summary.jpg)

---

## [安装](#install)

具体的安装步骤可以参见 [官方安装步骤](https://docs.docker.com/engine/installation/ubuntulinux/)

##### [确认版本号](#install-version)
首先需要检查你的系统版本号：

    $ uname -r

`Docker` 需要你的最低版本号为 `3.10`，否则会出现数据丢失等一系列问题。

##### [更新 APT 源](#install-apt)

随后需要更新你的 `apt` 源来使用最新版本的 `Docker`

添加新的 `gpg` 密钥：

    $ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D

打开或创建文件 `/etc/apt/sources.list.d/docker.list`，如果有内容需要清空，之后写入内容：

    $ cat /etc/apt/sources.list.d/docker.list
    deb https://apt.dockerproject.org/repo ubuntu-trusty main

这句话只适合 `Ubuntu 14.04` 版本，如果你是其他版本，需要看下：

    On Ubuntu Precise 12.04 (LTS)
    deb https://apt.dockerproject.org/repo ubuntu-precise main

    On Ubuntu Trusty 14.04 (LTS)
    deb https://apt.dockerproject.org/repo ubuntu-trusty main

    On Ubuntu Vivid 15.04
    deb https://apt.dockerproject.org/repo ubuntu-vivid main

    Ubuntu Wily 15.10
    deb https://apt.dockerproject.org/repo ubuntu-wily main

需要检查一下是否已经安装过以前的版本，如果有，则需要清理一下：

    $ apt-get purge lxc-docker

之后为了确保你的 `APT` 源正确，需要执行：

    $ apt-cache policy docker-engine

上面一系列步骤执行过后，可以确保以后 `Docker` 都会从新的源来进行下载了。

##### [安装Docker](#install-docker)

安装只需要几条命令：

    $ sudo apt-get update
    $ sudo apt-get install docker-engine
    $ sudo service docker start

这样 `Docker` 的服务也启动了。

我系统为 `Ubuntu`，如果你的系统不一样，可以参考官方文档的方法，按步骤进行即可。

##### [特定版本安装](#install-version)

这样安装的 `Docker` 为最新的版本，如果你想安装特定版本，等我试过再议 TODO

---

## [镜像](#images)

##### [简介](#images-summary)

和虚拟机镜像一样，可以想象它就是安装系统的那个ISO文件。但这个是可以变化的，可以打包成各种版本的镜像。

##### [查看镜像](#images-look)

    $ sudo docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    

可以看到，目前没有任何的镜像存在

##### [下载镜像](#images-pull)

    $ sudo docker pull ubuntu:14.04

如果第一次 `pull` 需要一段时间来等待，之后再 `pull` 有重叠的镜像时就会快很多，因为镜像是增量式存储的。

下载完之后查看一下现有的镜像：

    $ sudo docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    ubuntu              14.04               d55e68e6cc9c        4 days ago          187.9 MB

##### [创建镜像](#images-create)

创建镜像有很多方法，用户可以从 Docker Hub 获取已有镜像并更新，也可以利用本地文件系统创建一个。

###### [修改已有镜像](#images-create-hand)

先使用下载的镜像启动容器

    $ sudo docker run -it ubuntu:14.04 /bin/bash
    root@c4241128e523:/#

注意：记住容器的 ID，稍后还会用到。

给容器添加一个应用 `python-pip`

    root@c4241128e523:/# apt-get install python-pip

当结束后，我们使用 exit 来退出，现在我们的容器已经被我们改变了，使用 docker commit 命令来提交更新后的副本。

    $ sudo docker commit -a xingming -m 'install pip' c4241128e523 xingming/pip
    450225ccea656d22354cd5e7295c0d7ad15590509129a25e9ca1888c2b282903

之后查看一下刚刚创建的镜像

    $ sudo docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
    xingming/pip        latest              450225ccea65        7 seconds ago       278.2 MB
    ubuntu              14.04               d55e68e6cc9c        5 days ago          187.9 MB

###### [通过Dockerfile](#images-create-dockerfile)

`Dockerfile` 支持很多种关键词，我在后面部分会详细说一下。

##### [镜像标签](#images-tag)

以用 docker tag 命令来修改镜像的标签。

    $ sudo docker tag 5db5f8471261 xingming/pip

##### [存储和载入镜像](#images-sl)

###### [存储镜像](#images-sl-save)

如果要导出镜像到本地文件，可以使用 docker save 命令。

    $ sudo docker images
    REPOSITORY TAG IMAGE ID CREATED VIRTUAL SIZE
    ubuntu 14.04 c4ff7513909d 5 weeks ago 225.4 MB
    ...
    $sudo docker save -o ubuntu_14.04.tar ubuntu:14.04

###### [载入镜像](#images-sl-load)

可以使用 docker load 从导出的本地文件中再导入到本地镜像库，例如

    $ sudo docker load --input ubuntu_14.04.tar

或

    $ sudo docker load < ubuntu_14.04.tar

这将导入镜像以及其相关的元数据信息（包括标签等）。

##### [移除镜像](#images-rmi)

如果要移除本地的镜像，可以使用 docker rmi 命令。注意 docker rm 命令是移除容器。

    $ sudo docker rmi xingming/pip

> 注意：在删除镜像之前要先用 docker rm 删掉依赖于这个镜像的所有容器。

---

## [容器](#container)

##### [简介](#container-summary)

如果再用虚拟机来类比的话，容器就是一个运行起来的虚拟机了，他用的是镜像文件，启动起来就叫容器。

可以把容器看做是一个简易版的 Linux 环境（包括 root 用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

> 注：镜像是只读的，容器在启动的时候创建一层可写层作为最上层。

##### [启动容器](#container-run)

> 这个地方多说一句，算是这几天使用 Docker 的总结吧
> 如果通过 Dockerfile Build 出的镜像，Dockerfile 的最后一句应该是个 CMD 命令，且这个命令应该执行一个守护运行的脚本或者命令
> 这样在运行容器的时候就可以不用指定运行的命令了。
> 类似 sudo docker run -d -p xxx:xxx --add-host a.b.c:1.2.3.4 --restart=always xingming/pip

当利用 docker run 来创建容器时，Docker 在后台运行的标准操作包括：

> 检查本地是否存在指定的镜像，不存在就从公有仓库下载
> 利用镜像创建并启动一个容器
> 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
> 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
> 从地址池配置一个 ip 地址给容器
> 执行用户指定的应用程序
> 执行完毕后容器被终止

启动容器有多种参数可选，具体的查看 `sudo docker run --help`，常用的有三种模式：

###### [一次运行模式](#container-run-one)

    $ sudo docker run ubuntu:14.04 /bin/echo 'hello, world!'
    hello, world!

###### [交互运行方式](#container-run-many)

    $ sudo docker run -it ubuntu:14.04 /bin/bash
    root@426d0c491bd9:/# echo 'hello, world!'
    hello, world!
    root@426d0c491bd9:/#

###### [后台运行方式](#container-run-back)

    $ sudo docker run -d ubuntu:14.04 /bin/sh -c "while true;do echo hello;sleep 1;done;"
    1ca48c01674f384488ad86b45c3b647233bb21da9b2bd61aa9b11fc81ade2833

查看正在运行的容器：

    $ sudo docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
    1ca48c01674f        ubuntu:14.04        "/bin/sh -c 'while tr"   44 seconds ago      Up 43 seconds                           determined_bhabha

查看某一个容器的日志：

    $ sudo docker logs --tail 3 determined_bhabha
    hello
    hello
    hello

还有很多的使用方法请用 `--help` 进行查看。

##### [进入容器](#container-in)

当容器在后台运行时，有时我们需要进入容器，方法如下：

###### [attach](#container-in-attach)

    $ sudo docker attach determined_bhabha
    hello
    hello
    hello

`attach` 方式有个问题，就是你退出的时候容器也跟着停止了。

###### [exec](#container-in-exec)

    $ sudo docker exec -it loving_meninsky /bin/sh
    # pwd
    /
    # ls
    bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
    # exit

`exec` 方式呢，退出的时候容器还依然继续运行。

一般了解这些使用方法在平常应用中就足够了，如果想知道更多有关两者的详细使用可以通过 `--help` 来查看，或者 `google` 之

##### [停止容器](#container-stop)

    $ sudo docker stop dockername

##### [删除容器](#container-rm)

    $ sudo docker rm dockername

##### [启动容器](#container-start)

    $ sudo docker start dockername

---

## [仓库](#repository)

仓库（Repository）是集中存放镜像的地方。

一个容易混淆的概念是注册服务器（Registry）。实际上注册服务器是管理仓库的具体服务器，每个服务器上可以有多个仓库，而每个仓库下面有多个镜像。从这方面来说，仓库可以被认为是一个具体的项目或目录。例如对于仓库地址 dl.dockerpool.com/ubuntu 来说， dl.dockerpool.com 是注册服务器地址， ubuntu 是仓库名。

大部分时候，并不需要严格区分这两者的概念。

##### [公共仓库](#repository-public)

目前 Docker 官方维护了一个公共仓库 Docker Hub，其中已经包括了超过 15,000 的镜像。大部分需求，都可以通过在 Docker Hub 中直接下载镜像来实现。

先登录一下 `Docker Hub` :

    $ sudo docker login
    Username: xingming
    Password:
    Email: p.mars@163.com
    WARNING: login credentials saved in /home/xingming/.docker/config.json
    Login Succeeded

之后使用上面创建的镜像来 `push` 到服务器。

    $ sudo docker push xingming/pip

经过漫长的等待之后，终于push上去了。

    $ sudo docker search xingming
    NAME           DESCRIPTION   STARS     OFFICIAL   AUTOMATED
    xingming/pip                 0

这里 image 没有前缀，所以默认的就是 docker.io 的公有仓库，其实你可以先打一个标签

    $ sudo docker tag xingming/pip registry.xiaoh.me:5000/xingming/pip

这时候，如果 push 这个镜像，他会将前面的 host 提取出来，往这个仓库去提交。

##### [私有仓库](#repository-personal)

安装私有仓库：

    $ sudo apt-get install -y build-essential python-dev libevent-dev python-pip liblzma-dev swig
    $ sudo pip install docker-registry

TODO

---

## [Dockerfile](#dockerfile)

使用 `docker commit` 来扩展一个镜像比较简单，但是不方便在一个团队中分享。我们可以使用 `docker build` 来创建一个新的镜像。为此，首先需要创建一个 `Dockerfile`，包含一些如何创建镜像的指令。

`Dockerfile` 由一行行命令语句组成，并且支持以 # 开头的注释行。一般的，`Dockerfile` 分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

新建一个目录和一个 `Dockerfile`

    $ mkdir dpip
    $ cd dpip
    $ touch Dockerfile

`Dockerfile` 中每一条指令都创建镜像的一层，例如：

    # This is a comment
    FROM ubuntu:14.04
    MAINTAINER xingming <http://www.xiaoh.me>
    RUN apt-get update
    RUN apt-get -y install python-pip

`Dockerfile` 基本的语法是

* 使用 # 来注释
* `FROM` 指令告诉 `Docker` 使用哪个镜像作为基础
* 接着是维护者的信息
* `RUN` 开头的指令会在创建中运行，比如安装一个软件包，在这里使用 `apt-get` 来安装了一些软件

编写完成 `Dockerfile` 后可以使用 `docker build` 来生成镜像。

    $ sudo docker build -t xingming/pip:v2 .

其中 `-t` 标记来添加 `tag`，指定新的镜像的用户信息。 “.” 是 `Dockerfile` 所在的路径（当前目录），也可以替换为一个具体的 `Dockerfile` 的路径。

可以看到 `build` 进程在执行操作。它要做的第一件事情就是上传这个 `Dockerfile` 内容，因为所有的操作都要依据 `Dockerfile` 来进行。 然后，`Dockfile` 中的指令被一条一条的执行。每一步都创建了一个新的容器，在容器中执行指令并提交修改（就跟之前介绍过的 `docker commit` 一样）。当所有的指令都执行完毕之后，返回了最终的镜像 `id`。所有的中间步骤所产生的容器都被删除和清理了。

> 注意一个镜像不能超过 127 层

其它的 `Dockerfile` 命令：

指令的一般格式为 `INSTRUCTION arguments` ，指令包括 `FROM` 、 `MAINTAINER` 、 `RUN` 等。

###### [FROM](#dockerfile-from)

格式为 `FROM <image>` 或 `FROM <image>:<tag>` 。

第一条指令必须为 `FROM` 指令。并且，如果在同一个 `Dockerfile` 中创建多个镜像时，可以使用多个 `FROM` 指令（每个镜像一次）。

###### [MAINTAINER](#dockerfile-maintainer)

格式为 `MAINTAINER <name>` ，指定维护者信息。

###### [RUN](#dockerfile-run)

格式为 `RUN <command>` 或 `RUN ["executable", "param1", "param2"]` 。

前者将在 `shell` 终端中运行命令，即 /bin/sh -c ；后者则使用 `exec` 执行。指定使用其它终端可以通过第二种方式实现，例如 `RUN ["/bin/bash", "-c", "echo hello"]` 。

每条 RUN 指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用 \ 来换行。

###### [CMD](#dockerfile-cmd)

支持三种格式

* CMD ["executable","param1","param2"] 使用 exec 执行，推荐方式；
* CMD command param1 param2 在 /bin/sh 中执行，提供给需要交互的应用；
* CMD ["param1","param2"] 提供给 ENTRYPOINT 的默认参数；

指定启动容器时执行的命令，每个 `Dockerfile` 只能有一条 `CMD` 命令。如果指定了多条命令，只有最后一条会被执行。

如果用户启动容器时候指定了运行的命令，则会覆盖掉 `CMD` 指定的命令。

###### [EXPOSE](#dockerfile-expose)

格式为 `EXPOSE <port> [<port>...]` 。

告诉 `Docker` 服务端容器暴露的端口号，供互联系统使用。在启动容器时需要通过 `-P`，`Docker` 主机会自动分配一个端口转发到指定的端口。

###### [ENV](#dockerfile-env)

格式为 `ENV <key> <value>` 。 指定一个环境变量，会被后续 `RUN` 指令使用，并在容器运行时保持。

例如

    ENV PG_MAJOR 9.3
    ENV PG_VERSION 9.3.4
    RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
    ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH

运行时指定环境变量：

    $ sudo docker run -it -e "ENV_DB_PORT=5432" xingming/ubuntu /bin/bash

###### [ADD](#dockerfile-add)

格式为 `ADD <src> <dest>` 。

该命令将复制指定的 <src> 到容器中的 <dest> 。 其中 <src> 可以是 `Dockerfile` 所在目录的一个相对路径；也可以是一个 `URL`；还可以是一个 `tar` 文件（自动解压为目录）。

###### [COPY](#dockerfile-copy)

格式为 `COPY <src> <dest>` 。

复制本地主机的 <src> （为 `Dockerfile` 所在目录的相对路径）到容器中的 <dest> 。

当使用本地目录为源目录时，推荐使用 `COPY` 。

###### [ENTRYPOINT](#dockerfile-entrypoint)

两种格式：

    ENTRYPOINT ["executable", "param1", "param2"]
    ENTRYPOINT command param1 param2 （shell中执行）。

配置容器启动后执行的命令，并且不可被 `docker run` 提供的参数覆盖。

每个 `Dockerfile` 中只能有一个 `ENTRYPOINT` ，当指定多个时，只有最后一个起效。

###### [VOLUME](#dockerfile-volume)

格式为 `USER daemon`

指定运行容器时的用户名或 `UID`，后续的 `RUN` 也会使用指定用户。

当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，例如： `RUN groupadd -r postgres && useradd -r -g postgres postgres` 。要临时获取管理员权限可以使用 `gosu` ，而不推荐 `sudo` 。

###### [WORKDIR](#dockerfile-workdir)

格式为 `WORKDIR /path/to/workdir` 。

为后续的 `RUN` 、 `CMD` 、 `ENTRYPOINT` 指令配置工作目录。

可以使用多个 `WORKDIR` 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。例如

    WORKDIR /a
    WORKDIR b
    WORKDIR c
    RUN pwd

则最终路径为 `/a/b/c` 。

###### [ONBUILD](#dockerfile-onbuild)

格式为 `ONBUILD [INSTRUCTION]` 。

配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。

例如，`Dockerfile` 使用如下的内容创建了镜像 `image-A` 。

    [...]
    ONBUILD ADD . /app/src
    ONBUILD RUN /usr/local/bin/python-build --dir /app/src
    [...]

如果基于 `image-A` 创建新的镜像时，新的 `Dockerfile` 中使用 `FROM image-A` 指定基础镜像时，会自动执行 `ONBUILD` 指令内容，等价于在后面添加了两条指令。

    FROM image-A
    #Automatically run the following
    ADD . /app/src
    RUN /usr/local/bin/python-build --dir /app/src

使用 `ONBUILD` 指令的镜像，推荐在标签中注明，例如 `ruby:1.9-onbuild` 。

---

关于 `Docker` 就先介绍这些，已经足够日常使用，等有更多的发现，再来补充

---

### END



