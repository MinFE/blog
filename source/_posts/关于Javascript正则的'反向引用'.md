title: 关于Javascript正则的'反向引用'
subtitle: 了解关于javascript正则'反向引用'的一些知识及使用。
cover: http://oo12ugek5.bkt.clouddn.com/blog/images/maxresdefault.jpg
date: 2016-03-09
categories: 日常学习
tags:
  - 学习
  - javascript
author:
    nick: minfive
    github: https://github.com/Mrminfive

---

> 前言：一直忙着找实习单位，有几月没有写过博客，也没有学习过，终于找到一个勉强合适的，还是赶紧加油吧，别被别人甩太远了！ 本来是在看着阿树博客里的jQuery规范的，不知道怎么搞得，稀里糊涂的就看到正则去了，把学到的东西总结一下。

------

### 首先先看下面这个例子：

``` javascript
"abcd".match(/(a(bc)d)/);
//result: ["abcd", "abcd", "bc"]
```

这个例子说明看括号匹配顺序是按**左括号**计算的。（这是别人的理解，我的理解是匹配顺序是按**从外到里从左到右**计算的，每个反向引用都由一个编号或名称来标识，并通过“\编号”表示法进行引用，外面的组的编号靠前。也就是说引用分组是编号排列是**从外到内**编排。）

### 再看这个例子

``` javascript
'aaa'.match(/(a\1)/);
//result: ['a']  
'aaaaaaa'.match(/(a\1\1\1\1\1)/);  
//result: ['a']
```

由这个例子可以看出`/(a\1)/`在第一个括号中使用`\1`引用是没有意义的，同时在chrome中的测试结果表明，无论在第n个括号中有几个`\n`都会被忽略。（既从最外层括号往里层数）

### 最后

基于上述两个例子的解读，咱们可以把下边的例子进行转换

``` javascript
'aaabbbcccdddeeefff'.match(/[abc]\1/g);//null  
'aaabbbcccdddeeefff'.match(/([abc])\1/g);//["aa", "bb", "cc"]  
'aaabbbcccdddeeefff'.match(/(([abc])\1)\1/g);//["aa", "bb", "cc"]  
'aaabbbcccdddeeefff'.match(/(([abc])\1)\2/g);//["aa", "bb", "cc"]  
'aaabbbcccdddeeefff'.match(/((([abc])\1)\2)\3/g);//["aaa", "bbb", "ccc"]
```

可简化为

``` javascript
'aaabbbcccdddeeefff'.match(/[abc]\1/g);//null  
'aaabbbcccdddeeefff'.match(/([abc])\1/g);//["aa", "bb", "cc"]  
'aaabbbcccdddeeefff'.match(/(([abc]))\1/g);//["aa", "bb", "cc"]  
'aaabbbcccdddeeefff'.match(/(([abc]))\2/g);//["aa", "bb", "cc"]  
'aaabbbcccdddeeefff'.match(/((([abc]))\2)\3/g);//["aaa", "bbb", "ccc"]
```

### 参考文档

* [如何理解javascript正则“反向引用”](https://segmentfault.com/q/1010000000580762)