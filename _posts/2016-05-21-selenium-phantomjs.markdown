---
layout:     post
title:      "使用selenium和phantomjs实现解析js的网页"
subtitle:   "微信公众号文章抓取方法"
date:       2016-05-21 21:03:41
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
# File Name: weixin_czzj.py
# Author: xiaoh
# Mail: xiaoh@about.me
# Created Time:  2016-05-18 22:21:01
#############################################

###########################################################
#                                                         #
#                         _oo8oo_                         #
#                        o8888888o                        #
#                        88" . "88                        #
#                        (| -_- |)                        #
#                        0\  =  /0                        #
#                      ___/'==='\___                      #
#                    .' \\|     |// '.                    #
#                   / \\|||  :  |||// \                   #
#                  / _||||| -:- |||||_ \                  #
#                 |   | \\\  -  /// |   |                 #
#                 | \_|  ''\---/''  |_/ |                 #
#                 \  .-\__  '-'  __/-.  /                 #
#               ___'. .'  /--.--\  '. .'___               #
#            ."" '<  '.___\_<|>_/___.'  >' "".            #
#           | | :  `- \`.:`\ _ /`:.`/ -`  : | |           #
#           \  \ `-.   \_ __\ /__ _/   .-` /  /           #
#       =====`-.____`.___ \_____/ ___.`____.-`=====       #
#                         `=---=`                         #
#                                                         #
#                佛祖保佑         永无BUG                 #
#                                                         #
########################################################### 

from selenium import webdriver
import time, pymysql

class Item():
    def __init__(self, title, url, postdate, summary):
        self.title = title
        self.url = url
        self.postdate = postdate
        self.summary = summary

    def print_exc(self):
        print 'title   :' + self.title
        print 'url     :' + self.url
        print 'postdate:' + self.postdate
        print 'summary :' + self.summary


def weixin_load(app_name):
    driver=webdriver.PhantomJS(executable_path="./phantomjs") #在这里设置你的phantomjs路径

    driver.set_window_size(1920,1080) #设置浏览器的大小，弄得比较大，显示内容全啊
    driver.get('http://weixin.sogou.com/')
    driver.find_element_by_id("upquery").send_keys(app_name)
    driver.find_element_by_class_name("swz2").click()

    now_handle=driver.current_window_handle
    driver.find_element_by_xpath('/html/body/div[2]/div[3]/div[1]/div[1]/div/div[2]/div/div[1]').click()

    time.sleep(10)

    all_handle=driver.window_handles

    try:
        for handle in all_handle:
            if handle!=now_handle:
                driver.switch_to_window(handle)
                driver.set_window_size(1920, 1080) #设置浏览器的大小，弄得比较大，显示内容全啊

                div =  driver.find_elements_by_xpath('//div[@class="weui_media_box appmsg"]')

                for node in div:
                    h4 = node.find_element_by_xpath('div[1]/h4[@class="weui_media_title"]')
                    p1 = node.find_element_by_xpath('div[1]/p[@class="weui_media_desc"]')
                    p2 = node.find_element_by_xpath('div[1]/p[@class="weui_media_extra_info"]')

                    url = 'http://mp.weixin.qq.com' + h4.get_attribute('hrefs')
                    item = Item(h4.text, url, p2.text, p1.text)
                    item.print_exc()
    except Exception as e:
        print e.message
    finally:
        driver.quit()

def main():
    weixin_load(u"你的公众号名称")

if __name__ == "__main__":
    main()
```

---

### END


