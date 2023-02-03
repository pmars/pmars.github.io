---
layout:     post
title:      "Python学习笔记"
subtitle:   "Python学习过程总结"
date:       2016-02-03 17:56:50
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - Python
---

### 目录

0. [Python解释器](#pythonsolver)
0. [输出Prettytable](#prettytableoutput)
0. [Set的交并](#setsetjb)
0. [函数名](#functionname)
0. [数据类型检查](#isinstance)
0. [Python下标循环](#pythonenumerate)
0. [Map/Reduce](#mapreducemapreduce)
0. [filter](#filterfilter)
0. [sorted](#sortedsorted)
0. [装饰器](#decorator)
0. [偏函数](#partial)
0. [模块](#module)
0. [面向对象](#object)
0. [面向对象高级编程](#objecthigher)
0. [错误，调试，测试](#errordebugtest)
0. [io编程](#ioiocoding)
0. [进程和线程](#threading)
0. [常用的内建函数](#innerfunction)
0. [网络编程](#networkcoding)

---

> 打算回顾一下Python的各种知识点，如果有不了解或者记忆模糊的地方会记在这里
> 回顾的教程地址为 [Python2.7教程](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000)

---

#### [Python解释器](#solver)


Python的语言规范是开源的，理论上谁都可以写，只是比较麻烦，比较流行的Python解释器有以下几种：

###### CPython

Cpython相当于官方版本，安装了Python2.7之后，这个解释器就一同安装了，因为是C语言开发的，所以叫Cpython，可以在命令行下运行Python，就可以进入解释器

###### IPython

IPython是基于CPython之上的一个交互式解释器，就是说，这是指在交互上有所提高，本质上还是Cpython。要说不一样，就是提示符了，Cpython用的是 `>>>`，而IPython用的是 `In [序号]`

###### PyPy

PyPy是Python的执行速度出现的，采用了 [JIT技术](http://en.wikipedia.org/wiki/Just-in-time_compilation)，对Python代码进行动态**编译**，所以可以显著的提高Python代码的执行速度。

虽然大多数代码都可以执行，但还是和CPython有所不同，所以有的时候两个解释器下运行相同的代码回产生不同过的结果。可以看看 [PyPy和CPython的不同点](http://pypy.readthedocs.org/en/latest/cpython_differences.html)

###### JPython

JPython是运行在Java平台上的Python解释器，可以直接把Python代码编译成Java字节码运行

###### IronPython

对应JPython，IronPython是运行在.NET平台的Python解释器，可以直接把Python代码解释称.NET字节码

* 这些解释器里面最流行的还是官方的CPython
* 如果要和Java或者.NET平台进行交互，最好是用网络调用的接口来实现，确保各程序之间的独立性。

---

#### [输出Prettytable](#output)


看到输出的地方，想到了今天用到的 Prettytable，其实是要输出 SQL 查询的结果用到的，他可以将输出放到一个表格里面

    In [4]: from prettytable import PrettyTable
    In [5]: x = PrettyTable(['name', 'age', 'addr'])
    In [6]: x.add_row(['xiaoh', 323, 'beijing'])
    In [7]: x.add_row(['xiaoh', 323, 'beijingd'])
    In [8]: x.add_row(['xiaoh', 323, 'beijingd'])
    In [9]: x.add_row(['xiaoh', 323, 'beijingd'])
    In [10]: x.add_row(['xiaoh', 323, 'beijingd'])
    In [11]: print x
    +-------+-----+----------+
    |  name | age |   addr   |
    +-------+-----+----------+
    | xiaoh | 323 | beijing  |
    | xiaoh | 323 | beijingd |
    | xiaoh | 323 | beijingd |
    | xiaoh | 323 | beijingd |
    | xiaoh | 323 | beijingd |
    +-------+-----+----------+

Python里面输入可以用 `raw_input()` 来实现

---

#### [Set的交并](#setjb)


set可以看成数学意义上的无序和无重复元素的集合，因此，两个set可以做数学意义上的交集、并集等操作：

    >>> s1 = set([1, 2, 3])
    >>> s2 = set([2, 3, 4])
    >>> s1 & s2
    set([2, 3])
    >>> s1 | s2
    set([1, 2, 3, 4]

---

#### [函数名](#functionname)


函数名其实就是指向一个函数对象的引用，完全可以把函数名赋给一个变量，相当于给这个函数起了一个“别名”：

    >>> a = abs # 变量a指向abs函数
    >>> a(-1) # 所以也可以通过a调用abs函数
    1

---

#### [数据类型检查](#isinstance)


数据类型检查可以用内置函数isinstance实现

    if not isinstance(x, (int, float)):
        raise TypeError('bad operand type')

---

#### [Python下标循环](#enumerate)


如果要对list实现类似Java那样的下标循环怎么办？Python内置的enumerate函数可以把一个list变成索引-元素对，这样就可以在for循环中同时迭代索引和元素本身：

    >>> for i, value in enumerate(['A', 'B', 'C']):
    ...     print i, value
    ...
    0 A
    1 B
    2 C

---

#### [Map/Reduce](#mapreduce)


这个虽然以前看过，不过不用就忘了，其实Python提供了两个行数 `map()` and `reduce()`，两个函数分别对传进来的数据做处理，具体如下：

###### map

map会将传进来的数据传递给function，之后返回结果的列表

    >>> def f(x):
    ...     return x * x
    ...
    >>> map(f, [1, 2, 3, 4, 5, 6, 7, 8, 9])
    [1, 4, 9, 16, 25, 36, 49, 64, 81]

    >>> map(str, [1, 2, 3, 4, 5, 6, 7, 8, 9])
    ['1', '2', '3', '4', '5', '6', '7', '8', '9']

###### reduce

reduce也是将传进来的数据挨个给function，开始给两个，之后算出来的结果和后面的继续传进去。reduce把结果继续和序列的下一个元素做累积计算

    reduce(f, [x1, x2, x3, x4]) = f(f(f(x1, x2), x3), x4)

    >>> def fn(x, y):
    ...     return x * 10 + y
    ...
    >>> reduce(fn, [1, 3, 5, 7, 9])
    13579

###### 两个配合使用

    def str2int(s):
        def fn(x, y):
            return x * 10 + y
        def char2num(s):
            return {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}[s]
        return reduce(fn, map(char2num, s))

或者使用lambda表达式来实现reduce的函数

    def char2num(s):
        return {'0': 0, '1': 1, '2': 2, '3': 3, '4': 4, '5': 5, '6': 6, '7': 7, '8': 8, '9': 9}[s]

    def str2int(s):
        return reduce(lambda x,y: x*10+y, map(char2num, s))

---

#### [filter](#filter)


filter() 可以做为一个过滤函数，传进去的第一个函数需要返回True or False，来对后面的序列进行过滤。

    def is_odd(n):
        return n % 2 == 1
    
    filter(is_odd, [1, 2, 4, 5, 6, 9, 10, 15])
    # 结果: [1, 5, 9, 15]

    In [15]: filter(lambda s: s, ['', None, 'a', 'b', ''])
    Out[15]: ['a', 'b']

---

#### [sorted](#sorted)


sorted()，可以给一个序列排序，第二个参数可以传递一个比较函数来辅助排序

    In [16]: l = [2,1,6,4,7,3,2]

    In [17]: sorted(l)
    Out[17]: [1, 2, 2, 3, 4, 6, 7]

    In [19]: sorted(l, lambda x,y:-1 if x > y else 1 if x < y else 0)
    Out[19]: [7, 6, 4, 3, 2, 2, 1]

    In [27]: l = [{'age': 2}, {'age': 6}, {'age': 3}, {'age': 7}, {'age': 4}]

    In [28]: sorted(l, lambda x,y:-1 if x['age'] < y['age'] else 1 if x['age'] > y['age'] else 0)
    Out[28]: [{'age': 2}, {'age': 3}, {'age': 4}, {'age': 6}, {'age': 7}]

---

#### [装饰器](#decorator)


函数实则是一个对象，他有一个 `__name__` 的属性，可以拿到函数的名字

    In [46]: def now():
       ....:     print '2016-02-03'

    In [47]: now.__name__
    Out[47]: 'now'

    In [48]: f=now

    In [49]: f.__name__
    Out[49]: 'now'

假设我们要增强now()函数的功能，比如，在函数调用前后自动打印日志，但又不希望修改now()函数的定义，这种在代码运行期间动态增加功能的方式，称之为“装饰器”（Decorator）。

本质上，decorator就是一个返回函数的高阶函数。所以，我们要定义一个能打印日志的decorator，可以定义如下：

    def log(func):
        def wrapper(*args, **kw):
            print 'call %s():' % func.__name__
            return func(*args, **kw)
        return wrapper

观察上面的log，因为它是一个decorator，所以接受一个函数作为参数，并返回一个函数。我们要借助Python的@语法，把decorator置于函数的定义处：

    @log
    def now():
        print '2016-02-03'

调用now()函数，不仅会运行now()函数本身，还会在运行now()函数前打印一行日志：

    >>> now()
    call now():
    2016-02-03

把@log放到now()函数的定义处，相当于执行了语句：

    now = log(now)

由于log()是一个decorator，返回一个函数，所以，原来的now()函数仍然存在，只是现在同名的now变量指向了新的函数，于是调用now()将执行新函数，即在log()函数中返回的wrapper()函数。

wrapper()函数的参数定义是 `(*args, **kw)`，因此，wrapper()函数可以接受任意参数的调用。在wrapper()函数内，首先打印日志，再紧接着调用原始函数。

如果decorator本身需要传入参数，那就需要编写一个返回decorator的高阶函数，写出来会更复杂。比如，要自定义log的文本：

def log(text):
    def decorator(func):
        def wrapper(*args, **kw):
            print '%s %s() now. args:%s' % (text, func.__name__, args)
            return func(*args, **kw)
        return wrapper
    return decorator

这个3层嵌套的decorator用法如下：

    @log('execute')
    def now():
        print '2016-02-03'

执行结果如下：

    >>> now()
    execute now() now. args:
    2016-02-03

和两层嵌套的decorator相比，3层嵌套的效果是这样的：

    >>> now = log('execute')(now)

我们来剖析上面的语句，首先执行log('execute')，返回的是decorator函数，再调用返回的函数，参数是now函数，返回值最终是wrapper函数。

以上两种decorator的定义都没有问题，但还差最后一步。因为我们讲了函数也是对象，它有 `__name__` 等属性，但你去看经过decorator装饰之后的函数，它们的 `__name__` 已经从原来的'now'变成了'wrapper'：

    >>> now.__name__
    'wrapper'

因为返回的那个wrapper()函数名字就是'wrapper'，所以，需要把原始函数的 `__name__` 等属性复制到wrapper()函数中，否则，有些依赖函数签名的代码执行就会出错。

不需要编写 `wrapper.__name__ = func.__name__` 这样的代码，Python内置的 `functools.wraps` 就是干这个事的，所以，一个完整的decorator的写法如下：

    import functools

    def log(func):
        @functools.wraps(func)
        def wrapper(*args, **kw):
            print 'call %s() now. args:%s' % (func.__name__, args)
            return func(*args, **kw)
        return wrapper

或者针对带参数的decorator：

    import functools

    def log(action='excute'):
        def decorator(func):
            @functools.wraps(func)
            def wrapper(*args, **kw):
                print '%s %s() now. args:%s' % (action, func.__name__, args)
                return func(*args, **kw)
                print '%s %s() done. result:%s' % (action, func.__name__, result)
            return wrapper
        return decorator

    @log()
    def dos(msg):
        print msg
        return True

    excute dos() now. args:('this is dos function',)
    this is dos function
    excute dos() done. result:True

`import functools` 是导入functools模块。模块的概念稍候讲解。现在，只需记住在定义 `wrapper()` 的前面加上 `@functools.wraps(func)` 即可。

在面向对象（OOP）的设计模式中，decorator被称为装饰模式。OOP的装饰模式需要通过继承和组合来实现，而Python除了能支持OOP的decorator外，直接从语法层次支持decorator。Python的decorator可以用函数实现，也可以用类实现。

decorator可以增强函数的功能，定义起来虽然有点复杂，但使用起来非常灵活和方便。

---

#### [偏函数](#Partial)


偏函数 `functools` 提供的一个模块，它可以修改既有函数的参数，以致于调用的时候能少传递参数，降低调用函数的难度。

    >>> import functools
    >>> int2 = functools.partial(int, base=2)
    >>> int2('1000000')
    64
    >>> int2('1010101')
    85

---

#### [模块](#module)


如果想让一个文件夹成为一个模块，就必须在文件夹里面加入一个 `__init__.py` 的文件，这个文件可以空，也可以添加各种代码，这个文件本身就是一个模块，模块名为文件夹的名字。

在我们编写的模块里面通常会有下面这句话：

    if __name__=='__main__':
        cli()

在命令行运行模块时，解释器就会将特殊变量 `__name__` 置为 `__main__`，这样就可以执行下面的代码。而如果我们在其他模块里面引用这个模块的时候，就不会设置变量，使 if 语句失效，从而不执行任何代码。

在引用模块的时候我们可以通过 `as` 为抹开设置一个别名，比较有用的作用如下：

    try:
        import cStringIO as StringIO
    except ImportError: # 导入失败会捕获到ImportError
        import StringIO

就是在有 从StringIO的时候就引用这个，没有的话就引用下面的。为了代码效率吧。

Python 引用模块时，默认情况下，Python解释器会搜索当前目录、所有已安装的内置模块和第三方模块，搜索路径存放在sys模块的path变量中：

    $ import sys
    $ sys.path
    ............输出一堆东西，太多了..................

如果我们需要添加自己的搜索目录，有两种方式。

* 一是使用`sys.path.append('/your/path')`来添加，这种方法在运行时修改，运行结束后失效。
* 二是直接修改环境变量 PYTHONPATH，该环境变量的内容会被自动添加到模块搜索路径中。设置方式与设置Path环境变量类似。注意只需要添加你自己的搜索路径，Python自己本身的搜索路径不受影响。

---

#### [面向对象](#object)


如果要让内部属性不被外部访问，可以把属性的名称前加上两个下划线 `__`，在Python中，实例的变量名如果以 `__` 开头，就变成了一个私有变量（private），只有内部可以访问，外部不能访问

需要注意的是，在Python中，变量名类似 `__xxx__` 的，也就是以双下划线开头，并且以双下划线结尾的，是特殊变量，特殊变量是可以直接访问的，不是private变量，所以，不能用 `__name__` 、`__score__` 这样的变量名。

有些时候，你会看到以一个下划线开头的实例变量名，比如 `_name`，这样的实例变量外部是可以访问的，但是，按照约定俗成的规定，当你看到这样的变量时，意思就是，“虽然我可以被访问，但是，请把我视为私有变量，不要随意访问”。

双下划线开头的实例变量是不是一定不能从外部访问呢？其实也不是。不能直接访问 `__name` 是因为Python解释器对外把 `__name` 变量改成了 `_{ClassName}__name`，所以，仍然可以通过 `_{ClassName}__name` 来访问 `__name` 变量。

但是强烈建议你不要这么干，因为不同版本的Python解释器可能会把 `__name` 改成不同的变量名。

总的来说就是，Python本身没有任何机制阻止你干坏事，一切全靠自觉。

在继承关系中，如果一个实例的数据类型是某个子类，那它的数据类型也可以被看做是父类。但是，反过来就不行

对于一个变量，我们只需要知道它是父类类型，无需确切地知道它的子类型，就可以放心地调用父类方法，而具体调用的方法是作用在父类或是子类对象上，由运行时该对象的确切类型决定，这就是多态真正的威力：调用方只管调用，不管细节，而当我们新增一种父类的子类时，只要确保父类方法（重写）编写正确，不用管原来的代码是如何调用的。这就是著名的“开闭”原则：

* 对扩展开放：允许新增父类的子类；
* 对修改封闭：不需要修改依赖父类类型的函数。

继承还可以一级一级地继承下来，就好比从爷爷到爸爸、再到儿子这样的关系。而任何类，最终都可以追溯到根类object，这些继承关系看上去就像一颗倒着的树。

继承可以把父类的所有功能都直接拿过来，这样就不必重零做起，子类只需要新增自己特有的方法，也可以把父类不适合的方法覆盖重写；

有了继承，才能有多态。在调用类实例方法的时候，尽量把变量视作父类类型，这样，所有子类类型都可以正常被接收；

旧的方式定义Python类允许不从object类继承，但这种编程方式已经严重不推荐使用。任何时候，如果没有合适的类可以继承，就继承自object类。

`type()` 通过type函数可以获取实例的类型，Python把每种type类型都定义好了常量，放在types模块里，使用之前，需要先导入：

    >>> import types
    >>> type('abc')==types.StringType
    True
    >>> type(u'abc')==types.UnicodeType
    True
    >>> type([])==types.ListType
    True
    >>> type(str)==types.TypeType
    True

最后注意到有一种类型就叫TypeType，所有类型本身的类型就是TypeType，比如：

    >>> type(int)==type(str)==types.TypeType
    True

`isinstance()` 对于class的继承关系来说，使用type()就很不方便。我们要判断class的类型，可以使用isinstance()函数。isinstance()就可以告诉我们，一个对象是否是某种类型。能用type()判断的基本类型也可以用isinstance()判断：

    >>> isinstance('a', str)
    True
    >>> isinstance(u'a', unicode)
    True
    >>> isinstance('a', unicode)
    False

并且还可以判断一个变量是否是某些类型中的一种，比如下面的代码就可以判断是否是str或者unicode：

    >>> isinstance('a', (str, unicode))
    True
    >>> isinstance(u'a', (str, unicode))
    True

由于str和unicode都是从basestring继承下来的，所以，还可以把上面的代码简化为：

    >>> isinstance(u'a', basestring)
    True

`dir()` 如果要获得一个对象的所有属性和方法，可以使用dir()函数，它返回一个包含字符串的list，比如，获得一个str对象的所有属性和方法：

    >>> dir('ABC')
    ['__add__', '__class__', '__contains__', '__delattr__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getnewargs__', '__getslice__', '__gt__', '__hash__', '__init__', '__le__', '__len__', '__lt__', '__mod__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmod__', '__rmul__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '_formatter_field_name_split', '_formatter_parser', 'capitalize', 'center', 'count', 'decode', 'encode', 'endswith', 'expandtabs', 'find', 'format', 'index', 'isalnum', 'isalpha', 'isdigit', 'islower', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'partition', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill']

类似 `__xxx__` 的属性和方法在Python中都是有特殊用途的，比如 `__len__` 方法返回长度。在Python中，如果你调用len()函数试图获取一个对象的长度，实际上，在len()函数内部，它自动去调用该对象的 `__len__()` 方法，所以，下面的代码是等价的：

    >>> len('ABC')
    3
    >>> 'ABC'.__len__()
    3

我们自己写的类，如果也想用len(myObj)的话，就自己写一个__len__()方法：

    >>> class MyObject(object):
    ...     def __len__(self):
    ...         return 100
    ...
    >>> obj = MyObject()
    >>> len(obj)
    100

仅仅把属性和方法列出来是不够的，配合getattr()、setattr()以及hasattr()，我们可以直接操作一个对象的状态：

    >>> class MyObject(object):
    ...     def __init__(self):
    ...         self.x = 9
    ...     def power(self):
    ...         return self.x * self.x
    ...
    >>> obj = MyObject()

紧接着，可以测试该对象的属性：

    >>> hasattr(obj, 'x') # 有属性'x'吗？
    True
    >>> obj.x
    9
    >>> hasattr(obj, 'y') # 有属性'y'吗？
    False
    >>> setattr(obj, 'y', 19) # 设置一个属性'y'
    >>> hasattr(obj, 'y') # 有属性'y'吗？
    True
    >>> getattr(obj, 'y') # 获取属性'y'
    19
    >>> obj.y # 获取属性'y'
    19

如果试图获取不存在的属性，会抛出AttributeError的错误：

    >>> getattr(obj, 'z') # 获取属性'z'
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: 'MyObject' object has no attribute 'z'

可以传入一个default参数，如果属性不存在，就返回默认值：

    >>> getattr(obj, 'z', 404) # 获取属性'z'，如果不存在，返回默认值404
    404

也可以获得对象的方法：

    >>> hasattr(obj, 'power') # 有属性'power'吗？
    True
    >>> getattr(obj, 'power') # 获取属性'power'
    <bound method MyObject.power of <__main__.MyObject object at 0x108ca35d0>>
    >>> fn = getattr(obj, 'power') # 获取属性'power'并赋值到变量fn
    >>> fn # fn指向obj.power
    <bound method MyObject.power of <__main__.MyObject object at 0x108ca35d0>>
    >>> fn() # 调用fn()与调用obj.power()是一样的
    81

通过内置的一系列函数，我们可以对任意一个Python对象进行剖析，拿到其内部的数据。要注意的是，只有在不知道对象信息的时候，我们才会去获取对象信息。如果可以直接写：

    sum = obj.x + obj.y

就不要写：

    sum = getattr(obj, 'x') + getattr(obj, 'y')

一个正确的用法的例子如下：

    def readImage(fp):
        if hasattr(fp, 'read'):
            return readData(fp)
        return None

假设我们希望从文件流fp中读取图像，我们首先要判断该fp对象是否存在read方法，如果存在，则该对象是一个流，如果不存在，则无法读取。hasattr()就派上了用场。

请注意，在Python这类动态语言中，有read()方法，不代表该fp对象就是一个文件流，它也可能是网络流，也可能是内存中的一个字节流，但只要read()方法返回的是有效的图像数据，就不影响读取图像的功能。

---

#### [面向对象高级编程](#objecthigher)

正常情况下，当我们定义了一个class，创建了一个class的实例后，我们可以给该实例绑定任何属性和方法，这就是动态语言的灵活性。先定义class：

    >>> class Student(object):
    ...     pass

然后，尝试给实例绑定一个属性：

    >>> s = Student()
    >>> s.name = 'Michael' # 动态给实例绑定一个属性
    >>> print s.name
    Michael

还可以尝试给实例绑定一个方法：

    >>> def set_age(self, age): # 定义一个函数作为实例方法
    ...     self.age = age
    ...
    >>> from types import MethodType
    >>> s.set_age = MethodType(set_age, s, Student) # 给实例绑定一个方法
    >>> s.set_age(25) # 调用实例方法
    >>> s.age # 测试结果
    25

但是，给一个实例绑定的方法，对另一个实例是不起作用的：

    >>> s2 = Student() # 创建新的实例
    >>> s2.set_age(25) # 尝试调用方法
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: 'Student' object has no attribute 'set_age'

为了给所有实例都绑定方法，可以给class绑定方法：

    >>> def set_score(self, score):
    ...     self.score = score
    ...
    >>> Student.set_score = MethodType(set_score, None, Student)

给class绑定方法后，所有实例均可调用：

    >>> s.set_score(100)
    >>> s.score
    100
    >>> s2.set_score(99)
    >>> s2.score
    99

通常情况下，上面的 `set_score` 方法可以直接定义在class中，但动态绑定允许我们在程序运行的过程中动态给class加上功能，这在静态语言中很难实现。

但是，如果我们想要限制class的属性怎么办？比如，只允许对Student实例添加name和age属性。

为了达到限制的目的，Python允许在定义class的时候，定义一个特殊的 `__slots__` 变量，来限制该class能添加的属性：

    >>> class Student(object):
    ...     __slots__ = ('name', 'age') # 用tuple定义允许绑定的属性名称

然后，我们试试：

    >>> s = Student() # 创建新的实例
    >>> s.name = 'Michael' # 绑定属性'name'
    >>> s.age = 25 # 绑定属性'age'
    >>> s.score = 99 # 绑定属性'score'
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    AttributeError: 'Student' object has no attribute 'score'

由于'score'没有被放到 `__slots__` 中，所以不能绑定score属性，试图绑定score将得到AttributeError的错误。

使用 `__slots__` 要注意，`__slots__` 定义的属性仅对当前类起作用，对继承的子类是不起作用的：

    >>> class GraduateStudent(Student):
    ...     pass
    ...
    >>> g = GraduateStudent()
    >>> g.score = 9999

除非在子类中也定义 `__slots__`，这样，子类允许定义的属性就是自身的 `__slots__` 加上父类的 `__slots__`。

有的时候我们写的函数来设置类中的一个变量，但这样吊用起来比较麻烦，这个时候我们就可以用到了 `@property` 了，他可以将一个方法变成属性调用，之后用 `@score.setter`，负责把一个setter方法变成属性赋值，例如：

    class Student(object):

        @property
        def score(self):
            return self._score

        @score.setter
        def score(self, value):
            if not isinstance(value, int):
                raise ValueError('score must be an integer!')
            if value < 0 or value > 100:
                raise ValueError('score must between 0 ~ 100!')
            self._score = value

    >>> s = Student()
    >>> s.score = 60 # OK，实际转化为s.set_score(60)
    >>> s.score # OK，实际转化为s.get_score()
    60
    >>> s.score = 9999
    Traceback (most recent call last):
      ...
    ValueError: score must between 0 ~ 100!

注意到这个神奇的@property，我们在对实例属性操作的时候，就知道该属性很可能不是直接暴露的，而是通过getter和setter方法来实现的。

还可以定义只读属性，只定义getter方法，不定义setter方法就是一个只读属性：

    class Student(object):

        @property
        def birth(self):
            return self._birth

        @birth.setter
        def birth(self, value):
            self._birth = value

        @property
        def age(self):
            return 2014 - self._birth

上面的birth是可读写属性，而age就是一个只读属性，因为age可以根据birth和当前时间计算出来。

@property广泛应用在类的定义中，可以让调用者写出简短的代码，同时保证对参数进行必要的检查，这样，程序运行时就减少了出错的可能性。

通过多重继承，一个子类就可以同时获得多个父类的所有功能。

Mixin的目的就是给一个类增加多个功能，这样，在设计类的时候，我们优先考虑通过多重继承来组合多个Mixin的功能，而不是设计多层次的复杂的继承关系。

Python自带的很多库也使用了Mixin。举个例子，Python自带了TCPServer和UDPServer这两类网络服务，而要同时服务多个用户就必须使用多进程或多线程模型，这两种模型由ForkingMixin和ThreadingMixin提供。通过组合，我们就可以创造出合适的服务来。

由于Python允许使用多重继承，因此，Mixin就是一种常见的设计。只允许单一继承的语言（如Java）不能使用Mixin的设计。

打印一个类的信息，只需要定义好 `__str__()` 方法，返回一个好看的字符串就可以了：

    >>> class Student(object):
    ...     def __init__(self, name):
    ...         self.name = name
    ...     def __str__(self):
    ...         return 'Student object (name: %s)' % self.name
    ...
    >>> print Student('Michael')
    Student object (name: Michael)

而上面的方法解决不了，直接赋值的情况，例如：

    >>> s = Student('Michael')
    >>> s
    <__main__.Student object at 0x109afb310>

这是因为直接显示变量调用的不是 `__str__()`，而是 `__repr__()`，两者的区别是 `__str__()` 返回用户看到的字符串，而 `__repr__()` 返回程序开发者看到的字符串，也就是说，`__repr__()` 是为调试服务的。

解决办法是再定义一个`__repr__()`。但是通常`__str__()`和`__repr__()`代码都是一样的，所以，有个偷懒的写法：

    class Student(object):
        def __init__(self, name):
            self.name = name
        def __str__(self):
            return 'Student object (name=%s)' % self.name
        __repr__ = __str__

如果一个类想被用于for ... in循环，类似list或tuple那样，就必须实现一个 `__iter__()` 方法，该方法返回一个迭代对象，然后，Python的for循环就会不断调用该迭代对象的next()方法拿到循环的下一个值，直到遇到StopIteration错误时退出循环。

我们以斐波那契数列为例，写一个Fib类，可以作用于for循环：

    class Fib(object):
        def __init__(self):
            self.a, self.b = 0, 1 # 初始化两个计数器a，b

        def __iter__(self):
            return self # 实例本身就是迭代对象，故返回自己

        def next(self):
            self.a, self.b = self.b, self.a + self.b # 计算下一个值
            if self.a > 100000: # 退出循环的条件
                raise StopIteration();
            return self.a # 返回下一个值

现在，试试把Fib实例作用于for循环：

    >>> for n in Fib():
    ...     print n
    ...
    1
    1
    2
    3
    5
    ...
    46368
    75025

如果希望将类看作 list，tuple, set, dict等结构的话，比如需要取 Fib()[2] 值得话就需要实现 `__getitem__()` 方法。

    class Fib(object):
        def __getitem__(self, n):
            a, b = 1, 1
            for x in range(n):
                a, b = b, a + b
            return a

这个是一个简单的例子，因为传进来的参数可以是其他类型的，并且可以实现各种功能。

如果把对象看成dict，`__getitem__()` 的参数也可能是一个可以作key的object，例如str。

与之对应的是 `__setitem__()` 方法，把对象视作list或dict来对集合赋值。最后，还有一个 `__delitem__()` 方法，用于删除某个元素。

总之，通过上面的方法，我们自己定义的类表现得和Python自带的list、tuple、dict没什么区别，这完全归功于动态语言的“鸭子类型”，不需要强制继承某个接口。

`__getattr__` 是在类属性没有的时候进行调用，来处理工作，已有的属性，不会在 `__getattr__` 中查找。

    class Student(object):

        def __init__(self):
            self.name = 'Michael'

        def __getattr__(self, attr):
            if attr=='score':
                return 99

    >>> s = Student()
    >>> s.name
    'Michael'
    >>> s.score
    99

注意到任意调用如s.abc都会返回None，这是因为我们定义的 `__getattr__` 默认返回就是None。要让class只响应特定的几个属性，我们就要按照约定，抛出AttributeError的错误：

    class Student(object):

        def __getattr__(self, attr):
            if attr=='age':
                return lambda: 25
            raise AttributeError('\'Student\' object has no attribute \'%s\'' % attr)

这实际上可以把一个类的所有属性和方法调用全部动态化处理了，不需要任何特殊手段。

这种完全动态调用的特性有什么实际作用呢？作用就是，可以针对完全动态的情况作调用。

比如，用在动态的url生成上面，如果要写SDK，给每个URL对应的API都写一个方法，那得累死，而且，API一旦改动，SDK也要改。利用完全动态的`__getattr__`，我们可以写出一个链式调用：

    class Chain(object):

        def __init__(self, path=''):
            self._path = path

        def __getattr__(self, path):
            return Chain('%s/%s' % (self._path, path))

        def __str__(self):
            return self._path

试试：

    >>> Chain().status.user.timeline.list
    '/status/user/timeline/list'

这样，无论API怎么变，SDK都可以根据URL实现完全动态的调用，而且，不随API的增加而改变！

任何类，只需要定义一个 `__call__()` 方法，就可以直接对实例进行调用。请看示例：

    class Student(object):
        def __init__(self, name):
            self.name = name

        def __call__(self):
            print('My name is %s.' % self.name)

调用方式如下：

    >>> s = Student('Michael')
    >>> s()
    My name is Michael.

`__call__()` 还可以定义参数。对实例进行直接调用就好比对一个函数进行调用一样，所以你完全可以把对象看成函数，把函数看成对象，因为这两者之间本来就没啥根本的区别。

如果你把对象看成函数，那么函数本身其实也可以在运行期动态创建出来，因为类的实例都是运行期创建出来的，这么一来，我们就模糊了对象和函数的界限。

那么，怎么判断一个变量是对象还是函数呢？其实，更多的时候，我们需要判断一个对象是否能被调用，能被调用的对象就是一个Callable对象，比如函数和我们上面定义的带有 `__call()__` 的类实例：

    >>> callable(Student())
    True
    >>> callable(max)
    True
    >>> callable([1, 2, 3])
    False
    >>> callable(None)
    False
    >>> callable('string')
    False

通过callable()函数，我们就可以判断一个对象是否是“可调用”对象。

正常情况下，我们都用class Xxx...来定义类，但是，type()函数也允许我们动态创建出类来，也就是说，动态语言本身支持运行期动态创建类，这和静态语言有非常大的不同，要在静态语言运行期创建类，必须构造源代码字符串再调用编译器，或者借助一些工具生成字节码实现，本质上都是动态编译，会非常复杂。

    >>> def fn(self, name='world'): # 先定义函数
    ...     print('Hello, %s.' % name)
    ...
    >>> Hello = type('Hello', (object,), dict(hello=fn)) # 创建Hello class
    >>> h = Hello()
    >>> h.hello()
    Hello, world.
    >>> print(type(Hello))
    <type 'type'>
    >>> print(type(h))
    <class '__main__.Hello'>

要创建一个class对象，type()函数依次传入3个参数：

* class的名称；
* 继承的父类集合，注意Python支持多重继承，如果只有一个父类，别忘了tuple的单元素写法；
* class的方法名称与函数绑定，这里我们把函数fn绑定到方法名hello上。

通过type()函数创建的类和直接写class是完全一样的，因为Python解释器遇到class定义时，仅仅是扫描一下class定义的语法，然后调用type()函数创建出class。

---

#### [错误，调试，测试](#errordebugtest)

Python内置的try...except...finally用来处理错误十分方便。出错时，会分析错误信息并定位错误发生的代码位置才是最关键的。

程序也可以主动抛出错误，让调用者来处理相应的错误。但是，应该在文档中写清楚可能会抛出哪些错误，以及错误产生的原因。

调试的方法有多种，常见的有 print, assert, logging, pdb等

用print最大的坏处是将来还得删掉它，想想程序里到处都是print，运行结果也会包含很多垃圾信息。

`assert n != 0, 'n is zero!'` assert的意思是，表达式n != 0应该是True，否则，后面的代码就会出错。

程序中如果到处充斥着assert，和print相比也好不到哪去。不过，启动Python解释器时可以用-O参数来关闭assert，关闭后，你可以把所有的assert语句当成pass来看。

logging允许你指定记录信息的级别，有debug，info，warning，error等几个级别，当我们指定level=INFO时，logging.debug就不起作用了。同理，指定level=WARNING后，debug和info就不起作用了。这样一来，你可以放心地输出不同级别的信息，也不用删除，最后统一控制输出哪个级别的信息。

logging的另一个好处是通过简单的配置，一条语句可以同时输出到不同的地方，比如console和文件。

启动Python的调试器pdb，让程序以单步方式运行，可以随时查看运行状态。` python -m pdb err.py`

几个简单命令可以完成各种操作:

* l 查看代码
* n 执行下一条命令
* p 查看变量
* c 继续运行
* q 退出

这种通过pdb在命令行调试的方法理论上是万能的，但实在是太麻烦了，如果有一千行代码，要运行到第999行得敲多少命令啊。这个时候我们可以使用 `pdb.set_trace()` 来设置一个断点，运行代码，程序会自动在pdb.set_trace()暂停并进入pdb调试环境，可以用命令p查看变量，或者用命令c继续运行

检查代码是否正确的方法最好是进行单元测试，可以用python提供的 unittest，当然也可以用 pytest都可以。

Python内置的“文档测试”（doctest）模块可以直接提取注释中的代码并执行测试。

doctest严格按照Python交互式命令行的输入和输出来判断测试结果是否正确。只有测试异常的时候，可以用...表示中间一大段烦人的输出。

让我们用doctest来测试上次编写的Dict类：

    class Dict(dict):
        '''
        Simple dict but also support access as x.y style.

        >>> d1 = Dict()
        >>> d1['x'] = 100
        >>> d1.x
        100
        >>> d1.y = 200
        >>> d1['y']
        200
        >>> d2 = Dict(a=1, b=2, c='3')
        >>> d2.c
        '3'
        >>> d2['empty']
        Traceback (most recent call last):
            ...
        KeyError: 'empty'
        >>> d2.empty
        Traceback (most recent call last):
            ...
        AttributeError: 'Dict' object has no attribute 'empty'
        '''
        def __init__(self, **kw):
            super(Dict, self).__init__(**kw)

        def __getattr__(self, key):
            try:
                return self[key]
            except KeyError:
                raise AttributeError(r"'Dict' object has no attribute '%s'" % key)

        def __setattr__(self, key, value):
            self[key] = value

    if __name__=='__main__':
        import doctest
        doctest.testmod()

doctest非常有用，不但可以用来测试，还可以直接作为示例代码。通过某些文档生成工具，就可以自动把包含doctest的注释提取出来。用户看文档的时候，同时也看到了doctest。

当模块正常导入时，doctest不会被执行。只有在命令行运行时，才执行doctest。所以，不必担心doctest会在非测试环境下执行。

---

#### [io编程](#iocoding)

如果文件很小，read()一次性读取最方便；如果不能确定文件大小，反复调用read(size)比较保险；如果是配置文件，调用readlines()最方便

open() 默认都是读取文本文件，并且是ASCII编码的文本文件。要读取二进制文件，比如图片、视频等等，用'rb'模式打开文件即可

要读取非ASCII编码的文本文件，就必须以二进制模式打开，再解码。比如GBK编码的文件：

    >>> f = open('/test/gbk.txt', 'rb')
    >>> u = f.read().decode('gbk')
    >>> u
    u'\u6d4b\u8bd5'
    >>> print u
    测试

Python还提供了一个codecs模块帮我们在读文件时自动转换编码，直接读出unicode：

    import codecs
    with codecs.open('/test/gbk.txt', 'r', 'gbk') as f:
        f.read() # u'\u6d4b\u8bd5'

你可以反复调用write()来写入文件，但是务必要调用f.close()来关闭文件。当我们写文件时，操作系统往往不会立刻把数据写入磁盘，而是放到内存缓存起来，空闲的时候再慢慢写入。只有调用close()方法时，操作系统才保证把没有写入的数据全部写入磁盘。忘记调用close()的后果是数据可能只写了一部分到磁盘，剩下的丢失了。所以，还是用with语句来得保险：

    with open('/test/test.txt', 'w') as f:
        f.write('Hello, world!')

要写入特定编码的文本文件，请效仿codecs的示例，写入unicode，由codecs自动转换成指定编码。

###### os模块

* os.name 操作系统的名字
* os.uname() 系统的详细信息，uname()函数在Windows上不提供，也就是说，os模块的某些函数是跟操作系统相关的
* os.environ 环境变量
* os.getenv('PATH') 获取某个环境变量的值
* os.path.abspath('.') 查看目录的绝对量路径
* os.path.join('/test', 'testdir') 合并路径
* os.mkdir('/test/abc') 创建文件夹
* os.rmdir('/test/abc') 删除文件夹
* os.path.split('/test/abc/test.txt') 拆分路径，result:('/test/abc', 'test.txt')
* os.path.splitext('/test/abc/test.txt') 获取扩展名，result:('/text/abc/test', '.txt')
* os.rename('test.txt', 'test.py') 对文件重命名
* os.remove('test.py') 删除文件

但是复制文件的函数居然在os模块中不存在！原因是复制文件并非由操作系统提供的系统调用。理论上讲，我们通过上一节的读写文件可以完成文件复制，只不过要多写很多代码。

幸运的是shutil模块提供了copyfile()的函数，你还可以在shutil模块中找到很多实用函数，它们可以看做是os模块的补充。

Python提供两个模块来实现序列化：cPickle和pickle。这两个模块功能是一样的，区别在于cPickle是C语言写的，速度快，pickle是纯Python写的，速度慢，跟cStringIO和StringIO一个道理。用的时候，先尝试导入cPickle，如果失败，再导入pickle

我们尝试把一个对象序列化并写入文件：

    >>> d = dict(name='Bob', age=20, score=88)
    >>> pickle.dumps(d)
    "(dp0\nS'age'\np1\nI20\nsS'score'\np2\nI88\nsS'name'\np3\nS'Bob'\np4\ns."

pickle.dumps()方法把任意对象序列化成一个str，然后，就可以把这个str写入文件。或者用另一个方法pickle.dump()直接把对象序列化后写入一个file-like Object：

    >>> f = open('dump.txt', 'wb')
    >>> pickle.dump(d, f)
    >>> f.close()

看看写入的dump.txt文件，一堆乱七八糟的内容，这些都是Python保存的对象内部信息。

当我们要把对象从磁盘读到内存时，可以先把内容读到一个str，然后用pickle.loads()方法反序列化出对象，也可以直接用pickle.load()方法从一个file-like Object中直接反序列化出对象。我们打开另一个Python命令行来反序列化刚才保存的对象：

    >>> f = open('dump.txt', 'rb')
    >>> d = pickle.load(f)
    >>> f.close()
    >>> d
    {'age': 20, 'score': 88, 'name': 'Bob'}

当然，这个变量和原来的变量是完全不相干的对象，它们只是内容相同而已。

Pickle的问题和所有其他编程语言特有的序列化问题一样，就是它只能用于Python，并且可能不同版本的Python彼此都不兼容，因此，只能用Pickle保存那些不重要的数据，不能成功地反序列化也没关系。

Python语言特定的序列化模块是pickle，但如果要把序列化搞得更通用、更符合Web标准，就可以使用json模块。

json模块的dumps()和loads()函数是定义得非常好的接口的典范。当我们使用时，只需要传入一个必须的参数。但是，当默认的序列化或反序列机制不满足我们的要求时，我们又可以传入更多的参数来定制序列化或反序列化的规则，既做到了接口简单易用，又做到了充分的扩展性和灵活性。

---

#### [进程和线程](#threading)

###### 多进程

Unix/Linux操作系统提供了一个fork()系统调用，它非常特殊。普通的函数调用，调用一次，返回一次，但是fork()调用一次，返回两次，因为操作系统自动把当前进程（称为父进程）复制了一份（称为子进程），然后，分别在父进程和子进程内返回。

子进程永远返回0，而父进程返回子进程的ID。这样做的理由是，一个父进程可以fork出很多子进程，所以，父进程要记下每个子进程的ID，而子进程只需要调用getppid()就可以拿到父进程的ID。

Python的os模块封装了常见的系统调用，其中就包括fork，可以在Python程序中轻松创建子进程：

    # multiprocessing.py
    import os

    print 'Process (%s) start...' % os.getpid()
    pid = os.fork()
    if pid==0:
        print 'I am child process (%s) and my parent is %s.' % (os.getpid(), os.getppid())
    else:
        print 'I (%s) just created a child process (%s).' % (os.getpid(), pid)

运行结果如下：

    Process (876) start...
    I (876) just created a child process (877).
    I am child process (877) and my parent is 876.

有了fork调用，一个进程在接到新任务时就可以复制出一个子进程来处理新任务，常见的Apache服务器就是由父进程监听端口，每当有新的http请求时，就fork出子进程来处理新的http请求。

###### multiprocessing

如果你打算编写多进程的服务程序，Unix/Linux无疑是正确的选择。由于Windows没有fork调用，难道在Windows上无法用Python编写多进程的程序？

由于Python是跨平台的，自然也应该提供一个跨平台的多进程支持。multiprocessing模块就是跨平台版本的多进程模块。

multiprocessing模块提供了一个Process类来代表一个进程对象，下面的例子演示了启动一个子进程并等待其结束：

    from multiprocessing import Process
    import os
    
    # 子进程要执行的代码
    def run_proc(name):
        print 'Run child process %s (%s)...' % (name, os.getpid())
    
    if __name__=='__main__':
        print 'Parent process %s.' % os.getpid()
        p = Process(target=run_proc, args=('test',))
        print 'Process will start.'
        p.start()
        p.join()
        print 'Process end.'

执行结果如下：

    Parent process 928.
    Process will start.
    Run child process test (929)...
    Process end.

创建子进程时，只需要传入一个执行函数和函数的参数，创建一个Process实例，用start()方法启动，这样创建进程比fork()还要简单。

join()方法可以等待子进程结束后再继续往下运行，通常用于进程间的同步。

###### Pool

如果要启动大量的子进程，可以用进程池的方式批量创建子进程：

    from multiprocessing import Pool
    import os, time, random

    def long_time_task(name):
        print 'Run task %s (%s)...' % (name, os.getpid())
        start = time.time()
        time.sleep(random.random() * 3)
        end = time.time()
        print 'Task %s runs %0.2f seconds.' % (name, (end - start))

    if __name__=='__main__':
        print 'Parent process %s.' % os.getpid()
        p = Pool()
        for i in range(5):
            p.apply_async(long_time_task, args=(i,))
        print 'Waiting for all subprocesses done...'
        p.close()
        p.join()
        print 'All subprocesses done.'

执行结果如下：

    Parent process 669.
    Waiting for all subprocesses done...
    Run task 0 (671)...
    Run task 1 (672)...
    Run task 2 (673)...
    Run task 3 (674)...
    Task 2 runs 0.14 seconds.
    Run task 4 (673)...
    Task 1 runs 0.27 seconds.
    Task 3 runs 0.86 seconds.
    Task 0 runs 1.41 seconds.
    Task 4 runs 1.91 seconds.
    All subprocesses done.

代码解读：

对Pool对象调用join()方法会等待所有子进程执行完毕，调用join()之前必须先调用close()，调用close()之后就不能继续添加新的Process了。

请注意输出的结果，task 0，1，2，3是立刻执行的，而task 4要等待前面某个task完成后才执行，这是因为Pool的默认大小在我的电脑上是4，因此，最多同时执行4个进程。这是Pool有意设计的限制，并不是操作系统的限制。如果改成：

    p = Pool(5)

就可以同时跑5个进程。

由于Pool的默认大小是CPU的核数，如果你不幸拥有8核CPU，你要提交至少9个子进程才能看到上面的等待效果。

###### 进程间通信

Process之间肯定是需要通信的，操作系统提供了很多机制来实现进程间的通信。Python的multiprocessing模块包装了底层的机制，提供了Queue、Pipes等多种方式来交换数据。

我们以Queue为例，在父进程中创建两个子进程，一个往Queue里写数据，一个从Queue里读数据：

    from multiprocessing import Process, Queue
    import os, time, random
    
    # 写数据进程执行的代码:
    def write(q):
        for value in ['A', 'B', 'C']:
            print 'Put %s to queue...' % value
            q.put(value)
            time.sleep(random.random())
    
    # 读数据进程执行的代码:
    def read(q):
        while True:
            value = q.get(True)
            print 'Get %s from queue.' % value
    
    if __name__=='__main__':
        # 父进程创建Queue，并传给各个子进程：
        q = Queue()
        pw = Process(target=write, args=(q,))
        pr = Process(target=read, args=(q,))
        # 启动子进程pw，写入:
        pw.start()
        # 启动子进程pr，读取:
        pr.start()
        # 等待pw结束:
        pw.join()
        # pr进程里是死循环，无法等待其结束，只能强行终止:
        pr.terminate()

运行结果如下：

    Put A to queue...
    Get A from queue.
    Put B to queue...
    Get B from queue.
    Put C to queue...
    Get C from queue.

在Unix/Linux下，multiprocessing模块封装了fork()调用，使我们不需要关注fork()的细节。由于Windows没有fork调用，因此，multiprocessing需要“模拟”出fork的效果，父进程所有Python对象都必须通过pickle序列化再传到子进程去，所有，如果multiprocessing在Windows下调用失败了，要先考虑是不是pickle失败了。

###### 多线程

Python的标准库提供了两个模块：thread和threading，thread是低级模块，threading是高级模块，对thread进行了封装。绝大多数情况下，我们只需要使用threading这个高级模块。

启动一个线程就是把一个函数传入并创建Thread实例，然后调用start()开始执行：

    import time, threading
    
    # 新线程执行的代码:
    def loop():
        print 'thread %s is running...' % threading.current_thread().name
        n = 0
        while n < 5:
            n = n + 1
            print 'thread %s >>> %s' % (threading.current_thread().name, n)
            time.sleep(1)
        print 'thread %s ended.' % threading.current_thread().name
    
    print 'thread %s is running...' % threading.current_thread().name
    t = threading.Thread(target=loop, name='LoopThread')
    t.start()
    t.join()
    print 'thread %s ended.' % threading.current_thread().name

执行结果如下：

    thread MainThread is running...
    thread LoopThread is running...
    thread LoopThread >>> 1
    thread LoopThread >>> 2
    thread LoopThread >>> 3
    thread LoopThread >>> 4
    thread LoopThread >>> 5
    thread LoopThread ended.
    thread MainThread ended.

由于任何进程默认就会启动一个线程，我们把该线程称为主线程，主线程又可以启动新的线程，Python的threading模块有个 `current_thread()` 函数，它永远返回当前线程的实例。主线程实例的名字叫MainThread，子线程的名字在创建时指定，我们用LoopThread命名子线程。名字仅仅在打印时用来显示，完全没有其他意义，如果不起名字Python就自动给线程命名为Thread-1，Thread-2……

###### lock

多线程和多进程最大的不同在于，多进程中，同一个变量，各自有一份拷贝存在于每个进程中，互不影响，而多线程中，所有变量都由所有线程共享，所以，任何一个变量都可以被任何一个线程修改，因此，线程之间共享数据最大的危险在于多个线程同时改一个变量，把内容给改乱了。

创建一个锁就是通过threading.Lock()来实现

    balance = 0
    lock = threading.Lock()

    def run_thread(n):
        for i in range(100000):
            # 先要获取锁:
            lock.acquire()
            try:
                # 放心地改吧:
                change_it(n)
            finally:
                # 改完了一定要释放锁:
                lock.release()

当多个线程同时执行lock.acquire()时，只有一个线程能成功地获取锁，然后继续执行代码，其他线程就继续等待直到获得锁为止。

获得锁的线程用完后一定要释放锁，否则那些苦苦等待锁的线程将永远等待下去，成为死线程。所以我们用try...finally来确保锁一定会被释放。

锁的好处就是确保了某段关键代码只能由一个线程从头到尾完整地执行，坏处当然也很多，首先是阻止了多线程并发执行，包含锁的某段代码实际上只能以单线程模式执行，效率就大大地下降了。其次，由于可以存在多个锁，不同的线程持有不同的锁，并试图获取对方持有的锁时，可能会造成死锁，导致多个线程全部挂起，既不能执行，也无法结束，只能靠操作系统强制终止。

###### 多核CPU

python的多线程无法跑满多个CPU核心，这是因为Python的线程虽然是真正的线程，但解释器执行代码时，有一个GIL锁：Global Interpreter Lock，任何Python线程执行前，必须先获得GIL锁，然后，每执行100条字节码，解释器就自动释放GIL锁，让别的线程有机会执行。这个GIL全局锁实际上把所有线程的执行代码都给上了锁，所以，多线程在Python中只能交替执行，即使100个线程跑在100核CPU上，也只能用到1个核。

GIL是Python解释器设计的历史遗留问题，通常我们用的解释器是官方实现的CPython，要真正利用多核，除非重写一个不带GIL的解释器。

所以，在Python中，可以使用多线程，但不要指望能有效利用多核。如果一定要通过多线程利用多核，那只能通过C扩展来实现，不过这样就失去了Python简单易用的特点。

不过，也不用过于担心，Python虽然不能利用多线程实现多核任务，但可以通过多进程实现多核任务。多个Python进程有各自独立的GIL锁，互不影响。

多线程编程，模型复杂，容易发生冲突，必须用锁加以隔离，同时，又要小心死锁的发生。

Python解释器由于设计时有GIL全局锁，导致了多线程无法利用多核。多线程的并发在Python中就是一个美丽的梦。

在多线程环境下，每个线程都有自己的数据。一个线程使用自己的局部变量比使用全局变量好，因为局部变量只有线程自己能看见，不会影响其他线程，而全局变量的修改必须加锁。

全局变量 `local_school` 就是一个ThreadLocal对象，每个Thread对它都可以读写student属性，但互不影响。你可以把 `local_school` 看成全局变量，但每个属性如 `local_school.student` 都是线程的局部变量，可以任意读写而互不干扰，也不用管理锁的问题，ThreadLocal内部会处理。

可以理解为全局变量 `local_school` 是一个dict，不但可以用 `local_school.student`，还可以绑定其他变量，如 `local_school.teacher` 等等。

ThreadLocal最常用的地方就是为每个线程绑定一个数据库连接，HTTP请求，用户身份信息等，这样一个线程的所有调用到的处理函数都可以非常方便地访问这些资源。

###### 进程 vs 线程

要实现多任务，通常我们会设计Master-Worker模式，Master负责分配任务，Worker负责执行任务，因此，多任务环境下，通常是一个Master，多个Worker。

如果用多进程实现Master-Worker，主进程就是Master，其他进程就是Worker。

如果用多线程实现Master-Worker，主线程就是Master，其他线程就是Worker。

多进程模式最大的优点就是稳定性高，因为一个子进程崩溃了，不会影响主进程和其他子进程。（当然主进程挂了所有进程就全挂了，但是Master进程只负责分配任务，挂掉的概率低）著名的Apache最早就是采用多进程模式。

多进程模式的缺点是创建进程的代价大，在Unix/Linux系统下，用fork调用还行，在Windows下创建进程开销巨大。另外，操作系统能同时运行的进程数也是有限的，在内存和CPU的限制下，如果有几千个进程同时运行，操作系统连调度都会成问题。

多线程模式通常比多进程快一点，但是也快不到哪去，而且，多线程模式致命的缺点就是任何一个线程挂掉都可能直接造成整个进程崩溃，因为所有线程共享进程的内存。在Windows上，如果一个线程执行的代码出了问题，你经常可以看到这样的提示：“该程序执行了非法操作，即将关闭”，其实往往是某个线程出了问题，但是操作系统会强制结束整个进程。

在Windows下，多线程的效率比多进程要高，所以微软的IIS服务器默认采用多线程模式。由于多线程存在稳定性的问题，IIS的稳定性就不如Apache。为了缓解这个问题，IIS和Apache现在又有多进程+多线程的混合模式，真是把问题越搞越复杂。

###### 线程切换

无论是多进程还是多线程，只要数量一多，效率肯定上不去

###### 计算密集型 vs. IO密集型

是否采用多任务的第二个考虑是任务的类型。我们可以把任务分为计算密集型和IO密集型。

计算密集型任务的特点是要进行大量的计算，消耗CPU资源，比如计算圆周率、对视频进行高清解码等等，全靠CPU的运算能力。这种计算密集型任务虽然也可以用多任务完成，但是任务越多，花在任务切换的时间就越多，CPU执行任务的效率就越低，所以，要最高效地利用CPU，计算密集型任务同时进行的数量应当等于CPU的核心数。

计算密集型任务由于主要消耗CPU资源，因此，代码运行效率至关重要。Python这样的脚本语言运行效率很低，完全不适合计算密集型任务。对于计算密集型任务，最好用C语言编写。

第二种任务的类型是IO密集型，涉及到网络、磁盘IO的任务都是IO密集型任务，这类任务的特点是CPU消耗很少，任务的大部分时间都在等待IO操作完成（因为IO的速度远远低于CPU和内存的速度）。对于IO密集型任务，任务越多，CPU效率越高，但也有一个限度。常见的大部分任务都是IO密集型任务，比如Web应用。

IO密集型任务执行期间，99%的时间都花在IO上，花在CPU上的时间很少，因此，用运行速度极快的C语言替换用Python这样运行速度极低的脚本语言，完全无法提升运行效率。对于IO密集型任务，最合适的语言就是开发效率最高（代码量最少）的语言，脚本语言是首选，C语言最差。

###### 异步IO

考虑到CPU和IO之间巨大的速度差异，一个任务在执行的过程中大部分时间都在等待IO操作，单进程单线程模型会导致别的任务无法并行执行，因此，我们才需要多进程模型或者多线程模型来支持多任务并发执行。

现代操作系统对IO操作已经做了巨大的改进，最大的特点就是支持异步IO。如果充分利用操作系统提供的异步IO支持，就可以用单进程单线程模型来执行多任务，这种全新的模型称为事件驱动模型，Nginx就是支持异步IO的Web服务器，它在单核CPU上采用单进程模型就可以高效地支持多任务。在多核CPU上，可以运行多个进程（数量与CPU核心数相同），充分利用多核CPU。由于系统总的进程数量十分有限，因此操作系统调度非常高效。用异步IO编程模型来实现多任务是一个主要的趋势。

对应到Python语言，单进程的异步编程模型称为协程，有了协程的支持，就可以基于事件驱动编写高效的多任务程序

###### 分布式进程

在Thread和Process中，应当优选Process，因为Process更稳定，而且，Process可以分布到多台机器上，而Thread最多只能分布到同一台机器的多个CPU上。

Python的multiprocessing模块不但支持多进程，其中managers子模块还支持把多进程分布到多台机器上。一个服务进程可以作为调度者，将任务分布到其他多个进程中，依靠网络通信。由于managers模块封装很好，不必了解网络通信的细节，就可以很容易地编写分布式多进程程序。

举个例子：如果我们已经有一个通过Queue通信的多进程程序在同一台机器上运行，现在，由于处理任务的进程任务繁重，希望把发送任务的进程和处理任务的进程分布到两台机器上。怎么用分布式进程实现？

原有的Queue可以继续使用，但是，通过managers模块把Queue通过网络暴露出去，就可以让其他机器的进程访问Queue了。

我们先看服务进程，服务进程负责启动Queue，把Queue注册到网络上，然后往Queue里面写入任务：

    # taskmanager.py
    
    import random, time, Queue
    from multiprocessing.managers import BaseManager
    
    # 发送任务的队列:
    task_queue = Queue.Queue()
    # 接收结果的队列:
    result_queue = Queue.Queue()
    
    # 从BaseManager继承的QueueManager:
    class QueueManager(BaseManager):
        pass
    
    # 把两个Queue都注册到网络上, callable参数关联了Queue对象:
    QueueManager.register('get_task_queue', callable=lambda: task_queue)
    QueueManager.register('get_result_queue', callable=lambda: result_queue)
    # 绑定端口5000, 设置验证码'abc':
    manager = QueueManager(address=('', 5000), authkey='abc')
    # 启动Queue:
    manager.start()
    # 获得通过网络访问的Queue对象:
    task = manager.get_task_queue()
    result = manager.get_result_queue()
    # 放几个任务进去:
    for i in range(10):
        n = random.randint(0, 10000)
        print('Put task %d...' % n)
        task.put(n)
    # 从result队列读取结果:
    print('Try get results...')
    for i in range(10):
        r = result.get(timeout=10)
        print('Result: %s' % r)
    # 关闭:
    manager.shutdown()

请注意，当我们在一台机器上写多进程程序时，创建的Queue可以直接拿来用，但是，在分布式多进程环境下，添加任务到Queue不可以直接对原始的task_queue进行操作，那样就绕过了QueueManager的封装，必须通过manager.get_task_queue()获得的Queue接口添加。

然后，在另一台机器上启动任务进程（本机上启动也可以）：

    # taskworker.py
    
    import time, sys, Queue
    from multiprocessing.managers import BaseManager
    
    # 创建类似的QueueManager:
    class QueueManager(BaseManager):
        pass
    
    # 由于这个QueueManager只从网络上获取Queue，所以注册时只提供名字:
    QueueManager.register('get_task_queue')
    QueueManager.register('get_result_queue')
    
    # 连接到服务器，也就是运行taskmanager.py的机器:
    server_addr = '127.0.0.1'
    print('Connect to server %s...' % server_addr)
    # 端口和验证码注意保持与taskmanager.py设置的完全一致:
    m = QueueManager(address=(server_addr, 5000), authkey='abc')
    # 从网络连接:
    m.connect()
    # 获取Queue的对象:
    task = m.get_task_queue()
    result = m.get_result_queue()
    # 从task队列取任务,并把结果写入result队列:
    for i in range(10):
        try:
            n = task.get(timeout=1)
            print('run task %d * %d...' % (n, n))
            r = '%d * %d = %d' % (n, n, n*n)
            time.sleep(1)
            result.put(r)
        except Queue.Empty:
            print('task queue is empty.')
    # 处理结束:
    print('worker exit.')

任务进程要通过网络连接到服务进程，所以要指定服务进程的IP。

现在，可以试试分布式进程的工作效果了。先启动taskmanager.py服务进程：

    $ python taskmanager.py 
    Put task 3411...
    Put task 1605...
    Put task 1398...
    Put task 4729...
    Put task 5300...
    Put task 7471...
    Put task 68...
    Put task 4219...
    Put task 339...
    Put task 7866...
    Try get results...

taskmanager进程发送完任务后，开始等待result队列的结果。现在启动taskworker.py进程：

    $ python taskworker.py 127.0.0.1
    Connect to server 127.0.0.1...
    run task 3411 * 3411...
    run task 1605 * 1605...
    run task 1398 * 1398...
    run task 4729 * 4729...
    run task 5300 * 5300...
    run task 7471 * 7471...
    run task 68 * 68...
    run task 4219 * 4219...
    run task 339 * 339...
    run task 7866 * 7866...
    worker exit.

taskworker进程结束，在taskmanager进程中会继续打印出结果：

    Result: 3411 * 3411 = 11634921
    Result: 1605 * 1605 = 2576025
    Result: 1398 * 1398 = 1954404
    Result: 4729 * 4729 = 22363441
    Result: 5300 * 5300 = 28090000
    Result: 7471 * 7471 = 55815841
    Result: 68 * 68 = 4624
    Result: 4219 * 4219 = 17799961
    Result: 339 * 339 = 114921
    Result: 7866 * 7866 = 61873956

这个简单的Manager/Worker模型有什么用？其实这就是一个简单但真正的分布式计算，把代码稍加改造，启动多个worker，就可以把任务分布到几台甚至几十台机器上，比如把计算`n*n`的代码换成发送邮件，就实现了邮件队列的异步发送。

Queue对象存储在哪？注意到taskworker.py中根本没有创建Queue的代码，所以，Queue对象存储在taskmanager.py进程中：

![](http://www.xiaoh.me/img/post-python-study-process1.png)

而Queue之所以能通过网络访问，就是通过QueueManager实现的。由于QueueManager管理的不止一个Queue，所以，要给每个Queue的网络调用接口起个名字，比如`get_task_queue`。

authkey有什么用？这是为了保证两台机器正常通信，不被其他机器恶意干扰。如果taskworker.py的authkey和taskmanager.py的authkey不一致，肯定连接不上。

Python的分布式进程接口简单，封装良好，适合需要把繁重任务分布到多台机器的环境下。

注意Queue的作用是用来传递任务和接收结果，每个任务的描述数据量要尽量小。比如发送一个处理日志文件的任务，就不要发送几百兆的日志文件本身，而是发送日志文件存放的完整路径，由Worker进程再去共享的磁盘上读取文件。


---

#### [常用的内建函数](#innerfunction)


###### namedtuple

namedtuple是一个函数，它用来创建一个自定义的tuple对象，并且规定了tuple元素的个数，并可以用属性而不是索引来引用tuple的某个元素。

这样一来，我们用namedtuple可以很方便地定义一种数据类型，它具备tuple的不变性，又可以根据属性来引用，使用十分方便。

可以验证创建的Point对象是tuple的一种子类：

    >>> isinstance(p, Point)
    True
    >>> isinstance(p, tuple)
    True
    
类似的，如果要用坐标和半径表示一个圆，也可以用namedtuple定义：

    # namedtuple('名称', [属性list]):
    Circle = namedtuple('Circle', ['x', 'y', 'r'])

###### deque

deque是为了高效实现插入和删除操作的双向列表，适合用于队列和栈：

    >>> from collections import deque
    >>> q = deque(['a', 'b', 'c'])
    >>> q.append('x')
    >>> q.appendleft('y')
    >>> q
    deque(['y', 'a', 'b', 'c', 'x'])

deque除了实现list的append()和pop()外，还支持appendleft()和popleft()，这样就可以非常高效地往头部添加或删除元素。

###### defaultdict

使用dict时，如果引用的Key不存在，就会抛出KeyError。如果希望key不存在时，返回一个默认值，就可以用defaultdict：

    >>> from collections import defaultdict
    >>> dd = defaultdict(lambda: 'N/A')
    >>> dd['key1'] = 'abc'
    >>> dd['key1'] # key1存在
    'abc'
    >>> dd['key2'] # key2不存在，返回默认值
    'N/A'

注意默认值是调用函数返回的，而函数在创建defaultdict对象时传入。

除了在Key不存在时返回默认值，defaultdict的其他行为跟dict是完全一样的。

###### OrderedDict

使用dict时，Key是无序的。在对dict做迭代时，我们无法确定Key的顺序。

如果要保持Key的顺序，可以用OrderedDict：

    >>> from collections import OrderedDict
    >>> d = dict([('a', 1), ('b', 2), ('c', 3)])
    >>> d # dict的Key是无序的
    {'a': 1, 'c': 3, 'b': 2}
    >>> od = OrderedDict([('a', 1), ('b', 2), ('c', 3)])
    >>> od # OrderedDict的Key是有序的
    OrderedDict([('a', 1), ('b', 2), ('c', 3)])

注意，OrderedDict的Key会按照插入的顺序排列，不是Key本身排序：

    >>> od = OrderedDict()
    >>> od['z'] = 1
    >>> od['y'] = 2
    >>> od['x'] = 3
    >>> od.keys() # 按照插入的Key的顺序返回
    ['z', 'y', 'x']

OrderedDict可以实现一个FIFO（先进先出）的dict，当容量超出限制时，先删除最早添加的Key：

    from collections import OrderedDict

    class LastUpdatedOrderedDict(OrderedDict):

        def __init__(self, capacity):
            super(LastUpdatedOrderedDict, self).__init__()
            self._capacity = capacity

        def __setitem__(self, key, value):
            containsKey = 1 if key in self else 0
            if len(self) - containsKey >= self._capacity:
                last = self.popitem(last=False)
                print 'remove:', last
            if containsKey:
                del self[key]
                print 'set:', (key, value)
            else:
                print 'add:', (key, value)
            OrderedDict.__setitem__(self, key, value)

###### Counter

Counter是一个简单的计数器，例如，统计字符出现的个数：

    >>> from collections import Counter
    >>> c = Counter()
    >>> for ch in 'programming':
    ...     c[ch] = c[ch] + 1
    ...
    >>> c
    Counter({'g': 2, 'm': 2, 'r': 2, 'a': 1, 'i': 1, 'o': 1, 'n': 1, 'p': 1})

Counter实际上也是dict的一个子类，上面的结果可以看出，字符'g'、'm'、'r'各出现了两次，其他字符各出现了一次。

###### Base64

Base64是一种任意二进制到文本字符串的编码方法，常用于在URL、Cookie、网页中传输少量二进制数据。

Base64是一种通过查表的编码方法，不能用于加密，即使使用自定义的编码表也不行。

Base64适用于小段内容的编码，比如数字证书签名、Cookie的内容等。

    >>> import base64
    >>> base64.b64encode('binary\x00string')
    'YmluYXJ5AHN0cmluZw=='
    >>> base64.b64decode('YmluYXJ5AHN0cmluZw==')
    'binary\x00string'
    >>> base64.b64encode('i\xb7\x1d\xfb\xef\xff')
    'abcd++//'
    >>> base64.urlsafe_b64encode('i\xb7\x1d\xfb\xef\xff')
    'abcd--__'
    >>> base64.urlsafe_b64decode('abcd--__')
    'i\xb7\x1d\xfb\xef\xff'
    # 标准Base64:
    'abcd' -> 'YWJjZA=='
    # 自动去掉=:
    'abcd' -> 'YWJjZA'
    >>> base64.b64decode('YWJjZA==')
    'abcd'
    >>> base64.b64decode('YWJjZA')
    Traceback (most recent call last):
      ...
    TypeError: Incorrect padding
    >>> safe_b64decode('YWJjZA')
    'abcd'

###### struct

Python提供了一个struct模块来解决str和其他二进制数据类型的转换。

struct的pack函数把任意数据类型变成字符串：

    >>> import struct
    >>> struct.pack('>I', 10240099)
    '\x00\x9c@c'

pack的第一个参数是处理指令，`'>I'` 的意思是：

>表示字节顺序是big-endian，也就是网络序，I表示4字节无符号整数。

后面的参数个数要和处理指令一致。

unpack把str变成相应的数据类型：

    >>> struct.unpack('>IH', '\xf0\xf0\xf0\xf0\x80\x80')
    (4042322160, 32896)

根据 `>IH` 的说明，后面的str依次变为I：4字节无符号整数和H：2字节无符号整数。

###### hashlib

Python的hashlib提供了常见的摘要算法，如MD5，SHA1等等。

    import hashlib

    md5 = hashlib.md5()
    md5.update('how to use md5 in python hashlib?')
    print md5.hexdigest()

计算结果如下：

    d26a53750bc40b38b65a520292f69306

如果数据量很大，可以分块多次调用update()，最后计算的结果是一样的：

    md5 = hashlib.md5()
    md5.update('how to use md5 in ')
    md5.update('python hashlib?')
    print md5.hexdigest()

    import hashlib

    sha1 = hashlib.sha1()
    sha1.update('how to use sha1 in ')
    sha1.update('python hashlib?')
    print sha1.hexdigest()

###### itertools

Python的内建模块itertools提供了非常有用的用于操作迭代对象的函数。

itertools模块提供的全部是处理迭代功能的函数，它们的返回值不是list，而是迭代对象，只有用for循环迭代的时候才真正计算。

* itertools.count(1) 创建一个无限的迭代器
* itertools.cycle('ABC') 把传入的一个序列无限重复下去
* itertools.repeat('A', 10) 把一个元素无限重复下去，不过如果提供第二个参数就可以限定重复次数

无限序列只有在for迭代时才会无限地迭代下去，如果只是创建了一个迭代对象，它不会事先把无限个元素生成出来，事实上也不可能在内存中创建无限多个元素。

无限序列虽然可以无限迭代下去，但是通常我们会通过takewhile()等函数根据条件判断来截取出一个有限的序列：

    >>> natuals = itertools.count(1)
    >>> ns = itertools.takewhile(lambda x: x <= 10, natuals)
    >>> for n in ns:
    ...     print n
    ...
    打印出1到10

chain()可以把一组迭代对象串联起来，形成一个更大的迭代器：

    for c in chain('ABC', 'XYZ'):
        print c
    # 迭代效果：'A' 'B' 'C' 'X' 'Y' 'Z'

groupby()把迭代器中相邻的重复元素挑出来放在一起：

    >>> for key, group in itertools.groupby('AAABBBCCAAA'):
    ...     print key, list(group) # 为什么这里要用list()函数呢？
    ...
    A ['A', 'A', 'A']
    B ['B', 'B', 'B']
    C ['C', 'C']
    A ['A', 'A', 'A']

实际上挑选规则是通过函数完成的，只要作用于函数的两个元素返回的值相等，这两个元素就被认为是在一组的，而函数返回值作为组的key。如果我们要忽略大小写分组，就可以让元素'A'和'a'都返回相同的key：

    >>> for key, group in itertools.groupby('AaaBBbcCAAa', lambda c: c.upper()):
    ...     print key, list(group)
    ...
    A ['A', 'a', 'a']
    B ['B', 'B', 'b']
    C ['c', 'C']
    A ['A', 'A', 'a']

imap()和map()的区别在于，imap()可以作用于无穷序列，并且，如果两个序列的长度不一致，以短的那个为准。

    >>> for x in itertools.imap(lambda x, y: x * y, [10, 20, 30], itertools.count(1)):
    ...     print x
    ...
    10
    40
    90

注意imap()返回一个迭代对象，而map()返回list。当你调用map()时，已经计算完毕：

    >>> r = map(lambda x: x*x, [1, 2, 3])
    >>> r # r已经计算出来了
    [1, 4, 9]

当你调用imap()时，并没有进行任何计算：

    >>> r = itertools.imap(lambda x: x*x, [1, 2, 3])
    >>> r
    <itertools.imap object at 0x103d3ff90>
    # r只是一个迭代对象

必须用for循环对r进行迭代，才会在每次循环过程中计算出下一个元素：

    >>> for x in r:
    ...     print x
    ...
    1
    4
    9

###### PIL 

PIL：Python Imaging Library，已经是Python平台事实上的图像处理标准库了。PIL功能非常强大，但API却非常简单易用。

PIL的ImageDraw提供了一系列绘图方法，让我们可以直接绘图。比如要生成字母验证码图片：

    import Image, ImageDraw, ImageFont, ImageFilter
    import random
    
    # 随机字母:
    def rndChar():
        return chr(random.randint(65, 90))
    
    # 随机颜色1:
    def rndColor():
        return (random.randint(64, 255), random.randint(64, 255), random.randint(64, 255))
    
    # 随机颜色2:
    def rndColor2():
        return (random.randint(32, 127), random.randint(32, 127), random.randint(32, 127))
    
    # 240 x 60:
    width = 60 * 4
    height = 60
    image = Image.new('RGB', (width, height), (255, 255, 255))
    # 创建Font对象:
    font = ImageFont.truetype('Arial.ttf', 36)
    # 创建Draw对象:
    draw = ImageDraw.Draw(image)
    # 填充每个像素:
    for x in range(width):
        for y in range(height):
            draw.point((x, y), fill=rndColor())
    # 输出文字:
    for t in range(4):
        draw.text((60 * t + 10, 10), rndChar(), font=font, fill=rndColor2())
    # 模糊:
    image = image.filter(ImageFilter.BLUR)
    image.save('code.jpg', 'jpeg');

---

#### [网络编程](#networkcoding)

###### tcp编程

服务器端：

    #!/usr/bin/python
    #-*- coding:utf-8 -*-
    
    #############################################
    # File Name: server.py
    # Author: xiaoh
    # Mail: xiaoh@about.me
    # Created Time:  2016-02-16 12:58:57
    #############################################
    
    import socket, threading, time
    
    def tcplink(sock, addr):
        print 'Accept new connection from %s:%s...' % addr
        sock.send('Welcome!')
        while True:
            data = sock.recv(1024)
            time.sleep(1)
            if data == 'exit' or not data:
                break
            sock.send('Hello, %s!' % data)
        sock.close()
        print 'Connection from %s:%s closed.' % addr
    
    def main():
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.bind(('127.0.0.1', 9999))
        s.listen(2)
        print 'Waiting for connection...'
        while True:
            sock, addr = s.accept()
            t = threading.Thread(target=tcplink, args=(sock, addr))
            t.start()
        print 'Program will be quit now.'
    
    if __name__ == "__main__":
        main()

客户端：

    #!/usr/bin/python
    #-*- coding:utf-8 -*-
    
    #############################################
    # File Name: client.py
    # Author: xiaoh
    # Mail: xiaoh@about.me
    # Created Time:  2016-02-16 13:24:49
    #############################################
    
    import socket
    
    def main():
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect(('127.0.0.1', 9999))
        print s.recv(1024)
        for data in ['xiaoh', 'xingming', 'huo']:
            s.send(data)
            print s.recv(1024)
        s.send('exit')
        s.close()
    
    if __name__ == "__main__":
        main()

###### udp 编程

服务器端：

    #!/usr/bin/python
    #-*- coding:utf-8 -*-
    
    #############################################
    # File Name: server_udp.py
    # Author: xiaoh
    # Mail: xiaoh@about.me
    # Created Time:  2016-02-16 13:41:58
    #############################################
    
    import socket
    
    def main():
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        print 'Bind UDP on 9999...'
        s.bind(('127.0.0.1', 9999))
        while True:
            data, addr = s.recvfrom(1024)
            print 'Received from %s:%s.' % addr
            s.sendto('Hello, %s!' % data, addr)
        print 'Program will be quit now.'
    
    if __name__ == "__main__":
        main()

客户端：

    #!/usr/bin/python
    #-*- coding:utf-8 -*-
    
    #############################################
    # File Name: client_udp.py
    # Author: xiaoh
    # Mail: xiaoh@about.me
    # Created Time:  2016-02-16 13:43:24
    #############################################
    
    import socket
    
    def main():
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        for data in ['xiaoh', 'xingming', 'huo']:
            # 发送数据:
            s.sendto(data, ('127.0.0.1', 9999))
            # 接收数据:
            print s.recv(1024)
        s.close()
    
    if __name__ == "__main__":
        main()

---

### END



