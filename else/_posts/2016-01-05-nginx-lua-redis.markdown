---
layout:     post
title:      "高并发服务器构建方案"
subtitle:   "Nginx+Lua+Redis(OpenResty)配合使用"
date:       2016-01-05 04:28:13
author:     "xiaoh"
header-img: "img/post-bg-nginx-lua-redis.jpg"
tags:
    - 服务器
    - Nginx
    - Redis
    - Lua
---

> 项目里面需要限制访问，之后采用 `token` 的方式来访问，这就需要验证这个 `token` 是否合法，之后有个想法就是在 `nginx` 里面就做一次 `token` 的判断，后来这个想法觉得也不是很靠谱，因为有的时候我们需要将所有的访问都打日志，所以一直就放下去了。
> 最近有时间，就研究了一下。

---

### 目录

0. [OpenResty安装](#openrestyinstall)
0. [使用](#using)
0. [测试](#testing)
0. [我的需求](#request)

---

最近在找能够让三者可以在一起工作的方式，网上也有很多的解决方案，大致分为两种：

* 每个模块分别安装，这样需要在安装 `nginx` 的时候将所有的 `lua` 模块，`redis` 模块都编译进去，这样就是比较麻烦
* 有个简单的方法，就是用大神的 [OpenResty](http://openresty.org/) 这个模块有详细的文档，他里面包含了需要的 `nginx` `lua` `redis` 模块，我们可以直接安装这个来使用

### [OpenResty安装](#install)

具体的安装方法，在 [OpenResty](http://openresty.org/) 有详细的说明，首先我们需要下载源码，你可以在 `OpenResty` 的官方页面去选择一个版本，之后解压，安装即可。

    $ apt-get install libreadline-dev libncurses5-dev libpcre3-dev libssl-dev perl make build-essential
    $ wget https://openresty.org/download/ngx_openresty-1.9.7.1.tar.gz
    $ tar xvf ngx_openresty-1.9.7.1.tar.gz
    $ cd ngx_openresty-1.9.7.1
    $ ./configure
    $ make
    $ make install

安装之后，会在 `/usr/local/openresty` 下面生成相关的文件，下面会有一个 `/usr/local/openresty/nginx/sbin/nginx`，我们可以使用这个来做代理。

先设置一下环境变量 `export PATH=/usr/local/openresty/nginx/sbin:$PATH`，之后就可以使用了

---

### [使用](#using)

先查看一下使用方法 `nginx -h`

```
$ nginx -h
nginx version: openresty/1.9.7.1
Usage: nginx [-?hvVtTq] [-s signal] [-c filename] [-p prefix] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
  -p prefix     : set prefix path (default: /usr/local/openresty/nginx/)
  -c filename   : set configuration file (default: conf/nginx.conf)
  -g directives : set global directives out of configuration file
```

之后选择一个运行 `nginx` 的位置，因为启动之后会生成好多临时文件夹：

    $ mkdir work
    $ cd work
    $ mkdir logs conf

写一个配置文件，因为用的其实是 `nginx`，所以配置文件和 `nginx` 的其实是一样的。

```
$ cat conf/nginx.conf
worker_processes  1;
error_log logs/error.log;
events {
    worker_connections 1024;
}
http {
    server {
        listen 8080;
        location / {
            default_type text/html;
            content_by_lua '
                ngx.say("hello, world")
            ';
        }
    }
}
```

启动 `nginx` :

    $ nginx -p `pwd` -c `pwd`/conf/nginx.conf

---

### [测试](#testing)

具体的测试方法昨天的博客已经有所介绍，可以参考 [四种服务器测试方法](http://www.xiaoh.me/2016/01/04/web-testing/)，博客里面的测试的服务就是用 `openresty` 搭建的。

    $ curl http://localhost:8080

---

### [我的需求](#request)

具体来说我的需求分为几点：

* 能够在 `nginx` 里面获取到 `token` 数据
* 在 `lua` 里面获取 `redis` 里面的数据
* 判断两个 `token` 是否一致
* 返回相应的结果，之后给 `nginx`
* `nginx` 针对结果执行不同的逻辑

具体的配置文件我卸载了两个文件里面：

`nginx` 配置文件，`$ cat conf/nginx.conf`：

    worker_processes  4;
    error_log /path/to/service/logs/error.log;

    events {
        worker_connections 1024;
    }

    http {
        client_max_body_size 10M;
        client_body_buffer_size 10M;

        server {
            listen 8000;
            lua_code_cache on;

            location ~* ^/proxy/([0-9]+) {
                internal;
                proxy_pass http://127.0.0.1:$1;
            }

            location ~* ^/v1/api {
                proxy_pass http://127.0.0.1:8001;
            }

            location ~* ^/v1/([^/]+)/([^/]+)/?$ {
                content_by_lua_file /path/to/service/conf/flume.lua;
            }
        }
    }

`lua` 代码文件，`$ cat conf/flume.lua`：

    local redis = require "resty.redis"
    local cjson = require "cjson"
    local fun = require "fun"
    local cache = redis:new()
    local ok, err = cache:connect("127.0.0.1", "6379")
    local header = ngx.req.get_headers()
    local user = header["X-USERNAME"]
    local token = header["X-AUTH-TOKEN"]
    local uri = ngx.var.uri
    ngx.req.read_body()
    local body = ngx.req.get_body_data()
    local request_method = ngx.var.request_method

    if body == ngx.null or body == nil then
        ngx.say(header['Content-Length'])
        ngx.say('body is nil')
        ngx.exit(ngx.HTTP_INTERNAL_SERVER_ERROR)
    end

    if not ok then
        ngx.exit(ngx.HTTP_INTERNAL_SERVER_ERROR)
    end

    if request_method == "GET" then
        ngx.exit(ngx.HTTP_FORBIDDEN)
    end

    cache:set_timeout(60000)
    local res, err = cache:get(user)

    if res == ngx.null or res == nil then
        ngx.exit(ngx.HTTP_FORBIDDEN)
    end

    if res ~= token then
        ngx.exit(ngx.HTTP_FORBIDDEN)
    end

    db_table_name = uri
    db_table_name = db_table_name:gsub("/[vV]1/", "")
    db_table_name = db_table_name:gsub("/$", "")
    db_table_name = db_table_name:gsub("/", "_")

    local port, err = cache:get(db_table_name)
    if port == ngx.null or port == nil then
        ngx.say("table:\'"..uri.."\' not set any port now.")
    end

    function cvt(x)
        return {body=cjson.encode(x)}
    end

    body_json = cjson.decode(body)
    body_fun = fun.map(cvt, body_json)
    body_str = cjson.encode(fun.totable(body_fun))

    local proxy_url = "/proxy/"..port
    local res = ngx.location.capture(proxy_url, {
            method = ngx.HTTP_POST,
            body = body_str
        })

    ngx.say(res.status)
    ngx.say(res.body)

    if res.status ~= ngx.HTTP_OK then
        ngx.say("service run failed, proxy pass failed")
        return
    end

    local pool_max_idle_time = 10000
    local pool_size = 100
    local ok, err = cache:set_keepalive(pool_max_idle_time, pool_size)

这段代码其实和Flume的RestAPI相配合的，详细的 FLUME RESTAPI 请查看：<http://www.xiaoh.me/2016/01/28/flume-service/>

lua 代码里面引用了一个 openresty 没有的组件，fun，这个文件我会更新到github上面，如果你想用的话，需要下载下来，之后放到 `/usr/local/openresty/lualib/` 文件夹下，文件内容详情参见 <https://github.com/pmars/tools/tree/master/lualib>

具体的就不多解释了，写的比较简单，不过完成了我的需求，如果有更多的应用需求，我会继续更新。

---

### END


