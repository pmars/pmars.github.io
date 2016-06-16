---
layout:     post
title:      "简单的弹幕聊天系统构建"
subtitle:   "使用WebSocket+Tornado完成弹幕聊天功能"
date:       2016-06-16 23:24:36
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - tornado
    - python
    - websocket
---

> 今天了一个简单的弹幕聊天的DEMO，后台是Tornado+Websocket，项目地址:<https://github.com/pmars/barchat>

---

如果你只想看看代码，知道这个是怎么做的，那么直接去github上面看代码即可，下面只是设计到了一些理论知识而已。

聊到这个弹幕聊天的工具，必须解释一下WebSocket和http的对比。

##### WebSocket是HTML5出的东西（协议），也就是说HTTP协议没有变化，或者说没关系，但HTTP是不支持持久连接的（长连接，循环连接的不算）

首先HTTP有1.1和1.0之说，也就是所谓的keep-alive，把多个HTTP请求合并为一个，但是Websocket其实是一个新协议，跟HTTP协议基本没有关系，只是为了兼容现有浏览器的握手规范而已，也就是说它是HTTP协议上的一种补充可以通过这样一张图理解

![](http://www.xiaoh.me/img/post-bar-chat.png)

有交集，但是并不是全部。

另外Html5是指的一系列新的API，或者说新规范，新技术。Http协议本身只有1.0和1.1，而且跟Html本身没有直接关系。。

##### Websocket是什么样的协议，具体有什么优点

传统的HTTP协议虽然用的比较广泛（事实上互联网的绝大多数请求都是HTTP协议的），但还是有一定的不足：

1) HTTP的生命周期通过Request来界定，也就是一个Request 一个Response，那么在HTTP1.0中，这次HTTP请求就结束了。

在HTTP1.1中进行了改进，使得有一个keep-alive，也就是说，在一个HTTP连接中，可以发送多个Request，接收多个Response。但是请记住 Request = Response ， 在HTTP中永远是这样，也就是说一个request只能有一个response。而且这个response也是被动的，不能主动发起。

首先Websocket是基于HTTP协议的，或者说借用了HTTP的协议来完成一部分握手。

在握手阶段是一样的(from wikipedia)

    GET /chat HTTP/1.1
    Host: server.example.com
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
    Sec-WebSocket-Protocol: chat, superchat
    Sec-WebSocket-Version: 13
    Origin: http://example.com

这段类似HTTP协议的握手请求中，多了几个东西

    Upgrade: websocket
    Connection: Upgrade

这个就是Websocket的核心了，告诉Apache、Nginx等服务器：* 发起的是Websocket协议*

    Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
    Sec-WebSocket-Protocol: chat, superchat
    Sec-WebSocket-Version: 13

首先，`Sec-WebSocket-Key` 是一个`Base64 encode`的值，这个是浏览器随机生成的，告诉服务器验证是不是真的是Websocket助理

然后，`Sec_WebSocket-Protocol` 是一个用户定义的字符串，用来区分同URL下，不同的服务所需要的协议。

最后，`Sec-WebSocket-Version` 是告诉服务器所使用的Websocket Draft（协议版本）

> 在最初的时候，Websocket协议还在 Draft 阶段，各种奇奇怪怪的协议都有，而且还有很多期奇奇怪怪不同的东西，什么Firefox和Chrome用的不是一个版本之类的，当初Websocket协议太多可是一个大难题。。不过现在还好，已经定下来啦~大家都使用的一个东西~ 

然后服务器会返回下列东西，表示已经接受到请求， 成功建立Websocket啦！

    HTTP/1.1 101 Switching Protocols
    Upgrade: websocket
    Connection: Upgrade
    Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
    Sec-WebSocket-Protocol: chat

这里开始就是HTTP最后负责的区域了，告诉客户，我已经成功切换协议啦~

    Upgrade: websocket
    Connection: Upgrade

依然是固定的，告诉客户端即将升级的是Websocket协议，而不是mozillasocket，lurnarsocket或者shitsocket。

然后，`Sec-WebSocket-Accept` 这个则是经过服务器确认，并且加密过后的 `Sec-WebSocket-Key`。

后面的，`Sec-WebSocket-Protocol` 则是表示最终使用的协议。

至此，HTTP已经完成它所有工作了，接下来就是完全按照Websocket协议进行了。

##### Websocket的作用

在使用Websocket之前呢，客户端想从服务器监测信息的变化，主要使用两种方式，1,Ajex轮询，2,Long Poll

Ajax轮询主要就是隔一段时间客户端去访问一下服务器，看看是否有数据更新，这样处理有信息不及时，带宽压力大，服务器压力大等问题

Long Poll 采取的是阻塞模型（一直打电话，没收到就不挂电话），也就是说，客户端发起连接后，如果没消息，就一直不返回Response给客户端。直到有消息才返回，返回完之后，客户端再次建立连接，周而复始

这两种方式都是因为HTTP协议的一个特点：*被动性*,就是说，服务器没有办法直接发送信息给客户端。

而Websocket的出现就解决了这个问题，他可以持久连接，并且可以双向发送信息。

Websocket只需要一次HTTP握手，所以说整个通讯过程是建立在一次连接/状态中，也就避免了HTTP的非状态性，服务端会一直知道你的信息，直到你关闭请求。

---

> 以上就是WebSocket相对与HTTP的不同之处

---

##### 弹幕系统

理解了上面的WebSocket的功能，弹幕系统就很容易理解了，在服务器和客户端建立一个WebSocket连接，这样用户A发送弹幕会通过WebSocket发送给服务器，服务器在通过WebSocket的链接发送给所有连接在服务器上的用户。

代码详见 <https://github.com/pmars/barchat>

---

### END


