---
layout: post
title:  WebSocket及轮询
date:   2016-12-07 20:58:00 +0800
categories: JS
tag: HTML5
---

* content
{:toc}

### 1. WebSocket

先说说轮询。

#### ajax轮询

ajax轮询的原理非常简单，客户端定时向服务器发送Ajax请求，服务器接到请求后马上返回响应信息并关闭连接。

#### long poll(长轮询)

long poll 其实原理跟 ajax轮询差不多，都是采用轮询的方式，不过采取的是阻塞模型（一直打电话，没收到就不挂电话），也就是说，客户端发起连接后，如果没消息，就一直不返回Response给客户端。直到有新消息才返回响应信息才关闭连接。客户端处理完响应信息后再向服务器发送新的请求，周而复始。

#### WebSocket 机制

WebSocket 是 HTML5一种新的协议。它实现了浏览器与服务器全双工通信，能更好的节省服务器资源和带宽并达到实时通讯，它建立在 TCP 之上，同 HTTP 一样通过 TCP 来传输数据，但是它和 HTTP 最大不同是：

- WebSocket 是一种双向通信协议，在建立连接后，WebSocket 服务器和 Browser/Client Agent 都能主动的向对方发送或接收数据，就像 Socket 一样；
- WebSocket 需要类似 TCP 的客户端和服务器端通过握手连接，连接成功后才能相互通信。

相对于传统 HTTP 每次请求-应答都需要客户端与服务端建立连接的模式，WebSocket 是类似 Socket 的 TCP 长连接的通讯模式，一旦 WebSocket 连接建立后，后续数据都以帧序列的形式传输。在客户端断开 WebSocket 连接或 Server 端断掉连接前，不需要客户端和服务端重新发起连接请求。

在客户端，new WebSocket 实例化一个新的 WebSocket 客户端对象，连接类似 ws://yourdomain:port/path 的服务端 WebSocket URL，WebSocket 客户端对象会自动解析并识别为 WebSocket 请求，从而连接服务端端口，执行双方握手过程，客户端发送数据格式类似：

	//WebSocket 客户端连接报文
	GET /chat HTTP/1.1
	Host: server.example.com
	Upgrade: websocket
	Connection: Upgrade
	Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
	Sec-WebSocket-Protocol: chat, superchat
	Sec-WebSocket-Version: 13
	Origin: http://example.com

可以看到，客户端发起的 WebSocket 连接报文类似传统 HTTP 报文。

`Upgrade：websocket` 参数值表明这是 WebSocket 类型请求。 

`Sec-WebSocket-Key` 是一个Base64 encode的值，这个是浏览器随机生成的，要求服务端必须返回一个对应加密的“Sec-WebSocket-Accept”应答，否则客户端会抛出“Error during WebSocket handshake”错误，并关闭连接。

`Sec_WebSocket-Protocol` 是一个用户定义的字符串，用来区分同URL下，不同的服务所需要的协议。

`Sec-WebSocket-Version` 是告诉服务器所使用的Websocket Draft（协议版本）。

服务端收到报文后返回的数据格式类似：

	//WebSocket 服务端响应报文
	HTTP/1.1 101 Switching Protocols
	Upgrade: websocket
	Connection: Upgrade
	Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
	Sec-WebSocket-Protocol: chat

`Sec-WebSocket-Accept` 这个则是经过服务器确认，并且加密过的 Sec-WebSocket-Key，后返回客户端的。

### 2. WebSocket 客户端实现

	var ws = new WebSocket(“ws://localhost:8080”);
	ws.onopen = function()
	{
	  console.log(“open”);
	  ws.send(“hello”);
	};
	ws.onmessage = function(evt)
	{
	  console.log(evt.data)
	};
	ws.onclose = function(evt)
	{
	  console.log(“WebSocketClosed!”);
	};
	ws.onerror = function(evt)
	{
	  console.log(“WebSocketError!”);
	};

1. `var ws = new WebSocket(“ws://localhost:8080”)`

申请一个WebSocket对象，参数是需要连接的服务器端的地址，同http协议使用http://开头一样，WebSocket协议的URL使用ws://开头，另外安全的WebSocket协议使用wss://开头。

2. `ws.onopen = function() { console.log(“open”)}`

当websocket创建成功时，即会触发onopen事件

`ws.send(“hello”)`

用于叫消息发送到服务端 

3. `ws.onmessage = function(evt) { console.log(evt.data) }`

当客户端收到服务端发来的消息时，会触发onmessage事件，参数evt.data中包含server传输过来的数据

4. `ws.onclose = function(evt) { console.log(“WebSocketClosed!”); }`

当客户端收到服务端发送的关闭连接的请求时，触发onclose事件

5. `ws.onerror = function(evt) { console.log(“WebSocketError!”); }`

如果出现连接，处理，接收，发送数据失败的时候就会触发onerror事件
