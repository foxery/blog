---
title: "解决vue开发模式下的跨域问题"
date: 2018-03-30T17:26:47+08:00
categories:
- technique
- vue
tags:
- vue
- cross domain
keywords:
- cross domain
thumbnailImage: /img/camera.jpg
---

<!--more-->
前端本地开发，进行接口请求时，会遇到跨域问题，解决方法如下：  
1. 打开`config/index.js`,在`proxyTable`中添写如下代码: 

    ```
    proxyTable: {
      '/apitest': {
        target: 'https://cnodejs.org/api/v1', // 你接口的域名
        secure: false,      // 如果是https接口，需要配置这个参数
        changeOrigin: true,     // 如果接口跨域，需要进行这个参数配置
        pathRewrite: { 
          '^/apitest': '' //把域名换成target，代理域名为空
          } 
      }
    }
    ```  

    请注意，pathRewrite下的代理域名一定要设置为空，否则会出现请求接口404的问题  
2. 请求接口时，可以这样写：

    ```
    var prefix = "/apitest";

    export function GetTopices(page, tab, limit, mdrender) {
        return Get(prefix + '/topics', {
            page: page,
            tab: tab,
            limit: limit,
            mdrender: mdrender
        });
    }
    ```    
      
3. 打包部署上线时，要把前缀换回来  
    `var prefix = "https://cnodejs.org/api/v1";`    