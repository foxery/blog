---
title: "Hugo博客的创建与运行"
date: 2018-02-07T10:10:49+08:00
categories:
- technique
- blog
tags:
- 博客
keywords:
- hugo
thumbnailImage: /img/cover1.jpg
---

<!--more-->
### 一、安装

* [下载地址](https://github.com/gohugoio/hugo/releases)
* 去高级系统配置里将运行文件加入环境变量
* `$ hugo.exe version`
* [官方QuickStart](http://gohugo.io/getting-started/quick-start/)

### 二、创建

#### 1.创建项目
`$ hugo new site your_project_name`

#### 2.选择主题

``` javascript
$ cd your_project_name
$ git init  
$ git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke  
# Edit your config.toml configuration file  
# and add the Ananke theme.  
$ echo 'theme = "ananke"' >> config.toml`
```

* 可以从[这里](https://themes.gohugo.io/)选择你想要的主题  
然后 :  
`$ cd themes`  
`$ git clone your_themes_url your_themes_name`  
* 按照exampleSite里的config.toml进行修改  

### 三、运行
`$ hugo server -D`  

### 四、创建文章  
`$ hugo new blog/your-project-name.md`  

### 五、部署
 `$ cd your_project_name`  
 `$ hugo`  

然后会生成一个静态文件夹public,将此文件发布出去即可。