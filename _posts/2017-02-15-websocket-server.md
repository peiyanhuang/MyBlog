---
layout: post
title:  node搭建WebSocket服务
date:   2017-02-15 20:58:00 +0800
categories: Node
tag: Node
---

* content
{:toc}

### 1. 安装ws模块

	npm install --save ws

注意：`package.json` 中的`name`不要与模块有重名.

### 2. 创建Sever

在项目里面新建一个server.js，创建服务，指定8181端口。

```
var WebSocketServer  = require('ws').Server;
var ws = new WebSocketServer({port: 8181});

ws.on('connection', function(socket) {
	console.log('a user connected');
	socket.on('open', function() {
		// console.log('server is opened');
	});

	socket.on('close', function() {
		// console.log('server is closed');
	});

	socket.on('message', function(msg) {
		var obj = JSON.parse(msg);
		socket.name = obj.userid;

		if (obj.type === "opening") {					//opening message
			if (!onlineUsers.hasOwnProperty(obj.userid)) {
				onlineUsers[obj.userid] = obj.username;
				onlineCount++;
			}
			clients.push({"id":socket.name,"ws":socket});
			var temp = {
					"type":"opening", 
					"onlineUsers":onlineUsers, 
					"onlineCount":onlineCount, 
					"user":obj
				};

			wsSend(temp);
			console.log(obj.username + ' add the Home');

		}else if (obj.type === "message") {				//talk message
			wsSend(obj);
			console.log(obj.username + ' say ' + obj.content);

		}else if (obj.type === "closing") {				//closing message
			if (onlineUsers.hasOwnProperty(socket.name)) {
				delete onlineUsers[socket.name];
				onlineCount--;
			}
			var outMSg = {
					"type":'closing', 
					"onlineUsers":onlineUsers, 
					"onlineCount":onlineCount, 
					"user":obj
				};
			for (var i = 0; i < clients.length; i++) {
				if(clients[i].id === socket.name){
					clients.splice(i, 1);
				}
			}

			wsSend(outMSg);
			console.log(obj.username + " leave home");
		}
	});
});
```

消息发送函数:

```
function wsSend(cont) {			//send message to all user
    for (var i = 0; i < clients.length; i++) {
        var clientSocket = clients[i].ws;
        clientSocket.send(JSON.stringify(cont));
    }
}
```

注意：此处`send`发送消息只能发送`文本`或`二进制`数据，JSON数据要通过`JSON.stringify()`和`JSON.parse()`转换。

运用`ws`模块创建websocket[源码](https://github.com/peiyanhuang/learn/tree/master/nodejs-learn/node-webSocket-server/ws)

运用`socket.io`模块创建websocket[源码](https://github.com/peiyanhuang/learn/tree/master/nodejs-learn/node-webSocket-server/socket.io)
