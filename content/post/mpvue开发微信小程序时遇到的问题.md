---
title: "mpvue开发微信小程序时遇到的问题"
date: 2019-06-20T14:31:47+08:00
categories:
- category
- vue
tags:
- mpvue
- 微信小程序
keywords:
- tech
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
<!-- toc -->

# 1.input、textarea使用v-model绑定时，当快速频繁操作输入键盘时，文本内容会延迟显示，甚至会反复出现已删除文本，显示错乱

## 解决方案:  
使用`v-model.lazy`  

## 注意事项:  
`lazy`修饰符会被`Vue`编译成`change`事件,而`mpvue`会把`change`事件转换成`blur`事件，由于`blur`事件触发在点击提交`之后`，这样会导致：如果用户输入了最后一个输入框(使用lazy修饰符的)，没有将光标移开，而是直接点击提交，则最后一个提交会被忽略。  

因此，需要在数据提交的时候，做一个延时，使用`setTimeout`大概延时个`100`毫秒