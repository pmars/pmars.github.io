---
layout:     post
title:      "简易爬虫学习"
subtitle:   "一个简单的爬虫是如何运行的"
date:       2016-05-21 00:11:41
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - Spider
    - BeautifulSoup
---

### 目录

0. [控制器](#controller)
0. [下载器](#downloader)
0. [解析器](#parser)
0. [URL管理器](#urlmanager)
0. [输出器](#outputer)
0. [文档](#document)

---

> 好长时间没有更新博客了，最近做了一些爬虫的整理，虽然以前做过好多，但没有整理过，最近这段时间想要提炼一下Python爬虫的东西。

---

### [控制器](#controller)

控制器作为爬虫的整合工具，起到调度的作用，具有将爬虫各个组件协调起来工作的能力。

```Python
#!/usr/bin/python
#-*- coding:utf-8 -*-

#############################################
# File Name: spider.py
# Author: xiaoh
# Mail: xiaoh@about.me
# Created Time:  2016-05-20 23:31:49
#############################################

from url_manager import *
from html_downloader import *
from html_parser import *
from html_outputer import *

class SpiderMain():
    def __init__(self):
        self.urls = UrlManager()
        self.downloader = HtmlDownloader()
        self.parser = HtmlParser()
        self.outputer = HtmlOutputer()

    def craw(self, start_url):
        self.urls.add_new_url(start_url)
        count = 1
        while self.urls.has_new_url():
            try:
                new_url = self.urls.get_new_url()
                html_cont = self.downloader.download(new_url)
                new_urls, new_data = self.parser.parse(new_url, html_cont)
                self.urls.add_new_urls(new_urls)
                self.outputer.collect_data(new_data)

                if count == 2000:
                    break

                print 'Download %d page, title: %s' % (count, new_data['title'])

                count = count + 1
            except Exception as e:
                print type(e)
                print e.message

        self.outputer.output_html()

if __name__ == "__main__":
    start_url = "http://baike.baidu.com/view/6687996.htm"
    spider = SpiderMain()
    spider.craw(start_url)
```

---

### [下载器](#downloader)

下载器则需要进行页面的下载工作，简单的下载器直接调用库函数即可，但复杂一些的就需要考虑到限制IP（使用代理），查看Cookies（模拟登录），脚本加密（模拟浏览器执行）等等方面，这里只简单介绍最基础的下载器，更深入的后续会在博客里面进行更新。

```Python
#!/usr/bin/python
#-*- coding:utf-8 -*-

#############################################
# File Name: html_downloader.py
# Author: xiaoh
# Mail: xiaoh@about.me
# Created Time:  2016-05-21 19:00:54
#############################################

import requests

class HtmlDownloader():
    def download(self, url):
        if url is None:
            return None
        req = requests.get(url)
        if req.status_code != 200:
            return None
        return req.content
```

---

### [解析器](#parser)

解析器是从网页内容中获取需要的内容的功能，这部分也可以有很多种处理方式，分大方向的话，分别是模糊匹配（正则表达式）和结构化解析（html.parser/lxml)，按照速度来说，模糊匹配会更快一些，但正则可能会带来一定的难度，结构化解析倒是很简单，我们这里用到的就是结构化解析方法（正则写的太多了，烦）。

而结构化解析有一个第三方的库叫 `BeautifulSoup` 他可以更好的进行网页的解析（其实就是用起来简单）

```Python
#!/usr/bin/python
#-*- coding:utf-8 -*-

#############################################
# File Name: html_parser.py
# Author: xiaoh
# Mail: xiaoh@about.me
# Created Time:  2016-05-21 19:03:28
#############################################

import re
import urlparse
from bs4 import BeautifulSoup

class HtmlParser():

    def parse(self, url, content):
        if url is None or content is None:
            return
        soup = BeautifulSoup(content, 'html.parser', from_encoding='utf-8')
        new_urls = self._get_new_urls(url, soup)
        new_data = self._get_new_data(url, soup)
        return new_urls, new_data

    def _get_new_urls(self, url, soup):
        links = soup.find_all('a', href=re.compile(r'/view/\d+\.htm'))
        new_urls = set()
        for link in links:
            new_url = link['href']
            new_full_url = urlparse.urljoin(url, new_url)
            new_urls.add(new_full_url)
        return new_urls

    def _get_new_data(self, url, soup):
        title = soup.find('dd', class_="lemmaWgt-lemmaTitle-title").find('h1')
        summary = soup.find('div', class_="lemma-summary")
        return {
            "title":title.text,
            "summary":summary.text,
            "url":url
        }
```

---

### [URL管理器](#manager)

URL管理器主要工作就是存储URL列表，并进行URL去重

```Python
#!/usr/bin/python
#-*- coding:utf-8 -*-

#############################################
# File Name: url_manager.py
# Author: xiaoh
# Mail: xiaoh@about.me
# Created Time:  2016-05-21 18:53:35
#############################################


class UrlManager():
    def __init__(self):
        self.new_urls = set()
        self.old_urls = set()

    def add_new_url(self, url):
        if url is None:
            return
        if url in self.new_urls or url in self.old_urls:
            return
        self.new_urls.add(url)

    def add_new_urls(self, urls):
        if urls is None or len(urls) == 0:
            return
        for url in urls:
            self.add_new_url(url)

    def get_new_url(self):
        if not self.has_new_url():
            return None
        new_url = self.new_urls.pop()
        self.old_urls.add(new_url)
        return new_url

    def has_new_url(self):
        return len(self.new_urls) != 0
```

---

### [输出器](#outputer)

输出器则是将下载的内容进行整理输出的模块，这部分可以自定义做各种入库或者写入文件的操作。

```Python
#!/usr/bin/python
#-*- coding:utf-8 -*-

#############################################
# File Name: html_outputer.py
# Author: xiaoh
# Mail: xiaoh@about.me
# Created Time:  2016-05-21 19:31:36
#############################################


class HtmlOutputer():
    def __init__(self):
        self.datas = []

    def collect_data(self, data):
        if data is None:
            return
        self.datas.append(data)

    def output_html(self):
        fout = open('output.html', 'w')

        fout.write('<html><body><table>')
        for data in self.datas:
            fout.write('<tr>')
            fout.write('<td>%s</td>' % data['url'])
            fout.write('<td>%s</td>' % data['title'].encode('utf-8'))
            fout.write('<td>%s</td>' % data['summary'].encode('utf-8'))
            fout.write('</tr>')
        fout.write('</html></body></table>')
```

---

### [文档](#document)

本博客内容主要学习了网络视频教程的总结和练习，详细内容可以查看以下链接

* <http://www.imooc.com/learn/563>
* <https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html>

### END

