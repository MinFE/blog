---
title: 基于Inline-Block布局+vertical-Align的研究
subtitle: 基于Inline-Block布局+vertical-Align的一些个人理解。
cover: http://oo12ugek5.bkt.clouddn.com/blog/images/issue-02.png
date: 2016-03-22
categories: 日常学习
tags:
  - 学习
  - 百度ife
author:
    nick: minfive
    github: https://github.com/Mrminfive
---

> 前言：这个探究主要源于ife任务6的一个布局困扰，继而产生以下这些探究及结论，如解释有误或有新的见解，请及时与我联系，谢谢大家的捧场。

------

### 问题引入

有需求才会有解决方案，那么，这个需求是什么呢？

这是任务6布局的两个点

![img1][1]
![img2][2]

`review`了许多同学的代码，实现方式基本局限于两种，`position`定位、`float+内外边距`再者就是两者结合，那么又没有第三种更为简单的适合的方法呢？
答案是有的，那就是基于`inlink-block`+`vertical-align`的方式


### inline-block + vertical-align

什么是`inline-block`相信大家比我还要清楚，但用来布局的话还有几个重要的点需要大家着重注意的：

> * inline-block是行内块级元素，因此这种布局仅适用于单排布局（这点大家应该不会有太多异议吧）
> * inline-block布局+vertical-align的关键点在于`valign特性`的使用，因此对于`vertical-align`的理解尤为关键（具体可参照[张鑫旭大神关于vertical-align的理解][3]）


### 解决方法

基于上述技术，我写了一个新的解决方法，具体如下：

html代码部分：

``` html
<header class="header">
    <div class="header-mark">
        <span>ife.baidu.com</span>
    </div>
    <div class="header-date">
        <span>2016.03</span>
    </div>
</header>
```

css代码部分：

``` css
/*头部*/
.header{
    font-size: 12px;
    vertical-align: bottom;
}
    .header-mark{
        float: left;
        width: 110px;
        height: 61px;
        display: inline-block;
        background-color: #d45d5c;
    }
    .header-mark::before{
        content:'';
        line-height: 58px;
    }
    .header-mark>span{
        vertical-align: bottom;
    }
    .header-date{
        float:right;
        height: 61px;
        display: inline-block;
        color: #d45d5c;
    }
    .header-date::before{
        content:'';
        line-height: 58px;
        margin-left: -0.5em;
    }
    .header-date>span{
        vertical-align: bottom;
    }
```

原理其实很简单，利用伪元素去做基准线，然后其它元素以伪元素为基准进行排布，方便快捷，更重要的是这种方式维护起来也跟方便


### 结语

> 第一次写学习笔记，写的不好的地方请大家多多包涵，也请大家不吝啬给出意见，共同学习，共同进步，谢谢大家！


[1]: http://oo12ugek5.bkt.clouddn.com/blog/images/issue-01.png
[2]: http://oo12ugek5.bkt.clouddn.com/blog/images/issue-02.png
[3]: http://www.zhangxinxu.com/wordpress/2010/05/%E6%88%91%E5%AF%B9css-vertical-align%E7%9A%84%E4%B8%80%E4%BA%9B%E7%90%86%E8%A7%A3%E4%B8%8E%E8%AE%A4%E8%AF%86%EF%BC%88%E4%B8%80%EF%BC%89/