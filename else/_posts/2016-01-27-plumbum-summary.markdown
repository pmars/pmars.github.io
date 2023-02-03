---
layout:     post
title:      "Plumbum模块简介"
subtitle:   "快速了解Plumbum，Python脚本调用命令行模块"
date:       2016-01-27 10:33:37
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - Plumbum
    - Python
---

> 为了在 Python 程序中调用其他脚本或者可执行程序，我们尝试过很多『subprocess』的封装，但『plumbum』模式很轻松就击败了它们。它提供了非常易用的语法，可以轻松地以跨平台的方式执行本地或者远程命令，获取输出或者错误代码。如果这还不够，你还可以组合它们（shell 管道的方式），而且它还提供了创建命令行应用的接口。

---

### 目录

0. [本地命令](#localcommand)
0. [管道](#pipeline)
0. [重定向](#redirect)
0. [退出码](#exitcode)
0. [运行](#run)
0. [sudo](#sudosudo)

---

### [本地命令](#localcommand)


这部分允许你将本地已有的命令加载进来，在python代码中调用

    In [2]: from plumbum import local

    In [3]: local['cat']
    Out[3]: LocalCommand(/bin/cat)

    In [4]: local['ls']
    Out[4]: LocalCommand(/bin/ls)

    In [5]: local['ls']()
    Out[5]: u'bin\ngitpro\nhelp\npyvirt\n'

    In [6]: local['pip']
    Out[6]: LocalCommand(/home/xingming/pyvirt/bin/pip)
    
    In [7]: ls = local['ls']

    In [8]: ls()
    Out[8]: u'bin\ngitpro\nhelp\npyvirt\n'

    In [9]: ls('-a')
    Out[9]: u'.\n..\n.ansible.cfg\n.bash_history\n.bashrc\nbin\n.bundler\n.cache\n.config\n.dircolors\n.distlib\n.gem\n.gitconfig\ngitpro\nhelp\n.ipython\n.pip\n.profile\n.pypirc\npyvirt\n.rediscli_history\n.screwjack.cfg\n.sqlite_history\n.ssh\n.tmux.conf\n.vim\n.viminfo\n.vimrc\n'

    In [10]: ls('-a').splitlines()
    Out[10]:
    [u'.',
     u'..',
     u'.ansible.cfg',
     u'.bash_history',
     u'.bashrc',
     u'bin',
     u'.bundler',
     u'.cache',
     u'.config',
     u'.dircolors',
     u'.distlib',
     u'.gem',
     u'.gitconfig',
     u'gitpro',
     u'help',
     u'.ipython',
     u'.pip',
     u'.profile',
     u'.pypirc',
     u'pyvirt',
     u'.rediscli_history',
     u'.screwjack.cfg',
     u'.sqlite_history',
     u'.ssh',
     u'.tmux.conf',
     u'.vim',
     u'.viminfo',
     u'.vimrc']


可以看到，上面的local可以将本地的命令加载，并且可以作为函数来执行。

    In [6]: local.get('python3.4', '/usr/lib/python3.4', 'otherpath', '...')
    Out[6]: LocalCommand(/usr/bin/python3.4)

上面的get函数替代了[]，get可以解决如果命令不存在的情况下的默认查找位置，并且可以添加多个位置。

你也可以直接将命令加载进来

    In [13]: from plumbum.cmd import grep, cat, ls, head

    In [14]: head('.vimrc').splitlines()
    Out[14]:
    [u'set nocompatible',
     u'set autoindent             "\u81ea\u52a8\u7f29\u8fdb',
     u'set cindent',
     u'set showmatch              "\u4ee3\u7801\u5339\u914d',
     u'set softtabstop=4',
     u'set shiftwidth=4',
     u'set tabstop=4',
     u'set noet',
     u'set nu',
     u'set hlsearch               "\u68c0\u7d22\u65f6\u9ad8\u4eae\u663e\u793a\u5339\u914d\u9879']
    
    In [21]: head['-n', '3', '.vimrc']().splitlines()
    Out[21]:
    [u'set nocompatible',
     u'set autoindent             "\u81ea\u52a8\u7f29\u8fdb',
     u'set cindent']

---

### [管道](#pipeline)


plumbum 允许你将多个命令通过 `|` 连接起来，用起来和命令行一样

    In [30]: (ls | grep['.py'] | head['-n', '1'])().splitlines()
    Out[30]: [u'fapi.py']

    In [31]: py = ls | grep['.py'] | head['-n', '1']
    In [32]: py
    Out[32]: Pipeline(Pipeline(LocalCommand(/bin/ls), BoundCommand(LocalCommand(/bin/grep), ['.py'])), BoundCommand(LocalCommand(/usr/bin/head), ['-n', '1']))

    In [33]: py()
    Out[33]: u'fapi.py\n'

---

### [重定向](#redirect)


plumbum 允许你通过 `<` or `>` 来进行数据重定向处理

    In [36]: import sys

    In [37]: ((grep['hello'] < sys.stdin) > 'tmp.txt')()
    hello
    hi
    hello world!
    bye
    hello my dear!      < Ctrl+d pressed
    Out[37]: u''

    In [38]: cat['tmp.txt']().splitlines()
    Out[38]: [u'hello', u'hello world!', u'hello my dear!']

---

### [退出码](#exitcode)


plumbum 可以查看退出状态

    ProcessExecutionError: Command line: ['/bin/cat', 'adf']
    Exit code: 1
    Stderr:  | /bin/cat: adf: No such file or directory

    In [67]: cat['adf'].run(retcode=None)
    Out[67]: (1, u'', u'/bin/cat: adf: No such file or directory\n')

---

### [运行](#run)


`run()` 函数运行之后输出一个tuple(retcode, stdout, stderr)，如：

    In [77]: ls.run('-a')
    Out[77]:
    (0,
     u'.\n..\n.ansible.cfg\n.bash_history\n.bashrc\nbin\n.bundler\n.cache\n.config\n.dircolors\n.distlib\n.gem\n.gitconfig\ngitpro\nhelp\n.ipython\n.pip\n.profile\n.pypirc\npyvirt\n.rediscli_history\n.screwjack.cfg\n.sqlite_history\n.ssh\n.tmux.conf\n.vim\n.viminfo\n.vimrc\n',
     u'')

`popen()` 方式可以在后台运行命令，他会返回一个 `subprocess.Popen` 的对象

    In [78]: p = ls.popen('-a')
    In [79]: p.communicate()
    Out[79]:
    ('.\n..\n.ansible.cfg\n.bash_history\n.bashrc\nbin\n.bundler\n.cache\n.config\n.dircolors\n.distlib\n.gem\n.gitconfig\ngitpro\nhelp\n.ipython\n.pip\n.profile\n.pypirc\npyvirt\n.rediscli_history\n.screwjack.cfg\n.sqlite_history\n.ssh\n.tmux.conf\n.vim\n.viminfo\n.vimrc\n','')

    In [82]: p = ls.popen('-a')

    In [83]: p.stdout.read()
    Out[83]: '.\n..\n.ansible.cfg\n.bash_history\n.bashrc\nbin\n.bundler\n.cache\n.config\n.dircolors\n.distlib\n.gem\n.gitconfig\ngitpro\nhelp\n.ipython\n.pip\n.profile\n.pypirc\npyvirt\n.rediscli_history\n.screwjack
    .cfg\n.sqlite_history\n.ssh\n.tmux.conf\n.vim\n.viminfo\n.vimrc\n'

    In [84]: p.wait()
    Out[84]: 0

    In [85]: p.terminate()

可以通过 FG, BG 来设定前端执行还是后台执行，当然你也可以用 NOHUP

    In [88]: from plumbum import FG, BG

    In [89]: ls['-a'] & FG
    .   .ansible.cfg   .bashrc  .bundler  .config     .distlib  .gitconfig  help      .pip      .pypirc  .rediscli_history  .sqlite_history  .tmux.conf  .viminfo ..  .bash_history  bin      .cache    .dircolors  .gem      gitpro      .ipython  .profile  pyvirt   .screwjack.cfg     .ssh             .vim        .vimrc

    In [90]: ls['-a'] & BG
    Out[90]: <Future ['/bin/ls', '-a'] (running)>

    In [91]: f = _

    In [92]: f.ready()
    Out[92]: True

    In [93]: f.stdout
    Out[93]: u'.\n..\n.ansible.cfg\n.bash_history\n.bashrc\nbin\n.bundler\n.cache\n.config\n.dircolors\n.distlib\n.gem\n.gitconfig\ngitpro\nhelp\n.ipython\n.pip\n.profile\n.pypirc\npyvirt\n.rediscli_history\n.screwjack.cfg\n.sqlite_history\n.ssh\n.tmux.conf\n.vim\n.viminfo\n.vimrc\n'

---

### [sudo](#sudo)


plumbum 支持sudo来运行命令

    In [100]: from plumbum.cmd import sudo

    In [101]: sl = sudo[ls['-l']]

    In [102]: sl()
    Out[102]: u'total 16\ndrwxrwxr-x  2 xingming xingming 4096  1\u6708 20 10:34 bin\ndrwxrwxr-x 14 xingming xingming 4096  1\u6708 26 14:50 gitpro\ndrwxrwxr-x 12 xingming xingming 4096  1\u6708 27 14:05 help\ndrwxrwxr-x  7 xingming xingming 4096  1\u6708 27 11:32 pyvirt\n'

---

具体的使用方式就介绍这么多，具体可以参见：<http://plumbum.readthedocs.org/en/latest/>

---

另附 `subprocess.Popen` 运行命令行的方法：

    import subprocess

    def runCmd(cmd):
        return subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE).stdout.read()

---

### END


