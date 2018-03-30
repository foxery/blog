---
title: "在vue项目中使用sass"
date: 2018-03-30T09:00:13+08:00
categories:
- technique
- vue
tags:
- vue
- sass
keywords:
- vue
- sass
thumbnailImage: /img/cover3.jpg
---

<!--more-->
### 安装
1. 安装sass的依赖包  
    ``` 
    cnpm i node-sass --save-dev
    cnpm i sass-loader --save-dev
    ```  
    使用save会在package.json中自动添加
2. 添加配置  
    找到build文件夹下的`webpack.base.conf.js`,找到`rule`  
    ```
    module: {
        rules: [
                    //...默认及其他
                    {
                        test: /\.scss$/,
                        loaders: ["style", "css", "sass"]
                    }
                ]
            }
    ```     
### 引用公共scss文件  
1. 方法一  
据说是官方推荐，还没试过  
    1. 下载依赖  
    `cnpm i sass-resources-loader --save-dev`  
    2. 修改配置  
    找到`build/utils.js`  
      
    ```
    scss: generateLoaders('sass').concat(
        {
            loader: 'sass-resources-loader',
            options: {
            resources: path.resolve(__dirname, '../src/assets/scss/base.scss')  //注意自己的路径
            }
        }
    )
    ```  
2. 方法二  
    在`main.js`中引入，直接`import '../src/assets/scss/base.scss'`   
    但是这时候发现引入报错  
    发现只要把`webpack.base.conf.js`新添加的配置去掉即可  
    即以下这段：  
    ```
    {
        test: /\.scss$/,
        loaders: ["style", "css", "sass"]
    }
    ```  
### sass重置文件  
    [normalize.scss](/resource/_normalize.scss)