---
layout: post
title:  点击劫持(ClickJacking)
date:   2018-08-13 19:58:00 +0800
categories: 网络安全
tag: 其它
---

* content
{:toc}

### 1.什么是点击劫持技术

点击劫持 (Clickjacking) 技术又称为界面伪装攻击 (UI redress attack )，是一种视觉上的欺骗手段。攻击者使用一个或多个透明的 iframe 覆盖在一个正常的网页上，然后诱使用户在该网页上进行操作，当用户在不知情的情况下点击透明的 iframe 页面时，用户的操作已经被劫持到攻击者事先设计好的恶意按钮或链接上。攻击者既可以通过点击劫持设计一个独立的恶意网站，执行钓鱼攻击等；也可以与 XSS 和 CSRF 攻击相结合，突破传统的防御措施，提升漏洞的危害程度。

### 2.点击劫持原理及实例

攻击者实施攻击的一般步骤是：

1) 黑客创建一个网页利用iframe包含目标网站;
2) 隐藏目标网站，使用户无法察觉到目标网站存在;
3) 构造网页，诱骗用户点击特定按钮;
4) 用户在不知情的情况下点击按钮，触发执行恶意网页的命令。

2.1 目标网页的隐藏：

CSS 隐藏的原理是利用 CSS 技术控制网页内容显示的效果。其中 opacity 参数表示元素的透明度，取值范围为0~1，默认值为1表示不透明， 取值为0时元素在网页中完全透明显示。当设置目标 iframe 的opacity 属性小于或等于0.1，用户就无法看到含恶意代码的目标网页。

双 iframe 隐藏使用内联框架和外联框架。内联框架的主要功能是载入目标网页，并将目标网页定位到特定按钮或者链接。外联框架的主要功能是筛选，只显示内联框架中特定的按钮。

2.2 点击操作劫持

在成功隐藏目标网页后，攻击者下一个目标是欺骗用户点击特定的按钮，最简单实用的方法是使用社会工程学。例如，将攻击按钮外观设计成类似QQ消息的提示按钮，诱使用户点击从而触发攻击行为。另外一种思路是使用脚本代码以及其他技术增加用户点击特定按钮的概率。主要方法如JavaScript实现鼠标跟随技术、按键劫持 (Stroke jacking) 技术等。

2.3 拖拽（Drag and Drop）技术

主流的浏览器都有 drag-and-drop API 接口，供网站开发人员创建交互式网页。但是，这些 API 接口在设计时没有考虑很多的安全性问题，导致通过拖拽就可以实现跨域操作。利用拖拽技术，攻击者可以突破很多已有的安全防御措施，

利用拖拽技术，攻击者可以轻易将文本注入到目标网页。在实际实施过程中，攻击者欺骗用户选择输入框的内容，完成拖拽操作。另外一种方式是，通过浏览器的 API 接口将 iframe 中的内容拖拽到目标网页的 text area 中，攻击者就可以获得用户网页中存在的敏感信息。

2.4 触屏劫持

这是在移动设备触发的点击劫持事件。

### 3.点击劫持漏洞的防御

1. X-FRAME-OPTIONS 机制：该机制有3个选项：`DENY`、`SAMEORIGIN` 和 `ALLOW-FORM origin`。`DENY` 表示任何网页都不能使用 iframe 载入该网页，`SAMEORIGIN` 表示符合同源策略的网页可以使用 iframe 载入该网页。若值为 `ALLOW-FORM`，则可以定义允许 frame 加载的页面地址。
2. 使用 FrameBusting 代码：点击劫持攻击需要首先将目标网站载入到恶意网站中，使用 iframe 载入网页是最有效的方法。Web安全研究人员针对 iframe 特性提出 Frame Busting 代码，使用 JavaScript 脚本阻止恶意网站载入网页。如果检测到网页被非法网页载入，就执行自动跳转功能。Frame Busting代码是一种有效防御网站被攻击者恶意载入的方法，网站开发人员使用Frame Busting代码阻止页面被非法载入。需要指出的情况是，如果用户浏览器禁用JavaScript脚本，那么FrameBusting代码也无法正常运行。所以，该类代码只能提供部分保障功能。

业界普通采用这种做法:

```js
if(top.location != location)
{
　　top.location = self.location;
}
```

frame busting 一般由一个条件表达式和纠正动作(即跳转)组成。即将顶层页面导航到当前页面。frame busting 的条件判断语句:

```js
if (top != self)
if (top.location != self.location)
if (top.location != location)
if (parent.frames.length > 0)
if (window != top)
if (window.top !== window.self)
if (window.self != window.top)
if (parent && parent != window)
if (parent && parent.frames && parent.frames.length>0)
if((self.parent&&!(self.parent===self))&&(self.parent.frames.length!=0))
```

frame busting 的纠正动作代码:

 ```js
top.location = self.location
top.location.href = document.location.href
top.location.href = self.location.href
top.location.replace(self.location)
top.location.href = window.location.href
top.location.replace(document.location)
top.location.href = window.location.href
top.location.href = "URL"
document.write('')
top.location = location
top.location.replace(document.location)
top.location.replace('URL')
top.location.href = document.location
top.location.replace(window.location.href)
top.location.href = location.href
self.parent.location = document.location
parent.location.href = self.document.location
top.location.href = self.location
top.location = window.location
top.location.replace(window.location.pathname)
window.top.location = window.self.location
setTimeout(function(){document.body.innerHTML='';},1);
window.self.onload = function(evt){document.body.innerHTML='';}
var url = window.location.href; top.location.replace(url)
```

参考：

* [frame busting 各种姿势，防护总结](https://zhuanlan.zhihu.com/p/27310909)