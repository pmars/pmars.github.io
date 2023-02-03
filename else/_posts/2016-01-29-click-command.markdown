---
layout:     post
title:      "Python Click Command 快速入门"
subtitle:   "快速理解Python Click模块"
date:       2016-01-29 15:13:54
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - Python
    - Click
---

### 目录

0. [安装](#install)
0. [Command](#commandcommand)
0. [Group](#groupgroup)
0. [Parameters](#parametersparameters)
0. [安装工具](#setup)
0. [Options](#optionsoption)
0. [Arguments](#argumentsarguments)

---

> 在写命令行工具的时候都会用到参数解析模块，Click就是一个参数解析模块，今天学习一下
> Click的文档在此：[CLICK文档](http://click.pocoo.org/5/quickstart/)

---

### [安装](#install)


Click的安装很简单，直接 `pip install click` 即可

官方推荐你安装 `virtualenv`，我本地是安装了的，你可以安装一下：`sudo pip install virtualenv` or `sudo apt-get install python-virtualenv`

之后找一个地方创建一个虚拟目录即可，`virtualenv pyvirt`，之后加载一下：`. pyvirt/bin/activate`，之后 pip 安装的东西都会在这里面了，想换一套的话，可以直接给这个虚拟目录删掉。

---

### [Command](#command)


Click 通过 `click.command()` 来整理一个命令行工具，通过click.option()来添加命令行的参数，可以看一下下面的例子：

    import click

    @click.command()
    @click.option('--count', default=1, help='Number of greetings.')
    @click.option('--name', prompt='Your name',
                  help='The person to greet.')
    def hello(count, name):
        """Simple program that greets NAME for a total of COUNT times."""
        for x in range(count):
            click.echo('Hello %s!' % name)

    if __name__ == '__main__':
        hello()

测试一下：

    $ python hello.py --count=2
    Your name: xiaoh
    Hello xiaoh!
    Hello xiaoh!

    $ python hello.py --help
    Usage: hello.py [OPTIONS]

      Simple program that greets NAME for a total of COUNT times.

    Options:
      --count INTEGER  Number of greetings.
      --name TEXT      The person to greet.
      --help           Show this message and exit.

代码里面用的是 click.echo() 而不是 print() ，这是因为 click.echo() 可以解决一些环境问题，比如说 python2 or python3.

---

### [Group](#group)


Click 通过 group 来创建一个命令行组，也就是说它可以有各种参数来解决相同类别的不同问题

    
    import click
    
    @click.group()
    def cli():
        pass
    
    @click.command()
    def initdb():
        click.echo('Initialized the database')
    ····
    @click.command()
    def dropdb():
        click.echo('Droped the database')
    
    cli.add_command(initdb)
    cli.add_command(dropdb)
    
    if __name__ == "__main__":
        cli()

可以测试一下：

    $ python hello.py
    Usage: hello.py [OPTIONS] COMMAND [ARGS]...

    Options:
      --help  Show this message and exit.

    Commands:
      dropdb
      initdb
    $ python hello.py initdb
    Initialized the database
    $ python hello.py dropdb
    Droped the database

###### pass\_content

在Group里面可以添加 pass\_content 选项，这样的话，他就会把所有的输入添加到一个对象中去，并且作为第一个参数传递给函数：

    @click.group()
    @click.option('--debug/--no-debug', default=False)
    @click.pass_context
    def cli(ctx, debug):
        ctx.obj['DEBUG'] = debug

    @cli.command()
    @click.pass_context
    def sync(ctx):
        click.echo('Debug is %s' % (ctx.obj['DEBUG'] and 'on' or 'off'))

    if __name__ == '__main__':
        cli(obj={})

更多使用方法请参考 <http://click.pocoo.org/5/commands/>

---

### [Parameters](#parameters)


Click 通过 click.option() 添加可选参数，通过 click.argument() 来添加有可能可选的参数

以下几点是两个的区别：

* 需要提示补全输入的时候使用 option()
* 标志位(flag or acts) 使用 option()
* option的值可以从环境变量获取，而argument不行
* option的值会在帮助里面列出，而argument不能

简单看一个例子:

    @click.command()
    @click.option('--force', default=True, help='drop db anyway')
    @click.argument('name')
    def dropdb(force, name):
        click.echo(force)
        click.echo('Droped the database:%s' % name)

测试一下：

    $ python hello.py dropdb dbname
    True
    Droped the database:dbname
    (pyvirt)xingming@ubuntu:~/help/blogs/click$ python hello.py dropdb --force=False dbname
    False
    Droped the database:dbname
    (pyvirt)xingming@ubuntu:~/help/blogs/click$ python hello.py dropdb
    Usage: hello.py dropdb [OPTIONS] NAME

    Error: Missing argument "name".

##### 参数类型

Click支持一下参数类型

| STR      | click.STRING | Unicode  |
| INT      | click.INT    | Integers |
| Floating | click.FLOAT  | floating |
| Istanbul | click.BOOL   | boolean  |
| UUID     | click.UUID   | uuid.UUID|

还有就是 

* click.File(mode='R', encoding=no , errors='strict', lazy=no , atomic=false)
* click.Path(exists=False, file\_okay=True, dir\_okay=True, writable=False, readable=True, resolve\_path=False)
* click.Choice(['a','b'])
* click.IntRange

click.File 的例子：

    @click.argument('input', type=click.File('rb'))
    @click.argument('output', type=click.File('wb'))

click.Path 的用法和File是一样的，就不多写了

click.Choice的例子：

    @click.command()
    @click.option('--hash-type', type=click.Choice(['md5', 'sha1']))
    def digest(hash_type):
        click.echo(hash_type)

    =======================================
    $ digest --hash-type=md5
    md5

    $ digest --hash-type=foo
    Usage: digest [OPTIONS]

    Error: Invalid value for "--hash-type": invalid choice: foo. (choose from md5, sha1)

    $ digest --help
    Usage: digest [OPTIONS]

    Options:
      --hash-type [md5|sha1]
      --help                  Show this message and exit.

click.IntRange()的例子：

    @click.command()
    @click.option('--count', type=click.IntRange(0, 20, clamp=True))
    @click.option('--digit', type=click.IntRange(0, 10))
    def repeat(count, digit):
        click.echo(str(digit) * count)

    if __name__ == '__main__':
        repeat()

    =========================================
    $ repeat --count=1000 --digit=5
    55555555555555555555
    $ repeat --count=1000 --digit=12
    Usage: repeat [OPTIONS]

    Error: Invalid value for "--digit": 12 is not in the valid range of 0 to 10.

##### 参数名称

Click 的参数名称有几种形式，不过用的最多的应该就是 `('-n', '--name')` 之后变量名称会赋值给参数name

---

### [安装工具](#setup)


如果你的模块里面用到了Click，并且向打一个pypi包的话，你需要按照下面的方式来填写 setup.py

当你的打包脚本和你的模块脚本在一个文件夹的时候：

    yourscript.py
    setup.py

setup.py 写法如下:

    from setuptools import setup

    setup(
        name='yourscript',
        version='0.1',
        py_modules=['yourscript'],
        install_requires=[
            'Click',
        ],
        entry_points='''
            [console_scripts]
            yourscript=yourscript:cli
        ''',
    )

当你的脚本比较复杂，比如：

    yourpackage/
        __init__.py
        main.py
        utils.py
        scripts/
            __init__.py
            yourscript.py

setup.py 写法如下：

    from setuptools import setup, find_packages

    setup(
        name='yourpackage',
        version='0.1',
        packages=find_packages(),
        include_package_data=True,
        install_requires=[
            'Click',
        ],
        entry_points='''
            [console_scripts]
            yourscript=yourpackage.scripts.yourscript:cli
        ''',
    )

其实就是换了一个路径而已。

---

### [Options](#option)


##### Basic Value Options

最简单的参数就是一个的时候，参数如果有类型，就按照类型来，如果没有就按照默认值来。

    @click.command()
    @click.option('--n', default=1)
    def dots(n):
        click.echo('.' * n)

简单测试一下：

    $ python hello.py dots --n=4
    ....

##### Multi Value Options

    @click.command()
    @click.option('--pos', nargs=2, type=float)
    def findme(pos):
        click.echo('%s / %s' % pos)

测试:

    $ python hello.py findme --pos 3.4 5.0
    3.4 / 5.0

##### Tuples as Multi Value Options

    @click.command()
    @click.option('--item', type=(unicode, int))
    def putitem(item):
        click.echo('name:%s, id=%s' % item)

测试:

    $ python hello.py putitem --item xiaoh 12
    name:xiaoh, id=12

这个是一个tuple类型的参数，里面有两个值，其实就相当于也设置了 nargs=2 这个参数。

##### Multiple Options

    @click.command()
    @click.option('--message', '-m', multiple=True)
    def commit(message):
        click.echo('\n'.join(message))

测试：

    $ python hello.py commit -m a -m b -m c -m d
    a
    b
    c
    d
    $ python hello.py commit -m hello -m world
    hello
    world

##### Counting

这个貌似只能给 verbosity flags 使用

    @click.command()
    @click.option('-v', '--verbose', count=True)
    def log(verbose):
        click.echo('Verbosity: %s' % verbose)

测试:

    $ python hello.py log -vvvvvv
    Verbosity: 6

##### Boolean Flags

布尔类型有多种设置方式：

    @click.command()
    @click.option('--shout/--no-shout', default=False)
    def info(shout):
        rv = sys.platform
        if shout:
            rv = rv.upper() + '!!!!111'
        click.echo(rv)

测试：

    $ info --shout
    LINUX2!!!!111
    $ info --no-shout
    linux2

还有就是一个变量的：

    @click.command()
    @click.option('--shout', is_flag=True)
    def info1(shout):
        rv = sys.platform
        if shout:
            rv = rv.upper() + '!!!!111'
        click.echo(rv)

测试：

    $ python hello.py info1
    linux2
    $ python hello.py info1 --shout
    LINUX2!!!!111

还有一种：

    @click.command()
    @click.option('/debug;/no-debug')
    def info2(debug):
        click.echo('debug=%s' % debug)

测试：

    $ python hello.py info2 /debug
    debug=True
    $ python hello.py info2 /no-debug
    debug=False
    $ python hello.py info2
    debug=False

##### Feature Switches

这个很厉害，看例子:

    @click.command()
    @click.option('--upper', 'transformation', flag_value='upper',
                  default=True)
    @click.option('--lower', 'transformation', flag_value='lower')
    def info3(transformation):
        click.echo(getattr(sys.platform, transformation)())

测试：

    $ python hello.py info3
    LINUX2
    $ python hello.py info3 --lower
    linux2
    $ python hello.py info3 --upper
    LINUX2

##### Choice Options

这个在上面已经说过了，在贴一次：

    @click.command()
    @click.option('--hash-type', type=click.Choice(['md5', 'sha1']))
    def digest(hash_type):
        click.echo(hash_type)

测试:

    $ python hello.py digest

    $ python hello.py digest --hash-type md3
    Usage: hello.py digest [OPTIONS]

    Error: Invalid value for "--hash-type": invalid choice: md3. (choose from md5, sha1)
    $ python hello.py digest --hash-type md5
    md5

##### Prompting

这个在一开始的例子中就有涉及，如果设置为True的话，就提示你输入

    @click.command()
    @click.option('--name', prompt=True)
    def hello(name):
        click.echo('Hello %s!' % name)

测试:

    $ hello --name=Xiaoh
    Hello Xiaoh!
    $ hello
    Name: Xiaoh.me
    Hello Xiaoh.me!

或者你可以换一种问法：

    @click.command()
    @click.option('--name', prompt='Your name please')
    def hello(name):
        click.echo('Hello %s!' % name)

这个就不测试了。

##### Password Prompts

这个指定 hide\_input=True 就可以隐藏输入，用来做密码还是蛮好的。

    @click.command()
    @click.option('--password', prompt=True, hide_input=True,
                  confirmation_prompt=True)
    def encrypt(password):
        click.echo('Encrypting password to %s' % password.encode('rot13'))

测试:

    $ python hello.py encrypt
    Password:
    Repeat for confirmation:
    Encrypting password to kvnbu

Click 也提供了 password\_option() 来处理输入密码的问题:

    @click.command()
    @click.password_option()
    def encrypt(password):
        click.echo('Encrypting password to %s' % password.encode('rot13'))

和上面的测试一样，都看不到，就不贴了。

##### Dynamic Defaults for Prompts

自动获取默认值

    @click.command()
    @click.option('--username', prompt=True,
                  default=lambda: os.environ.get('USER', ''))
    def hello1(username):
        print("Hello,", username)

测试:

    $ export USER=xiaoh
    $ python hello.py hello1
    Username [xiaoh]:
    ('Hello,', u'xiaoh')
    $ python hello.py hello1
    Username [xiaoh]: xingming
    ('Hello,', u'xingming')
    $ python hello.py hello1 --username xiaoh.me
    ('Hello,', u'xiaoh.me')

##### Callbacks and Eager Options

有时候你可能需要通过要给选项就运行到另一个函数，运行完之后就退出，而不是继续运行后面的东西，比如 --version

    def print_version(ctx, param, value):
        if not value or ctx.resilient_parsing:
            return
        click.echo('Version 1.0')
        ctx.exit()

    @click.command()
    @click.option('--version', is_flag=True, callback=print_version,
                  expose_value=False, is_eager=True)
    def hello2():
        click.echo('Hello World!')

测试:

    $ python hello.py hello2
    Hello World!
    $ python hello.py hello2 --version
    Version 1.0

这个很有用啊，我将他放到了 click.group()下面，之后就可以从外层来使用了。

    def print_version(ctx, param, value):
        if not value or ctx.resilient_parsing:
            return
        click.echo('Version 1.0')
        ctx.exit()

    @click.group()
    @click.option('--version', is_flag=True, callback=print_version,
                  expose_value=False, is_eager=True)
    def cli():
        pass

测试:

    $ python hello.py --version
    Version 1.0

##### Yes Parameters

这个就是问你是不是要这么做啊，是就继续，不是就go dead...

    def abort_if_false(ctx, param, value):
        if not value:
            ctx.abort()

    @click.command()
    @click.option('--yes', is_flag=True, callback=abort_if_false,
                  expose_value=False,
                  prompt='Are you sure you want to drop the db?')
    def dropdb():
        click.echo('Dropped all tables!')

测试：

    $ dropdb
    Are you sure you want to drop the db? [y/N]: n
    Aborted!
    $ dropdb --yes
    Dropped all tables!

这个其实也挺常用的，所以Click也提供了另一种解决方案：`confirmation_option()`

    @click.command()
    @click.confirmation_option(help='Are you sure you want to drop the db?')
    def dropdb():
        click.echo('Dropped all tables!'

##### Values from Environment Variables

    @click.command()
    @click.option('--username')
    def greet(username):
        click.echo('Hello %s!' % username)

    if __name__ == '__main__':
        greet(auto_envvar_prefix='GREETER')

测试：

    $ export GREETER_USERNAME=xiaoh
    $ greet
    Hello xiaoh!

当然还可以在Option里面指定环境变量的名字：

    @click.command()
    @click.option('--username')
    def greet(username):
        click.echo('Hello %s!' % username)

    if __name__ == '__main__':
        greet(auto_envvar_prefix='GREETER')

测试:

    $ export GREETER_USERNAME=xiaoh
    $ greet
    Hello xiaoh!

后面还有几个，就不打算罗列了，用到的机会不大感觉，IntRange这个在上面说过了就不写了。

那么所有的Option()的方法基本上就在这了

---

### [Arguments](#Arguments)

##### Basic Arguments

    @click.command()
    @click.argument('filename')
    def touch(filename):
        click.echo(filename)

测试：

    $ python hello.py touch
    Usage: hello.py touch [OPTIONS] FILENAME

    Error: Missing argument "filename".
    $ python hello.py touch abc
    abc

##### Variadic Arguments

多个参数的情况下，可以指定 nargs 的值，如果是-1的话就代表多个（\*的意思）

    @click.command()
    @click.argument('src', nargs=-1)
    @click.argument('dst', nargs=1)
    def copy(src, dst):
        for fn in src:
            click.echo('move %s to folder %s' % (fn, dst))

测试:

    $ python hello.py copy a b c d e
    move a to folder e
    move b to folder e
    move c to folder e
    move d to folder e

##### File Arguments

    @click.command()
    @click.argument('input', type=click.File('rb'))
    @click.argument('output', type=click.File('wb'))
    def inout(input, output):
        while True:
            chunk = input.read(1024)
            if not chunk:
                break
            output.write(chunk)

测试：

    $ python hello.py inout - hello.txt
    hello
    ^C
    Aborted!
    $ python hello.py inout hello.txt -
    hello

##### File Path Arguments

    @click.command()
    @click.argument('f', type=click.Path(exists=True))
    def touch1(f):
        click.echo(click.format_filename(f))

测试:

    $ python hello.py touch1 hellodfadf
    Usage: hello.py touch1 [OPTIONS] F

    Error: Invalid value for "f": Path "hellodfadf" does not exist.
    $ python hello.py touch1 hello.txt
    hello.txt

##### Environment Variables

    @click.command()
    @click.argument('src', envvar='SRC', type=click.File('r'))
    def echo(src):
        click.echo(src.read())

测试:

    $ python hello.py echo hello.txt
    hello

    $ export SRC=hello.txt
    $ echo

    $ python hello.py echo
    hello

##### Option-Like Arguments

这边也可以设置类似于 Option 的参数

    @click.command()
    @click.argument('files', nargs=-1, type=click.Path())
    def touch(files):
        for filename in files:
            click.echo(filename)

测试:

    $ touch -- -foo.txt bar.txt
    -foo.txt
    bar.txt

> 可用的所有的情况都已经说了。。。。

---

贴一下自己测试的代码:

    #!/home/xingming/pyvirt/bin/python
    #-*- coding:utf-8 -*-
    
    #############################################
    # File Name: hello.py
    # Author: xiaoh
    # Mail: xiaoh@about.me 
    # Created Time:  2016-01-29 15:00:47
    #############################################
    
    import os, sys, click
    
    def print_version(ctx, param, value):
        if not value or ctx.resilient_parsing:
            return
        click.echo('Version 1.0')
        ctx.exit()
    
    @click.group()
    @click.option('--version', is_flag=True, callback=print_version,
                  expose_value=False, is_eager=True)
    def cli():
        pass
    
    @click.command()
    @click.option('--n', default=1)
    def dots(n):
        click.echo('.' * n)
        
    @click.command()
    @click.option('--pos', nargs=2, type=float)
    def findme(pos):
        click.echo('%s / %s' % pos)
    
    @click.command()
    @click.option('--item', type=(unicode, int))
    def putitem(item):
        click.echo('name:%s, id=%s' % item)
    
    @click.command()
    @click.option('--message', '-m', multiple=True)
    def commit(message):
        click.echo('\n'.join(message))
    
    @click.command()
    @click.option('-v', '--verbose', count=True)
    def log(verbose):
        click.echo('Verbosity: %s' % verbose)
    
    @click.command()
    @click.option('--shout/--no-shout', default=False)
    def info(shout):
        rv = sys.platform
        if shout:
            rv = rv.upper() + '!!!!111'
        click.echo(rv)
    
    @click.command()
    @click.option('--shout', is_flag=True)
    def info1(shout):
        rv = sys.platform
        if shout:
            rv = rv.upper() + '!!!!111'
        click.echo(rv)
    
    @click.command()
    @click.option('/debug;/no-debug')
    def info2(debug):
        click.echo('debug=%s' % debug)
    
    @click.command()
    @click.option('--upper', 'transformation', flag_value='upper',
                  default=True)
    @click.option('--lower', 'transformation', flag_value='lower')
    def info3(transformation):
        click.echo(getattr(sys.platform, transformation)())
    
    @click.command()
    @click.option('--hash-type', type=click.Choice(['md5', 'sha1']))
    def digest(hash_type):
        click.echo(hash_type)
    
    @click.command()
    @click.option('--name', prompt=True)
    def hello(name):
        click.echo('Hello %s!' % name)
    
    @click.command()
    @click.option('--password', prompt=True, hide_input=True,
                  confirmation_prompt=True)
    def encrypt(password):
        click.echo('Encrypting password to %s' % password.encode('rot13'))
    
    @click.command()
    @click.password_option()
    def encrypt1(password):
        click.echo('Encrypting password to %s' % password.encode('rot13'))
    
    @click.command()
    @click.option('--username', prompt=True,
                  default=lambda: os.environ.get('USER', ''))
    def hello1(username):
        print("Hello,", username)
    
    @click.command()
    @click.option('--version', is_flag=True, callback=print_version,
                  expose_value=False, is_eager=True)
    def hello2():
        click.echo('Hello World!')
    
    def abort_if_false(ctx, param, value):
        if not value:
            ctx.abort()
    
    @click.command()
    @click.option('--yes', is_flag=True, callback=abort_if_false,
                  expose_value=False,
                  prompt='Are you sure you want to drop the db?')
    def dropdb():
        click.echo('Dropped all tables!')
    
    @click.command()
    @click.argument('filename')
    def touch(filename):
        click.echo(filename)
    
    @click.command()
    @click.argument('src', nargs=-1)
    @click.argument('dst', nargs=1)
    def copy(src, dst):
        for fn in src:
            click.echo('move %s to folder %s' % (fn, dst))
    
    @click.command()
    @click.argument('input', type=click.File('rb'))
    @click.argument('output', type=click.File('wb'))
    def inout(input, output):
        while True:
            chunk = input.read(1024)
            if not chunk:
                break
            output.write(chunk)
    
    @click.command()
    @click.argument('f', type=click.Path(exists=True))
    def touch1(f):
        click.echo(click.format_filename(f))
    
    @click.command()
    @click.argument('src', envvar='SRC', type=click.File('r'))
    def echo(src):
        click.echo(src.read())
    
    @click.command()
    @click.argument('files', nargs=-1, type=click.Path())
    def touch2(files):
        for filename in files:
            click.echo(filename)
    
    cli.add_command(touch2)
    cli.add_command(echo)
    cli.add_command(touch1)
    cli.add_command(inout)
    cli.add_command(copy)
    cli.add_command(touch)
    cli.add_command(dropdb)
    cli.add_command(hello2)
    cli.add_command(hello1)
    cli.add_command(encrypt1)
    cli.add_command(encrypt)
    cli.add_command(dots)
    cli.add_command(findme)
    cli.add_command(putitem)
    cli.add_command(commit)
    cli.add_command(log)
    cli.add_command(info)
    cli.add_command(info1)
    cli.add_command(info2)
    cli.add_command(info3)
    cli.add_command(digest)
    cli.add_command(hello)
    
    if __name__ == "__main__":
        cli()

---

### END



