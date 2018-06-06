---
title: "在vue框架中进行微信公众号开发"
date: 2018-05-31T17:37:21+08:00
categories:
- technique
- js
tags:
- vue
- wechat
keywords:
- wechat
thumbnailImage: /img/default.jpg
---

<!--more-->

<!-- toc -->

# 1.引入微信JSSDK
    cnpm install weixin-js-sdk --save

    import wx from "weixin-js-sdk";
# 2.微信配置
{{< codeblock "App.vue" "js" "" "" >}}wx.config({
    debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
    appId: /*后台返回*/, // 必填，公众号的唯一标识
    timestamp: /*后台返回*/, // 必填，生成签名的时间戳
    nonceStr: /*后台返回*/, // 必填，生成签名的随机串
    signature: /*后台返回*/, // 必填，签名
    jsApiList: [
        "onMenuShareTimeline",
        "onMenuShareAppMessage",
        "onMenuShareQQ",
        "onMenuShareWeibo",
        "chooseWXPay",
        "hideAllNonBaseMenuItem",
        "showMenuItems",
        "checkJsApi"
    ] // 必填，需要使用的JS接口列表
});
wx.ready(function() {
    wx.hideAllNonBaseMenuItem(); //隐藏所有非基础按钮接口
    wx.checkJsApi({
        jsApiList: [
        "onMenuShareTimeline",
        "onMenuShareAppMessage",
        "onMenuShareQQ",
        "onMenuShareWeibo",
        "chooseWXPay"
        ], // 需要检测的JS接口列表，所有JS接口列表见附录2,
        fail: function(res) {
            //微信版本不支持部分功能
        }
    });
});
wx.error(function(res) {
    //微信验证失败
});
{{< /codeblock >}}
# 3.微信接口
1. 分享接口
{{< codeblock "demo.vue" "js" "" "" >}}wx.ready(function() {
    //分享到朋友圈
    wx.onMenuShareTimeline({
        title: "", // 分享标题
        desc: "",
        link: "", // 分享链接，该链接域名或路径必须与当前页面对应的公众号JS安全域名一致
        imgUrl: "", // 分享图标,必须是绝对路径
        success: function() {
            // 用户确认分享后执行的回调函数
        },
        cancel: function() {
            // 用户取消分享后执行的回调函数
        }
    });
});
{{< /codeblock >}}
2. 在微信内置浏览器内调用支付接口
{{< codeblock "demo.vue" "js" "" "" >}}wx.ready(function() {
    wx.chooseWXPay({
        timestamp: /*后台返回*/, // 支付签名时间戳，注意微信jssdk中的所有使用timestamp字段均为小写。但最新版的支付后台生成签名使用的timeStamp字段名需大写其中的S字符
        nonceStr: /*后台返回*/, // 支付签名随机串，不长于 32 位
        package: /*后台返回*/, // 统一支付接口返回的prepay_id参数值，提交格式如：prepay_id=\*\*\*）
        signType: /*后台返回*/, // 签名方式，默认为'SHA1'，使用新版支付需传入'MD5'
        paySign: /*后台返回*/, // 支付签名
        success: function(res) {
        //支付成功
        },
        fail: function(res) {
        //支付失败
        },
        cancel: function() {
        //支付取消
        }
    });
});
{{< /codeblock >}}
3. 在移动端浏览器内调用支付接口  

  后台会通过接口返回给你一串url，只要将这串url拼接下，后面加上支付成功后你希望回跳的地址，然后直接访问该地址即可  

    var url=res +"&redirect_url=" +encodeURIComponent(payUrl);
    window.location.href = url;
    