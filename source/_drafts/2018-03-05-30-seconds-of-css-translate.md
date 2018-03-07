---
title: 30s css 片段翻译
subtitle: 这是对 github 上《30-seconds-of-css》的翻译整理，整理了放到博客上供大家学习参考。
date: 2018-03-05
cover: http://oo12ugek5.bkt.clouddn.com/blog/images/18-03-05/30-seconds-of-css-cover.png
categories: 日常学习
tags:
    - css

author:
  nick: minfive
  link: 'https://github.com/Mrminfive'
---

<style>
  .label {
    margin-bottom: -21px !important;
    color: #333;
  }
  .label::before {
    content: attr(data-label);
    display: inline-block;
    padding: 0px 5px;
    border: 1px solid #272822;
    border-bottom: 0;
  }
  .demo-container {
    background: #f5f6f9;
    border-radius: .25rem;
    padding: .75rem 1.25rem !important;
    position: relative;
    margin-bottom: 21px !important;
    margin-top: 40px !important;
  }
  .demo-container::before {
    content: "Demo";
    display: block;
    position: absolute;
    left: 0;
    bottom: 100%;
    color: #333;
    font-weight: bold;
  }
</style>

#### Bouncing loader（弹跳加载器）

创建弹跳加载器动画。

<div class="label" data-label="html"></div>

``` html
<div class="bouncing-loader">
  <div></div>
  <div></div>
  <div></div>
</div>
```

<div class="label" data-label="css"></div>

``` css
@keyframes bouncing-loader {
  from {
    opacity: 1;
    transform: translateY(0);
  }
  to {
    opacity: 0.1;
    transform: translateY(-1rem);
  }
}
.bouncing-loader {
  display: flex;
  justify-content: center;
}
.bouncing-loader > div {
  width: 1rem;
  height: 1rem;
  margin: 3rem 0.2rem;
  background: #8385aa;
  border-radius: 50%;
  animation: bouncing-loader 0.6s infinite alternate;
}
.bouncing-loader > div:nth-of-type(2) {
  animation-delay: 0.2s;
}
.bouncing-loader > div:nth-of-type(3) {
  animation-delay: 0.4s;
}
```

<style>
  @keyframes bouncing-loader {
  from {
    opacity: 1;
    transform: translateY(0);
  }
  to {
    opacity: 0.1;
    transform: translateY(-1rem);
  }
}
.bouncing-loader {
  display: flex;
  justify-content: center;
}
.bouncing-loader > div {
  width: 1rem;
  height: 1rem;
  margin: 3rem 0.2rem;
  background: #8385aa;
  border-radius: 50%;
  animation: bouncing-loader 0.6s infinite alternate;
}
.bouncing-loader > div:nth-of-type(2) {
  animation-delay: 0.2s;
}
.bouncing-loader > div:nth-of-type(3) {
  animation-delay: 0.4s;
}
</style>

<div class="demo-container"><div class="bouncing-loader"><div></div><div></div><div></div></div></div>