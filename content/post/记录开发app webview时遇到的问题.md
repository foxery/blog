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