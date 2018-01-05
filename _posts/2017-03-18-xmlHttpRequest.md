---
layout: post
title:  XMLHttpRequest 详解
date:   2017-03-18 20:38:00 +0800
categories: JS
tag: JS
---

* content
{:toc}

### 1. XMLHttpRequest的发展历程

我们通常将 `Ajax` 等同于 `XMLHttpRequest`，但细究起来它们两个是属于不同的概念。

两者的关系可以理解为我们使用 `XMLHttpRequest` 对象来发送一个 `Ajax` 请求。

`XMLHttpRequest` 一开始只是微软浏览器提供的一个接口，后来各大浏览器纷纷效仿也提供了这个接口，再后来W3C对它进行了标准化，提出了 `XMLHttpRequest` 标准。`XMLHttpRequest` 标准又分为 `Level 1` 和 `Level 2`。

老版本的XMLHttpRequest对象有以下几个缺点：

- 只支持文本数据的传送，无法用来读取和上传二进制文件。
- 传送和接收数据时，没有进度信息，只能提示有没有完成。
- 受到"同域限制"（Same Origin Policy），只能向同一域名的服务器请求数据。

`Level 2`对`Level 1` 进行了改进，`XMLHttpRequest Level 2`中新增了以下功能：

- 可以发送跨域请求，在服务端允许的情况下；
- 支持发送和接收二进制数据；
- 新增formData对象，支持发送表单数据；
- 发送和获取数据时，可以获取进度信息；
- 可以设置请求的超时时间.

使用XMLHttpRequest发送Ajax请求的简单示例代码

```
function sendAjax() {
	//构造表单数据
	var formData = new FormData();
	formData.append('username', 'johndoe');
	formData.append('id', 123456);

	//创建xhr对象 
	var xhr = new XMLHttpRequest();

	//设置xhr请求的超时时间
	xhr.timeout = 3000;

	//设置响应返回的数据格式
	xhr.responseType = "text";

	//创建一个 post 请求，采用异步
	xhr.open('POST', '/server', true);

	//注册相关事件回调处理函数
	xhr.onload = function(e) { 
		if(this.status == 200||this.status == 304){
		    alert(this.responseText);
		}
	};
	xhr.ontimeout = function(e) { ... };
	xhr.onerror = function(e) { ... };
	xhr.upload.onprogress = function(e) { ... };

	//发送数据
	xhr.send(formData);
}
```

#### 1.1 XMLHttpRequest对象的主要属性

- `xhr.readyState`：XMLHttpRequest对象的状态，等于4表示数据已经接收完毕。

```
0	UNSENT (未打开)	open()方法还未被调用.
1	OPENED  (未发送)	send()方法还未被调用.
2	HEADERS_RECEIVED (已获取响应头)	send()方法已经被调用, 响应头和响应状态已经返回.
3	LOADING (正在下载响应体)	响应体下载中; responseText中已经获取了部分数据.
4	DONE (请求完成)	整个请求过程已经完毕.
```

- `xhr.status`：服务器返回的状态码，等于200表示一切正常。

- `xhr.statusText`：服务器返回的状态信息,包含一个状态码和原因短语 (例如 "200 OK"). 只读.

- `xhr.responseType`: 设置该值能够改变响应类型。就是告诉服务器你期望的响应格式。

```
"" (空字符串)		字符串(默认值)
"arraybuffer"		ArrayBuffer(数组)
"blob"				Blob(二进制)
"document"			Document(xml, html)
"json"				JavaScript 对象，解析自服务器传递回来的JSON 字符串。
"text"				字符串
```

- `xhr.responseText`：服务器返回的文本数据

- `xhr.responseXML`：服务器返回的XML格式的数据

- `xhr.withCredentials`: 表明在进行跨站(cross-site)的访问控制(Access-Control)请求时，是否使用认证信息(例如cookie或授权的header)。 默认为 false。

#### 1.2 设置request header

```
var xhr = new XMLHttpRequest();
xhr.open('GET', url);

xhr.setRequestHeader('X-Test', 'one');
xhr.setRequestHeader('X-Test', 'two');
// 最终request header中"X-Test"为: one, two

xhr.send();
```

**注意点**：

1.方法的第一个参数 header 大小写不敏感，即可以写成content-type，也可以写成Content-Type，甚至写成content-Type;

2.Content-Type的默认值与具体发送的数据类型有关

3.`setRequestHeader`必须在`open()`方法之后，`send()`方法之前调用，否则会抛错；

4.`setRequestHeader`可以调用多次，最终的值不会采用覆盖override的方式，而是采用追加append的方式。

#### 1.3 获取response header

`xhr`提供了2个用来获取响应头部的方法：`getAllResponseHeaders` 和 `getResponseHeader` 。前者是获取 `response` 中的所有 `header` 字段，后者只是获取某个指定 `header` 字段的值。另外，`getResponseHeader(header)`的 `header` 参数不区分大小写。

这两个方法看着简单，但有不少限制：

1.W3C的 xhr 标准中做了限制，规定客户端无法获取 `response` 中的 `Set-Cookie`、`Set-Cookie2`这2个字段，无论是同域还是跨域请求；

2.W3C 的 cors 标准对于跨域请求也做了限制，规定对于跨域请求，客户端允许获取的`response header`字段只限于`simple response header`和`Access-Control-Expose-Headers`

`simple response header`:包括的 `header` 字段有：`Cache-Control`,`Content-Language`,`Content-Type`,`Expires`,`Last-Modified`,`Pragma`;

`Access-Control-Expose-Headers`:首先得注意是`Access-Control-Expose-Headers`进行跨域请求时响应头部中的一个字段，对于同域请求，响应头部是没有这个字段的。这个字段中列举的 header 字段就是服务器允许暴露给客户端访问的字段。

#### 1.4 指定xhr.response的数据类型

有2种方法可以实现，一个是`level 1`就提供的 `overrideMimeType()` 方法，另一个是`level 2`才提供的 `xhr.responseType` 属性。

```
var xhr = new XMLHttpRequest();
//向 server 端获取一张图片
xhr.open('GET', '/path/to/image.png', true);

//将响应数据按照纯文本格式来解析，字符集替换为用户自己定义的字符集
xhr.overrideMimeType('text/plain; charset=x-user-defined');

xhr.onreadystatechange = function(e) {
  	if (this.readyState == 4 && this.status == 200) {
    	//通过 responseText 来获取图片文件对应的二进制字符串
    	var binStr = this.responseText;
    	//逐个将字节还原为二进制数据
    	for (var i = 0, len = binStr.length; i < len; ++i) {
      		var c = binStr.charCodeAt(i);
      		var byte = c & 0xff; 
    	}
  	}
};

xhr.send();
```

或者

```
	xhr.responseType = 'blob';
```

#### 1.5 设置请求的超时时间

```
xhr.timeout = 3000;
```

什么时候才算是请求开始 ？

`xhr.onloadstart` 事件触发的时候，也就是你调用 `xhr.send()` 方法的时候开始计算。`xhr.loadend` 事件触发的时候算结束。

当`xhr`为同步请求时，有如下限制：

- `xhr.timeout`必须为 0;
- `xhr.withCredentials` 必须为 false;
- `xhr.responseType` 必须为"",（注意置为"text"也不允许）

#### 1.6 FormData对象

`ajax`操作往往用来传递表单数据。为了方便表单处理，HTML 5新增了一个`FormData`对象，可以模拟表单。

首先，新建一个FormData对象。

```
var formData = new FormData();
```

然后，为它添加表单项

```
formData.append('username', '张三');
formData.append('id', 123456);
```

最后，直接传送这个FormData对象。这与提交网页表单的效果，完全一样。

```
xhr.send(formData);
```

FormData对象也可以用来获取网页表单的值:

```
var form = document.getElementById('myform');
var formData = new FormData(form);

formData.append('secret', '123456'); // 可以继续添加一个表单项
```

还可以用`FormData` [上传文件](https://peiyanhuang.github.io/MyBlog/2017/03/20/H5-file/#文件上传)

#### 1.7 进度信息

可以通过 `onprogress` 事件来实时显示进度，默认情况下这个事件每 `50ms` 触发一次。需要注意的是，上传过程和下载过程触发的是不同对象的 `onprogress` 事件：

下载的`progress`事件属于`XMLHttpRequest`对象，上传的`progress`事件属于`XMLHttpRequest.upload`对象。

```
xhr.onprogress = updateProgress; 			//下载
xhr.upload.onprogress = updateProgress;		//上传

function updateProgress(event) {
	if (event.lengthComputable) {
		var percentComplete = event.loaded / event.total;
	}
}
```

上面的代码中，`event.total` 是需要传输的总字节，`event.loaded` 是已经传输的字节。如果 `event.lengthComputable` 不为真，则 `event.total` 等于 `0`。

与`progress`事件相关的，还有其他五个事件，可以分别指定回调函数：

### 2. Events

|  事件  			|	触发条件  |
| :----  			| :----      |
|onreadystatechange |	每当xhr.readyState改变时触发；但xhr.readyState由非0值变为0时不触发。|
|onloadstart		| 调用xhr.send()方法后立即触发，若xhr.send()未被调用则不会触发此事件。|
|onprogress			| xhr.upload.onprogress在上传阶段(即xhr.send()之后，xhr.readystate=2之前)触发，每50ms触发一次；xhr.onprogress在下载阶段（即xhr.readystate=3时）触发，每50ms触发一次。|
|onload				| 当请求成功完成时触发，此时xhr.readystate=4 |
|onloadend			| 当请求结束（包括请求成功和请求失败）时触发 | 
|onabort			| 当调用xhr.abort()后触发 |
|ontimeout			| xhr.timeout不等于0，由请求开始即onloadstart开始算起，当到达xhr.timeout所设置时间请求还未结束即onloadend，则触发此事件。 |
|onerror			| 在请求过程中，若发生Network error则会触发此事件（若发生Network error时，上传还没有结束，则会先触发xhr.upload.onerror，再触发xhr.onerror；若发生Network error时，上传已经结束，则只会触发xhr.onerror）。注意，只有发生了网络层级别的异常才会触发此事件，对于应用层级别的异常，如响应返回的xhr.statusCode是4xx时，并不属于Network error，所以不会触发onerror事件，而是会触发onload事件。|

`XMLHttpRequest.abort()` 方法将终止请求，如果该请求已被发出。当一个请求被终止，它的 `readyState` 属性将被置为 `0`（ UNSENT ），但是并不触发 `readystatechange` 事件。

参考：

[ W3C的xhr 标准](https://www.w3.org/TR/XMLHttpRequest/)

[MDN XMLHttpRequest](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest#Events)

[阮一峰 XMLHttpRequest Level 2 使用指南](http://www.ruanyifeng.com/blog/2012/09/xmlhttprequest_level_2.html)