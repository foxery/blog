---
title: "解决在微信浏览器内使用vue Router进行跳转时ios端url未改变的问题"
date: 2018-11-27T17:05:19+08:00
categories:
- problem
- vue
tags:
- vue-router
- 微信公众号
- ios
keywords:
- ios
- 路由跳转
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
背景：  
在微信公众号开发中，新增一个支付宝支付的需求，当选择支付宝支付时，需要跳转到一个支付宝支付说明页，在这个页面引导用户通过微信菜单栏从浏览器内打开该页面。但是发现，浏览器内打开的地址一直是项目最初的访问地址，导致无法调起支付宝支付。

问题描述：  
在微信浏览器内，ios手机端，使用vue-router进行页面跳转，跳转后的页面通过微信菜单复制链接，复制出来的链接仍是没有跳转之前的链接，但是通过程序输出的链接是跳转后的正确链接。  

问题原因：  
猜测大概是ios手机端，在微信浏览器内，无法识别#号键后面的url的更改。  

解决思路：  
判断如果是ios手机端，则进行`window.location.herf`的跳转，否则仍使用`router.push()`进行跳转。  

{{< codeblock "test.vue" "vue" "" "vue" >}}const browser = {
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

...  
methods:{
    ...
    if (browser.versions.ios) {
        window.location.href = ``;
        } else {
        this.$router.push({});
        }
    ...
}

{{< /codeblock >}}