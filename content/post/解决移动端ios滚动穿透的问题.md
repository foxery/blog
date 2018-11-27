---
title: "解决移动端ios滚动穿透的问题"
date: 2018-11-27T15:25:26+08:00
categories:
- technique
- vue
tags:
- vue
- ios
keywords:
- 滚动穿透
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

问题描述：  
在ios端，当页面可以滚动，并且弹出的弹窗也可以滚动时，在弹窗区域滚动时底部的被遮盖区域也在滚动。  

问题原因：  
滚动穿透。   

解决思路：  
弹窗显示时禁止body的滚动事件，弹窗关闭时恢复body的滚动事件。


{{< codeblock "test.vue" "vue" "" "vue" >}}methods:{
    handler: function(e) {
      e.preventDefault();
    },
    closeTouch() {
      document
        .getElementsByTagName("body")[0]
        .addEventListener("touchmove", this.handler, { passive: false }); //阻止默认事件
    },
    openTouch() {
      document
        .getElementsByTagName("body")[0]
        .removeEventListener("touchmove", this.handler, { passive: false }); //打开默认事件
    }
}
{{< /codeblock >}}  

在显示弹窗的事件内调用`closeTouch()`，在隐藏弹窗的事件内调用`openTouch()`即可  

若使用UI框架，如`mint-ui`时，关闭弹窗时没有回调事件，则监听弹窗的model即可  

{{< codeblock "test.vue" "vue" "" "vue" >}}watch: {
    visible: function(news, olds) {
      if (news) {
        this.closeTouch();
      } else {
        this.openTouch();
      }
    }
  },
{{< /codeblock >}}