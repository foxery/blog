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
    
   此方法在webview内就无法判断Android与ios，返回的userAgent是一样的

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

# 5.ios出于安全机制，不允许自动触发input的focus事件，只能用户触发

# 6.解决 iphone下 input type="search" 弹出虚拟键盘 不能显示 搜索 问题
在input 外面包裹一层form即可
{{< codeblock "demo.js" "js" "" "" >}}<form action="javascript:return true;">       
    <input name= "seach" type ="search"  placeholder=""/>
</form>{{< /codeblock >}}

在vue中监听手机点击键盘搜索按钮的事件，用`@keyup.enter`即可

# 7.解决移动端点击image标签内的图片会放大预览图片问题
{{< codeblock "demo.js" "js" "" "" >}}img.addEventListener('click',function(e){
　　e.preventDefault();
});{{< /codeblock >}}
注意，这里click和touchend事件都可以，但是不可以是touchstart和touchmove事件  
因为使用touchstart和touchmove事件的时候，假如页面顶部有个超级大的banner图，那么当横屏显示或者类似于ipad等屏幕宽度大于高度的情况下，整个banner图都占满了屏幕，这个时候页面没法滑动。因为你用touchstart和touchmove禁止掉了图片的默认行为，所以手指怎么滑动，页面都没反应的。刚好这个滑动的行为触发了touchstart和touchmove

# 8.ios在调起微信支付后，无法自己返回到app的问题
在ios的webview内，当发起微信支付成功后，无论是取消支付还是支付成功后，系统都会自动打开Safari，停留在浏览器内，无法自主返回到app内  

webview内调起微信支付，是调用微信的H5支付接口，此时前端只需要将后台接口返回的data内url拼接后进行跳转处理即可
{{< codeblock "demo.js" "js" "" "" >}}var payUrl=window.location.href;
var url =data.mweb_url +"&redirect_url=" +encodeURIComponent(payUrl); //redirect_url后面的链接是支付调起后回跳的链接，需要进行编码
window.location.href = url;
{{< /codeblock >}}

因此，需要在回跳的链接上加个参数，与ios端约定好一个协议，在回跳页面加载完成后判断是否存在这个参数，有的话就发起ios协议跳转到app  
用vue来举例：
{{< codeblock "demo.vue" "vue" "" "" >}}var payUrl=window.location.href+"?rouse=gcysuser://";
var url =data.mweb_url +"&redirect_url=" +encodeURIComponent(payUrl); //redirect_url后面的链接是支付调起后回跳的链接，需要进行编码
window.location.href = url;

//在回跳页面的mounted钩子内
if (this.$route.query && this.$route.query.rouse) {
    window.location.href = this.$route.query.rouse;
}
{{< /codeblock >}}

# 9.如何使在vue内需要引用的文件不进行编译压缩
在与app交互中，遇到一个需求，前端需要调起app端的分享功能，当前端发起一份分享协议给app时，app需要请求我这边的接口以此获取需要分享的信息内容。但是发现，如果在src项目内部引用，js文件会被编译压缩，app端就找不到我这边的接口，因为此时接口已经被编译成类似于`a()`这种形式。

其实要解决这个问题很简单，通过vue-cli构建的项目中，与src同级的还有一个`static`文件夹，将js文件放在这里，就不会被编译压缩，然后在`index.html`内直接引用即可。

{{< codeblock "index.html" "html" "" "html" >}}...
<div id="app"></div>

<script type="text/javascript" src="static/js/share.js"></script>
...
{{< /codeblock >}}

# 10.关于new Date()转换时间在iOS中不生效问题  
js中使用`new Date('2019-06-20 23:59:59').getTime()`，Android端正常转换，但是在ios端没有正确转换成时间戳。  
原因是ios端不支持这种时间格式。  
将时间格式转换成`'2019/06/20 23:59:59'`即可  

# 11.关于input输入框在iOS中获取到焦点之后界面上移无法回落问题
在ios端，会遇到input输入框获取到焦点之后，软键盘自动顶起界面，但是失去焦点之后无法回落的问题  

解决方法如下：  
{{< codeblock "demo.js" "js" "" "js" >}}const scrollHeight = document.documentElement.scrollTop || document.body.scrollTop || 0;

window.scrollTo(0, Math.max(scrollHeight - 1, 0));
{{< /codeblock >}}  

# 12.canvas在移动端出现锯齿的问题  
原因应该是手机的宽度是720像素的, 而canvas是按照小于720像素画出来的, 所以在720像素的手机上显示时, 这个canvas的内容其实是经过拉伸的, 所以会出现模糊和锯齿。  

解决方法是：  
在canvas标签中设置了`width="200",height="200"`之外, 还需要在canvas标签上设置`style="width:100%;"`  

# 13.ios端webview内，当A页面滚动超过一屏后跳转到B页面，再返回A页面时会出现局部区域被遮盖的情况  

1. 出现步骤  
    1.  
{{< codeblock "app.vue" "vue" "" "vue" >}}<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<style>
html, body {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
  position: relative;
}
#app{
  position: relative;
  width: 100%;
  height: 100%;
  background: #fff;
}
</style>
{{< /codeblock >}}  

    2. 在ios端webview内访问A页面，滚动超过一屏后跳转到B页面，再返回A页面时，会出现如下情况(黄色部分就是被遮盖部分)：  
    ![](/img/ios_back.png)  
    3.点击页面或者继续进行滚动操作后，页面才会恢复正常  

2. 猜测原因  
因为设置html、body高度是100%，从而造成了 #app 撑开父级，但浏览器默认滚动的scroll是body，当使用go history (-1)时，无法复原     

3. 解决方案   
让 #app 重新成为scroll的对象 
{{< codeblock "app.vue" "vue" "" "vue" >}}<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<style>
html, body {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
  position: relative;
}
#app{
  width: 100%;
  height: 100%;
  background: #fff;
  overflow: scroll;
  -webkit-overflow-scrolling: touch;
  position: absolute;
  left:0;
  top:0;
}
</style>
{{< /codeblock >}} 