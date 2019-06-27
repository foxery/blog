---
title: "如何调试app webview"
date: 2018-05-31T17:06:06+08:00
categories:
- program
- webview
tags:
- 调试
keywords:
- 调试
thumbnailImage: /img/default.jpg
---

<!--more-->

<!-- toc -->

# 1.调试安卓端的webview
1. 下载安装[chrome canary](http://www.google.cn/chrome/browser/canary.html)


2. **打开webview的调试模式** 
  
        webView.setWebContentsDebuggingEnabled(true);
3. 打开金色谷歌浏览器，在地址栏输入`chrome://inspect/#devices`  
4. 打开app，即可在`Remote Target`看到页面链接，点击`inspect`即可调试

# 2.调试ios端的webview
1. 在苹果电脑上设置safari浏览器  
打开Safari偏好设置，选中“高级菜单“，在页面最下方看到“在菜单中显示开发菜单”的复选框，在复选框内打钩，这样设置完毕就能在Safari菜单中看到开发菜单了
![](/img/safari.jpg)
2. 设置iPhone  
打开手机设置->Safari->高级（最下面）->Web检查器打开，JavaScript开关打开
![](/img/iphone.jpg)
3. iPhone连接到mac上，打开Safari浏览器，运行手机app里面的web页面，在开发菜单中选择连接的手机，找到调试的网页，就能在Safari里面调试了   
![](/img/link.jpg)        