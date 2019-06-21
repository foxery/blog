---
title: "混合开发中与app的交互方案"
date: 2019-02-21T10:25:24+08:00
categories:
- technique
- webview
tags:
- 混合开发
- 交互
keywords:
- WebViewJavascriptBridge
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

<!-- toc -->

# 1.js调用app，无返回值

这里Android和iOS的方式是一样的，都是使用很简单的schema 的方式。

具体步骤如下：

    1. 前端与客户端约定好一个 schema 的名称，比如返回就叫做 goback
    2. 前端通过 location.href="goback://" 或 iframe.src="goback://" 的方式发起请求
    3. 客户端拦截 goback 这个 schema，执行返回逻辑
    4. 如果需要向客户端传递参数，直接在该 goback:// 后面拼接参数即可，客户端可以进行解析

# 2.使用 WebViewJavascriptBridge,有返回值

iOS客户端与前端进行交互时，如果前端需要获取iOS端传回的返回值，或者iOS端需要调用前端的js方法，就可以通过`WebViewJavascriptBridge`来实现。

具体以vue为例：

1. 创建 src/util/bridge.js 文件，用于封装 WebViewJavascriptBridge
    {{< codeblock "src/util/bridge.js" "js" "" "" >}}function setupWebViewJavascriptBridge(callback) {
	  if (window.WebViewJavascriptBridge) {
	    return callback(window.WebViewJavascriptBridge)
	  }
	  if (window.WVJBCallbacks) {
	    return window.WVJBCallbacks.push(callback)
	  }
	  window.WVJBCallbacks = [callback]
	  let WVJBIframe = document.createElement('iframe');
	  WVJBIframe.style.display = 'none'
	  WVJBIframe.src = 'https://__bridge_loaded__'
	  document.documentElement.appendChild(WVJBIframe);
	  setTimeout(() => {
	    document.documentElement.removeChild(WVJBIframe)
	  }, 0)
	}
	export default {
	  callhandler(name, data, callback) {
	    setupWebViewJavascriptBridge(function (bridge) {
	      bridge.callHandler(name, data, callback)
	    })
	  },
	  registerhandler(name, callback) {
	    setupWebViewJavascriptBridge(function (bridge) {
	      bridge.registerHandler(name, function (data, responseCallback) {
	        callback(data, responseCallback)
	      })
	    })
	  }
	}
    {{< /codeblock >}}

2. 在 main.js 中引入该文件
{{< codeblock "main.js" "js" "" "" >}}import Bridge from './util/bridge.js'
Vue.prototype.$bridge = Bridge
{{< /codeblock >}}

3.在组件内调用

1. 前端调用iOS端方法（第一个参数为前端和iOS端约定好的方法名，第二个参数为前端传给iOS端的参数，第三个参数为iOS端返回的数据）
{{< codeblock "demo.vue" "vue" "" "" >}}this.$bridge.callhandler('JSCallApp',params, (data) => {
    alert('JSCallApp:', data);
});
{{< /codeblock >}}
2. iOS端调用前端js方法
{{< codeblock "demo.vue" "vue" "" "" >}}this.$bridge.registerhandler('AppCallJs', (data, responseCallback) => {
    alert('AppCallJs:', data)
    responseCallback(data)
});
{{< /codeblock >}}

# 3.客户端调用前端js方法
Android和iOS有一个简单的调用方法，就是前端只要在js文件里定义好一个函数，客户端就可以调用

具体步骤如下：  

  1. 新建一个 share.js 文件，代码如下（其中）：

	{{< codeblock "share.js" "js" "" "" >}}var share_obj = {
	"img_url": "",//分享图标
	"title": "",//分享标题
	"page_type": "",//分享内容
	"share_url": "",//分享链接
	"sms_text": ""//短信发送文本
};

//分享对象-安卓
function share_android() {
	var param = share_param();
	param.title = param.title ? param.title : '';
	param.page_type = param.page_type ? param.page_type : '';
	param.share_url = param.share_url ? param.share_url : '';
	param.img_url = param.img_url ? param.img_url : '';
	param.sms_text = param.sms_text ? param.sms_text : '';
	window.webView.info(param.title, param.page_type, param.share_url, param.img_url, param.sms_text);
}

//分享对象-ios
function share_ios() {
	var param = share_param(),
	str = '';
	if (typeof param.share_url === 'undefined') {
		str = str + param.title + '#' + param.img_url;
	} else {
		str = str + param.title + '#' + param.img_url + '#' + param.share_url;
	}

	if (typeof param.page_type != 'undefined') {
		str = str + '#' + param.page_type;
	}
	if (param.sms_text) {
		str += '#' + param.sms_text;
	}
	return str;
}

function share_param() {
	//提前将分享信息存储于localStorage中
	var shareInfo = JSON.parse(localStorage.getItem("share"));
	share_obj.title = shareInfo.title;
	share_obj.share_url = shareInfo.share_url;
	share_obj.img_url = shareInfo.img_url;
	share_obj.page_type = shareInfo.page_type;
	share_obj.sms_text = shareInfo.sms_text;
	return share_obj;
}
	{{< /codeblock >}}

2. ios端调用`share_ios()`,android端调用`share_android`即可  

# 4.native注入js方法，有返回值  
1.ios端可以通过提前注入的方式，将信息注入到webview中,web前端可以通过约定好的方法获取到需要的信息  

		1.ios端与web前端定义好一个方法名,如 `getInfo`
		2.ios端将信息注入
		3.前端调用时，直接`window.getInfo`即可，或者`window.getInfo()`,返回值约定json格式或者object格式都行,怎么调用，返回值是什么，都由和ios的约定决定

2.android原理相似，调用方式也类似	  

# 5.js调用native，传参  
当前端与客户端进行交互且需要前端传递数据给客户端时，方法3比较被动，且数据不好维护跟踪，因此可以使用这种方法  

1.ios端调用方法  
		`window.webkit.messageHandlers.方法名.postMessage(传递参数)`   
		例如:`window.webkit.messageHandlers.share.postMessage(JSON.stringify(shareInfo))`  

2.android  
		`let jsonTemp = JSON.stringify(shareInfo);window.webView.share(jsonTemp);`  
		`这里需要注意下，android端必须先转换成json格式再赋值，否则android端接收到的参数会是undefined`
