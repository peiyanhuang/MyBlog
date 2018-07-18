---
layout: post
title:  EventSource(服务端推送)
date:   2018-07-18 19:58:00 +0800
categories: JS
tag: JS
---

* content
{:toc}

### 1.服务端推送

在应用层的 HTTP 协议实现中，请求是由客户端发起的。服务端主动发起请求向客户端发送信息。当前解决服务端推送的方案有这几个：

1. 客户端长轮询
2. websocket双向连接

`eventSource` 是用来解决 web 上服务器端向客户端推送消息的问题的。不同于 ajax 轮询的复杂和 websocket 的资源占用过大，`eventSource` 是一个轻量级的，易使用的消息推送api。

### 2.EventSource简析

#### 2.1 浏览器端

浏览器端，需要创建一个 EventSource 对象，并且传入一个服务端的接口 URI 作为参数。

    var eventSource = new EventSource(URI);

一旦成功初始化了一个事件源,就可以开始监听它的消息了:

```js
eventSource.onmessage = function(e) {
  var newElement = document.createElement("li");
  newElement.innerHTML = "message: " + e.data;
  eventList.appendChild(newElement);
}
```

默认 EventSource 对象通过侦听 `message` 事件获取服务端传来的消息，`open` 事件则在 http 连接建立后触发，`error` 事件会在通信错误（连接中断、服务端返回数据失败）的情况下触发。同时，EventSource 规范允许服务端指定自定义事件，客户端侦听该事件即可。

上面的代码监听了那些从服务器发送来的所有没有指定事件类型的消息(没有 `event` 字段的消息)。也可以使用 `addEventListener()` 方法来监听其他类型的事件:

```js
eventSource.addEventListener("ping", function(e) {
  var newElement = document.createElement("li");
  var obj = JSON.parse(e.data);
  newElement.innerHTML = "ping at " + obj.time;
  eventList.appendChild(newElement);
}, false);
```

这段代码也类似，只是只有在服务器发送的消息中包含一个值为 `ping` 的 `event` 字段的时候才会触发对应的处理函数,也就是将 `data` 字段的字段值解析为 JSON 数据,然后在页面上显示出所需要的内容.

#### 2.2 服务器端

服务器端发送的响应内容应该使用值为 `text/event-stream` 的MIME类型。

服务端返回数据需要特殊的格式，它分为四种消息类型：

```
event,
data,
id,
retry
```

其中，`event` 指定自定义消息的名称，如 event: customMessage

`data` 消息的数据字段。如果该条消息包含多个 data 字段，则客户端会用换行符把它们连接成一个字符串来作为字段值

`id` 事件ID，会成为当前 EventSource 对象的内部属性的属性值

`retry` 一个整数值，指定了重新连接的时间(单位为毫秒)，如果该字段值不是整数，则会被忽略

每个字段都有名称，紧接着有个 `:`。当出现一个没有名称的字段而只有 `:` 时，这就会被服务端理解为”注释“，并不会被发送至浏览器端，如 `:commision`。

### 参考：

[服务端事件EventSource揭秘](https://www.cnblogs.com/accordion/p/7764460.html)

[使用服务器发送事件](https://developer.mozilla.org/zh-CN/docs/Server-sent_events/Using_server-sent_events)