---
layout:     post
title:      "Linux下有趣的命令"
subtitle:   "Linux下好玩的命令，Interesting Command of Linux"
date:       2015-12-13 11:55:00
author:     "Xiaoh"
header-img: "img/post-command-cmatrix.jpg"
tags:
    - Ubuntu
---

> 周末闲来无事，搜索了一些有趣的命令，看起来会没什么用途，其实还是可以小装一下的，哈哈哈哈

---

### Catalog


1.  [cmatrix](#cmatrix)
3.  [banner toilet figlet](#banner-toilet-figlet)
4.  [fortune](#fortune)
5.  [cowsay cowthink](#cowsay-cowthink)
6.  [aafire](#aafire)
7.  [sl](#sl)
8.  [moo](#moo)
9.  [bb](#bb)
9.  [linuxlogo](#linuxlogo)
10. [else](#else)

---

###### cmatrix

看过《黑客帝国》的应该都熟悉满是字节流的屏幕，`cmatrix` 就可以做到这个。

安装 `cmatrix`：

    $ sudo apt-get install cmatrix

执行效果：

![](http://www.xiaoh.me/img/post-command-cmatrix.jpg)

你还可以指定参数，输入 `cmatrix -h` 进行查看，具体的就不多说了。

---

###### banner toilet figlet

这三个命令可以用特殊字符打印字符串：

    $ banner xiaoh.me
    
     #    #     #      ##     ####   #    #          #    #  ######
      #  #      #     #  #   #    #  #    #          ##  ##  #
       ##       #    #    #  #    #  ######          # ## #  #####
       ##       #    ######  #    #  #    #   ###    #    #  #
      #  #      #    #    #  #    #  #    #   ###    #    #  #
     #    #     #    #    #   ####   #    #   ###    #    #  ######
    
    $ toilet xiaoh.me
    
              "                  #
     m   m  mmm     mmm    mmm   # mm          mmmmm   mmm
      #m#     #    "   #  #" "#  #"  #         # # #  #"  #
      m#m     #    m"""#  #   #  #   #         # # #  #""""
     m" "m  mm#mm  "mm"#  "#m#"  #   #    #    # # #  "#mm"
    
    
    $ figlet xiaoh.me
          _             _
    __  _(_) __ _  ___ | |__    _ __ ___   ___
    \ \/ / |/ _` |/ _ \| '_ \  | '_ ` _ \ / _ \
     >  <| | (_| | (_) | | | |_| | | | | |  __/
    /_/\_\_|\__,_|\___/|_| |_(_)_| |_| |_|\___|

再来一张彩色的：

![](http://www.xiaoh.me/img/post-command-toilet.jpg)

是不是看起来很炫，更多的用户可以通过 `--help` 来查看。

查看字体用 showfigfonts 即可！

---

###### fortune

这个命令会随机输出一句英文句子，如果你想输出汉语的诗词也可以多安装 `fortune-zh` 这个包

    $ sudo apt-get install fortune
    $ sudo apt-get install fortune-zh

执行以下效果:

    $ fortune
    There was a phone call for you.
    $ fortune
    So you're back... about time...
    $ fortune
    《破山寺后禅院》
    作者：常建
    清晨入古寺，初日照高林。
    曲径通幽处，禅房花木深。
    山光悦鸟性，潭影空人心。
    万籁此俱寂，惟闻钟磬音。

---

###### cowsay cowthink

这个命令可以调出来一头牛，它会说话和思考，安装:

    $ sudo apt-get install cowsay

执行效果：

    $ cowsay xiaoh.me
     __________
    < xiaoh.me >
     ----------
            \   ^__^
             \  (oo)\_______
                (__)\       )\/\
                    ||----w |
                    ||     ||
    $ cowthink xiaoh.me
     __________
    ( xiaoh.me )
     ----------
            o   ^__^
             o  (oo)\_______
                (__)\       )\/\
                    ||----w |
                    ||     ||

这个命令可以和上面的 `fortune` 一起来使用：

    $ fortune | cowthink
     __________________________________
    ( You are going to have a new love )
    ( affair.                          )
     ----------------------------------
            o   ^__^
             o  (oo)\_______
                (__)\       )\/\
                    ||----w |
                    ||     ||

应该还有其他动物说话的命令，我没有细查，如果你知道，请告诉我。

还有个更高端点的 `xcowsay`，你也可以安装试一下，会出现一头真牛。。。

---

###### aafire

这个命令可以让你的终端燃气一把火，安装:

    $ sudo apt-get install libaa-bin

执行效果：

![](http://www.xiaoh.me/img/post-command-aafire.jpg)

---

###### sl

这个命令让你的终端跑起来一个小火车，安装：

    $ sudo apt-get install sl

执行效果：

![](http://www.xiaoh.me/img/post-command-sl.jpg)

---

###### moo

这个命令不需要安装，我们直接看执行效果：

    $ apt-get moo
                     (__)
                     (oo)
               /------\/
              / |    ||
             *  /\---/\
                ~~   ~~
    ..."Have you mooed today?"...

    $ apt-build moo
             (__)    ~
             (oo)   /
         _____\/___/
        /  /\ / /
       ~  /  * /
         / ___/
    *----/\
        /  \
       /   /
      ~    ~
    ..."Have you danced today? Discow!"...

---

###### bb

这个命令太炫酷了，特别推荐一下，哈哈，安装：

    $ sudo apt-get install bb

执行效果部分截图：

![](http://www.xiaoh.me/img/post-command-bb1.jpg)
![](http://www.xiaoh.me/img/post-command-bb2.jpg)
![](http://www.xiaoh.me/img/post-command-bb3.jpg)

---

###### linuxlogo

可以输出你的 `Linux` 的 `logo`，安装：

    $ sudo apt-get install linuxlogo

效果图：

![](http://www.xiaoh.me/img/post-command-linuxlogo.jpg)

---

###### asciiquarium

这个命令如果有时间一定要玩，这个让你的屏幕成为一个水族馆，满满的装X情节啊。

安装方法：

    $ wget http://search.cpan.org/CPAN/authors/id/K/KB/KBAUCOM/Term-Animation-2.4.tar.gz
    $ tar -zxvf Term-Animation-2.4.tar.gz
    $ cd Term-Animation-2.4/
    $ apt-get install libcurses-perl
    $ perl Makefile.PL && make && make test
    $ sudo make install

上面是安装了基础库而已，下面才是真正的安装(其实就是download而已)

    $ wget http://www.robobunny.com/projects/asciiquarium/asciiquarium.tar.gz
    $ tar -zxvf asciiquarium.tar.gz

下载完了，执行：

    $ cd asciiquarium_1.0/
    $ ./asciiquarium

我这里只是测试，如果你没事就像用的话，最好给他放到 /usr/local/bin 下面去，就不用每次都到根目录下面运行了。

![](http://www.xiaoh.me/img/post-command-asciiquarium.jpg)

---

###### else

还有很多其他的命令，由于虚拟机没有 `display`，就不多说了，你可以自己去尝试一下。

    $ xeyes     # 会出来两个大眼珠子
    $ oneko     # 跳出来一直兔子
    $ factor    # 因式分解
    $ cal       # 打印日历
    $ ddate     # 把日历转换成其他的什么历
    $ yes       # 输出很多个y，可以用来对付选择很多y/n的应用

---

### END



