---
layout: post
title:  node 热更新
date:   2019-02-11 19:00:00 +0800
categories: node
tag: node
---

* content
{:toc}

在开发 Node.js 实现的 HTTP 应用时会发现，无论你修改了代码的哪一部份，都必须终止 Node.js 再重新运行才会奏效。这是因为 Node.js 只有在第一次引用到某部份时才会去解析脚本文件，以后都会直接访问内存，避免重复载入。Node.js 的这种设计虽然有利于提高性能，却不利于开发调试，因为我们在开发过程中总是希望修改后立即看到效果，而不是每次都要终止进程并重启。

### 方案一：fs.watch

使用 node 原生的 `fs.watch` 方法监听文件改动，所谓的“热重载”也不过是及时清除内存中的文件缓存。示例如下：

```js
const fs = require('fs'),
    path = require('path'),
    projectRootPath = path.resolve(__dirname, './src');

const watch = project => {
    require('./src'); // 启动 APP，自动检索到 src/index.js
    try { // 监听文件夹
        fs.watch(project, { recursive: true }, cacheClean);
    } catch(e) {
        console.error('watch file error');
    }
}
// 清除缓存
const cacheClean = () => {
    Object.keys(require.cache).forEach(function (id) {
        if (/[\/\\](src)[\/\\]/.test(id)) {
            delete require.cache[id];
        }
    });
}
// 启动开发模式
watch(projectRootPath);
```

注意：在服务器入口文件 `src/index.js` 中引用中间件时需要套一层函数，并使用 `require` 的方式引入模块才能清除缓存。比如：

```js
// 引入中间件或控制器
app.use(async (ctx, next) => {
    await require('./controllers/main.js')(ctx);
});
// 或引入路由
app.use(async (ctx, next) => {
    await require('./router').routes()(ctx, next)
});
```

### 方案二：应用进程管理器

此处以 PM2 为例，supervisor、forever 等类似的进程管理工具异曲同工，这里不再赘述。

PM2 是一款带有负载均衡功能的 Node 应用进程管理器，具有 `--watch` 配置项，用来监听应用目录的变化，一旦发生变化，立即重启。他是真正意义上的重启，不是热替换。

```js
pm2 start ecosystem.config.js

<!-- ecosystem.config.js -->
module.exports = {
  apps: [{
    name        : "worker",
    script      : "./worker.js",
    watch       : true,
    env: {
      "NODE_ENV": "development",
    },
    env_production : {
       "NODE_ENV": "production"
    }
  }]
}
```

[pm2 Documentation](http://pm2.keymetrics.io/docs/usage/application-declaration/)

### 方案三：开发插件

[nodemon](https://github.com/remy/nodemon) 和 node-dev 都是可用于 node.js 开发版插件，提供简单易用的开发环境。以 nodemon 为例：

```json
"scripts": {
    "start": "nodemon ./bin/www"
}
```