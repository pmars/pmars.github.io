---
layout:     post
title:      "Python 操作电子邮件方法"
subtitle:   "smtp和Pop3发送和接收电子邮件方法简介"
date:       2016-02-16 14:22:34
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - Python
    - Email
---

> 这篇文章大概就是一个源码分享，介绍了用Python来发送电子邮件的方式
> 文章参考：[Python电子邮件](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/0013868325601402299d1e941914a21990ac7861ef4bc2d000)

---

### SMTP发送邮件

SMTP是发送邮件的协议，Python内置对SMTP的支持，可以发送纯文本邮件、HTML邮件以及带附件的邮件。

Python对SMTP支持有smtplib和email两个模块，email负责构造邮件，smtplib负责发送邮件。

以下是各种电子邮件模式的发送代码：

    #!/usr/bin/python
    #-*- coding:utf-8 -*-
    
    #############################################
    # File Name: send_email.py
    # Author: xiaoh
    # Mail: xiaoh@about.me
    # Created Time:  2016-02-16 14:55:55
    #############################################
    
    import smtplib
    import ConfigParser
    from email import encoders
    from email.header import Header
    from email.mime.base import MIMEBase
    from email.mime.text import MIMEText
    from email.mime.multipart import MIMEMultipart
    from email.utils import parseaddr, formataddr
    
    cfg = ConfigParser.ConfigParser()
    
    cfg.read('email.conf')
    from_addr = cfg.get('email', 'from')
    password = cfg.get('email', 'pass')
    smtp_serv = cfg.get('email', 'smtp')
    to_addr = cfg.get('email', 'sdto')
    
    def _format_addr(s):
        name, addr = parseaddr(s)
        return formataddr(( \
            Header(name, 'utf-8').encode(), \
            addr.encode('utf-8') if isinstance(addr, unicode) else addr))
    
    def send_text():
        msg = MIMEText('hello, send by script coding by xiaoh.', 'plain', 'utf-8')
        msg['From'] = _format_addr(u'xiaoh <%s>' % from_addr)
        msg['To'] = _format_addr(u'管理员 <%s>' % to_addr)
        msg['Subject'] = Header(u'来自SMTP的问候……', 'utf-8').encode()
        send(msg)
    
    def send_html():
        msg = MIMEText('<html><body><h1>Hello</h1>' +
            '<p>welcome to my <a href="http://www.xiaoh.me">Blog</a>...</p>' +
            '</body></html>', 'html', 'utf-8')
        msg['From'] = _format_addr(u'xiaoh <%s>' % from_addr)
        msg['To'] = _format_addr(u'管理员 <%s>' % to_addr)
        msg['Subject'] = Header(u'来自SMTP的问候……', 'utf-8').encode()
        send(msg)
    
    def send_attachment():
        msg = MIMEMultipart()
        msg['From'] = _format_addr(u'xiaoh <%s>' % from_addr)
        msg['To'] = _format_addr(u'管理员 <%s>' % to_addr)
        msg['Subject'] = Header(u'来自SMTP的问候……', 'utf-8').encode()
        # 邮件正文是MIMEText:
        msg.attach(MIMEText('send with file...', 'plain', 'utf-8'))
        # 添加到MIMEMultipart:
        msg.attach(read_image())
        send(msg)
    
    def send_image():
        msg = MIMEMultipart()
        msg['From'] = _format_addr(u'xiaoh <%s>' % from_addr)
        msg['To'] = _format_addr(u'管理员 <%s>' % to_addr)
        msg['Subject'] = Header(u'来自SMTP的问候……', 'utf-8').encode()
        # 邮件正文是MIMEText:
        msg.attach(MIMEText('<html><body><h1>Hello</h1>' +
                        '<p><a href=http://www.xiaoh.me><img src="cid:0"></a></p>' +
                        '</body></html>', 'html', 'utf-8'))
        # 添加到MIMEMultipart:
        msg.attach(read_image())
        send(msg)
    
    def send_multi():
        msg = MIMEMultipart('alternative')
        msg['From'] = _format_addr(u'xiaoh <%s>' % from_addr)
        msg['To'] = _format_addr(u'管理员 <%s>' % to_addr)
        msg['Subject'] = Header(u'来自SMTP的问候……', 'utf-8').encode()
        # 邮件正文是MIMEText:
        msg.attach(MIMEText('hello, this is my script.', 'plain', 'utf-8'))
        msg.attach(MIMEText('<html><body><h1>Hello</h1>' +
                        '<p><a href=http://www.xiaoh.me><img src="cid:0"></a></p>' +
                        '</body></html>', 'html', 'utf-8'))
        # 添加到MIMEMultipart:
        msg.attach(read_image())
        msg.attach(read_image())
        send(msg)
    
    def read_image():
        # 添加附件就是加上一个MIMEBase，从本地读取一个图片:
        with open('/home/xingming/gitpro/blogs/img/post-bg-default.jpg', 'rb') as f:
            # 设置附件的MIME和文件名，这里是png类型:
            mime = MIMEBase('image', 'png', filename='test.png')
            # 加上必要的头信息:
            mime.add_header('Content-Disposition', 'attachment', filename='test.png')
            mime.add_header('Content-ID', '<0>')
            mime.add_header('X-Attachment-Id', '0')
            # 把附件的内容读进来:
            mime.set_payload(f.read())
            # 用Base64编码:
            encoders.encode_base64(mime)
            return mime
    
    def send(msg):
        server = smtplib.SMTP(smtp_serv, 25) # 25为SMTP协议的默认端口
        server.set_debuglevel(1)
        server.login(from_addr, password)
        server.sendmail(from_addr, [to_addr], msg.as_string())
        server.quit()
    
    def main():
    #    send_text()
    #    send_html()
    #    send_attachment()
    #    send_image()
        send_multi()
        print 'hello'
    
    if __name__ == "__main__":
        main()

---

### POP接收邮件

Python内置一个poplib模块，实现了POP3协议，可以直接用来收邮件。

注意到POP3协议收取的不是一个已经可以阅读的邮件本身，而是邮件的原始文本，这和SMTP协议很像，SMTP发送的也是经过编码后的一大段文本。

要把POP3收取的文本变成可以阅读的邮件，还需要用email模块提供的各种类来解析原始文本，变成可阅读的邮件对象。

所以，收取邮件分两步：

第一步：用poplib把邮件的原始文本下载到本地；

第二部：用email解析原始文本，还原为邮件对象。

    #!/usr/bin/python
    #-*- coding:utf-8 -*-
    
    #############################################
    # File Name: recv_email.py
    # Author: xiaoh
    # Mail: xiaoh@about.me
    # Created Time:  2016-02-16 17:33:29
    #############################################
    
    import ConfigParser
    import poplib
    import email
    from email.parser import Parser
    from email.header import decode_header
    from email.utils import parseaddr
    
    cfg = ConfigParser.ConfigParser()
    
    cfg.read('email.conf')
    email = cfg.get('pop', 'email')
    password = cfg.get('pop', 'pass')
    pop_serv = cfg.get('pop', 'pop')
    
    def recv():
        # 连接到POP3服务器:
        server = poplib.POP3(pop_serv)
        # 可以打开或关闭调试信息:
        # server.set_debuglevel(1)
        # 可选:打印POP3服务器的欢迎文字:
        print(server.getwelcome())
        # 身份认证:
        server.user(email)
        server.pass_(password)
        # stat()返回邮件数量和占用空间:
        print('Messages: %s. Size: %s' % server.stat())
        # list()返回所有邮件的编号:
        resp, mails, octets = server.list()
        # 可以查看返回的列表类似['1 82923', '2 2184', ...]
        # print(mails)
        # 获取最新一封邮件, 注意索引号从1开始:
        index = len(mails)
        print index
        resp, lines, octets = server.retr(index)
        # lines存储了邮件的原始文本的每一行,
        # 可以获得整个邮件的原始文本:
        msg_content = '\r\n'.join(lines)
        # 稍后解析出邮件:
        msg = Parser().parsestr(msg_content)
        print_info(msg)
        # 可以根据邮件索引号直接从服务器删除邮件:
        # server.dele(index)
        # 关闭连接:
        server.quit()
    
    # indent用于缩进显示:
    def print_info(msg, indent=0):
        if indent == 0:
            # 邮件的From, To, Subject存在于根对象上:
            for header in ['From', 'To', 'Subject']:
                value = msg.get(header, '')
                if value:
                    if header=='Subject':
                        # 需要解码Subject字符串:
                        value = decode_str(value)
                    else:
                        # 需要解码Email地址:
                        hdr, addr = parseaddr(value)
                        name = decode_str(hdr)
                        value = u'%s <%s>' % (name, addr)
                print('%s%s: %s' % ('  ' * indent, header, value))
        if (msg.is_multipart()):
            # 如果邮件对象是一个MIMEMultipart,
            # get_payload()返回list，包含所有的子对象:
            parts = msg.get_payload()
            for n, part in enumerate(parts):
                print('%spart %s' % ('  ' * indent, n))
                print('%s--------------------' % ('  ' * indent))
                # 递归打印每一个子对象:
                print_info(part, indent + 1)
        else:
            # 邮件对象不是一个MIMEMultipart,
            # 就根据content_type判断:
            content_type = msg.get_content_type()
            if content_type=='text/plain' or content_type=='text/html':
                # 纯文本或HTML内容:
                content = msg.get_payload(decode=True)
                # 要检测文本编码:
                charset = guess_charset(msg)
                if charset:
                    content = content.decode(charset)
                print('%sText: %s' % ('  ' * indent, content + '...'))
            else:
                # 不是文本,作为附件处理:
                print('%sAttachment: %s' % ('  ' * indent, content_type))
    
    def decode_str(s):
        value, charset = decode_header(s)[0]
        if charset:
            value = value.decode(charset)
        return value
    
    def guess_charset(msg):
        # 先从msg对象获取编码:
        charset = msg.get_charset()
        if charset is None:
            # 如果获取不到，再从Content-Type字段获取:
            content_type = msg.get('Content-Type', '').lower()
            pos = content_type.find('charset=')
            if pos >= 0:
                charset = content_type[pos + 8:].strip()
        return charset
    
    def main():
        recv()
        print 'hello'
    
    if __name__ == "__main__":
        main()

---


### END


