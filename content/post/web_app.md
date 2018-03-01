---
title: "开发webApp时遇到的问题记录"
date: 2018-02-27T17:27:47+08:00
categories:
- technique
- webApp
tags:
- webApp
keywords:
- webApp
thumbnailImage: /img/camera.jpg
---

<!--more-->
### 1.区分安卓与ios

这个很容易,`window.android`即可区分

### 2.长按功能的实现
原理很简单,为触摸事件加一个事件延迟,在设定时间之后清除延迟事件即可。代码如下:  

``` javascript
var timeout=0;//设置一个定时器
$.on("touchstart", function(){
    timeout = setTimeout(function () {
        something();
    }, 400);
});
$.on("touchend", function () {
    var me = this;
    clearTimeout(timeout);
});
```