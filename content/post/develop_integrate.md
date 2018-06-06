---
title: "整合项目开发过程中遇到的各种小问题"
date: 2018-05-30T15:30:50+08:00
categories:
- technique
- js
tags:
- js
keywords:
- css
thumbnailImage: /img/profile.jpg
---

<!--more-->

<!-- toc -->

# 1.只针对打印模式修改css
{{< codeblock "demo.html" "xml" "" "" >}}<style type="text/css" media="print">
    img
    {
        display:none;
    }
</style>
{{< /codeblock >}}

# 2.CSS实现单行、多行文本溢出显示省略号
{{< codeblock "demo.css" "css" "" "" >}}// 单行省略
.single{
    overflow: hidden;
    text-overflow:ellipsis;
    white-space: nowrap;
}
//多行省略 因使用了WebKit的CSS扩展属性，该方法适用于WebKit浏览器及移动端
.multi{
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 3;
    overflow: hidden;
}
{{< /codeblock >}}

# 3.js判断浏览器类型，以及是否为手机端
{{< codeblock "demo.js" "js" "" "" >}}var browser={
    versions:function(){
    var u = window.navigator.userAgent;
    return {
        trident: u.indexOf('Trident') > -1, //IE内核
        presto: u.indexOf('Presto') > -1, //opera内核
        webKit: u.indexOf('AppleWebKit') > -1, //苹果、谷歌内核
        gecko: u.indexOf('Gecko') > -1 && u.indexOf('KHTML') == -1, //火狐内核
        mobile: !!u.match(/AppleWebKit.*Mobile.*/)||!!u.match(/AppleWebKit/), //是否为移动终端
        ios: !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/), //ios终端
        android: u.indexOf('Android') > -1 || u.indexOf('Linux') > -1, //android终端或者uc浏览器
        iPhone: u.indexOf('iPhone') > -1 || u.indexOf('Mac') > -1, //是否为iPhone或者安卓QQ浏览器
        iPad: u.indexOf('iPad') > -1, //是否为iPad
        webApp: u.indexOf('Safari') == -1 ,//是否为web应用程序，没有头部与底部
        weixin: u.indexOf('MicroMessenger') > -1 //是否为微信浏览器
        };
    }()
}
console.log(" 是否为移动终端: "+browser.versions.mobile);
console.log(" ios终端: "+browser.versions.ios);
console.log(" android终端: "+browser.versions.android);
console.log(" 是否为iPhone: "+browser.versions.iPhone);
console.log(" 是否iPad: "+browser.versions.iPad);
{{< /codeblock >}}

# 4.对接支付宝支付接口
通过后台接口返回一串字符串形式的html表单，将它渲染到页面上，由于是异步渲染的表单，所以需要手动触发submit事件
{{< codeblock "demo.vue" "js" "" "" >}}<template>
    <div v-html="ZFBForm"></div>
</template>

<script>
    export default {
        data() {
            return {
                ZFBForm: null
            };
        },
        mounted(){
            this.ZFBForm = res;
            this.$nextTick(() => {
                document.forms["alipaysubmit"].submit();
            });
        }
    }
</script>
{{< /codeblock >}}