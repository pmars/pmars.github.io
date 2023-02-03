---
layout:     post
title:      "Ubuntu系统的简单配置"
subtitle:   "初始化Ubuntu系统，简单配置VIM,TMUX"
date:       2015-12-12 23:55:00
author:     "Xiaoh"
header-img: "img/post-bg-ubuntu.jpg"
tags:
    - Ubuntu
---

> 这几天重新安装了虚拟机系统，每次一些初始化的设置都记不得，干脆把这些设置直接写成脚本，以后初始化就方便了

---

目前来说最稳定的版本应该就是 `14.04` 版本的了。这里我选择了桌面版的进行安装。

你可以到 [http://pan.baidu.com/s/1i3YGBvZ](http://pan.baidu.com/s/1i3YGBvZ) 来下载安装包。
此篇文章涉及到的脚本地址在 [https://github.com/pmars/linux-config](https://github.com/pmars/linux-config)

---

### 安装

> 下面的安装步骤你不一定要和我的一样，各取所需即可

我的用 `Pyper-V` 来安装的虚拟机，你如果用 `Vmware` 或者其他的管理器的话也无所谓，基本上过程是一样的。

先给虚拟机起一个名字：

![](http://www.xiaoh.me/img/post-ubuntu-install1.jpg)

修改一下内存大小，由于我只是在做测试就没有修改：

![](http://www.xiaoh.me/img/post-ubuntu-install2.jpg)

选择网络连接方式，这里我选的是桥接，具体桥接的意思请 [google](https://www.google.com/?gws_rd=ssl#q=%E6%A1%A5%E6%8E%A5) 之：

![](http://www.xiaoh.me/img/post-ubuntu-install3.jpg)

之后就是修改存放虚拟机文件的目录，可以选择硬盘的大小：

![](http://www.xiaoh.me/img/post-ubuntu-install4.jpg)

选择虚拟机安装的来源，我这里已经将安装包下载到了本地，直接选择即可：

![](http://www.xiaoh.me/img/post-ubuntu-install5.jpg)

最后在看一下总体的信息，选择完成即可开始安装了：

![](http://www.xiaoh.me/img/post-ubuntu-install6.jpg)

这个时候回到了 `Hyper-V` 的页面，右键刚刚设置的虚拟机，选择启动，当虚拟机状态变为运行的时候，邮件点击连接：

![](http://www.xiaoh.me/img/post-ubuntu-install7.jpg)

现在到了安装页面了，我在这安装的英文版的：

![](http://www.xiaoh.me/img/post-ubuntu-install8.jpg)

确定一下机器的配置是否达标，现在不达标的很少了：

![](http://www.xiaoh.me/img/post-ubuntu-install9.jpg)

选择磁盘处理方式，我这里是虚拟机里面安装的，无所谓了：

![](http://www.xiaoh.me/img/post-ubuntu-install10.jpg)

选择你所在的位置，这个涉及到时间而已：

![](http://www.xiaoh.me/img/post-ubuntu-install11.jpg)

选择键盘的模式:

![](http://www.xiaoh.me/img/post-ubuntu-install12.jpg)

设置账户密码：

![](http://www.xiaoh.me/img/post-ubuntu-install13.jpg)

OK，开始进入安装步骤：

![](http://www.xiaoh.me/img/post-ubuntu-install14.jpg)

经过漫长的等待，系统终于安装完毕，重启下：

![](http://www.xiaoh.me/img/post-ubuntu-restart.jpg)

---

### 进入命令行模式

`Ubuntu` 进入命令行模式，需要按住 `ctrl+alt` + `F2 ~ F6` 之一即可：

输入账号密码，就可以进入命令行模式了

---

### 初始化

初始化的脚本我上传到了 `github`，所以你需要先安装 `git`，之后再 `clone` 代码到本地：

```
$ sudo apt-get install -y git
$ git clone https://github.com/pmars/linux-config.git
```

之后你需要直接调用里面 `init-ubuntu.sh` 来初始化你的系统即可

```
$ bash linux-config/init-ubuntu.sh
```

![](/img/post-ubuntu-script.jpg)

脚本执行完毕，系统也就初始化完成了。

---

### 展示

写了这么多，没有效果是不行的

看一下我的目录的调色，目录原来是深蓝色，对于我这色弱的眼睛来说简直就是欺负人，所以我调成了黄色：

![](http://www.xiaoh.me/img/post-ubuntu-show1.jpg)

我的 `VIM` 配色：

![](http://www.xiaoh.me/img/post-ubuntu-show2.jpg)

当然还少不了我的 `TMUX` 设置：

![](http://www.xiaoh.me/img/post-ubuntu-show3.jpg)

---

### END


