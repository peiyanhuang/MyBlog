---
layout: post
title:  MVC、MVP和MVVM
date:   2017-02-14 18:58:00 +0800
categories: Others
tag: Others
---

* content
{:toc}

[原文链接](http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html)

### 一、MVC

MVC模式的意思是软件可以分成三个部分。

![image](http://image.beekka.com/blog/2015/bg2015020105.png)

	视图（View）：用户界面。
	控制器（Controller）：业务逻辑
	模型（Model）：数据保存

各部分之间的通信方式如下:

	View 传送指令到 Controller
	Controller 完成业务逻辑后，要求 Model 改变状态
	Model 将新的数据发送到 View，用户得到反馈

所有通信都是单向的。

### 二、MVP

MVP 模式将 Controller 改名为 Presenter，同时改变了通信方向。

![image](http://image.beekka.com/blog/2015/bg2015020109.png)

1. 各部分之间的通信，都是双向的。

2. View 与 Model 不发生联系，都通过 Presenter 传递。

3. View 非常薄，不部署任何业务逻辑，称为"被动视图"（Passive View），即没有任何主动性，而 Presenter非常厚，所有逻辑都部署在那里。

### 三、MVVM

MVVM 模式将 Presenter 改名为 ViewModel，基本上与 MVP 模式完全一致。

![image](http://image.beekka.com/blog/2015/bg2015020110.png)

唯一的区别是，它采用双向绑定（data-binding）：View的变动，自动反映在 ViewModel，反之亦然。