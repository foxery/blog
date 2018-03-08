---
title: "创建对象的过程以及方法"
date: 2018-03-07T10:18:33+08:00
categories:
- technique
- js
tags:
- object
- js
keywords:
- object
thumbnailImage: /img/workday.jpg
---

<!--more-->
### 过程
js中当new一个对象时,其过程到底是怎样的呢?  

``` javascript
  function Test() {};
  var test = new Test();
  // new的过程
  var test = {};
  this = test;
  Test.call(this);
  test.__proto__ = Test.prototype;
  return test;
```

每个实例对象(不论是普通对象还是函数对象)都有__proto__的属性(用于指向创建它的函数对象的原型对象prototype,原型对象prototype中都有个预定义的constructor属性，用来引用它的函数对象,即`Test.prototype.constructor === Test;//true`),每个函数对象都有prototype的属性,类型是object,即一个引用对象  
具体可参考!(https://segmentfault.com/a/1190000000532556)  

1. 当JavaScript引擎执行new操作时,会马上开辟一个块内存,创建一个空对象test,并将this指向这个对象  
2. 执行构造函数Test()，对这个空对象进行构造,将构造函数里的属性和方法都添加进去  
3. 给空对象test添加__proto__属性,指向构造函数Test()的prototype对象  

### 方法  
