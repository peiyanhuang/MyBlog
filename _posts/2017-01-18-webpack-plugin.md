---
layout: post
title:  Webpack plugin常用 
date:   2017-01-18 20:38:00 +0800
categories: 前端工具
tag: 前端工具
---

* content
{:toc}

###　NoErrorsPlugin

该插件可以当遇到错误时跳过，不终止webpack进程。

	plugins: [
		new webpack.NoErrorsPlugin()
	]

### CommonsChunkPlugin

提取公共脚本

```
entry: {
		"main": path.join(__dirname + "/app/main.js"),
		"hello": path.join(__dirname + "/app/hello.js"),
		"vendor": ['react']
	},


plugins: [
	    //提取公共脚本
	    new webpack.optimize.CommonsChunkPlugin('vendor',  'vendor.js')
  	],
```

### transfer-webpack-plugin

拷贝目录下的文件到输出目录

```
var TransferWebpackPlugin = require('transfer-webpack-plugin');
//其他节点省略    
plugins: [
    //把指定文件夹下的文件复制到指定的目录
    new TransferWebpackPlugin([
      {from: 'www'}
    ], path.resolve(__dirname,"src"))
  ]
```
