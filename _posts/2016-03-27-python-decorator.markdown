---
layout:     post
title:      "python装饰器入门与提高"
subtitle:   "理解PYTHON中的装饰器"
date:       2016-03-27 12:59:36
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - Python
---

### 目录

0. [开篇介绍](#firstsummary)
    0. [简介](#summary)
    0. [函数即对象](#needknow)
0. [装饰器的本质](#realmeaning)
0. [需注意的地方](#attention)
    0. [属性变化](#attribute)
    0. [使用inspect获取函数参数](#inspectinspect)
    0. [多个装饰器的调用顺序](#multi)
    0. [给装饰器传递参数](#parameters)
0. [装饰器的使用场景与缺点](#usingplaceandshort)
    0. [装饰器的使用场景](#usingplace)
    0. [缺点](#short)
0. [参考链接](#links)
0. [学习资料](#learningmore)

---

> 项目里面用到LOG的功能很常见，最近我想，可以将这个写成一个装饰器来做LOG的自动记录（打印函数的参数和生命周期信息），研究了一下装饰器，才有了如下的文章。

---

### [开篇介绍](#firstsummary)

##### [简介](#summary)

Python 2.2 开始提供了装饰器（decorator），装饰器作为修改函数的一种便捷方式，为程序员编写程序提供了便利性和灵活性，适当使用装饰器，能够有效的提高代码的可读性和可维护性，然而，装饰器并没有被广泛的使用，主要还是因为大多数人并不理解装饰器的工作机制。

装饰器本质上就是一个函数，这个函数接收其他函数作为参数，并将其以一个新的修改后的函数进行替换。概念比较抽象，一起来看两个装饰器的例子。

##### [函数即对象](#needknow)

要理解装饰器，需要理解下面两个Python的概念

###### Python的函数是对象

简单的例子来说明一下:

```Python
def shout(word="yes"):
    return word.capitalize()+"!"

print shout()
# outputs : 'Yes!'

# 作为一个对象，你可以讲函数赋值给另一个对象
scream = shout

# 注意到这里我们并没有使用括号：我们不是调用函数，而是将函数'shout'赋给变量'scream'
# 这意味着，你可以通过'scream'调用'shout'

print scream()
# outputs : 'Yes!'

# 不仅如此，你可以删除老的名称'shout'，但是通过'scream'依旧可以访问原有函数

del shout
try:
    print shout()
except NameError, e:
    print e
    #outputs: "name 'shout' is not defined"

print scream()
# outputs: 'Yes!'
```

###### 函数可以被定义在另一个函数里

代码说明

```Python
def talk():
    # 你可以定义一个函数
    def whisper(word="yes"):
        return word.lower()+"..."

    # ... 并且立刻调用
    print whisper()

# 每次当你调用"talk", 都会定义"whisper"
# 并且在"talk"中被调用
talk()
# outputs:
# "yes..."

#但是在"talk"外部，函数"whisper"不存在！
try:
    print whisper()
except NameError, e:
    print e
    #outputs : "name 'whisper' is not defined"*
```

###### 函数引用

理解了上面两个概念，就很容易理解这个概念：一个函数可以返回另一个函数

```Python
def getTalk(type="shout"):

    # 定义函数
    def shout(word="yes"):
        return word.capitalize()+"!"

    def whisper(word="yes") :
        return word.lower()+"...";

    # 返回函数
    if type == "shout":
        # 没有使用"()", 并不是要调用函数，而是要返回函数对象
        return shout
    else:
        return whisper

# 如何使用？

# 将函数返回值赋值给一个变量
talk = getTalk()

# 我们可以打印下这个函数对象
print talk
#outputs : <function shout at 0xb7ea817c>

# 这个对象是函数的返回值
print talk()
#outputs : Yes!

# 不仅如此，你还可以直接使用之
print getTalk("whisper")()
#outputs : yes...
```

同时，可以理解，一个函数也可以做为参数来进行传递

```Python
def doSomethingBefore(func):
    print "I do something before then I call the function you gave me"
    print func()

doSomethingBefore(scream)
#outputs:
#I do something before then I call the function you gave me
#Yes!
```

装饰器就是封装器，可以让你在被装饰函数之前或之后执行代码，而不必修改函数本身

---

### [装饰器的本质](#realmeaning)

先来看几段代码：

```Python
class Store(object):
    def get_food(self, username, food):
        if username != 'admin':
            raise Exception("This user is not allowed to get food")
        return self.storage.get(food)

    def put_food(self, username, food):
        if username != 'admin':
            raise Exception("This user is not allowed to put food")
        self.storage.put(food)
```

程序中多个函数需要检查用户是否为admin，我们可以独立出一个函数：

```Python
def check_is_admin(username):
    if username != 'admin':
        raise Exception("This user is not allowed to get food")

class Store(object):
    def get_food(self, username, food):
        check_is_admin(username)
        return self.storage.get(food)

    def put_food(self, username, food):
        check_is_admin(username)
        return self.storage.put(food)
```

这样看着就好看多了，不过还有没有更加简练的呢？

```Python
def check_is_admin(f):
    def wrapper(*args, **kwargs):
        if kwargs.get('username') != 'admin':
            raise Exception("This user is not allowed to get food")
        return f(*arg, **kargs)
    return wrapper

class Storage(object):
    @check_is_admin
    def get_food(self, username, food):
        return self.storage.get(food)

    @check_is_admin
    def put_food(self, username, food):
        return storage.put(food)
```

可以看到，我们将检查的逻辑单独拎出来，这样更容易理解函数的真实用意，将一些小事情独立出来，能够更好的让同事理解咱们的代码。

前面说过，** 装饰器本质上就是一个函数，这个函数接收其他函数作为参数，并将其以一个新的修改后的函数进行替换。** 下面这个例子能够更好地理解这句话。

```Python
def bread(func):
    def wrapper():
        print "</''''''\>"
        func()
        print "</______\>"
    return wrapper

sandwich_copy = bread(sandwich)
sandwich_copy()

输出结果如下：

</''''''\>
--ham--
</______\>
```

bread是一个函数，它接受一个函数作为参数，然后返回一个新的函数，新的函数对原来的函数进行了一些修改和扩展，且这个新函数可以当做普通函数进行调用。

下面这段代码和上面的程序输出结果一摸一样，只是用了python提供的装饰器语法，看起来更加简单直接。

```Python
def bread(func):
    def wrapper():
        print "</''''''\>"
        func()
        print "</______\>"
    return wrapper

@bread
def sandwich(food="--ham--"):
    print food
```

到这里，我们应该已经能够理解装饰器的作用和用法了，再强调一遍：装饰器本质上就是一个函数，这个函数接收其他函数作为参数，并将其以一个新的修改后的函数进行替换

---

### [需注意的地方](#attention)

##### [属性变化](#attribute)

装饰器动态创建的新函数替换原来的函数，但是，新函数缺少很多原函数的属性，如docstring和名字。

```Python
def is_admin(f):
    def wrapper(*args, **kwargs):
        if kwargs.get("username") != 'admin':
            raise Exception("This user is not allowed to get food")
        return f(*args, **kwargs)
    return wrapper

def foobar(username='someone'):
    """Do crazy stuff"""
    pass

@is_admin
def barfoo(username='someone'):
    """Do crazy stuff"""
    pass

def main():
    print foobar.func_doc
    print foobar.__name__

    print barfoo.func_doc
    print barfoo.__name__

if __name__ == '__main__':
    main()

输出结果：

Do crazy stuff
foobar

None
wrapper
```

我们定义了两个函数foobar与barfoo，其中，barfoo使用装饰器进行了封装，我们获取foobar与barfoo的docstring和函数名字，可以看到，使用了装饰器的函数，不能够正确获取函数原有的docstring与名字，为了解决这个问题，可以使用python内置的 functools模块。

```Python
import functools

def is_admin(f):
    @functools.wraps(f)
    def wrapper(*args, **kwargs):
        if kwargs.get("username") != 'admin':
            raise Exception("This user is not allowed to get food")
        return f(*arg, **kwargs)
    return wrapper
```

我们只需要增加一行代码，就能够正确地获取函数的属性。此外，我们也可以向下面这样：

```Python
def is_admin(f):
    def wrapper(*args, **kwargs):
        if kwargs.get("username") != 'admin':
            raise Exception("This user is not allowed to get food")
        return f(*args, **kwargs)
    return functools.wraps(f)(wrapper) # important
```

当然，个人推荐第一种方法，因为第一种方法可读性更强。

##### [使用inspect获取函数参数](#inspect)

考虑一下下面的程序的输出结果：

```Python
import functools

def check_is_admin(f):
    @functools.wraps(f)
    def wrapper(*args, **kwargs):
        print kwargs
        if kwargs.get('username') != 'admin':
            raise Exception("This user is not allowed to get food")
        return f(*args, **kwargs)
    return wrapper


@check_is_admin
def get_food(username, food='chocolate'):
    return "{0} get food: {1}".format(username, food)


def main():
    print get_food('admin')

if __name__ == '__main__':
    main()
```

这段程序会抛出一个异常，因为我们传入的'admin'是一个位置参数，而我们却去关键字参数(kwargs)获取用户名，因此，`kwargs.get('username')返回None，那么，权限检查发现，用户没有相应的权限，抛出异常。

为了提供一个更加智能的装饰器，我们需要使用python的inspect模块。inspect能够取出函数的签名，并对其进行操作，如下所示：

```Python
import functools
import inspect

def check_is_admin(f):
    @functools.wraps(f)
    def wrapper(*args, **kwargs):
        func_args = inspect.getcallargs(f, *args, **kwargs)
        print func_args
        if func_args.get('username') != 'admin':
            raise Exception("This user is not allowed to get food")
        return f(*args, **kwargs)
    return wrapper


@check_is_admin
def get_food(username, food='chocolate'):
    return "{0} get food: {1}".format(username, food)


def main():
    print get_food('admin')

if __name__ == '__main__':
    main()
```

承担主要工作的函数是inspect.getcallargs，它返回一个将参数名字和值作为键值对的字典，这程序清单7中，这个函数返回{'username':'admin', 'food':'chocolate'}。这意味着我们的装饰器不必检查参数username是基于位置的参数还是基于关键字的参数，而只需在字典中查找即可。

##### [多个装饰器的调用顺序](#multi)

多个装饰器的调用顺序也很好理解，其实多个装饰器就是在嵌套而已

```Python
def makebold(fn):
    def wrapped():
        return "<b>" + fn() + "</b>"
    return wrapped

def makeitalic(fn):
    def wrapped():
        return "<i>" + fn() + "</i>"
    return wrapped

@makebold
@makeitalic
def hello():
    return "hello world"

print hello() ## returns <b><i>hello world</i></b>
```

装饰器就是在外层进行了封装：

```Python
@bread
sandwich()

sandwich_copy = bread(sandwich)
```

那么，封装两层可以像这样理解：

```Python
@makebold
@makeitalic
hello()

hello-copy = makebold(makeitalic(helo))
```

因此，这样理解以后，对于多个装饰器的调用顺序，就不再有疑问了。

##### [给装饰器传递参数](#parameters)

下面通过一个官方的例子来看如何给装饰器传递参数。官方介绍了一个非常有用的装饰器，即设置超时器。如果函数调用超时，则抛出异常。

```Python
def timeout(seconds, error_message = 'Function call timed out'):
   def decorated(func):
       def _handle_timeout(signum, frame):
           raise TimeoutError(error_message)

       def wrapper(*args, **kwargs):
           signal.signal(signal.SIGALRM, _handle_timeout)
           signal.alarm(seconds)
           try:
               result = func(*args, **kwargs)
           finally:
               signal.alarm(0)
           return result

       return functools.wraps(func)(wrapper)

   return decorated
```

使用方法如下：

```Pytrhon
import time

@timeout(1, 'Function slow; aborted')
def slow_function():
    time.sleep(5)
```

对应于我们这篇博客一直讨论的例子，传递参数的代码如下所示：

```Python
def is_admin(admin='admin'):
    def decorated(f):
        @functools.wraps(f)
        def wrapper(*args, **kwargs):
            if kwargs.get("username") != admin:
                raise Exception("This user is not allowed to get food")
            return f(*args, **kwargs)
        return wrapper
    return decorated


@is_admin(admin='root')
def barfoo(username='someone'):
    """Do crazy stuff"""
    print '{0} get food'.format(username)


if __name__ == '__main__':
    barfoo(username='root')
```

---

### [装饰器的使用场景与缺点](#usingplaceandshort)

##### [装饰器的使用场景](#usingplace)

装饰器虽然语法比较复杂，但是，在一些场景下，也确实比较有用。包括：

* 注入参数（提供默认参数，生成参数）
* 记录函数行为（日志、缓存、计时什么的）
* 预处理／后处理（配置上下文什么的）
* 修改调用时的上下文（线程异步或者并行，类方法）

下面这个例子演示了上面提到的几种情况，如下所示：

```Python
def benchmark(func):
    """
    A decorator that prints the time a function takes
    to execute.
    """
    import time
    def wrapper(*args, **kwargs):
        t = time.clock()
        res = func(*args, **kwargs)
        print func.__name__, time.clock()-t
        return res
    return wrapper


def logging(func):
    """
    A decorator that logs the activity of the script.
    (it actually just prints it, but it could be logging!)
    """
    def wrapper(*args, **kwargs):
        res = func(*args, **kwargs)
        print func.__name__, args, kwargs
        return res
    return wrapper


def counter(func):
    """
    A decorator that counts and prints the number of times a function has been executed
    """
    def wrapper(*args, **kwargs):
        wrapper.count = wrapper.count + 1
        res = func(*args, **kwargs)
        print "{0} has been used: {1}x".format(func.__name__, wrapper.count)
        return res
    wrapper.count = 0
    return wrapper

@counter
@benchmark
@logging
def reverse_string(string):
    return str(reversed(string))
```

关于装饰器的例子，官方列出了一个长长的 [列表](https://wiki.python.org/moin/PythonDecoratorLibrary)，这里很多代码可以直接拿来使用，如果需要详细地了解装饰器的使用场景，可以学习一下这份 [列表](https://wiki.python.org/moin/PythonDecoratorLibrary)。

##### [缺点](#short)

* Decorators were introduced in Python 2.4, so be sure your code will be run on >= 2.4.
* Decorators slow down the function call. Keep that in mind.
* You cannot un-decorate a function. (There are hacks to create decorators that can be removed, but nobody uses them.) So once a function is decorated, it’s decorated for all the code.
* Decorators wrap functions, which can make them hard to debug.

---

### [参考链接](#links)

0. <http://stackoverflow.com/questions/739654/how-can-i-make-a-chain-of-function-decorators-in-python/1594484#1594484>
0. <http://www.wklken.me/posts/2013/07/19/python-translate-decorator.html>
0. <http://mingxinglai.com/cn/2015/08/python-decorator/>

---

### [学习资料](#learningmore)

0. [python decorator library](https://wiki.python.org/moin/PythonDecoratorLibrary)
0. [source code of flask](http://flask.pocoo.org/docs/0.10/)
0. [Magic decorator syntax for asynchronous code in Python](https://github.com/lalor/Tomorrow)
0. [A Python decorator that helps ensure that a Python Process is running only once](https://github.com/chriscannon/highlander)

---

### END


