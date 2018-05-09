---
layout: post
title:  ShadowSocks 自定义规则
date:   2018-05-09 19:58:00 +0800
categories: 开发者
tag: 开发者
---

* content
{:toc}

公司网 MDN 突然不能访问了，检查了一遍防火墙等，没发现问题。难道被墙了？shadowsock 开着啊。换一下全局代理试试。哦，可以了！

那怎么让在 PAC 模式下也能访问呢？在这研究下。

### 1.什么是PAC

下面是从维基百科摘录的关于 PAC 的解释：

代理自动配置（英语：Proxy auto-config，简称PAC）是一种网页浏览器技术，用于定义浏览器该如何自动选择适当的代理服务器来访问一个网址。

一个 PAC 文件包含一个JavaScript形式的函数 `FindProxyForURL(url, host)`。这个函数返回一个包含一个或多个访问规则的字符串。用户代理根据这些规则适用一个特定的代理其或者直接访问。当一个代理服务器无法响应的时候，多个访问规则提供了其他的后备访问方法。浏览器在访问其他页面以前，首先访问这个 PAC 文件。PAC 文件中的 URL 可能是手工配置的，也可能是是通过网页的网络代理自发现协议（Web Proxy Autodiscovery Protocol）自动配置的。

`shadowsocks.exe` 同级目录下有一个 `pac.txt` 文件，这正是我所说的 `pac` 配置文件。

### 2.ShadowSocks 规则格式说明

`ShadowSocks` 默认使用的 `GFWList` 规则，使用的是 `adblock plus` 的引擎，要想自己添加规则最好熟悉一下其规则，下面是 `ShadowSocks` 的 pac 规则。

[中文版： Adblock Plus 过滤规则](https://adblockplus.org/zh_CN/filters)
[英文版： Adblock Plus filters explained](https://adblockplus.org/en/filter-cheatsheet)

规则大概描述如下

1. 通配符支持，如 `*.example.com/*` 实际书写时可省略 `*` 如 `.example.com/` 意即 `*.example.com/*`
2. 正则表达式支持，以 `\` 开始和结束， 如 `\[\w]+:\/\/example.com\`
3. 例外规则 `@@`，如 `@@*.example.com/*` 满足 `@@` 后规则的地址不使用代理
4. 匹配地址开始和结尾 `|`，如 `|http://example.com`、`example.com|` 分别表示以 `http://example.com` 开始和以 `example.com` 结束的地址
5. `||` 标记，如 `||example.com` 则 `http://example.com` 、`https://example.com` 、`ftp://example.com` 等地址均满足条件，只用于匹配地址开头
6. 注释 `!` 如 `! Comment`
7. 分隔符 `^`，表示除了字母、数字或者 `_ - . %` 之外的任何字符。如 `http://example.com^` ，`http://example.com/` 和 `http://example.com:8000/` 均满足条件，而 `http://example.com.ar/` 不满足条件

### 3.PAC文件及user-rule文件

那么，如何讲一个网站添加到 PAC 中呢？在 Shadowsocks 里面，可以有如下两个方式：

1. 添加到 pac.txt 文件中

可以点击 ShadowSocks 选择『编辑编辑本地PAC文件』项，使用编辑器打开 pac.txt 文件，在文件的 `rules` 中加入你的规则。

2. 添加到 user-rule.txt 文件中

可以点击 ShadowSocks 选择『编辑GFWList的用户规则』项，使用编辑器打开 user-rule.txt 文件，在文件中加入你的规则，格式如下：

```
! Put user rules line by line in this file.
! See https://adblockplus.org/en/filter-cheatsheet
||amazonaws.com
||atom.io
```
