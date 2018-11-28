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
  }
{{< /codeblock >}}  

另外，注意一下`passive: false`。为什么要加这句呢？  

如果没有这句的话，控制台大概会有如下提示：  
{{< codeblock "console.js" "js" "" "js" >}}Unable to preventDefault inside passive event listener due to target being treated as passive. See https://www.chromestatus.com/features/5093566007214080
{{< /codeblock >}}  

由于浏览器必须要在执行事件处理函数之后，才能知道有没有调用过`preventDefault()`，这就导致了浏览器不能及时响应滚动，略有延迟。

所以为了让页面滚动的效果如丝般顺滑，从 chrome56 开始，在`window`、`document` 和 `body` 上注册的 `touchstart` 和 `touchmove` 事件处理函数，会默认为是 `passive: true`。浏览器忽略 `preventDefault()` 就可以第一时间滚动了。

举例：
`wnidow.addEventListener('touchmove', func)` 效果和下面一句一样
`wnidow.addEventListener('touchmove', func, { passive: true })`  

这就导致了一个问题:  
如果在以上这 3 个元素的 `touchstart` 和 `touchmove` 事件处理函数中调用 `e.preventDefault()` ，会被浏览器忽略掉，并不会阻止默认行为。  

因此，要加上`passive: false`，明确声明为不是被动的。  

另外，还有一种方法：  
应用 CSS 属性 `touch-action: none;` 这样任何触摸事件都不会产生默认行为，但是 touch 事件照样触发。关于[touch-action](https://w3c.github.io/pointerevents/#the-touch-action-css-property)