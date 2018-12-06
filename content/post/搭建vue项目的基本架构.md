---
title: "搭建vue项目的基本架构"
date: 2018-12-06T14:36:33+08:00
categories:
- technique
- vue
tags:
- vue
- architecture
keywords:
- vue
- architecture
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

<!-- toc -->
# 1.安装vue-cli

具体步骤参考[这一篇博文](https://foxery.github.io/2018/02/%E6%90%AD%E5%BB%BAvue%E8%84%9A%E6%89%8B%E6%9E%B6/)

# 2.修改开发环境和线上环境的配置  

1. 修改资源引用的配置路径  
{{< codeblock "config/index.js" "js" "" "js" >}}assetsPublicPath: './'
{{< /codeblock >}}
{{< codeblock "build/utils.js" "js" "" "js" >}}return ExtractTextPlugin.extract({
    use: loaders,
    fallback: 'vue-style-loader',
    publicPath:'../../' //<--注意此处路径
})
{{< /codeblock >}}

    具体原因请参考[这里](https://foxery.github.io/2018/02/%E8%AE%B0%E5%BD%95vue%E7%9A%84%E8%B8%A9%E5%9D%91%E4%B9%8B%E8%B7%AF%E4%BB%A5%E5%8F%8A%E7%BB%8F%E9%AA%8C%E5%88%86%E4%BA%AB/#6-%E9%A1%B9%E7%9B%AE%E6%89%93%E5%8C%85%E5%90%8E%E9%83%A8%E7%BD%B2%E5%88%B0%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E6%89%93%E5%BC%80%E4%B8%80%E7%89%87%E7%A9%BA%E7%99%BD-%E5%8F%91%E7%8E%B0%E6%98%AF%E6%96%87%E4%BB%B6%E5%BC%95%E7%94%A8%E8%B7%AF%E5%BE%84%E9%94%99%E8%AF%AF%E9%97%AE%E9%A2%98)  

2. 解决开发模式下的跨域问题  
{{< codeblock "config/index.js" "js" "" "js" >}}proxyTable: {
      '/apitest': { //跨域接口请求标记
        target: 'https://cnodejs.org/api/v1', // 开发环境下需要跨域请求接口的域名
        secure: false,      // 如果是https接口，需要配置这个参数
        changeOrigin: true,     // 如果接口跨域，需要进行这个参数配置
        pathRewrite: { 
          '^/apitest': '' //把域名换成target，代理域名为空，请注意，pathRewrite下的代理域名一定要设置为空，否则会出现请求接口404的问题
          } 
      }
    }
{{< /codeblock >}}
{{< codeblock "build/webpack.dev.conf.js" "js" "" "js" >}}plugins: [
    new webpack.DefinePlugin({
      API_HOST:'"/apitest"' //设置全局变量 注意单双引号
    })
]
{{< /codeblock >}}
{{< codeblock "build/webpack.prod.conf.js" "js" "" "js" >}}plugins: [
    new webpack.DefinePlugin({
      API_HOST:'"https://cnodejs.org/api/v1"' //设置线上环境的全局变量
    })
]
{{< /codeblock >}}
{{< codeblock "build/webpack.dev.build.conf.js" "js" "" "js" >}}plugins: [
    new webpack.DefinePlugin({
      API_HOST:'"https://cnodejs.org/api/v1"' //设置开发环境的全局变量
    })
]
{{< /codeblock >}}  
    详情请参考[这里](https://foxery.github.io/2018/02/%E8%AE%B0%E5%BD%95vue%E7%9A%84%E8%B8%A9%E5%9D%91%E4%B9%8B%E8%B7%AF%E4%BB%A5%E5%8F%8A%E7%BB%8F%E9%AA%8C%E5%88%86%E4%BA%AB/#7-%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3vue%E5%BC%80%E5%8F%91%E6%A8%A1%E5%BC%8F%E4%B8%8B%E7%9A%84%E8%B7%A8%E5%9F%9F%E9%97%AE%E9%A2%98)  

3. 增加开发/测试环境的打包配置

    通常一个项目需要多个环境进行开发测试，开发环境下请求的域名和线上环境请求的域名是不一样的，为方便分别打包线上环境和开发/测试环境，我们需要修改下打包配置。  

    很简单，直接复制一份build/build.js文件，重命名为build/build-dev.js,然后修改成如下配置：  
{{< codeblock "build/build-dev.js" "js" "" "js" >}}...
process.env.NODE_ENV = 'development'  
...
const webpackConfig = require('./webpack.dev.build.conf')
const spinner = ora('building for development...')
...
{{< /codeblock >}}  

    修改package.json文件，在scripts脚本内增加`"build-dev": "node build/build-dev.js"`  

    打包时，运行`npm run build-dev`,打包出来的文件是开发模式下；运行`npm run build`，打包的则是线上环境的文件。  

#  3.引入babel-polyfill,兼容低版本浏览器  
1. `npm i babel-polyfill --save-dev`
2. 引入
    最好在`import Vue from 'vue'`之前引入
{{< codeblock "main.js" "js" "" "js" >}}import "babel-polyfill";
{{< /codeblock >}} 

# 4.开启gzip 
具体请看[这一篇博文](https://foxery.github.io/2018/10/vue%E5%8D%95%E9%A1%B5%E5%BA%94%E7%94%A8%E9%A6%96%E5%B1%8F%E5%8A%A0%E8%BD%BD%E4%BC%98%E5%8C%96/#5-%E5%BC%80%E5%90%AFgzip)  

# 5.线上环境使用cdn
具体请看[这一篇博文](https://foxery.github.io/2018/10/vue%E5%8D%95%E9%A1%B5%E5%BA%94%E7%94%A8%E9%A6%96%E5%B1%8F%E5%8A%A0%E8%BD%BD%E4%BC%98%E5%8C%96/#3-%E4%BD%BF%E7%94%A8cdn)  

此外，再新建一份`index.dev.html`,内容如下：  
{{< codeblock "index.dev.html" "html" "" "html" >}}<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1.0">
  <title>vue-base</title>
</head>

<body>
  <div id="app"></div>
  <!-- built files will be auto injected -->
</body>

</html>
{{< /codeblock >}}   
修改配置文件
{{< codeblock "build/webpack.dev.build.conf.js" "js" "" "js" >}}...
new HtmlWebpackPlugin({
      filename: process.env.NODE_ENV === 'production'
        ? 'index.html'
        : config.build.index,
      template: 'index.dev.html',
      ...
    }),
{{< /codeblock >}} 
{{< codeblock "build/webpack.dev.conf.js" "js" "" "js" >}}...
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.dev.html',
      inject: true
    }),
{{< /codeblock >}} 
{{< codeblock "build/webpack.prod.conf.js" "js" "" "js" >}}...
  externals: {
    'vue': 'Vue',
    'vuex': 'Vuex',
    'vue-router': 'VueRouter',
    'axios': 'axios'
  },
  plugins: [
      ...
  ]
{{< /codeblock >}}
# 6.路由按需加载，使用热更新
增加`--hot`，如下：
{{< codeblock "package.json" "js" "" "js" >}}...
"scripts": {
    "dev": "webpack-dev-server --inline --progress --hot --config build/webpack.dev.conf.js",
    ...
  },
{{< /codeblock >}}
    将`import`路由的方式改成`const HelloWorld = r => require.ensure([], () => r(require('@/components/HelloWorld')), 'HelloWorld')`就可以实现按需加载了  

# 7.引入sass    