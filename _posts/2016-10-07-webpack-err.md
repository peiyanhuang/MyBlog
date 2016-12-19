---
layout: post
title:  Webpack 遇到的error
date:   2016-10-07 13:58:00 +0800
categories: 前端工具
tag: 前端工具
---

* content
{:toc}

### 1.ERROR in Entry module not found: Error: Cannot resolve module 'babel'

错误原因：没有相应的 babel-loader

	npm install --save-dev babel-loader

### 2. Error: Cannot resolve 'file' or 'directory'

错误原因：获取的文件没有后缀名

必须要检查webpack的resolve配置信息，如果没有配置resolve，必须添加在webpack配置文件中如下信息：
```
//  用来配置应用层的模块解析，即要被打包的模块
resolve: {
    //  第一项扩展非常重要，千万不要忽略，否则经常出现模块无法加载错误
    extensions: ['', '.js', '.es6', '.vue','.css']
},
```
通过以上配置，webpack会自动为请求的文件添加后缀名，从而保证请求的路径正确无误。

对上面的配置进行说明，即请求js、es6、vue文件时不需要添加后缀名，直接请求即可，如下。

	import {add} from './MathUtils'

### 3.CSS
webpack提供两个工具处理样式表，`css-loader` 和 `style-loader`，二者处理的任务不同，css-loader使你能够使用类似@import 和 url(...)的方法实现 require()的功能,style-loader将所有的计算后的样式加入页面中，二者组合在一起使你能够把样式表嵌入webpack打包后的JS文件中。