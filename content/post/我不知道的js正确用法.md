---
title: "我不知道的js正确用法"
date: 2019-07-01T15:24:49+08:00
categories:
- program
- js
tags:
- tag1
- tag2
keywords:
- tech
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

<!-- toc -->

# 1. 如何使 0.1 + 0.2 == 0.3 成立  

根据浮点数的定义，非整数的 Number 类型无法用 ==（=== 也不行）来比较。  

检查等式左右两边差的绝对值是否小于最小精度，才是正确的比较浮点数的方法。代码如下：  

    Math.abs(0.1 + 0.2 - 0.3) <= Number.EPSILON  

   