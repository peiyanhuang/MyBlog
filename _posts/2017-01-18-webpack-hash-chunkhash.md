---
layout: post
title:  Webpack--hash 和 chunkhash 的区别
date:   2017-01-18 18:38:00 +0800
categories: 前端工具
tag: 前端工具
---

* content
{:toc}

### hash 和 chunkhash 的区别

官方文档对于两者的定义：

    [hash] is replaced by the hash of the compilation.

hash代表的是`compilation`的hash值。

    [chunkhash] is replaced by the hash of the chunk.

`chunkhash`代表的是chunk的hash值。

`chunkhash`很好理解，chunk在Webpack中的含义我们都清楚，简单讲，chunk就是模块。chunkhash也就是根据模块内容计算出的hash值。

`compilation`对象代表某个版本的资源对应的编译进程。当使用Webpack的development中间件时，每次检测到项目文件有改动就会创建一个`compilation`，进而能够针对改动生产全新的编译文件。`compilation`对象包含当前模块资源、待编译文件、有改动的文件和监听依赖的所有信息。

与`compilation`对应的有个`compiler`对象，通过对比，可以帮助大家对`compilation`有更深入的理解。`compiler`对象代表的是配置完备的Webpack环境。 `compiler`对象只在Webpack启动时构建一次，由Webpack组合所有的配置项构建生成。

简单的讲，`compiler`对象代表的是不变的webpack环境，是针对webpack的；而`compilation`对象针对的是随时可变的项目文件，只要文件有改动，`compilation`就会被重新创建。

### hash

```
entry: {
    "main": path.join(__dirname + "/app/main.js"),
    "hello": path.join(__dirname + "/app/hello.js")
},
output: {
    path: path.join(__dirname + "/public"),
    filename: "[name].[hash].js"
},

//npm start
                        Asset     Size  Chunks             Chunk Names
hello.3526642e76ed3a029dad.js  1.43 kB       0  [emitted]  hello
 main.3526642e76ed3a029dad.js   753 kB       1  [emitted]  main
```

`hash`是compilation对象计算所得，而不是具体的项目文件计算所得。所以以上配置的编译输出文件，所有的文件名都会使用相同的`hash`。

这样带来的问题是，任何一个js文件改动都会影响另外 js 文件的最终文件名。上线后，另外文件的浏览器缓存也全部失效。

所以用到了`chunkhash`，`chunkhash`是根据具体模块文件的内容计算所得的`hash`值，所以某个文件的改动只会影响它本身的hash，不会影响其他文件。

### chunkhash

```
entry: {
    "main": path.join(__dirname + "/app/main.js"),
    "hello": path.join(__dirname + "/app/hello.js")
},
output: {
    path: path.join(__dirname + "/public"),
    filename: "[name].[chunkhash].js"
},

//npm start
                        Asset     Size  Chunks             Chunk Names
hello.dd21fd829522dcf2ee90.js  1.43 kB       0  [emitted]  hello
 main.4e8865ea2649ceefa3a3.js   753 kB       1  [emitted]  main
```