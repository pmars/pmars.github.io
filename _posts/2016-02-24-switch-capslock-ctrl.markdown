---
layout:     post
title:      "Ctrl与Caps Lock键的交换"
subtitle:   "Windows 下交换 Ctrl与Caps Lock键"
date:       2016-02-24 10:38:10
author:     "xiaoh"
header-img: "img/post-bg-default.jpg"
tags:
    - Tmux
---

> 用了一段时间的tmux了，它默认的全局按键是 Ctrl+b 而 ctrl 这两个键实在有点远，后来就给快捷键修改成了 Ctrl+a 这段时间尤其发现每次按键的时候都需要弯手腕，很不舒服，寻思着学一下大神修改一下 Caps Lock with Ctrl 会不会好一点呢，刚在windows下修改了，将方法记载在这里。

---

直接在注册表中修改键位映射关系

注册表位置：`[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout]` 如果没有此键，就新建一个

新建一个二进制值的Key，名叫：Scancode Map

输入如下的值：

```
00,00,00,00
00,00,00,00
03,00,00,00
3A,00,1D,00
1D,00,3A,00
00,00,00,00
```

![](http://www.xiaoh.me/img/post-switch-caps-ctrl-regedit.jpg)

前两行和最后一行，都是固定的，全部为0。

第三行，代表你需要有几行修改，我们进行一次交换，就是两行，之后加上最后一行，所以是3。

第四行，3A00，代表Caps Lock， 1D00，代表Ctrl。这一行，意思即为，将Ctrl修改为Caps Lock

第五行，就不用说了，意思刚好相反。

修改完毕后，重新登录Windows即可生效！

其实对我来说Caps Lock 没有什么大用，所以我只将其修改为了 Ctrl，现在两个Ctrl键了（其实就是为了过度）

下面附上各个键位值的参考：

    01 00  |   Escape 
    0F 00  |   Tab 
    3A 00  |   Caps Lock 
    38 00  |   Left Alt 
    1D 00  |   Left Ctrl 
    2A 00  |   Left Shift 
    5B E0  |   Left Windows 
    38 E0  |   Right Alt 
    1D E0  |   Right Ctrl 
    36 00  |   Right Shift 
    5C E0  |   Right Windows 
    0E 00  |   Backspace 
    53 E0  |   Delete 
    1C 00  |   Enter 
    39 00  |   Space 
    52 E0  |   Insert 
    47 E0  |   HOME 
    4F E0  |   End 
    45 00  |   Num Lock 
    51 E0  |   Page Down 
    49 E0  |   Page Up 
    46 00  |   Scroll Lock 

---

### END


