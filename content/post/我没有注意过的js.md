---
title: "我没有注意过的js"
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

# 2. 为什么 12.toString() 会报错  

JavaScript 规范中规定的数字直接量可以支持四种写法：十进制数、二进制整数、八进制整数和十六进制整数。  

十进制的 Number 可以带小数，小数点前后部分都可以省略，但是不能同时省略，例如：  

    .01
    12.
    12.01

因此就会引发一个问题  

    12.toString()

这时候 `12. `会被当做省略了小数点后面部分的数字而看成一个整体  

因此，需要在`12`后面加入空格或者用小括号括起来等方法，才能保证不报错，以下都可以避免这种情况产生：  

    12 .toString()
    (12).toString()
    12..toString()  

# 3. 在不使用递归的情况下，如何创建一个无限循环  

{{< codeblock "" "js" "" "js" >}}function sleep(duration) {
    return new Promise(function(resolve, reject) {
        setTimeout(resolve,duration);
    })
}
async function* foo(){
    i = 0;
    while(true) {
        await sleep(1000);
        yield i++;
    }
        
}
for await(let e of foo())
    console.log(e);

{{< /codeblock >}}  

这段代码定义了一个异步生成器函数，异步生成器函数每隔一秒生成一个数字，这是一个无限的生成器。 

接下来，使用 for await of 来访问这个异步生成器函数的结果，这形成了一个每隔一秒打印一个数字的无限循环。  

因为这个循环是异步的，并且有时间延迟，所以，这个无限循环的代码可以用于显示时钟等有意义的操作。  

    async 函数是可以暂停执行，等待异步操作的函数。

    异步生成器函数可以理解为可以同步调用异步函数，相当于把异步数据同步返回（这么说可能有点问题），  
    而不需要手动在callback/promise中处理数据。  