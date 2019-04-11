---
layout: post
title:  Webpack plugin常用 
date:   2017-01-18 20:38:00 +0800
categories: 前端工具
tag: 前端工具
---

* content
{:toc}

### NoErrorsPlugin

该插件可以当遇到错误时跳过，不终止webpack进程。

	plugins: [
		new webpack.NoErrorsPlugin()
	]

webpack2.x 要替换为 new webpack.NoEmitOnErrorsPlugin()

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

### html-webpack-plugin

这个插件用来简化创建服务于 webpack bundle 的 HTML 文件，尤其是对于在文件名中包含了 hash 值，而这个值在每次编译的时候都发生变化的情况。你既可以让这个插件来帮助你自动生成 HTML 文件，也可以使用 lodash 模板加载生成的 bundles，或者自己加载这些 bundles。

[利用webpack生成HTML普通网页&页面模板](https://segmentfault.com/a/1190000007126268#articleHeader1)

[html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin)

### extract-text-webpack-plugin

打包成独立文件

```
var ExtractTextPlugin = require('extract-text-webpack-plugin');
var extractCSS = new ExtractTextPlugin('style/[name].css');

var cssLoader = extractCSS.extract(['css']);

modules: {
	loaders: [
		{test: /\.css$/, loader: cssLoader}
	]
},
plugins:[
	extractCSS,
]
```

[extract-text-webpack-plugin](https://www.npmjs.com/package/extract-text-webpack-plugin)