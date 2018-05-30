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
#thumbnailImage: //example.com/image.jpg
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
