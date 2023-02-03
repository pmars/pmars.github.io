---
layout:     post
title:      "Linux下服务器压力测试工具介绍"
subtitle:   "ab, http_load, webbench, siege"
date:       2016-01-04 06:28:19
author:     "xiaoh"
header-img: "img/post-bg-web-testing.jpg"
tags:
    - 服务器
    - 测试
---

> 这段时间一直在照看孩子，也没有多少时间来写博客，今天是16年第一天上班，也开始了我的2016的生活，总觉得孩子的到来，给自己了很多的动力，我会努力将博客写好的。
> 今天想研究一下 nginx + lua + redis 整合起来能做点啥，后来发现了大神写的 openresty 这个工具，就简单弄了个web服务，之后看文档有测试的部分，后来查了一些，也测试了一些，所以简单写在这里。
> openresty 或者 nginx + lua + redis 的博客等日后再来补上。

---

### 目录

0. [ab](#abab)
0. [http\_load](#httploadhttpload)
0. [webbench](#webbenchwebbench)
0. [Siege](#siegesiege)

---

### [ab](#ab)

###### 安装

`ab` 是 `apache` 自带的一款功能强大的测试工具，安装了 `apache` 一般就自带了

    $ sudo apt-get install apache2-utils

###### 测试

    $ ab -c 1000 -n 10000 http://localhost:8080/     # 这个表示同时处理1000个请求并运行10000次

###### 缺陷

程序中有各种静态声明的固定长度的缓冲区。 另外，对命令行参数、服务器的响应头和其他外部输入的解析也很简单，这可能会有不良后果。

它没有完整地实现HTTP/1.x; 仅接受某些'预想'的响应格式。 strstr(3)的频繁使用可能会带来性能问题，即, 你可能是在测试ab而不是服务器的性能。

###### HELP

具体结果请自行测试，详细参数请参见 `ab`，部分解释如下：

```
Options are:
    -n requests     Number of requests to perform
                    在测试会话中所执行的请求个数。 默认时，仅执行一个请求，但通常其结果不具有代表意义
    -c concurrency  Number of multiple requests to make at a time
                    一次产生的请求个数。默认是一次一个
    -t timelimit    Seconds to max. to spend on benchmarking
                    This implies -n 50000
                    测试所进行的最大秒数。其内部隐含值是-n 50000。 它可以使对服务器的测试限制在一个固定的总时间以内。默认时，没有时间限制
    -s timeout      Seconds to max. wait for each response
                    Default is 30 seconds
                    单个请求最大延迟，默认为30s
    -b windowsize   Size of TCP send/receive buffer, in bytes
                    
    -B address      Address to bind to when making outgoing connections
    -p postfile     File containing data to POST. Remember also to set -T
                    包含了需要POST的数据的文件
    -u putfile      File containing data to PUT. Remember also to set -T
    -T content-type Content-type header to use for POST/PUT data, eg.
                    'application/x-www-form-urlencoded'
                    Default is 'text/plain'
                    POST数据所使用的Content-type头信息
    -v verbosity    How much troubleshooting info to print
                    设置显示信息的详细程度 - 4或更大值会显示头信息， 3或更大值可以显示响应代码(404, 200等), 2或更大值可以显示警告和其他信息
    -w              Print out results in HTML tables
                    以HTML表的格式输出结果。默认时，它是白色背景的两列宽度的一张表
    -i              Use HEAD instead of GET
                    执行HEAD请求，而不是GET
    -x attributes   String to insert as table attributes
                    设置<table>属性的字符串。 此属性被填入<table 这里 >
    -y attributes   String to insert as tr attributes
                    设置<tr>属性的字符串
    -z attributes   String to insert as td or th attributes
                    设置<td>属性的字符串
    -C attribute    Add cookie, eg. 'Apache=1234'. (repeatable)
                    对请求附加一个Cookie:行。 其典型形式是name=value的一个参数对。 此参数可以重复
    -H attribute    Add Arbitrary header line, eg. 'Accept-Encoding: gzip'
                    Inserted after all normal header lines. (repeatable)
                    对请求附加额外的头信息。 此参数的典型形式是一个有效的头信息行，其中包含了以冒号分隔的字段和值的对 (如, "Accept-Encoding: zip/zop;8bit")
    -A attribute    Add Basic WWW Authentication, the attributes
                    are a colon separated username and password.
                    对服务器提供BASIC认证信任。 用户名和密码由一个:隔开，并以base64编码形式发送。 无论服务器是否需要(即, 是否发送了401认证需求代码)，此字符串都会被发送
    -P attribute    Add Basic Proxy Authentication, the attributes
                    are a colon separated username and password.
                    对一个中转代理提供BASIC认证信任。 用户名和密码由一个:隔开，并以base64编码形式发送。 无论服务器是否需要(即, 是否发送了401认证需求代码)，此字符串都会被发送
    -X proxy:port   Proxyserver and port number to use
                    对请求使用代理服务器
    -V              Print version number and exit
                    显示版本号并退出
    -k              Use HTTP KeepAlive feature
                    启用HTTP KeepAlive功能，即, 在一个HTTP会话中执行多个请求。 默认时，不启用KeepAlive功能
    -d              Do not show percentiles served table.
                    不显示"percentage served within XX [ms] table"的消息(为以前的版本提供支持)
    -S              Do not show confidence estimators and warnings.
                    不显示中值和标准背离值， 而且在均值和中值为标准背离值的1到2倍时，也不显示警告或出错信息。 默认时，会显示 最小值/均值/最大值等数值。(为以前的版本提供支持)
    -q              Do not show progress when doing more than 150 requests
                    如果处理的请求数大于150， ab每处理大约10%或者100个请求时，会在stderr输出一个进度计数。 此-q标记可以抑制这些信息
    -l              Accept variable document length (use this for dynamic pages)
    -g filename     Output collected data to gnuplot format file.
                    把所有测试结果写入一个'gnuplot'或者TSV (以Tab分隔的)文件。 此文件可以方便地导入到Gnuplot, IDL, Mathematica, Igor甚至Excel中。 其中的第一行为标题
    -e filename     Output CSV file with percentages served
                    产生一个以逗号分隔的(CSV)文件， 其中包含了处理每个相应百分比的请求所需要(从1%到100%)的相应百分比的(以微妙为单位)时间。 由于这种格式已经“二进制化”，所以比'gnuplot'格式更有用
    -r              Don't exit on socket receive errors.
    -h              Display usage information (this message)
                    显示使用方法
    -Z ciphersuite  Specify SSL/TLS cipher suite (See openssl ciphers)
    -f protocol     Specify SSL/TLS protocol
                    (SSL3, TLS1, TLS1.1, TLS1.2 or ALL)
```

---

### [http\_load](#httpload)

`http_load` 以并行复用的方式运行，用以测试 `web` 服务器的吞吐量与负载。但是它不同于大多数压力测试工具，它可以以一个单一的进程运行，一般不会把客户机搞死。还可以测试 `HTTPS` 类的网站请求

###### 安装

    $ wget http://soft.vpser.net/test/http_load/http_load-12mar2006.tar.gz
    $ tar zxvf http_load-12mar2006.tar.gz
    $ cd http_load-12mar2006
    $ make && make install

###### 测试

需要将测试的链接写入到一个文件：

    $ cat urls.txt
    http://localhost:8080

测试命令：

    $ http_load -p 1000 -s 10 urls.txt          # -p 并发访问进程数  -s 访问时间  需要访问的URL文件
    73980 fetches, 160 max parallel, 1.4796e+06 bytes, in 10.0013 seconds
    20 mean bytes/connection
    7397.02 fetches/sec, 147940 bytes/sec
    msecs/connect: 0.955713 mean, 1002.4 max, 0.038 min
    msecs/first-response: 4.52099 mean, 206.347 max, 0.533 min
    HTTP response codes:
      code 200 -- 73980

###### 对比

测试结果中主要的指标是 fetches/sec、msecs/connect 这个选项，即服务器每秒能够响应的查询次数，用这个指标来衡量性能。似乎比 apache的ab准确率要高一些，也更有说服力一些。

Qpt-每秒响应用户数和response time，每连接响应用户时间。

测试的结果主要也是看这两个值。当然仅有这两个指标并不能完成对性能的分析，我们还需要对服务器的cpu、men进行分析，才能得出结论

###### HELP

参数其实可以自由组合，参数之间的选择并没有什么限制。比如你写成 `http_load -parallel 5 -seconds 300 urls.txt` 也是可以的。

具体命令可以参见 `http_load -h`，部分解释如下：

```
-parallel   简写-p ：含义是并发的用户进程数。
-fetches    简写-f ：含义是总计的访问次数
-rate       简写-p ：含义是每秒的访问频率
-seconds    简写-s ：含义是总计的访问时间
准备URL文件：urllist.txt，文件格式是每行一个URL，URL最好超过50－100个测试效果比较好，每行一个url
```

---

### [webbench](#webbench)

webbench是Linux下的一个网站压力测试工具，最多可以模拟3万个并发连接去测试网站的负载能力

###### 安装

    $ wget http://soft.vpser.net/test/webbench/webbench-1.5.tar.gz
    $ tar zxvf webbench-1.5.tar.gz
    $ cd webbench-1.5/
    $ make && make install

###### 测试

    $ webbench -c 1000 -t 10 http://localhost:8080      # -c 并发数 -t 运行测试时间 URL

###### 总结

这个给出的结果内容比较少，不如上面几个好用，不过这个程序倒是很小

###### HELP

更多的用法可以参考 `webbench -h`

---

### [Siege](#siege)

一款开源的压力测试工具，可以根据配置对一个WEB站点进行多用户的并发访问，记录每个用户所有请求过程的相应时间，并在一定数量的并发访问下重复进行

###### 安装

    $ wget http://soft.vpser.net/test/siege/siege-2.67.tar.gz
    $ tar -zxf siege-2.67.tar.gz
    $ cd siege-2.67/
    $ ./configure && make && make install

###### 测试

    $ siege -c 200 -r 10 http://localhost:8080

> 我写了个 `-r 10000`，看着他还在跑。。。。。。

###### HELP

具体的用法可以参考 `siege -h`

---

### END

---


