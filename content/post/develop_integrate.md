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

# 5.打印html时自动分页
在需要分页的地方加上以下css即可
这个属性好像对原始table结构更友好些，不知为什么，在div结构内存在table时，使用此属性，会出现table的thead在tbody内出现多余重叠现象，原因不明
{{< codeblock "demo.css" "css" "" "" >}}page-break-before:auto;
page-break-after:always;{{< /codeblock >}}  

# 6.高分辨率移动端下，img与img衔接顶部会出现1px的空隙问题
![](/img/1px.png)  
{{< codeblock "demo.html" "html" "" "" >}}<img src="img1.jpg" alt="" class="full-img">
<img src="img2.jpg" alt="" class="full-img">
{{< /codeblock >}}
{{< codeblock "demo.css" "css" "" "" >}}.full-img {
  width: 100%;
  display: block;
}
{{< /codeblock >}}  
尝试了许多方法：
1.{{< codeblock "demo.css" "css" "" "" >}}.full-img {
  width: 100%;
  line-height: 0;
  vertical-align:top; // 或者bottom
}
{{< /codeblock >}}    
2.在img外围包裹一个div，将div的`font-size`设为`0`

都无效  

最后只能加个`margin-top:-1px`来 hack 一下  
{{< codeblock "demo.css" "css" "" "" >}}.full-img {
  width: 100%;
  display: block;
  //hack 高分辨率手机下会出现img之间1px的间隙
  margin-top: -1px;
}
{{< /codeblock >}}

`不过话说回来，要解决两个inline-block元素之间的间距问题，将其父元素的font-size设为0真的很管用`