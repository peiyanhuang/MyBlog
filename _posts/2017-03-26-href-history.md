---
layout: post
title:  URL导航
date:   2017-03-26 20:38:00 +0800
categories: 2017
tag: JS
---

* content
{:toc}

### 1. Location

```
window.location === document.location //true

//location 

hash:"#Manual/Getting_Started/Creating_a_scene"
host:"threejs.org"
hostname:"threejs.org"
href:"https://threejs.org/docs/index.html?q=links#Manual/Getting_Started/Creating_a_scene"
origin:"https://threejs.org"
pathname:"/docs/index.html"
port:""
protocol:"https:"
search:"?q=links"
```

以上为 `https://threejs.org/docs/index.html?q=links#Manual/Getting_Started/Creating_a_scene` 的location对象

提取URL搜索字符串中的参数：

```
function urlArgs(){
	var arg = {};
	var query = location.search.substring(1);
	var paris = query.split('&');
	var len = paris.length;
	for(var i = 0; i < len; i++){
		var pos = paris[i].indexOf('=');
		if(pos == -1){
			continue;
		}
		var name = paris[i].substring(0, pos);
		var value = paris[i].substring(pos + 1);

		arg[name] = value;
	}
	return arg;
}
```

`Location` 的 `assign()` 和 `replace()` 方法都可以使窗口载入指定的URL的文档，不同的是 `replace()` 在载入文档之前会从浏览器历史中把当前文档删除。

也可以直接把URL赋给location来跳转：

```
location = "https://www.baidu.com";
```

还可以把相对URL赋给location，他们会相对当前URL来解析

```
location = "page.html"
```

`location.reload()` 会重载当前文档。

纯粹的片段标识符是相对URL的一种类型，他不会让浏览器载入新文档，只会使他滚动到文档的某个位置。`#top` 是个特殊的例子，如果文档中没有 `id` 为 `top` 的元素，他会让浏览器跳到文档的开始处。

`Location` 的分解属性也是可写的：

```
location.search = '?page=' + parama;
```

### 2. 历史记录管理

`window` 的 `history` 属性保存着该窗口的浏览历史。

`history` 对象：

```
history.back();		//与浏览器的后退按钮一样
history.forward();	//与浏览器的前进按钮一样
history.go(n);		//向前或向后n个记录
```

比较简单的管理历史记录的方法就是使用 `location.hash` 和 `hashChange` 事件 (支持 onhashchange IE8+、各种主流浏览器)。`onhashchange` 的触发条件是页面地址的 hash值发生改变，就会触发这个事件。

### 3. HTML5 History

HTML5 History API包括2个方法：history.pushState()和history.replaceState()，和1个事件：window.onpopstate。

`history.pushState(stateObject, title, url)`: 当一个web应用进入一个新的状态的时候，他会调用 `pushState` 方法将该状态添加到浏览器的游览历史记录里去。

第一个参数用于存储该url对应的状态对象

第二个参数是标题

第三个参数则是设定的url，表示当前状态的位置。一般设置为相对路径，如果设置为绝对路径时需要保证同源。

`replaceState`

该接口与`pushState`参数相同，含义也相同。唯一的区别在于`replaceState`是替换浏览器历史记录中的当前历史记录为设定的url。

当用户通过 “前进、后退” 按钮浏览保存的历史状态时，浏览器会触发 `window.onpopstate` 事件。其事件对象 `event.state` 包含传递给 `pushState()` 方法的状态对象的副本。


相关：

[History API 与浏览器历史堆栈管理](http://web.jobbole.com/87227/)