---
layout: post
title:  客户端检测
date:   2016-12-08 20:58:00 +0800
categories: JS
tag: ES6
---

* content
{:toc}

检测 Web 客户端的手段很多，而且各有利弊，但最重要的是，不到万不得已，就不要使用客户端检测。只要能够找到更通用的方法，就应该优先采用更通用的方法。

### 1. 能力检测

最常用也最为人们广泛接受的客户端检测形式是能力检测（又称特性检测）。能力检测的目标不是识别特定的浏览器，而是识别浏览器的能力。采用这种方式不必顾忌特定的浏览器如何，只要确定浏览器支持特定的能力，就可以给出解决方案。

能力检测的基本模式如下：

```
//例如检测 WebSocket 的支持
if( window.WebSocket ){
    //...
}
```

**三个重要的概念：**

- **第一个概念是先检测达成目的的最常用的特性。先检测最常用的特性，可以保证代码最优化，因为在多数情况下都可以避免测试多个条件。**
- **第二个概念是必须测试实际要用到的特性。一个特性存在，不一定意味着另一个特性也存在。**
- **检测某个或某几个特性并不能够确定浏览器。**

**实际上，根据浏览器不同将能力组合起来是更可取的方式。如果知道自己的应用程序需要使用某些特定的浏览器特性，那么最好是一次性检测所有相关特性，而不要分别检测。**

```
//确定浏览器是否支持 Netscape 风格的插件
var hasNSPlugins = !!(navigator.plugins && navigator.plugins.length );

//确定浏览器是否具有 DOM1 级规定的能力
var hasDOM1 = !!(document.getElementById && document.createElement 
                && document.getElementByTagName);
```

在实际开发中，应该将能力检测作为确定下一步解决方案的依据，而不是用它来判断用户使用的是什么浏览器。

### 2.怪癖检测

与能力检测类似，怪癖检测（quirks detection）的目标是识别浏览器的特殊行为。
但与能力检测不同，怪癖检测是想要知道浏览器存在什么缺陷。这个通常要运行一小段代码，以确定某一特性不能正常工作。

例如，IE中存在的一个 bug ，即如果某个实例属性与标记为 [[DontEnum]] 的某个原型属性同名，那么该实例属性将不会出现在 fon-in 循环当中。可以使用如下代码来检测这种“怪癖”：

```
var hasDontEnumQuirk = function(){
    var o = { toString : function(){}};

    for( var prop in o){
        if( prop == "toString"){
            return false;
        }
    }
    return true;
}();
```

以上代码通过一个匿名函数来测试该“怪癖”，函数中创建了一个带有 toString() 方法的对象。在正确的 ECMAScript 实现中，toString 应该在 for-in 循环中作为属性返回。

一般来说，“怪癖”都是个别浏览器所独有的，而且通常被归为 bug。
在相关浏览器的信版本中，这些问题可能会也可能不会被修复。由于检测“怪癖”涉及运行代码，因此建议仅检测那些对你有直接影响的“怪癖”，而且最好在脚本一开始就执行此类检测，以便尽早解决问题。

### 3. 用户代理检测

用户代理检测通过检测用户代理字符串来确定实际使用的浏览器。

在每一次HTTP请求过程中，用户代理字符串是作为响应首部发送的，而该字符串可以通过 JavaScript 的 navigator.userAgent 属性访问。

[用户代理检测](https://github.com/peiyanhuang/learn/blob/master/js/clientDetection/browser.js)

IE 浏览器的 Ua 有点乱，具有如下特征：

- IE6-10 通过“ MSIE ”字段正确地标识自身版本信息；
- IE11 通过“ rv ”字段标识自身版本信息；

简单的只要判断游览器类型是否是IE，可以用IE特有的ActiveXObject()对象(Edge不支持)：
	
```	
	function isIE(){
		if (!!window.ActiveXObject || "ActiveXObject" in window){
	        return true;
		}
	    else{
	        return false;
	    }
	}
```