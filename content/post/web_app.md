---
title: "记录开发app webview时遇到的问题"
date: 2018-02-27T17:27:47+08:00
categories:
- technique
- webview
tags:
- webview
keywords:
- webview
thumbnailImage: /img/camera.jpg
---

<!--more-->

<!-- toc -->

# 1.区分安卓与ios

    window.android //安卓

# 2.长按功能的实现
原理很简单,为触摸事件加一个事件延迟,在设定时间之后清除延迟事件即可。代码如下:  

{{< codeblock "demo.js" "js" "" "" >}}var timeout=0;//设置一个定时器
$.on("touchstart", function(){
    timeout = setTimeout(function () {
        something();
    }, 400);
});
$.on("touchend", function () {
    var me = this;
    clearTimeout(timeout);
});
{{< /codeblock >}}

# 3.ios 5s不支持flex属性,不支持transform:translate(x,y)属性

# 4.ios 触摸滑动卡顿问题
{{< codeblock "demo.css" "css" "" "" >}}* {
    -webkit-overflow-scrolling : touch;
}{{< /codeblock >}}