---
layout:     post
title:      "使用selenium和phantomjs实现解析js的网页"
subtitle:   "微信公众号文章抓取方法"
date:       2016-05-21 01:03:41
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - Spider
    - 微信
    - Selenium
    - PhantomJS
---

### 目录

0. [selenium](#seleniumselenium)
0. [phantomJS](#phantomjsphantomjs)

---

> 最近写了好多爬虫相关的项目，大部分还都是比较简单的直接可以抓取的，但微信公众号这个比较麻烦，知道是可以从[搜狗微信搜索](http://weixin.sogou.com)页面获取，但搜狗做了各种加密措施，所以只能依托与可以解析JS的类浏览器的框架来抓取了

> 开始使用的方法是Selenium+Firefox，但这需要启动一个浏览器，因为centos上面无法执行的关系，又研究了selenium+phantomJS的方法，下面简单介绍一下我对这些的理解

---

### [selenium](#selenium)

Selenium这个我理解就是各个浏览器的包装，将多个浏览器（如下所示）进行了封装，对外提供了统一的接口，这样更换模型就比较简单。

```Shell
In [1]: from selenium import webdriver

In [2]: webdriver.
webdriver.ActionChains         webdriver.Opera                webdriver.edge
webdriver.Android              webdriver.PhantomJS            webdriver.firefox
webdriver.BlackBerry           webdriver.Proxy                webdriver.ie
webdriver.Chrome               webdriver.Remote               webdriver.opera
webdriver.ChromeOptions        webdriver.Safari               webdriver.phantomjs
webdriver.DesiredCapabilities  webdriver.TouchActions         webdriver.remote
webdriver.Edge                 webdriver.android              webdriver.safari
webdriver.Firefox              webdriver.blackberry           webdriver.support
webdriver.FirefoxProfile       webdriver.chrome
webdriver.Ie                   webdriver.common
```

更多的内容可以在互联网上找到一堆，就不再罗列，详细使用方法，见代码更容易理解。

---

### [phantomJS](#phantomjs)

PhantomJS可以理解为一个没有前端的文字浏览器，他仅需要Webkit内核即可，可以类似与Firefox浏览器一样的操作进行网页解析。我在从Firefox转到PhantomJS的时候仅仅替换了名称和设置浏览器的大小而已（Firefox没有设置是因为本身我就最大化了，这个大小设计到页面js的执行，展示数据的内容）

在我按照官网通过 brew 按抓 phantomJS的时候，运行的时候，会出现 `get_element` 找不到的错误，搜索了一下，推荐直接从官网下载源码，之后解压缩，设置PATH路径即可（也可以不设置，在调用的时候传路径一样，只不过不灵活而已）

详细内容参见代码：

```Python
#!/usr/bin/python
#-*- coding:utf-8 -*-

#############################################
# File Name: weixin.py
# Author: xiaoh
# Mail: xiaoh@about.me
# Created Time:  2016-05-18 22:21:01
#############################################

import Queue
import selenium, requests, urlparse
from selenium import webdriver
import time, pymysql, sys, random, traceback
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities

que = Queue.Queue()
conn = pymysql.connect(
        host = 'bs.xiaoh.me',
        user = 'root',
        password = 'password',
        db = 'test',
        charset = 'utf8mb4',
        cursorclass = pymysql.cursors.DictCursor)

'''CREATE DATABASE `test` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;'''
'''create table weixin ( 
        id varchar(50) not null, 
        name varchar(50) not null, 
        title varchar(200) not null, 
        url varchar(200) not null, 
        date varchar(50) not null, 
        summary varchar(300) not null, 
        content text not null,
        primary key(id, title, data)
    );'''
class Item():
    def __init__(self, id, name, title, url, date, summary, content):
        self.id = id
        self.name = name
        self.title = title
        self.url = url
        self.date = date
        self.summary = summary
        self.content = content

    def print_exc(self):
        #print 'id      :' + self.id
        print 'name    :' + self.name
        print 'title   :' + self.title
        #print 'url     :' + self.url
        print 'date    :' + self.date
        #print 'summary :' + self.summary
        #print 'content :' + self.content

    def save(self):
        try:
            sql = 'insert into weixin values (%s, %s, %s, %s, %s, %s, %s)'
            data = (self.id, self.name, self.title, self.url, self.date, self.summary, self.content)
            with conn.cursor() as cur:
                cur.execute(sql, data)
            conn.commit()
        except pymysql.err.IntegrityError:
            print 'item has existed in database'
        except Exception as e:
            print type(e)
            print e.message

def article_content(url):
    req = requests.get(url)
    return req.content if req.status_code == 200 else ''

def create_driver():
    dcap = dict(DesiredCapabilities.PHANTOMJS)
#    dcap["phantomjs.page.settings.userAgent"] = "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.87 Safari/537.36"

    return webdriver.PhantomJS(
        executable_path = "/home/xingming/bin/phantomjs",
        desired_capabilities = dcap,
#        service_args = [
#            "--cookies-file={}".format("/tmp/cookies.txt")
#            ]
        )

def weixin_load():
    driver = create_driver()
    driver.maximize_window()

    # start load search page
    driver.get('http://weixin.sogou.com/')

    while not que.empty():
        app_name = que.get()
        if not isinstance(app_name, unicode):
            app_name = unicode(app_name, 'utf-8')
        print u'start crawl app name: %s' % app_name
        try:
            # input search words and click search button
            driver.find_element_by_id("upquery").clear()
            driver.find_element_by_id("upquery").send_keys(app_name)
            driver.find_element_by_class_name("swz2").click()

            # delay 3 second, loading the result
            time.sleep(3)

            text = driver.find_element_by_xpath('/html/body').text
            error_ip = u'您的访问过于频繁，为确认本次访问为正常用户行为，需要您协助验证'
            error_exist= u'无与“%s”相关的官方认证订阅号。' % app_name
            if text.find(error_ip) >= 0:
                print 'IP受限，请更换IP进行搜索，此次搜索即将退出'
                return
            elif text.find(error_exist) > 0:
                print '账号不存在，请更新查询关键词, 此次搜索即将退出'
                continue

            # get name and wechat id
            name = driver.find_element_by_xpath('//div[@class="results mt7"]/div[1]/div[2]/h3').text
            weid = driver.find_element_by_xpath('//label[@name="em_weixinhao"]').text

            # save current handle
            start_handle = driver.current_window_handle

            # click for articles
            app_node = driver.find_element_by_xpath('//div[@class="results mt7"]/div[1]')
            if app_node is None:
                print '没有找到对应的公众号节点，请检查xpath或搜索词'
                return
            app_node.click()

            # delay 5 second, loading the result
            time.sleep(5)

            # check new handle for message and save it
            for handle in driver.window_handles:
                if handle == start_handle:
                    continue

                driver.switch_to_window(handle)
                driver.maximize_window()

                try:
                    # get article list div node
                    div_list =  driver.find_elements_by_xpath('//div[@class="weui_media_bd"]')

                    for node in div_list:
                        try:
                            h4 = node.find_element_by_xpath('h4[@class="weui_media_title"]')
                            p1 = node.find_element_by_xpath('p[@class="weui_media_desc"]')
                            p2 = node.find_element_by_xpath('p[@class="weui_media_extra_info"]')

                            title = h4.text
                            url = urlparse.urljoin('http://mp.weixin.qq.com', h4.get_attribute('hrefs'))
                            date = p2.text
                            summary = p1.text
                            content = article_content(url)

                            item = Item(weid, name, title, url, date, summary, content)
                            item.print_exc()
                            item.save()
                        except Exception as e:
                            print type(e)
                            print e.message
                except Exception as e:
                    print type(e)
                    print driver.page_source
                    print e.message
                finally:
                    driver.close()
        except Exception as e:
            print type(e)
            print e.message
            traceback.print_exc()

        driver.switch_to_window(start_handle)
        time.sleep(10)
    driver.quit()

def main():
    f = open('words.csv')
    words = f.readlines()
    f.close()

    [que.put(word[:-1]) for word in words]
    times = 5
    while times:
        times = times - 1
        weixin_load()

if __name__ == "__main__":
    main()
```

---

### END


