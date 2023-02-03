---
layout:     post
title:      "Thrift架构介绍"
subtitle:   "跨语言服务部署架构之Thrift简介"
date:       2016-03-28 10:19:05
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - Thrift
---

### 目录

0. [架构](#arthitecture)
0. [传输](#translate)
    0. [支持的传输格式](#supportformat)
    0. [支持的数据传输方式](#supportmode)
    0. [支持的服务模型](#supportservice)
0. [Thrift安装](#thriftinstall)
0. [基本语法](#basicsyntax)
    0. [类型](#types)
    0. [枚举类型](#enum)
    0. [注释](#comment)
    0. [命名空间](#namespace)
    0. [文件包含](#include)
    0. [常量](#constant)
    0. [定义结构体](#structs)
    0. [定义服务](#designservice)
0. [生成代码](#generatecode)
    0. [Transport](#transporttransport)
    0. [Protocol](#protocolprotocol)
    0. [Processor](#processorprocessor)
    0. [Server](#serverserver)
    0. [应用举例](#examples)
    0. [实践经验](#realexample)
0. [利用Thrift部署服务](#thriftdeployservice)
    0. [thrift文件编写](#thriftwritefile)
0. [client端和server端代码编写](#clientservercoding)
0. [参考资料](#learningfrom)

---

> Thrift是一个跨语言的服务部署框架，最初由Facebook于2007年开发，2008年进入Apache开源项目。Thrift通过一个中间语言(IDL, 接口定义语言)来定义RPC的接口和数据类型，然后通过一个编译器生成不同语言的代码（目前支持C++,Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, Smalltalk和OCaml）,并由生成的代码负责RPC协议层和传输层的实现。

---

### [架构](#arthitecture)

Thrift实际上是实现了C/S模式，通过代码生成工具将接口定义文件生成服务器端和客户端代码（可以为不同语言），从而实现服务端和客户端跨语言的支持。用户在Thirft描述文件中声明自己的服务，这些服务经过编译后会生成相应语言的代码文件，然后用户实现服务（客户端调用服务，服务器端提供服务）便可以了。其中protocol（协议层, 定义数据传输格式，可以为二进制或者XML等）和transport（传输层，定义数据传输方式，可以为TCP/IP传输，内存共享或者文件共享等）被用作运行时库。

![](/img/post-thrift-arthitecture.jpg)

---

### [传输](#translate)

##### [支持的传输格式](#supportformat)

* TBinaryProtocol – 二进制格式.
* TCompactProtocol – 压缩格式
* TJSONProtocol – JSON格式
* TSimpleJSONProtocol –提供JSON只写协议, 生成的文件很容易通过脚本语言解析。
* TDebugProtocol – 使用易懂的可读的文本格式，以便于debug

##### [支持的数据传输方式](#supportmode)

* TSocket -阻塞式socker
* TFramedTransport – 以frame为单位进行传输，非阻塞式服务中使用。
* TFileTransport – 以文件形式进行传输。
* TMemoryTransport – 将内存用于I/O. java实现时内部实际使用了简单的ByteArrayOutputStream。
* TZlibTransport – 使用zlib进行压缩， 与其他传输方式联合使用。当前无java实现。

##### [支持的服务模型](#supportservice)

* TSimpleServer – 简单的单线程服务模型，常用于测试
* TThreadPoolServer – 多线程服务模型，使用标准的阻塞式IO。
* TNonblockingServer – 多线程服务模型，使用非阻塞式IO（需使用TFramedTransport数据传输方式）

---

### [Thrift安装](#install)

下载地址：<http://incubator.apache.org/thrift/download/>

* 安装要求：Unix/linux 系统，windows+cygwin
* C++语言：g++、boost
* java 语言：JDK、Apache Ant
* 其他语言：Python、PHP、Perl, etc…

编译安装：

```Shell
./configure 
make
make install
```

---

### [基本语法](#basicsyntax)

##### [类型](#types)

Thrift类型系统包括预定义基本类型，用户自定义结构体，容器类型，异常和服务定义

###### 基本类型

* bool：布尔类型(true or value)，占一个字节
* byte：有符号字节 
* i16:16位有符号整型
* i32:32位有符号整型
* i64:64位有符号整型
* double：64位浮点数
* string：未知编码或者二进制的字符串

注意，thrift不支持无符号整型，因为很多目标语言不存在无符号整型（如java）。

##### 容器类型

Thrift容器与类型密切相关，它与当前流行编程语言提供的容器类型相对应，采用java泛型风格表示的。Thrift提供了3种容器类型：

* List<t1>：一系列t1类型的元素组成的有序表，元素可以重复
* Set<t1>：一系列t1类型的元素组成的无序表，元素唯一
* Map<t1,t2>：key/value对（key的类型是t1且key唯一，value类型是t2）。

容器中的元素类型可以是除了service意外的任何合法thrift类型（包括结构体和异常）。

###### 结构体和异常

Thrift结构体在概念上同C语言结构体类型—-一种将相关属性聚集（封装）在一起的方式。在面向对象语言中，thrift结构体被转换成类。

异常在语法和功能上类似于结构体，只不过异常使用关键字exception而不是struct关键字声明。但它在语义上不同于结构体—当定义一个RPC服务时，开发者可能需要声明一个远程方法抛出一个异常。

###### 服务

服务的定义方法在语法上等同于面向对象语言中定义接口。Thrift编译器会产生实现这些接口的client和server桩。

###### 类型定义

Thrift支持C/C++风格的typedef:

```C
typedef i32 MyInteger   \\a
typedef Tweet ReTweet  \\b
```

说明：

0. 末尾没有逗号
0. struct可以使用typedef

##### [枚举类型](#enum)

可以像C/C++那样定义枚举类型，如：

```C
enum TweetType {
    TWEET,       //a
    RETWEET = 2, //b
    DM = 0xa,  //c
    REPLY
}        //d
struct Tweet {
    required i32 userId;
    required string userName;
    required string text;
    optional Location loc;
    optional TweetType tweetType = TweetType.TWEET // e
    optional string language = "english"
}
```

说明：

*  编译器默认从0开始赋值
*  可以赋予某个常量某个整数
*  允许常量是十六进制整数
*  末尾没有逗号
*  给常量赋缺省值时，使用常量的全称

注意，不同于protocol buffer，thrift不支持枚举类嵌套，枚举常量必须是32位的正整数

##### [注释](#comment)

Thrfit支持shell注释风格，C/C++语言中单行或者多行注释风格

```Shell
# This is a valid comment.
/*
* This is a multi-line comment.
* Just like in C.
*/
// C++/Java style single-line comments work just as well.
```

##### [命名空间](#namespace)

Thrift中的命名空间同C++中的namespace和java中的package类似，它们均提供了一种组织（隔离）代码的方式。因为每种语言均有自己的命名空间定义方式（如python中有module），thrift允许开发者针对特定语言定义namespace：

```C
namespace cpp com.example.project  // a
namespace java com.example.project // b
```

说明：

0. 转化成namespace com { namespace example { namespace project {
0. 转换成package com.example.project

##### [文件包含](#include)

Thrift允许thrift文件包含，用户需要使用thrift文件名作为前缀访问被包含的对象，如：

```C
include "tweet.thrift"           // a
...
struct TweetSearchResult {
    list<tweet.Tweet> tweets; // b
}
```

说明：

0. thrift文件名要用双引号包含，末尾没有逗号或者分号
0. 注意tweet前缀

##### [常量](#constant)

Thrift允许用户定义常量，复杂的类型和结构体可使用JSON形式表示。

```C
const i32 INT_CONST = 1234;    // a
const map<string,string> MAP_CONST = {"hello": "world", "goodnight": "moon"}
```

说明：分号是可选的，可有可无；支持十六进制赋值。

##### [定义结构体](#structs)

结构体由一系列域组成，每个域有唯一整数标识符，类型，名字和可选的缺省参数组成。如：

```C
struct Tweet {
    required i32 userId;                  // a
    required string userName;             // b
    required string text;
    optional Location loc;                // c
    optional string language = "english" // d
}
struct Location {                            // e
    required double latitude;
    required double longitude;
}
```

说明：

0. 每个域有一个唯一的，正整数标识符
0. 每个域可以标识为required或者optional（也可以不注明）
0. 结构体可以包含其他结构体
0. 域可以有缺省值
0. 一个thrift中可定义多个结构体，并存在引用关系

规范的struct定义中的每个域均会使用required或者optional关键字进行标识。如果required标识的域没有赋值，thrift将给予提示。如果optional标识的域没有赋值，该域将不会被序列化传输。如果某个optional标识域有缺省值而用户没有重新赋值，则该域的值一直为缺省值。

与service不同，结构体不支持继承，即，一个结构体不能继承另一个结构体。

##### [定义服务](#designservice)

在流行的序列化/反序列化框架（如protocol buffer）中，thrift是少有的提供多语言间RPC服务的框架。

Thrift编译器会根据选择的目标语言为server产生服务接口代码，为client产生桩代码。

```C
//“Twitter”与“{”之间需要有空格！！！
service Twitter {
// 方法定义方式类似于C语言中的方式，它有一个返回值，一系列参数和可选的异常
// 列表. 注意，参数列表和异常列表定义方式与结构体中域定义方式一致.
void ping(),                                    // a
bool postTweet(1:Tweet tweet);                  // b
TweetSearchResult searchTweets(1:string query); // c
// ”oneway”标识符表示client发出请求后不必等待回复（非阻塞）直接进行下面的操作，
// ”oneway”方法的返回值必须是void
oneway void zip()                               // d
}
```

说明：

0. 函数定义可以使用逗号或者分号标识结束
0. 参数可以是基本类型或者结构体，参数是只读的（const），不可以作为返回值！！！
0. 返回值可以是基本类型或者结构体
0. 返回值可以是void

注意，**函数中参数列表的定义方式与struct完全一样**

Service支持继承，一个service可使用extends关键字继承另一个service

### [生成代码](#generatecode)

本节介绍thrift产生各种目标语言代码的方式。本节从几个基本概念开始，逐步引导开发者了解产生的代码是怎么样组织的，进而帮助开发者更快地明白thrift的使用方法。
概念

Thrift的网络栈如下所示：

![](/img/post-thrift-stack.jpg)

##### [Transport](#transport)

Transport层提供了一个简单的网络读写抽象层。这使得thrift底层的transport从系统其它部分（如：序列化/反序列化）解耦。以下是一些Transport接口提供的方法：

```
open
close
read
write
flush
```

除了以上几个接口，Thrift使用ServerTransport接口接受或者创建原始transport对象。正如名字暗示的那样，ServerTransport用在server端，为到来的连接创建Transport对象。

```
open
listen
accept
close
```

##### [Protocol](#protocol)

Protocol抽象层定义了一种将内存中数据结构映射成可传输格式的机制。换句话说，Protocol定义了datatype怎样使用底层的Transport对自己进行编解码。因此，Protocol的实现要给出编码机制并负责对数据进行序列化。

Protocol接口的定义如下：

```
writeMessageBegin(name, type, seq)
writeMessageEnd()
writeStructBegin(name)
writeStructEnd()
writeFieldBegin(name, type, id)
writeFieldEnd()
writeFieldStop()
writeMapBegin(ktype, vtype, size)
writeMapEnd()
writeListBegin(etype, size)
writeListEnd()
writeSetBegin(etype, size)
writeSetEnd()
writeBool(bool)
writeByte(byte)
writeI16(i16)
writeI32(i32)
writeI64(i64)
writeDouble(double)
writeString(string)
name, type, seq = readMessageBegin()
readMessageEnd()
name = readStructBegin()
readStructEnd()
name, type, id = readFieldBegin()
readFieldEnd()
k, v, size = readMapBegin()
readMapEnd()
etype, size = readListBegin()
readListEnd()
etype, size = readSetBegin()
readSetEnd()
bool = readBool()
byte = readByte()
i16 = readI16()
i32 = readI32()
i64 = readI64()
double = readDouble()
string = readString()
```

下面是一些对大部分thrift支持的语言均可用的protocol：

0. binary：简单的二进制编码
0. Compact：具体见THRIFT-11
0. Json

##### [Processor](#processor)

Processor封装了从输入数据流中读数据和向数据数据流中写数据的操作。读写数据流用Protocol对象表示。Processor的结构体非常简单:

```
interface TProcessor {
    bool process(TProtocol in, TProtocol out) throws TException
}
```

与服务相关的processor实现由编译器产生。Processor主要工作流程如下：从连接中读取数据（使用输入protocol），将处理授权给handler（由用户实现），最后将结果写到连接上（使用输出protocol）。

##### [Server](#server)

Server将以上所有特性集成在一起：

0. 创建一个transport对象
0. 为transport对象创建输入输出protocol
0. 基于输入输出protocol创建processor
0. 等待连接请求并将之交给processor处理

##### [应用举例](#examples)

下面，我们讨论thrift文件产生的特定语言代码。下面给出thrift文件描述：

```
namespace cpp thrift.example
namespace java thrift.example
enum TweetType {
    TWEET,
    RETWEET = 2,
    DM = 0xa,
    REPLY
}
struct Location {
    required double latitude;
    required double longitude;
}
struct Tweet {
    required i32 userId;
    required string userName;
    required string text;
    optional Location loc;
    optional TweetType tweetType = TweetType.TWEET;
    optional string language = "english";
}
typedef list<Tweet> TweetList
struct TweetSearchResult {
    TweetList tweets;
}
const i32 MAX_RESULTS = 100;
service Twitter {
    void ping(),
    bool postTweet(1:Tweet tweet);
    TweetSearchResult searchTweets(1:string query);
    oneway void zip()
}
```

###### Java语言

**产生的文件**

一个单独的文件（Constants.java）包含所有的常量定义。

每个结构体，枚举或者服务各占一个文件

```Shell
$ tree gen-java
`– thrift
    `– example
    |– Constants.java
    |– Location.java
    |– Tweet.java
    |– TweetSearchResult.java
    |– TweetType.java
    `– Twitter.java
```

**类型**

thrift将各种基本类型和容器类型映射成java类型：

```C
bool: boolean
byte: byte
i16: short
i32: int
i64: long
double: double
string: String
list<t1>: List<t1>
set<t1>: Set<t1>
map<t1,t2>: Map<t1, t2>
```

**typedef**

Java不支持typedef，它只使用原始类型，如，在上面的例子中，产生的代码中，TweetSearchResult会被还原成list<Tweet> tweets

**Enum**

Thrift直接将枚举类型映射成java的枚举类型。用户可以使用geValue方法获取枚举常量的值。此外，编译器会产生一个findByValue方法获取枚举对应的数值。

**常量**

Thrift把所有的常量放在一个叫Constants的public类中，每个常量修饰符是public static final。

###### C++语言

**产生的文件**

所有变量均存放在一个.cpp/.h文件对中

所有的类型定义（枚举或者结构体）存放到另一个.cpp/.h文件对中

每一个service有自己的.cpp/.h文件

```C
$ tree gen-cpp
|– example_constants.cpp
|– example_constants.h
|– example_types.cpp
|– example_types.h
|– Twitter.cpp
|– Twitter.h
`– Twitter_server.skeleton.cpp
```

###### 其他语言

Python，Ruby，javascript等

##### [实践经验](#realexample)

thrift文件内容可能会随着时间变化的。如果已经存在的消息类型不再符合设计要求，比如，新的设计要在message格式中添加一个额外字段，但你仍想使用以前的thrift文件产生的处理代码。如果想要达到这个目的，只需：

* 不要修改已存在域的整数编号
* 新添加的域必须是optional的，以便格式兼容。对于一些语言，如果要为optional的字段赋值，需要特殊处理，比如对于C++语言，要为

```C
struct Example{
    i32 id,
    string name,
    optional age,
}
```

中的optional字段age赋值，需要将它的 `__isset` 值设为true，这样才能序列化并传输或者存储（不然optional字段被认为不存在，不会被传输或者存储），

如：

```Shell
Example example;
example.age=10,
example.__isset.age = true; //__isset是每个thrift对象的自带的public成员，来指定optional字段是否启用并赋值。
```
* 非required域可以删除，前提是它的整数编号不会被其他域使用。对于删除的字段，名字前面可添加 `OBSOLETE_`以防止其他字段使用它的整数编号。
* thrift文件应该是unix格式的（windows下的换行符与unix不同，可能会导致你的程序编译不过），如果是在window下编写的，可使用dos2unix转化为unix格式。
* 貌似当前的thrift版本（0.6.1）不支持常量表达式的定义（如 const i32 DAY = 24 * 60 * 60），这可能是考虑到不同语言，运算符不尽相同。

---

### [利用Thrift部署服务](#deployservice)

主要流程：

0. 编写服务说明，保存到.thrift文件
0. 根据需要， 编译.thrift文件，生成相应的语言源代码
0. 根据实际需要， 编写client端和server端代码。

##### [thrift文件编写](#writefile)

一般将服务放到一个.thrift文件中，服务的编写语法与C语言语法基本一致，在.thrift文件中有主要有以下几个内容：

0. 变量声明
0. 数据声明（struct）
0. 服务接口声明（service, 可以继承其他接口）。

下面分析Thrift的tutorial中带的例子tutorial.thrift

* 包含头文件：59行：`include “shared.thrift”`
* 指定目标语言：65行：`namespace cpp tutorial`
* 定义变量：80行：`const i32 INT32CONSTANT = 9853`
* 定义结构体：103行：

```C
struct Work {
    i32 num1 = 0,
    i32 num2,
    Operation op,
    optional string comment,
}
```

定义服务：

```C
service Calculator extends shared.SharedService {
void ping(),
    i32 add(1:i32 num1, 2:i32 num2),
    i32 calculate(1:i32 logid, 2:Work w) throws (1:InvalidOperation ouch),
    oneway void zip()
}
```

* 要生成C++代码：./thrift --gen cpp tutorial.thrift，结果代码存放在gen-cpp目录下
* 要生成java代码：./thrift --gen java tutorial.thrift，结果代码存放在gen-java目录下

---

### [client端和server端代码编写](#coding)

client端和sever端代码要调用编译.thrift生成的中间文件。下面分析cpp文件下面的CppClient.cpp和CppServer.cpp代码

![](/img/post-thrift-coding.jpg)

在client端，用户自定义CalculatorClient类型的对象（用户在.thrift文件中声明的服务名称是Calculator， 则生成的中间代码中的主类为CalculatorClient）， 该对象中封装了各种服务，可以直接调用（如client.ping()）, 然后thrift会通过封装的rpc调用server端同名的函数。

在server端，需要实现在.thrift文件中声明的服务中的所有功能，以便处理client发过来的请求。

---

### [参考资料](#learningfrom)

0. <http://diwakergupta.github.com/thrift-missing-guide/#_versioning_compatibility>
0. <http://dongxicheng.org/search-engine/thrift-framework-intro/>
0. <http://dongxicheng.org/search-engine/thrift-guide/>
0. <http://dongxicheng.org/search-engine/search-engine/thrift-rpc/>
0. <http://dongxicheng.org/search-engine/thrift-internals/>
0. <http://wiki.apache.org/thrift/>
0. <http://jnb.ociweb.com/jnb/jnbJun2009.html>
0. <http://blog.rushcj.com/tag/thrift/>
0. <http://www.vvcha.cn/c.aspx?id=31984>

---

### END


