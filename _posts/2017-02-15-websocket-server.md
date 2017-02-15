---
layout: post
title:  node搭建WebSocket服务
date:   2017-02-15 20:58:00 +0800
categories: JS
tag: Node
---

* content
{:toc}

### 1. 安装ws模块

	npm install --save ws

注意：`package.json` 中的`name`不要与模块有重名.

### 2. 创建Sever

在项目里面新建一个server.js，创建服务，指定8181端口，将收到的消息log出来。