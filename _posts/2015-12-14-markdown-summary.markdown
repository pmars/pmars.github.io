---
layout:     post
title:      "Markdown的基本语法"
subtitle:   "简单介绍几种Markdown的语法"
date:       2015-12-14 01:55:02
author:     "xiaoh"
header-img: "img/post-bg-markdown-summary.jpg"
tags:
    - Markdown
---

> 从博客开始创建到现在已经写过几天了，基本上对 `Markdown` 也有了部分了解，现在给我了解的总结一下

---

### 目录

1. [标题设置](#title)
2. [分割线](#line)
3. [块注释](#quote)
4. [强调](#qiang)
5. [无序列表](#wuxu)
6. [有序列表](#youxu)
7. [嵌套列表](#qiantao)
8. [链接](#link)
9. [图片](#image)
0. [自动链接](#autolink)
0. [代码块](#code)
0. [转义字符](#zhuanyi)

---

### [标题设置](#title)

标题设置分为两种，`Setext` and `Atx`

###### Setext

向文字下方添加三个或者更多的 `=` or `-` 来表示一级标题和二级标题

    一级标题
    ===
    二级标题
    ---

一级标题
===

二级标题
---

###### Atx

在文字开头加 `#`，通过 `#` 的数量来区分标题等级。

    # 一级标题
    ## 二级标题
    ### 三级标题
    #### 四级标题
    ##### 五级标题
    ###### 六级标题

# 一级标题

## 二级标题

### 三级标题

#### 四级标题

##### 五级标题

###### 六级标题

---

### [分割线](#line)

三个或者更多的 `-`，必须单独一行，可含空格

    ---

---

---

### [块注释](#quote)

也可以叫引用，通过字符 `>` 来表示，后面可以没有空格

    > 这里是注释
    >也就是引用

> 这里是注释
>也就是引用

---

### [强调](#qiang)

可以使用两种符号：`*`, `_`，单个是斜体，两个是粗体，符号可以跨行使用

    **你好**
    *xiaoh.me*
    _xingming_
    __pmars.github.io__
    _欢迎来看我的博客
    博客地址为xiaoh.me_

**你好**
*xiaoh.me*
_xingming_
__pmars.github.io__
_欢迎来看我的博客
博客地址为xiaoh.me_

---

### [无序列表](#wuxu)

可以用三种符号来表示：`+`, `-`, `*`，但不能混用（混用代表嵌套），需要注意的是，字符后必须有空格

```
- 无序列表
- 无序列表
- 无序列表
- 无序列表 
```

- 无序列表
- 无序列表
- 无序列表
- 无序列表 

```
+ 无序列表
+ 无序列表
+ 无序列表
+ 无序列表 
```

+ 无序列表
+ 无序列表
+ 无序列表
+ 无序列表

```
* 无序列表
* 无序列表
* 无序列表
* 无序列表
```

* 无序列表
* 无序列表
* 无序列表
* 无序列表

---

### [有序列表](#youxu)

使用数字后面跟上句号加上空格来表示有序列表。数字不可以省略，但可以无序

    1. 有序列表
    4. 有序列表
    2. 有序列表
    6. 有序列表

1. 有序列表
4. 有序列表
2. 有序列表
6. 有序列表

---

### [嵌套列表](#qiantao)

使用无序列表的三个字符：`-`, `+`, `*`，来嵌套使用，可循环，符号之后的空格不能少，符号后的空格也不能少

    - 嵌套列表
        + 嵌套列表
        + 嵌套列表
            * 嵌套列表
        + 嵌套列表
    - 回到最前

- 嵌套列表
    + 嵌套列表
    + 嵌套列表
        * 嵌套列表
    + 嵌套列表
- 回到最前

---

### [链接](#link)

链接也可以通过两种方式来查看

###### 内联方式

    click [blog](http://www.xiaoh.me) to www.xiaoh.me

click [blog](http://www.xiaoh.me) to www.xiaoh.me

###### 引用方式

    click [blog][1] to www.xiaoh.me and click [weibo][2] to weibo.com/topmars

click [blog][1] to www.xiaoh.me and click [weibo][2] to weibo.com/topmars

[1]: http://www.xiaoh.me "Xiaoh's Blog"
[2]: http://weibo.com/topmars "Weibo"

---

### [图片](#image)

图片和链接一样，也有两种方式

###### 内联方式

    ![](http://www.xiaoh.me/img/icon_wechat.png "Xiaoh's header Image")

![](http://www.xiaoh.me/img/icon_wechat.png "Xiaoh's header Image")

###### 引用方式

    ![][id]
    [id]: http://www.xiaoh.me/img/icon_wechat.png

![][id]

[id]: http://www.xiaoh.me/img/icon_wechat.png

---

### [自动链接](#autolink)

    <http://www.xiaoh.me>
    <http://weibo.com/topmars>

<http://www.xiaoh.me>
<http://weibo.com/topmars>

---

### [代码块](#code)

可以通过引用：

    ```
    code here
    ```

```
code here
```

也可以通过 `tab` 或者 四个空格来指定

        code here

---

    code here

---

### [转义字符](#zhuanyi)

    Markdown中的转义字符为\，转义的有：
    \\ 反斜杠
    \` 反引号
    \* 星号
    \_ 下划线
    \{\} 大括号
    \[\] 中括号
    \(\) 小括号
    \# 井号
    \+ 加号
    \- 减号
    \. 英文句号
    \! 感叹号

---

### END


