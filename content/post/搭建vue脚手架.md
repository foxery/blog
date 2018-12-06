---
title: "搭建vue脚手架"
date: 2018-02-07T10:13:59+08:00
categories:
- technique
- vue
tags:
- vue
- vue-cli
keywords:
- vue-cli
thumbnailImage: /img/cover3.jpg
---

<!--more-->
### 一、环境搭建
#### 1.下载 [git](https://git-scm.com/downloads)
#### 2.安装node.js
    从 [官网](https://nodejs.org/en/download/) 下载 

#### 3.安装淘宝镜像
    打开命令行工具
    `npm install -g cnpm --registry= https://registry.npm.taobao.org`
#### 4.安装webpack
    打开命令行工具 `npm install webpack -g`
#### 5.安装vue-cli脚手架构建工具
    打开命令行工具 `npm install vue-cli -g`

### 二、构建项目
    1. `vue init webpack your_project_name`  
        ![](/img/vue-cli.png)  
    2. `cd your_project_name`  
    3. `cnpm install`  
        因为自动构建过程中已存在package.json文件，所以这里直接安装依赖就行
    4. `cnpm install vue-router vue-resource --save`  
        安装 vue 路由模块 vue-router 和网络请求模块 vue-resource(现在的脚手架会自动安装vue-router，所以这一步可以省略)
    5. `npm run dev`  
        启动项目

### 三、遇到的问题
    1.错误:
        ? Project name (project) vue-cli · inquirer.prompt(...).then is not a function
      解决：
        用cnpm install -g vue-cli，重新安装就可以了          