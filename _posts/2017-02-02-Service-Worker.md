---
layout: post
title:  Service Worker
date:   2017-02-02 15:58:00 +0800
categories: JS
tag: JS
---

* content
{:toc}

### 1. Service worker介绍

`Service worker` 是一种由 Javascript 编写的浏览器端代理脚本，位于你的浏览器和服务器之间。当一个页面注册了一个 `Service worker`，它就可以注册一系列事件处理器来响应如网络请求和消息推送这些事件。`Service worker` 可以被用来管理缓存，当响应一个网络请求时可以配置为返回缓存还是从网络获取。由于 `Service worker` 是基于事件的，所以它只在处理这些事件的时候被调入内存，不用担心常驻内存占用资源导致系统变慢。

### 2. 实现的功能

目前sw还是一个草案，各个浏览器支持程度还不是很高，除了chrome 40、firefox以外，其他浏览器均不支持该功能，但是sw提供的逆天功能还是非常值得期待:

- 后台数据的同步
- 从其他域获取资源请求
- 接受计算密集型数据的更新，多页面共享该数据
- 客户端编译与依赖管理
- 后端服务的hook机制
- 根据URL模式，自定义模板
- 性能优化
- 消息推送
- 定时默认更新
- 地理围栏

其中最期待的还是通过HTTP请求的拦截，进而实现离线应用，提升页面的性能体验。

### 3. 使用Service Workers前的设置

浏览器: Google Chrome，Firefox，Opera 以及国内的各种双核浏览器都支持，但是 safari 不支持

`non-block`: `Service worker` 中的 Javascript 代码必须是非阻塞的，因为 `localStorage` 是阻塞性，所以不应该在 `Service Worker` 代码中使用 `localStorage`。

在已经支持`serivce workers`的浏览器的较新版本中，很多`serivce workers`的特性默认没有开启支持。如果你发现示例代码在代当前版本的浏览器中怎么样都无法正常运行，你可能需要开启一下浏览器的相关配置：

`Firefox Nightly`: 访问 `about:config` 并设置 `dom.serviceWorkers.enabled` 的值为 `true`; 重启浏览器；

`Chrome Canary`: 访问 `chrome://flags` 并开启 `experimental-web-platform-features`; 重启浏览器 (注意：有些特性在Chrome中没有默认开房支持)；

`Opera`: 访问 `opera://flags` 并开启 `ServiceWorker` 的支持; 重启浏览器。

另外，你需要通过`HTTPS`来访问你的页面代码 — 出于安全原因，`Service Workers`严格要求要在HTTPS下才能运行。Github是个用来测试的好地方，因为它就支持HTTPS。

### 4. Service worker 生命周期

![image]({{ '/images/sw/sw.png' | prepend: site.baseurl }})

1. `service worker`，通过`navigator.serviceWorker.register()`来加载和注册（一个脚本URL）。如果注册成功，`service worker` 在`ServiceWorkerGlobalScope`环境中运行；这是一个特殊的woker上下文运行环境，与主脚本的运行线程相独立，同时也没有访问DOM的能力。

2. `Service worker` 在 `Register` 时会触发 `Install` 事件，在 `Install` 时可以用来预先获取和缓存应用所需的资源并设置每个文件的缓存策略(可以开始`IndexDB`和`Cache`的相关操作流程)。

3. 当`oninstall`事件的处理流程执行完毕后，可以认为`service worker`安装完成了。

4. 下一步是激活。当`service worker`安装完成后，会接收到一个激活事件(`activate event`)。一旦 `Service worker` 处于 `activated` 状态，就可以完全控制应用的资源，对网络请求进行检查，修改网络请求，从网络上获取并返回内容或是返回由已安装的 `Service worker` 预告获取并缓存好的资源，甚至还可以生成内容并返回给网络语法。

5. `Service Worker` 现在可以控制页面了，但是只针对在成功注册( `register()` )了`service worker`后打开的页面。也就是说，页面打开时有没有`service worker`，决定了接下来页面的生命周期内受不受`service worker`控制。所以，只有当页面刷新后，之前不受`service worker`控制的页面才有可能被控制起来。

注意: `Service worker` 的控制从第二次页面访问开始。在首次加载页面时，所有资源都是从网络载的，Service worker 在首次加载时不会获取控制网络响应，它只会在后续访问页面时起作用。

### 5. 简单使用

从这个[项目地址](https://github.com/dominiccooney/cache-polyfill)拿到`chaches polyfill`。

这个`polyfill`支持`CacheStorate.match，Cache.add和Cache.addAll`，而现在`Chrome 49`以下实现的Cache API还没有支持这些方法。
在service worker中通过importScripts加载进来。被service worker加载的脚本文件会被自动缓存。

	importScripts('serviceworker-cache-polyfill.js');

#### 5.1 Service Worker的安装步骤

来个简单缓存文件的例子。首先在页面注册一个service worker，这个步骤告诉浏览器你的service worker脚本在哪里。

```
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js').then(function(registration) {
    // Registration was successful
    console.log('ServiceWorker registration successful with scope: ',    registration.scope);
  }).catch(function(err) {
    // registration failed :(
    console.log('ServiceWorker registration failed: ', err);
  });
}
```

上面的代码检查service worker API是否可用，如果可用，service worker /sw.js 被注册。

如果这个service worker已经被注册过，浏览器会自动忽略上面的代码。

现在你可以到 `chrome://inspect/#service-workers` 检查service worker是否对你的网站启用了。

在页面上完成注册步骤之后，让我们把注意力转到service worker的脚本里来，在这里面，我们要完成它的安装步骤。

在最基本的例子中，你需要为install事件定义一个callback，并决定哪些文件你想要缓存。

```
var CACHE_NAME = 'my-site-cache-v1';
// The files we want to cache
var urlsToCache = [
  '/',
  '/styles/main.css',
  '/script/main.js'
];

// Set the callback for the install step
self.addEventListener('install', function(event) {
    event.waitUntil(
    caches.open(CACHE_NAME)
      .then(function(cache) {
        console.log('Opened cache');
        return cache.addAll(urlsToCache);
      })
  	);
});
```

上面的代码中，我们通过caches.open打开我们指定的cache文件名，然后我们调用cache.addAll并传入我们的文件数组。这是通过一连串promise（caches.open 和 cache.addAll）完成的。event.waitUntil拿到一个promise并使用它来获得安装耗费的时间以及是否安装成功。

如果所有的文件都被缓存成功了，那么service worker就安装成功了。如果任何一个文件下载失败，那么安装步骤就会失败。这个方式允许你依赖于你自己指定的所有资源，但是这意味着你需要非常谨慎地决定哪些文件需要在安装步骤中被缓存。指定了太多的文件的话，就会增加安装失败率。

#### 5.2 怎样缓存和返回Request

你已经安装了service worker，你现在可以返回你缓存的请求了。

当service worker被安装成功并且用户浏览了另一个页面或者刷新了当前的页面，service worker将开始接收到fetch事件。下面是一个例子：

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request)
      .then(function(response) {
        // Cache hit - return response
        if (response) {
          return response;
        }

        return fetch(event.request);
      }
    )
  );
});
```

上面的代码里我们定义了fetch事件，在event.respondWith里，我们传入了一个由caches.match产生的promise.caches.match 查找request中被service worker缓存命中的response。

如果我们有一个命中的response，我们返回被缓存的值，否则我们返回一个实时从网络请求fetch的结果。这是一个非常简单的例子，使用所有在install步骤下被缓存的资源。

如果我们想要增量地缓存新的请求，我们可以通过处理fetch请求的response并且添加它们到缓存中来实现，例如：

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request)
      .then(function(response) {
        // Cache hit - return response
        if (response) {
          return response;
        }

        // IMPORTANT: Clone the request. A request is a stream and
        // can only be consumed once. Since we are consuming this
        // once by cache and once by the browser for fetch, we need
        // to clone the response
        var fetchRequest = event.request.clone();

        return fetch(fetchRequest).then(
          function(response) {
            // Check if we received a valid response
            if(!response || response.status !== 200 || response.type !== 'basic') {
              return response;
            }

            // IMPORTANT: Clone the response. A response is a stream
            // and because we want the browser to consume the response
            // as well as the cache consuming the response, we need
            // to clone it so we have 2 stream.
            var responseToCache = response.clone();

            caches.open(CACHE_NAME)
              .then(function(cache) {
                cache.put(event.request, responseToCache);
              });

            return response;
          }
        );
      })
    );
});
```

代码里我们所做事情包括：

```
1. 添加一个callback到fetch请求的 .then 方法中
2. 一旦我们获得了一个response，我们进行如下的检查：

	2.1 确保response是有效的
	2.2 检查response的状态是否是200
	2.3 保证response的类型是basic，这表示请求本身是同源的，非同源（即跨域）的请求也不能被缓存。

3.如果我们通过了检查，clone这个请求。这么做的原因是如果response是一个Stream，那么它的body只能被读取一次，所以我们得将它克隆出来，一份发给浏览器，一份发给缓存。
```

#### 5.3 如何更新一个Service Worker

你的service worker总有需要更新的那一天。当那一天到来的时候，你需要按照如下步骤来更新：

```
1. 更新你的service worker的JavaScript文件

	1.1 当用户浏览你的网站，浏览器尝试在后台下载service worker的脚本文件。只要服务器上的文件和本地文件有一个字节不同，它们就被判定为需要更新。

2. 更新后的service worker将开始运作，install event被重新触发。

3. 在这个时间节点上，当前页面生效的依然是老版本的service worker，新的servicer worker将进入"waiting"状态。
4. 当前页面被关闭之后，老的service worker进程被杀死，新的servicer worker正式生效。
5. 一旦新的service worker生效，它的activate事件被触发。
```

代码更新后，通常需要在activate的callback中执行一个管理cache的操作。因为你会需要清除掉之前旧的数据。我们在activate而不是install的时候执行这个操作是因为如果我们在install的时候立马执行它，那么依然在运行的旧版本的数据就坏了。

之前我们只使用了一个缓存，叫做`my-site-cache-v1`，其实我们也可以使用多个缓存的，例如一个给页面使用，一个给blog的内容提交使用。这意味着，在`install`步骤里，我们可以创建两个缓存，`pages-cache-v1`和`blog-posts-cache-v1`，在`activite`步骤里，我们可以删除旧的`my-site-cache-v1`。

下面的代码能够循环所有的缓存，删除掉所有不在白名单中的缓存。

```
self.addEventListener('activate', function(event) {

  var cacheWhitelist = ['pages-cache-v1', 'blog-posts-cache-v1'];

  event.waitUntil(
    caches.keys().then(function(cacheNames) {
      return Promise.all(
        cacheNames.map(function(cacheName) {
          if (cacheWhitelist.indexOf(cacheName) === -1) {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});
```

### 6 Cache API

`Cache API`就是对`http`的`request/response`进行缓存管理，是在`service worker`的规范中定义的，往往跟`service worker`一起操作使用，是实现web app离线应用的关键一环。但是`Cache API`又不依赖于`service worker`，可以单独在window下使用，。

在`window`对象下，`cache api`的操作封装在`caches`对象下面，里面的操作分为两类: 对`cache`的操作、对`cache`里面`http`的操作。下面简单说明下cache storage的相关操作

```
// open: 创建或打开一个cache
caches.open('test').then(cache => {
  return cache.add('/base.css')
}).then((val) => {
  console.log('create a cache and add "base.css" to it');
});
// 在cache storage查找缓存的资源
caches.match('/base.css').then(res => {
  if (!res) return 'can not find this http in caches storage';
  return res.text()
}).then((result) => {
  console.log(result)
});
// 得到所有的cache
caches.keys().then(name => {
  console.log('names: ', name)
});
// 得到某个cache
caches.delete('test').then(val => {
  console.log('delete success?: ', val)
});

// 对cache里面http进行操作
caches.open('test').then(cache => {
  // 添加缓存资源
  return cache.add('/base.css')
}).then((val) => {
  console.log('create a cache and add "base.css" to it');
});

caches.open('test').then(cache => {
  // 资源匹配
  return cache.match('/base.css')
}).then(res => {
  return res.text()
}).then(str => {
  console.log(str)
});
```

相关：

[使用 Service worker 实现加速/离线访问静态 blog 网站](http://www.liuhaihua.cn/archives/469454.html)  
[service worker：让 Web 应用变得逆天的起来](https://toutiao.io/posts/kuvl2w/preview)  
[【翻译】Service Worker 入门](https://www.w3ctech.com/topic/866)  
[Service Worker初体验](http://web.jobbole.com/84792/)  
