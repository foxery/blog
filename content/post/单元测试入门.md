---
title: "单元测试入门"
date: 2018-04-03T15:17:42+08:00
categories:
- technique
- js
tags:
- unit test
- mocha
keywords:
- unit test
thumbnailImage: /img/cover1.jpg
---

<!--more-->
### 安装Mocha
    `cnpm i mocha -g`  
## 测试
    在与项目文件夹的同级目录下，新建一个test文件夹，测试脚本与待测试的脚本同名，后缀名改为`.test.js`,结构如下：      

    .
    +-- app
    |   +-- index.js
    +-- test
    |   +-- index.test.js

    index.js 内容如下：  

    ```js
    var calculate = {
        add: function(a, b){
            return a + b;
        },
        divide: function(a, b){
            return a / b;
        }
    }

    module.exports = calculate;
    ```

    index.test.js 内容如下：  

    ```js
    var calculate =  require('./../app/b');
    var expect = require('chai').expect;

    describe('Caculate', function() {
        describe('cal', function() {
            it('return add result', function() {
                expect(calculate.add(1, 2)).equal(3);
            });
            it('return divide result', function() {
                expect(calculate.divide(2, 2)).equal(1);
            });
        });
    });
    ```

    脚本下包括一个或多个describe块，每个describe块包括一个或多个it块。

    describe块称为“测试套件”，表示一组相关的测试。它是一个函数，第一个参数是测试套件的名称，第二个参数是实际执行的函数。

    it块称为“测试用例”，表示一个单独的测试，是测试的最小单位。

    `expect(calculate.add(1, 2)).equal(3);`语句叫做断言。所谓断言就是判断源码的实际执行结果与预期是否一致，如果不一致就抛出一个错误。

    所有测试用例都应该含有一个或多个断言。断言功能由断言库来实现，Mocha本身并不带有断言库，所以必须先引入断言库。上面代码引入的断言库是chai，并且指定使用它的expect断言风格。expect断言的优点就是很接近自然语言。

    发现`expect(calculate.add(1, 2)).equal(3);`与`expect(calculate.add(1, 2)).to.be.equal(3);`都一样。  

### 运行
    用命令行工具运行`mocha`命令，会出现以下结果：  
    ![](/img/mocha.png)      

    这样会默认运行test文件夹下的所有测试脚本，如果想单独运行，可以`$ mocha index.test.js`，也可以指定运行多个，如`mocha index.test.js b.test.js`。

    如果test文件夹下存在子目录，则需要执行`$ mocha --recursive`才能执行所有的测试脚本。

### 异步测试

    Mocha默认每个测试用例最多执行2000毫秒，如果到时没有得到结果，就报错。对于涉及异步操作的测试用例，这个时间往往是不够的，需要用`-t`或`--timeout`参数指定超时门槛。

1. 延时

    ```js
    it('测试应该5000毫秒后结束', function(done) {
        var x = true;
        var f = function() {
            x = false;
            expect(x).to.be.not.ok;
            done(); // 通知Mocha测试结束
        };
        setTimeout(f, 4000);
    });
    ```  

    这需要运行`$ mocha -t 5000`  

    当测试结束的时候，必须显式调用`done()`函数，告诉Mocha测试结束了。否则，Mocha就无法知道，测试是否结束，会一直等到超时报错。

2. 异步请求  

    ```js
    var request = require('superagent');

    it('异步请求应该返回一个对象', function (done) {
        request
            .get('https://cnodejs.org/api/v1/topics')
            .end(function (err, res) {
                expect(res).to.be.an('object');
                done();
            });
    });
    ```  

    Mocha内置对Promise的支持，允许直接返回Promise，等到它的状态改变，再执行断言，而不用显式调用done方法。如下：  

    ```js
    var fetch = require('node-fetch');

    it('异步请求应该返回一个对象', function () {
        return fetch('https://cnodejs.org/api/v1/topics')
            .then(function (res) {
                return res.json();
            }).then(function (json) {
                expect(json).to.be.an('object');
            });
    });
    ```

### 文档

更具体的，可参考:  
[测试框架 Mocha 实例教程](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)     
[Chai.js断言库API中文文档](https://www.jianshu.com/p/f200a75a15d2) 