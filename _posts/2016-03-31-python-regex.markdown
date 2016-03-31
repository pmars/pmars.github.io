---
layout:     post
title:      "Python正则表达式简介"
subtitle:   "正则表达式在Python语言中的使用方法"
date:       2016-03-31 00:14:18
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - Python
    - Regex
---

### 目录

0. [简介](#summary)
0. [Python正则](#pythonregex)
    0. [match](#matchmatch)
    0. [findall](#findallfindall)
    0. [sub](#subsub)

---

> 从几年前在使用.NET语言的时候使用正则，到现在，有的时候用的多一些，有的时候用的比较少，不过，都没有离开正则，毕竟工作中很多内容都是在和字符串打交道。

---

### [简介](#summary)

简介方面我不再罗列，如果你想简单的学习正则表达式，你可以去看一下 [正则表达式30分钟入教程](http://www.cnblogs.com/pmars/archive/2011/12/29/2306517.html)

如果你对这些内容感兴趣，想系统的学习一下正则，你可以去看下面的博客：

* [\[ \] 字符组(Character Classes)](http://www.cnblogs.com/pmars/archive/2011/12/30/2307325.html)
* [正则基础之——捕获组（capture group）](http://www.cnblogs.com/pmars/archive/2011/12/30/2307507.html)
* [正则基础之——小数点](http://www.cnblogs.com/pmars/archive/2011/12/31/2308603.html)
* [正则基础之——NFA引擎匹配原理](http://www.cnblogs.com/pmars/archive/2011/12/31/2308614.html)
* [正则基础之——环视](http://www.cnblogs.com/pmars/archive/2011/12/31/2308616.html)
* [正则基础之——/b 单词边界](http://www.cnblogs.com/pmars/archive/2011/12/31/2308617.html)
* [正则应用之——日期正则表达式](http://www.cnblogs.com/pmars/archive/2011/12/31/2308618.html)
* [.NET正则基础之——.NET正则匹配模式](http://www.cnblogs.com/pmars/archive/2011/12/31/2308622.html)
* [ .NET正则基础之——平衡组](http://www.cnblogs.com/pmars/archive/2011/12/31/2308623.html)
* [正则基础之——非捕获组](http://www.cnblogs.com/pmars/archive/2011/12/31/2308637.html)
* [正则基础之——反向引用](http://www.cnblogs.com/pmars/archive/2011/12/31/2308828.html)
* [.NET正则基础——.NET正则类及方法应用](http://www.cnblogs.com/pmars/archive/2011/12/31/2308835.html)
* [.NET正则基础之——正则委托](http://www.cnblogs.com/pmars/archive/2011/12/31/2308837.html)
* [正则基础之——贪婪与非贪婪模式](http://www.cnblogs.com/pmars/archive/2011/12/31/2308842.html)
* [正则应用之——逆序环视探索](http://www.cnblogs.com/pmars/archive/2011/12/31/2308846.html)
* [正则匹配原理之——逆序环视深入](http://www.cnblogs.com/pmars/archive/2011/12/31/2308851.html)
* [正则基础之——神奇的转义](http://www.cnblogs.com/pmars/archive/2011/12/31/2308853.html)
* [正则表达式学习参考——总结](http://www.cnblogs.com/pmars/archive/2011/12/31/2308855.html)
* [C++中三种正则表达式比较（C regex，C ++regex，boost regex）](http://www.cnblogs.com/pmars/archive/2012/10/24/2736831.html)
* [令人惊艳的正则表达式](http://www.cnblogs.com/pmars/p/3515858.html)
* [Python正则表达式指南](http://www.cnblogs.com/pmars/p/4143354.html)

---

### [Python正则](#regex)

在日常使用正则的时候无非就几个模块而已，下面列举一下：

##### [match](#match)

```Python
import re
example = "Hello World, I am 24 years old."
pattern = re.compile(r"(\S+) (\S+) (\S*) (\S+) (\d+) (\S+) (\S+)")
m = pattern.match(example)
print m.group(0)    # Hello World, I am 24 years old.
print m.group(1)    # Hello
print m.group(2)    # World
```

```Python
m = re.match(r'(\S+) (\S+) (\S*) (\S+) (\d+) (\S+) (\S+)', example)
print m.group()
```

##### [findall](#findall)

```Python
import re
example = "Hello World, I am 24 years old."
pattern = re.compile(r"\d+")
m = pattern.findall(example)
print m # ['24']
```

##### [sub](#sub)

```Python
import re
example = "Hello World, I am 24 years old."
m = re.sub(r"\d+", "48", example)
print m
```

```Python
import re
example = "Hello World, I am 24 years old."
m = re.sub(r"(\d+)", lambda x: "[" + x.group(1) + "]", example)
print m # Hello World, I am [24] years old.
```

```Python
def my_replace(match):
    return "[" + match.group(1) + "]"
m = re.sub(r"(\d+)", my_replace, example)
```

---

### END


