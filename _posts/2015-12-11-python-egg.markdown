---
layout:     post
title:      "怎么制作pip安装包，Python Egg"
subtitle:   "制作Python Egg, 上传PyPI，pip安装下载，制作CLI工具"
date:       2015-12-11 00:13:00
author:     "Xiaoh"
header-img: "img/post-bg-python-egg.jpg"
tags:
    - Python
    - Ubuntu
---

> 最近项目组在写项目的 CLI 工具，已经接近尾声，想做成 pip 的安装包，所以才有了这篇文章
> 1，文章介绍了如何生成 Python Egg ，上传 PyPI 及其 pip 的安装测试
> 2，在后面的进阶部分会介绍简单的生成cli工具的方法


话说既然研究了如何做包，不想浪费这次机会，干脆水一个博客吧

---

# 制作

#### 整理项目

先创建一个项目的文件夹

    $ mkdir eds     # eds 是我项目的名称，你随意修改成自己的即可
    $ cd eds

在里面在创建一个 edssdk 的文件夹，这个文件夹的名称我故意创建的和上层目录不一样，以免误会，这个文件夹其实就是包名称了

    $ mkdir edssdk     # 这个文件夹就是包名称
    $ cd edssdk

这个时候就是写代码的时候了，如果项目逻辑简单，你可以选择在文件夹里面只创建一个 \_\_init\_\_.py 文件，将所有的函数写到此文件里

当然如果项目复杂，你可以多创建几个文件，这里我繁中取简，只创建一个其他文件。

    $ touch __init__.py     # 这个文件作用就是给这个文件夹打成包
    $ touch help.py         # 这里是你的逻辑代码了，我就简单写了

我写了两个函数在 help.py 中

    $ cat help.py
    #!/usr/bin/env python
    #-*- coding:utf-8 -*-
    
    #############################################
    # File Name: help.py
    # Author: xingming
    # Mail: huoxingming@gmail.com
    # Created Time:  2015-12-11 01:23:50 AM
    #############################################
    
    
    def sum(*values):
        s = 0
        for v in values:
            i = int(v)
            s = s + i
        print s
    
    def output():
        print 'http://xiaoh.me'


#### 制作PyPI包

现在项目逻辑已经完成，那么开始做 PyPI 的包了。

在 eds 文件夹中，创建 Egg 的配置文件 setup.py，并填写配置

    $ cat setup.py
    #!/usr/bin/env python
    #-*- coding:utf-8 -*-
    
    #############################################
    # File Name: setup.py
    # Author: xingming
    # Mail: huoxingming@gmail.com
    # Created Time:  2015-12-11 01:25:34 AM
    #############################################
    
    
    from setuptools import setup, find_packages
    
    setup(
        name = "edssdk",
        version = "0.0.1",
        keywords = ("pip", "datacanvas", "eds", "xiaoh"),
        description = "eds sdk",
        long_description = "eds sdk for python",
        license = "MIT Licence",
    
        url = "http://xiaoh.me",
        author = "xiaoh",
        author_email = "huoxingming@gmail.com",
    
        packages = find_packages(),
        include_package_data = True,
        platforms = "any",
        install_requires = []
    )

当你的包很复杂的时候，势必会引用其他的包，你需要将你所有引用的包名称写到 `install_requires` 这个里面：

    install_requires = ["requests"]

当然 setup 还有很多可以选择的项可以填，你可以 python setup.py --help 查看，也可以去看 [文档](http://docs.python.org/distutils/setupscript.html#additional-meta-data)

OK，看一下现在目录的结构：

    $ tree
    $ eds
    $ ├── edssdk
    $ │   ├── help.py
    $ │   └── __init__.py
    $ └── setup.py

这里面还需要说一下，setup 文件支持用配置文件来编写里面的参数

    $ cat setup.cfg
    [metadata]
    name = edssdk
    version = 0.1.1
    zip_safe = False
    
    description = eds sdk
    author = xiaoh
    author_email = huoxingming@gmail.com
    
    license = MIT Licence
    platforms = any
    
    [files]
    packages = find_packages()

更多的帮助信息你可以查看 [这篇文档](https://wiki.python.org/moin/CheeseShopTutorial#Package_Meta-Data)

#### 打包

打包这一步我认为比较简单，目前比较流行的2中打包的方式:

    $ python setup.py bdist_egg     # 生成类似 edssdk-0.0.1-py2.7.egg，支持 easy_install 
    $ python setup.py sdist         # 生成类似 edssdk-0.0.1.tar.gz，支持 pip

上面两条命令都会将文件生成到 dist 目录中

当然还有其他非主流格式或者其他选项，可以通过下面这个命令查看：

    $ python setup.py --help-commands

---

# 部署到PyPI

#### 注册PyPI包

我是直接在 SSH 下面进行操作的，你也可以通过 [网页](http://pypi.python.org/pypi?%3Aaction=submit_form) 来做，SSH 步骤：

    $ python setup.py register
    running register
    running egg_info
    writing dependency_links to eds_sdk.egg-info/dependency_links.txt
    writing eds_sdk.egg-info/PKG-INFO
    writing top-level names to eds_sdk.egg-info/top_level.txt
    reading manifest file 'eds_sdk.egg-info/SOURCES.txt'
    writing manifest file 'eds_sdk.egg-info/SOURCES.txt'
    running check
    We need to know who you are, so please choose either:
     1. use your existing login,
     2. register as a new user,
     3. have the server generate a new password for you (and email it to you), or
     4. quit
    Your selection [default 1]:
    
    Username: xingming
    Password:
    Registering edssdk to https://pypi.python.org/pypi
    Server response (200): OK
    I can store your PyPI login so future submissions will be faster.
    (the login will be stored in /home/xingming/.pypirc)
    Save your login (y/N)?y

关于 register 更详细的内容可以看 [PackageIndex](http://docs.python.org/distutils/packageindex.html)

#### 上传到PyPI

上传文件也是有 SSH 和 网页两种方法。

网页: 在浏览器登陆到 PyPI 点击 Your packages 中 Egg 的项目，然后选择某个版本的 files 即可看到上传界面。

* 下面这句话还是比较常用的，因为你的包以后更新的话，就不用再去注册了，直接使用下面的命令生成包并上传即可。

SSH:

    $ python setup.py sdist upload

这个又是一堆的输出信息，我就不罗列了。

#### 安装测试

用 pip 安装：

    $ pip install edssdk
    Downloading/unpacking edssdk
    Downloading edssdk-0.0.1.tar.gz
    Running setup.py (path:/home/huoxm/pyvirt/build/edssdk/setup.py) egg_info for package edssdk
    
    Installing collected packages: edssdk
    Running setup.py install for edssdk

测试：

    $ python -c "from edssdk import help; help.sum(3,5);help.output();"
    8
    http://xiaoh.me

OK, 现在这个 sdk 大家都可以用到了~~~~~

关于 upload 更详细的内容可以看 [Uploading](http://docs.python.org/distutils/uploading.html)

---

# Tips

###### setup.py 中调用当前目录的文件一定要加 MANIFEST.in 并将调用文件 include 进来

使用 python setup.py sdist 打包时，如果 setup.py 调用了当前目录中的文件(如README.rst):

    $ long_description = open('README.rst').read()

一定要在增加 MANIFEST.in 文件并将调用文件 include 进来，否则将导致用户在 pip install 时报文件找不到的错误，示例:

    $ cat MANIFEST.in
    include README.rst

更详情的可以看 [docs.python.org](http://docs.python.org/distutils/sourcedist.html#specifying-the-files-to-distribute)

###### 项目逻辑内容如果直接在 '\_\_init\_\_.py' 中完成的话，那么引用就可以更简便了。

---

# CLI工具制作

#### CLI命令行工具介绍

CLI（command-line interface，命令行界面）是指可在用户提示符下键入可执行指令的界面，它通常不支持鼠标，用户通过键盘输入指令，计算机接收到指令后，予以执行。

就我直观的理解就是，这就是给程序员使用的，可以在终端即使装XX的美妙工具。

#### CLI制作

由于是 CLI 的命令行工具，所以程序一定要有一个入口的位置，所以我在 help.py 里面添加了 main 函数：

    def main():
        print 'this is main()'
        print sys.argv[1:]
    
    if __name__ == "__main__":
        main()

这个 main 下面会在 setup.py 中用到，下面说一下 setup.py 的相关修改。

> 看英文文档真的是头大，所以后来干脆到 github 上面去找 CLI 的源码来看。发现每个 CLI 工具的 setup.py 中都会有 entry\_points 这个节点。

    entry_points = {
        'console_scripts': [
            'edssdk = edssdk.help:main'
        ]
    }

完整的 setup.py 为：

    from setuptools import setup, find_packages
    
    setup(
        name = "edssdk",
        version = "0.1.2",
        keywords = ("pip", "datacanvas", "eds", "xiaoh"),
        description = "eds sdk",
        long_description = "eds sdk for python",
        license = "MIT Licence",
    
        url = "http://xiaoh.me",
        author = "xiaoh",
        author_email = "huoxingming@gmail.com",
    
        packages = find_packages(),
        include_package_data = True,
        platforms = "any",
        install_requires = [],
    
        scripts = [],
        entry_points = {
            'console_scripts': [
                'edssdk = edssdk.help:main'
            ]
        }
    )

这个里面多了一个 entry\_points 节点，里面的 console\_scripts 指明了命令行工具的名称：

    edssdk = edssdk.help:main

等号前面指明了工具包的名称，等号后面的内容指明了程序的入口地址，当然，这个可以有多条记录，这样一个项目就可以制作多个命令行工具了

#### 制作PyPI

制作包的过程和上面的是一样的

    $ python setup.py sdist
    $ python setup.py register
    $ python setup.py sdist upload

#### 安装测试

通过 pip 更新一下包：

    $ pip install -U edssdk

测试工具包：

    $ edssdk 2345
    this is main()
    ['2345']

OK，方法已经找到了，CLI 工具也就好弄了。

---

### 记FCON模块的pypi打包过程

`fcon` 是刚刚完成的一个python模块，他可以查找指定目录下的符合一定规则文件名的内容中包含一定规则的行，并打印出来。说起来比较拗口，就是三个参数，文件目录，文件正则，字符正则，三个一综合就是输出结果了。

这个工具主要帮我解决查找部分内容的功能，写的博客里面有好多用的是默认的背景图，有时间的时候我就会换一下，这时候我就需要查出来那些用到了默认的。

你可以通过 `pip install fcon` 来安装使用。

这次打包过程还挺烦的，一直有问题，是因为我用到了 [Click模块](http://www.xiaoh.me/2016/01/29/click-command/) 之后就发现，我不用指定内置的运行函数(main)了，而且，我希望可以输出version，并且这个version在setup.py里面也可以使用，这就高出了好多问题。还好刚刚都解决了。

首先说，看一下我所有文件的结构：

    $ tree
    .
    ├── bin
    │   └── fcon
    ├── fcon
    │   └── __init__.py
    ├── README.md
    └── setup.py

    2 directories, 4 files

可以发现，我的 fcon 脚本放到了 bin 目录下，而 fcon 文件夹里面只有一个 `__init__.py` 文件，这是为了包含的时候不冲突，比如我在 fcon 脚本里面用到了这句话：

    try:
        import fcon
    except:
        sys.path.append(os.path.join(os.path.dirname(__file__), "../"))
        import fcon

    def output_version(ctx, param, value):
        if not value or ctx.resilient_parsing:
            return
        click.echo("Version: %s" % fcon.__version__)
        ctx.exit()

这个就是通过引用来设置 version 的部分，这样就实现了，version一处改，大家起开怀的要求了。`__init__.py` 如下：

    __version__ = '0.0.17'

对的，就这么一句话。

再看我的 setup.py 关键就是这个文件了：

    $ cat setup.py
    #!/usr/bin/env python
    #-*- coding:utf-8 -*-
    
    #############################################
    # File Name: setup.py
    # Author: xiaoh
    # Mail: p.mars@163.com
    # Created Time:  2015-12-11 01:25:34 AM
    #############################################
    
    from setuptools import setup, find_packages
    import fcon
    
    setup(
        name = "fcon",
        version = fcon.__version__,
        keywords = ("find", "fcon", "xiaoh"),
        description = "find content",
        long_description = "print files which contain the content you want to search.",
        license = "MIT Licence",
    
        url = "http://xiaoh.me",
        author = "xiaoh",
        author_email = "xiaoh@about.me",
    
        packages = ['fcon'],
        package_data = {
        },
        include_package_data = True,
        platforms = "any",
        install_requires = ["click"],
    
        scripts = ['bin/fcon']
    #    entry_points = {
    #        'console_scripts': [
    #            'fcon = bin/fcon'
    #        ]
    #    }
    )

注释的地方是我测试的过程。。。

就说这么多，关键还得是看脚本，看写法。

如果想看源码的话，可以去：<https://github.com/pmars/tools/tree/master/fcon>

---

### END


