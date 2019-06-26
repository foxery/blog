---
title: "关于vue-router的一些使用心得"
date: 2018-06-07T11:45:40+08:00
categories:
- program
- vue
tags:
- vue-router
keywords:
- watch
- title
thumbnailImage: /img/cover2.jpg
---

<!--more-->

<!-- toc -->

# 1.watch $route只有在改变当前动态路由时才会触发，进行其他路由跳转时并不会触发

# 2.给每个页面设置单独的title  
{{< codeblock "router/index.js" "js" "" "" >}}export default new Router({
    routes: [
    {          
      path: '/',
      name: 'Entrance',
      component: Entrance,
      meta: {
        title: '首页入口'
      }
    }
  ]
})
{{< /codeblock >}}
{{< codeblock "main.js" "js" "" "" >}}import router from './router'

router.beforeEach((to, from, next) => {
  /* 路由发生变化修改页面title */
  if (to.meta.title) {
    document.title = to.meta.title
  }
  next()
})
{{< /codeblock >}}
