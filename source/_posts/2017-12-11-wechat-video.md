---
title: 微信视频 h5 开发小结
subtitle: 近日，由于工作原因，需要开发一款视频类 h5。玩法很简单，用户滚页面至底部，切换全屏视频播放。
date: 2017-12-11
cover: http://oo12ugek5.bkt.clouddn.com/blog/images/17-12-11/WeChat-logo.jpg
categories: 经验总结
tags:
    - wechat

author:
  nick: minfive
  link: 'https://github.com/Mrminfive'
---

### 前言

近日，由于工作原因，需要开发一款视频类 h5。玩法很简单，用户滚页面至底部，切换全屏视频播放。
地址如下：

![10-08][vip10-08]


### 准备

#### 全屏同层播放

为了满足交互需求，需要达到全屏同层播放，具体配置如下：

``` html
<video
    id="my-video" 
    type="video/mp4" x5-video-player-fullscreen="false"
    webkit-playsinline
    playsinline
    x-webkit-airplay="allow" x5-video-player-type="h5"
    src="www.baidu.com"></video>
```

#### 视频预加载

由于该 H5 为视频类 H5 ，在移动端播放视频还是要考虑到加载速度及稳定播放等问题，因此笔者采用的是 “边播边加载” 的方式，利用 video 标签的 `canplaythrough` 事件作为页面加载完成的标志。

由于移动端原因， video 并不会主动去预加载用户未需求的资源，因此我们需要手动去触发 video 的预加载资源，

``` js
document.getElementById('my-video').play();
```

这样就可以了吗？

不，万恶的微信限制了必须用户行为才能播放媒体资源，因此我们只能再祭出万能 hack：

```
document.addEventListener('DOMContentLoaded', function() {
    let video = document.getElementById('my-video');

    function preload() {
        video.play();
        setTimeout(function () {
            video.pause();
        }, 200);
    }
    
    document.addEventListener("WeixinJSBridgeReady",  preload, false);
    if (typeof WeixinJSBridge == "object" && typeof WeixinJSBridge.invoke == "function") {
        WeixinJSBridge.invoke("getNetworkType", {}, preload);
    }
    
    preload(); 
});
```

### 槽点

#### 布局限制

安卓端由于无法像 ios 端完美的实现全屏播放，播放视频时手机依旧会进入媒体播放模式，这就导致了播放视频时会连带着媒体播放层的弹出，而媒体播放层则类似于 `position: fixed; top: 0; left: 0;` 的效果固定在页面顶部，每个媒体
播放层的弹出，都会预先将页面滚动至顶部，如果页面发生滚动，在安卓端播放视频就会出现短暂的黑屏。

**因此，微信视频类 H5 如果需要滚动存在，请采用局部滚动。**

#### 用户行为限制

安卓端下，滚动行为不归属在用户操作行为中，滚动事件中无法执行视频播放，在这个问题上，笔者使用 `touchstart` 对安卓端进行 hack。

### 参考文献

* [video 标签在微信浏览器的问题解决方法][video 标签在微信浏览器的问题解决方法]
* [微信内置浏览器 如何小窗不全屏播放视频？][微信内置浏览器 如何小窗不全屏播放视频？]


[vip10-08]: http://oo12ugek5.bkt.clouddn.com/blog/images/17-12-11/0c05c64de7df694e1b4fa5ea99f3ddb8.png
[video 标签在微信浏览器的问题解决方法]: http://www.cnblogs.com/baiyygynui/p/6323565.html
[微信内置浏览器 如何小窗不全屏播放视频？]: https://www.zhihu.com/question/36423771