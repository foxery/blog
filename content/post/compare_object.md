---
title: "如何比较两个对象是否相同"
date: 2018-02-09T16:12:34+08:00
categories:
- technique
- js
tags:
- object
keywords:
- 对象比较
thumbnailImage: /img/cover.jpg
---

<!--more-->
### 1. 比较对象
两个js对象之间的比较,需要遍历其属性,一一对比。  

``` javascript
function CompareObjectEqual(a, b) {
    if (!(a instanceof Object && b instanceof Object)) {
        return a === b;
    }

    var aProps = Object.getOwnPropertyNames(a);
    var bProps = Object.getOwnPropertyNames(b);

    if (aProps.length != bProps.length) {
        return false;
    }

    for (var i = 0; i < aProps.length; i++) {
        var propName = aProps[i];
        if ((a[propName] instanceof Object) && (b[propName] instanceof Object)) {
            CompareObjectEqual(a[propName], b[propName]);
        } else {
            if (a[propName] !== b[propName]) {
                return false;
            }
        }
    }

    return true;
}
```  