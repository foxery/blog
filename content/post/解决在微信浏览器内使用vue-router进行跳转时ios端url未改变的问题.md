---
title: "解决在微信浏览器内使用vue Router进行跳转时ios端url未改变的问题"
date: 2018-11-27T17:05:19+08:00
categories:
- technique
- vue
tags:
- vue-router
- wechat
keywords:
- 路由跳转
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
问题描述：  
在微信浏览器内，ios手机端，使用vue-router进行页面跳转，跳转后的页面通过微信菜单复制链接，复制出来的链接仍是没有跳转之前的链接，但是通过程序输出的链接是跳转后的正确链接。  

问题原因：  
猜测大概是ios手机端，在微信浏览器内，无法识别#号键后面的url的更改。