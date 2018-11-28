---
title: "vue单页应用首屏加载优化"
date: 2018-10-17T10:10:17+08:00
categories:
- technique
- vue
tags:
- vue
- optimization
keywords:
- 首屏优化
thumbnailImage: /img/cover2.jpg
---

<!--more-->

vue单页应用打包之后，渲染加载时由于是一次性加载整个应用，所以首次加载会特别慢，会出现长时间的白屏，极大影响用户体验，因此，我尝试了多种优化方法。
<!-- toc -->
# 1.按需加载
MintUI、ElementUI等UI框架都支持按需加载，直接在main.js内将需要用到的组件一一注册，或者只在需要用到的页面局部引用。  

修改前：  

{{< codeblock "main.js" "js" "" "js" >}}import MintUI from 'mint-ui'
import 'mint-ui/lib/style.css'

Vue.use(MintUI)
{{< /codeblock >}}

修改后：  

{{< codeblock "main.js" "js" "" "js" >}}import { Popup, Picker, Spinner, Swipe, SwipeItem, Lazyload } from 'mint-ui'

Vue.use(Lazyload);
Vue.component(Popup.name, Popup);
Vue.component(Picker.name, Picker);
Vue.component(Spinner.name, Spinner);
Vue.component(Swipe.name, Swipe);
Vue.component(SwipeItem.name, SwipeItem);
{{< /codeblock >}}  

# 2.异步组件
在路由定义中，将应用拆分为多个小模块，动态解析组件。

修改前：  

{{< codeblock "router/index.js" "js" "" "js" >}}import Index from '../views/index'

...
{
    path: '/index',
    name: 'Index',
    component: Index
}
...
{{< /codeblock >}}  

修改后：  

{{< codeblock "router/index.js" "js" "" "js" >}}const Index = r => require.ensure([], () => r(require('../views/index')), 'Index')

...
{
    path: '/index',
    name: 'Index',
    component: Index
}
...
{{< /codeblock >}}  

# 3.使用CDN
打包时，把vue、vuex、vue-router、axios等，换用国内的bootcdn 直接引入到根目录的index.html中。

在webpack设置中添加externals，忽略不需要打包的库。  

{{< codeblock "build/webpack.prod.conf.js" "js" "" "js" >}}...
externals: {
    'vue': 'Vue',
    'vuex': 'Vuex',
    'vue-router': 'VueRouter',
    'axios': 'axios'
  }
...
{{< /codeblock >}}

{{< codeblock "index.html" "html" "" "html" >}}<script src="//cdn.bootcss.com/vue/2.2.5/vue.min.js"></script>  
<script src="//cdn.bootcss.com/vue-router/2.3.0/vue-router.min.js"></script>
<script src="//cdn.bootcss.com/vuex/2.2.1/vuex.min.js"></script>  
<script src="//cdn.bootcss.com/axios/0.15.3/axios.min.js"></script>
{{< /codeblock >}}  

相应的，也就不需要import引用了，不然据说还是会打包进去（？这个没有深入研究对比过）  

修改前：  
{{< codeblock "store/index.js" "js" "" "js" >}}import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex)

//router/index.js
import Router from 'vue-router'
Vue.use(Router)
{{< /codeblock >}}

修改后：  
{{< codeblock "store/index.js" "js" "" "js" >}}//import Vue from 'vue';
//import Vuex from 'vuex';

//Vue.use(Vuex)

//router/index.js
//import Router from 'vue-router'
//Vue.use(Router)
{{< /codeblock >}}

# 4.在index.html页面加上loading，避免出现长时间的白屏状态  
{{< codeblock "index.html" "html" "" "html" >}}...
<div id="app">
//loading
</div>
...
{{< /codeblock >}}  

当然，能实现骨架屏最好，还可以用SSR服务端渲染，但是挺复杂的，我没能实现

# 5.开启Gzip
gzip大约能减少20%的大小，不过需要服务端配合开启，服务端若开启了gzip，在`Response Headers`里可以看到`Content-Encoding:gzip`

在开启vue的gzip之前，需要先安装依赖`compression-webpack-plugin`，但是注意，默认的安装版本是2.0，可能是版本太高了，最后build的时候会报错，所以安装低版本的就可以了，即运行`npm install --save-dev compression-webpack-plugin@1.1.11`

然后
{{< codeblock "config/index.js" "js" "" "js" >}}...
build:{
    ...
    productionGzip: true,
    productionGzipExtensions: ['js', 'css'],
    ...
}
...
{{< /codeblock >}}

# 6.图片懒加载  
使用`mint-ui`中的`Lazyload`可实现。

以上就是我优化首屏加载速度的全部内容。