---
layout:     post
title:      "将Flume做成Service"
subtitle:   "源码分享--Supervisor服务形式启动Flume做Web服务"
date:       2016-01-28 21:37:10
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - Flume
    - Supervisor
    - 服务器
---

> 这篇文章主要解决了，将FLume做成服务的形式，通过RestAPI来创建服务，监听不同的端口，来接受网络的数据。
> 需要解决的一点是，不同的服务（不同的数据表）需要启动不同的端口，而这端口不能公开出来，也就是说需要和表名称做一个映射
> 需要有用户权限检测，需要获取服务的状态

---

##### 简单分析一下

0. RestAPI 我打算用Python来做
0. 不同的端口，需要检测端口是否被占用，并且，需要指定可用端口空间
0. 端口不能公开，需要动态的通过表名称来获取端口，这就需要一个缓存的地方，这里用到Redis
0. 用户检测，这部分需要用户传递TOKEN来进行验证，而TOKEN必须可变，所以不能将其写死，这就需要用到Nginx+Lua，所以这里用到了OpenResty
0. 获取服务状态，就是可以监控Flume的运行，所以我用到了Supervisor

---

##### Supervisor配置

这整个设计感觉亮点最重要，Supervisor和OpenResty，首先看一下我Supervisor的配置

    # modify [program:flume-flask-service].directory
    # modify [include].files
    
    [unix_http_server]
    file=/tmp/flume_services_supervisor.sock          ; path to your socket file
    
    [supervisord]
    logfile=%(here)s/supervisord.log                ; supervisord log file
    logfile_maxbytes=100MB                          ; maximum size of logfile before rotation
    logfile_backups=10                              ; number of backed up logfiles
    loglevel=debug                                  ; info, debug, warn, trace
    pidfile=%(here)s/supervisord.pid                ; pidfile location
    nodaemon=false                                  ; run supervisord as a daemon
    minfds=1024                                     ; number of startup file descriptors
    minprocs=200                                    ; number of process descriptors
    user=root                                       ; default user
    childlogdir=%(here)s                            ; where child log files will live
    
    [rpcinterface:supervisor]
    supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface
    
    [supervisorctl]
    serverurl=unix:///tmp/flume_services_supervisor.sock  ; use a unix:// URL  for a unix socket
    
    [program:flume-flask-service]
    command=python flumeAPI.py
    directory=/path/to/service
    user = centos
    autorestart = true
    startsecs = 3
    stderr_logfile=/tmp/flume-flask-service.err
    stdout_logfile=/tmp/flume-flask-service.out
    
    [include]
    #files=%(here)s/conf.d/*.conf                  ; not work
    files=/path/to/service/conf.d/*.conf

关键是最后一行的设置，制定了所有的配置文件的目录，这样每次执行 `supervisor reload` 和 `supervisor update` 即可重新加载最新的配置。

启动supervisor：`supervisor -c /path/to/conf/supervisor.conf`

---

##### OpenResty

这一部分就不多说了，其中用到的内容都在我另一篇博客里面有写，传送门：[高并发服务器构建方案](http://www.xiaoh.me/2016/01/05/nginx-lua-redis/)

---

##### Flume Service

之后就是将启动Flume的功能携程RestAPI了，这部分直接分享源码：

    
具体见源码

    #!/home/xingming/pyvirt/bin/python
    #-*- coding:utf-8 -*-
    
    #############################################
    # File Name: flumeAPI.py
    # Author: xiaoh
    # Mail: xiaoh@about.me·
    # Created Time:  2016-01-20 16:50:20
    #############################################
    
    import requests
    from flask import Flask, request, jsonify
    from flumeHelper import *
    
    app = Flask(__name__)
    cmd = "python -c 'from flumeHelper import {function}; {function}(\"{db}\",\"{table}\");'"
    
    @app.route('/v1/api/<dbname>/<tablename>/create')
    def create(dbname, tablename):
        print 'create api running now.'
    
        out = runCmd(cmd.format(function='start', db=dbname, table=tablename))
        print out
        state = formatOutput(out)
        return status(state)
    
    @app.route('/v1/api/<dbname>/<tablename>/start')
    def start(dbname, tablename):
        print 'start api running now.'
        out = runCmd(cmd.format(function='start', db=dbname, table=tablename))
        print out
        state = formatOutput(out)
        return status(state)
    
    @app.route('/v1/api/<dbname>/<tablename>/stop')
    def stop(dbname, tablename):
        print 'stop api running now.'
        out = runCmd(cmd.format(function='stop', db=dbname, table=tablename))
        print out
        state = formatOutput(out)
        return status(state)
    
    @app.route('/v1/api/<dbname>/<tablename>/restart')
    def restart(dbname, tablename):
        print 'restart api running now.'
        out = runCmd(cmd.format(function='restart', db=dbname, table=tablename))
        print out
        state = formatOutput(out)
        return status(state)
    
    @app.route('/v1/api/<dbname>/<tablename>/status')
    def status(dbname, tablename):
        print 'status api running now.'
        out = runCmd(cmd.format(function='status', db=dbname, table=tablename))
        print out
        state = formatOutput(out)
        return status(state)
    
    @app.route('/v1/api/token', methods=['POST'])
    def token():
        if not request.json:
            print 'not json'
            return status('no json format', 10001)
        username = request.json.get('username', None)
        token = request.json.get('token', None)
        print username, token
        if not username or not token:
            return status('Please provide a username and token', 10002)
        redis_set(username, token, True)
        return status('set token done.')
    
    def status(msg, code=0):
        return jsonify({'message':msg, 'code':code})
    
    def formatOutput(out):
        sl = out.split('\n')
        state = sl[-2].split()
        if len(state) > 1:
            return "%s:%s" % (state[0], state[1])
        return "%s:%s" % (state[0], 'Exception')
    
        return subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE).stdout.read()
    
    def getToken():
        requests.get('http://hosttoservice/api/ehc/gettoken')
        pass
    
    if __name__ == "__main__":
        app.run(host='0.0.0.0', port=8001, debug=True)

以上就是所有公开出来的RestAPI，用Flask编写，Flask这个鬼真的适合小规模的服务器搭建哦。

下面是用到的关于Supervisor操作Flume的函数，觉得这里才是关键好么。。。。

    #!/home/xingming/pyvirt/bin/python
    #-*- coding:utf-8 -*-
    
    #############################################
    # File Name: flumeHelper.py
    # Author: xiaoh
    # Mail: xiaoh@about.me·
    # Created Time:  2016-01-20 16:30:58
    #############################################
    
    import redis
    import socket
    import subprocess
    import os, errno
    import ConfigParser
    from supervisor.supervisorctl import main as supervisor_ctl_main
    
    rd = redis.Redis()
    flumePorts = 'FLUME_PORTS'
    
    serviceRoot = '/path/to/service/'
    superConfFile = os.path.join(serviceRoot, "supervisor.conf")
    
    def _update_conf(dbName, tableName):
        serviceName = "%s_%s" % (dbName, tableName)
        ########################
        # generate flume.conf
        ########################
        rootDir = os.path.join(serviceRoot, 'services', serviceName)
        varDir = os.path.join(serviceRoot, 'var', serviceName)
        _mkdir_p(rootDir)
        _mkdir_p(varDir)
        flumeConf = os.path.join(rootDir, "flume.conf")
        checkpointDir = os.path.join(varDir, 'file-channel', 'checkpoint')
        dataDir = os.path.join(varDir, 'file-channel', 'data')
        newPort = _get_new_port()
        with open(flumeConf, 'w') as f:
            f.write(flumeTemplate.format(localPort=newPort, dbName=dbName, tableName=tableName, checkpointDir=checkpointDir, dataDir=dataDir))
    
        ##############################
        # generate supervisor handler
        ##############################
        superConfDir = os.path.join(serviceRoot, 'conf.d')
        flumeCmd = "flume-ng agent --conf {confPath} --conf-file {confFile} --name a1 -Dflume.root.logger=INFO,console -Djava.net.preferIPv4Stack=true".format(confPath=rootDir, confFile=flumeConf)
        superConf = {
            "command" : flumeCmd,
            "process_name" : serviceName,
            "directory" : rootDir,
            "stdout_logfile" : os.path.join(rootDir, "stdout_log"),
            "stderr_logfile" : os.path.join(rootDir, "stderr_log")
        }
        config = ConfigParser.RawConfigParser()
        sectionName = "program:"+serviceName
        config.add_section(sectionName)
        for key in superConf:
            config.set(sectionName, key, superConf[key])
        superConfPath = os.path.join(superConfDir, serviceName + ".conf")
        with open(superConfPath, "wb") as configfile:
            config.write(configfile)
        return newPort
    
    def restart(dbName, tableName):
        stop(dbName, tableName)
        start(dbName, tableName)
    
    def start(dbName, tableName):
        serviceName = "%s_%s" % (dbName, tableName)
        newPort = _update_conf(dbName, tableName)
    
        print "Executing : supervisorctl reread '%s' ..." % serviceName
        supervisor_ctl_main(args=["-c", superConfFile, "reread", serviceName])
        print "Executing : supervisorctl update '%s' ..." % serviceName
        supervisor_ctl_main(args=["-c", superConfFile, "update", serviceName])
        print "Executing : supervisorctl start '%s' ..." % serviceName
        supervisor_ctl_main(args=["-c", superConfFile, "start", serviceName])
        print "Executing : supervisorctl status '%s' ..." % serviceName
        supervisor_ctl_main(args=["-c", superConfFile, "status", serviceName])
        redis_set(serviceName, newPort)
    
    def stop(dbName, tableName):
        serviceName = "%s_%s" % (dbName, tableName)
        _redis_del_port(serviceName)
    
        print "Executing : supervisorctl stop '%s' ..." % serviceName
        supervisor_ctl_main(args=["-c", superConfFile, "stop", serviceName])
        print "Executing : supervisorctl status '%s' ..." % serviceName
        supervisor_ctl_main(args=["-c", superConfFile, "status", serviceName])
    
        _clear_files(serviceName)
    
    def status(dbName, tableName):
        serviceName = "%s_%s" % (dbName, tableName)
    
        print "Executing : supervisorctl status '%s' ..." % serviceName
        supervisor_ctl_main(args=["-c", superConfFile, "status", serviceName])
    
    def _clear_files(serviceName):
        rootDir = os.path.join(serviceRoot, 'services', serviceName)
        varDir = os.path.join(serviceRoot, 'var', serviceName)
        superConf = os.path.join(serviceRoot, 'conf.d', serviceName + '.conf')
        runCmd('rm -rf %s' % rootDir)
        runCmd('rm -rf %s' % varDir)
        runCmd('rm -rf %s' % superConf)
    
    def runCmd(cmd):
        return subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE).stdout.read()
    
    def _get_new_port(startPort=10000, endPort=60000):
        usedPorts = _redis_list_ports()
        for port in xrange(startPort, endPort):
            if str(port) in usedPorts or port_is_openning(port):
                continue
            return port
        raise Exception('PortOver', 'all port used')
    
    def redis_set(name, port, token=False):
        if not token:
            rd.sadd(flumePorts, port)
        rd.set(name, port)
    
    def _redis_get_port(name):
        return rd.get(name)
    
    def _redis_del_port(name):
        port = _redis_get_port(name)
        rd.delete(name)
        rd.srem(flumePorts, port)
    
    def _redis_list_ports():
        return rd.smembers(flumePorts)
    
    def port_is_openning(port):
        line = int(runCmd('netstat -an | grep %s | wc -l' % port).splitlines()[0])
        print 'Port:%s is Listen: (%s)' % (port, line > 0)
        return line > 0
    
    def _mkdir_p(path):
        try:
            os.makedirs(path)
        except OSError as exc: # Python >2.5
            if exc.errno == errno.EEXIST and os.path.isdir(path):
                pass
            else:·
                raise
    
    flumeTemplate = """
    # Web ------> Logger
    
    # Name the components on this agent
    a1.sources = r1
    a1.sinks = k1
    a1.channels = c1
    
    # Describe/configure the source
    a1.sources.r1.type = http
    a1.sources.r1.port = {localPort}
    a1.sources.r1.interceptors = i1
    a1.sources.r1.interceptors.i1.type = timestamp
    
    #######################
    # Sink : HDFS
    #######################
    a1.sinks.k1.type = hdfs
    a1.sinks.k1.hdfs.path = /ingestion/{dbName}/{tableName}/%Y-%m-%d/%H
    a1.sinks.k1.hdfs.filePrefix = ing
    a1.sinks.k1.hdfs.round = true
    a1.sinks.k1.hdfs.roundValue = 10
    a1.sinks.k1.hdfs.roundUnit = minute
    
    a1.sinks.k1.hdfs.rollInterval = 60
    a1.sinks.k1.hdfs.rollCount = 8192
    a1.sinks.k1.hdfs.idleTimeout = 60
    a1.sinks.k1.hdfs.writeFormat = Text
    a1.sinks.k1.hdfs.fileType = DataStream
    a1.sinks.k1.hdfs.fileSuffix = .json
    a1.sinks.k1.hdfs.rollSize = 1024000
    
    
    #######################
    # Channel
    #######################
    a1.channels.c1.capacity = 10000000
    a1.channels.c1.transactionCapacity = 100000
    a1.channels.c1.type = file
    a1.channels.c1.checkpointDir = {checkpointDir}
    a1.channels.c1.dataDirs = {dataDir}
    
    # Bind the source and sink to the channel
    a1.sources.r1.channels = c1
    a1.sinks.k1.channel = c1
    """

---

上面所有组件配合起来就完成了需求。具体不多说了，代码说明一切的节奏。

---

### END



