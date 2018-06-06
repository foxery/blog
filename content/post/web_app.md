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
1.  增加一个交互接口  

        window.android //需要安卓通过addJavascriptInterface定义一个接口，然后将name命名成android

2. 通过userAgent判断  

        const browser = {
            versions: function () {
                var u = window.navigator.userAgent;
                return {
                    trident: u.indexOf('Trident') > -1, //IE内核
                    presto: u.indexOf('Presto') > -1, //opera内核
                    webKit: u.indexOf('AppleWebKit') > -1, //苹果、谷歌内核
                    gecko: u.indexOf('Gecko') > -1 && u.indexOf('KHTML') == -1, //火狐内核
                    mobile: !!u.match(/AppleWebKit.*Mobile.*/) || !!u.match(/AppleWebKit/), //是否为移动终端
                    ios: !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/), //ios终端
                    android: u.indexOf('Android') > -1 || u.indexOf('Linux') > -1, //android终端或者uc浏览器
                    iPhone: u.indexOf('iPhone') > -1 || u.indexOf('Mac') > -1, //是否为iPhone或者安卓QQ浏览器
                    iPad: u.indexOf('iPad') > -1, //是否为iPad
                    webApp: u.indexOf('Safari') == -1,//是否为web应用程序，没有头部与底部
                    weixin: u.indexOf('MicroMessenger') > -1 //是否为微信浏览器
                };
            }()
        }        

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