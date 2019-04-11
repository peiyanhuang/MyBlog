---
layout: post
title:  使用 Resource Hints 提高页面性能
date:   2019-03-13 19:00:00 +0800
categories: 开发者
tag: 开发者
---

* content
{:toc}

Resource Hints 是 HTML 的 `link` 标签的一系列相关标准。具体包括 `dns-prefetch`, `preconnect`, `prefetch` 和 `prerender` 四个属性，下面将详细介绍。

### 1.DNS Prefetch

Resource Hint 主要通过使用 `link` 标签。`rel` 属性确定类型，`href` 属性则指定相应的源或资源 URL。DNS Prefetch 可以像下面这样使用：

```html
<link rel="dns-prefetch" href="//example.com">
```

DNS 的解析也是需要消耗时间的，DNS Prefetch 的作用就是提供 DNS 预解析，本页面无论是否存在此域名下的资源，都会对此域名进行 DNS 解析，当实际用到时，浏览器就无需再去解析。

[dns-prefetch 兼容性](https://caniuse.com/#search=dns-prefetch)

### 2.Preconnect

我们知道，要建立连接不仅需要 DNS 解析，还需要进行 TCP 协议握手，有些还会有 TLS/SSL 协议，这些都会导致连接的耗时。

Preconnect 的作用就是可以帮我们预先建立连接：

* 针对 HTTP：可以提前为 URL 建立早期链接，可提前完成包含 DNS 解析 + TCP 三次握手环节；
* 针对 HTTPS：可以提前为 URL 建立早期链接，可提前完成包括 DNS 解析 + TCP 三次握手 + TLS / SSL 握手环节

```html
<link rel="preconnect" href="//example.com">
<link rel="preconnect" href="//cdn.example.com" crossorigin>
```

[Preconnect 兼容性](https://caniuse.com/#search=Preconnect)

需要注意的是，标准并没有硬性规定浏览器一定要完成整个连接过程，浏览器可以视情况完成部分工作。

### 3.Prefetch

Prefetch 是一个低优先级的资源加载策略，此策略允许浏览器在后台（空闲时间）获取稍后可能需要的资源，并将它们存储在浏览器缓存中。由于仅仅是提前获取资源，因此浏览器不会对资源进行预处理，并且像 CSS 样式表、JavaScript 脚本这样的资源是不会自动执行并应用于当前文档的。

如果在执行预加载策略时，中途出现了高优先级的请求，则预加载的线程占用会立即释放，并且会将已经加载的全部删除，已让出线程来执行当前页面的内容。

```html
<link rel="prefetch" href="//example.com/next-page.html" as="document" crossorigin="use-credentials">
<link rel="prefetch" href="/library.js" as="script">
```

Prefetch 有一个 `as` 的可选属性，用来指定获取资源的类型。应用程序可以使用 `as` 属性来传递资源上下文，使得用户代理可以优化提取过程 - 例如，设置适当的请求标头，传输优先级等。下表列出了一些常用资源的 [`as` 属性值](https://www.w3.org/TR/preload/#as-attribute)：

| consumer | Preload directive |
| :--- | :--- |
| `<audio>` | `<link rel=preload as=audio href=...>` |
| `<video>` | `<link rel=preload as=video href=...>` |
| `<track>` | `<link rel=preload as=track href=...>` |
| `<script>, Worker's importScripts` | `<link rel=preload as=script href=...>` |
| `<link rel=stylesheet>, CSS @import` | `<link rel=preload as=style href=...>` |
| `CSS @font-face` | `<link rel=preload as=font href=...>` |
| `<img>, <picture>, srcset, imageset` | `<link rel=preload as=image href=...>` |
| `SVG's <image>, CSS *-image` | `<link rel=preload as=image href=...>` |
| `XHR, fetch` | `<link rel=preload as=fetch crossorigin href=...>` |
| `Worker, SharedWorker` | `<link rel=preload as=worker href=...>` |
| `<embed>` | `<link rel=preload as=embed href=...>` |
| `<object>` | `<link rel=preload as=object href=...>` |
| `<iframe>, <frame>` | `<link rel=preload as=document href=...>` |

### 4.Prerender

Prerender 会预处理资源，浏览器都会作为 HTML 进行处理。浏览器除了会去获取资源，还可能会预处理（MAY preprocess）该资源，而该 HTML 页面依赖的其他资源，像 `<script>`、`<style>` 等页面所需资源也可能会被处理。但是预处理会由于浏览器或当前机器、网络情况的不同而被不同程度地推迟。例如，会根据 CPU、GPU 和内存的使用情况，以及请求操作的幂等性而选择不同的策略或阻止该操作。

注意，由于这些预处理操作的不可控性，当你只是需要能够预先获取部分资源来加速后续可能出现的网络请求时，建议使用 Prefetch。当使用 Prerender 时，为了保证兼容性，目标页面可以监听 `visibilitychange` 事件并使用 `document.visibilityState` 来判断页面状态。

```html
<link rel="prerender" href="//example.com/next-page.html">
```

### 5.Resource Hint的具体使用方式

1. 使用文档 `head` 中的 `link` 元素

```html
<link rel="prefetch" href="./nextpage.js" as="script">
```

2. HTTP Link 头字段

通过使用 [HTTP Link Header](https://tools.ietf.org/html/rfc5988)来使用 Resource Hint

Link 主要由两部分组成—— `URI-Reference` 和 `link-param`。`URI-Reference` 相当于 `link` 元素中的 `href` 属性；`link-param` 则包括了 `rel`、`title`、`type` 等一系列元素属性，使用 `;` 分割。因此可以在 html 文件的响应头中添加以下部分:

```js
Link: </nextpage.js>; rel="prefetch"; as="script"
```

3. 动态添加

```js
var hint = document.createElement("link");
hint.rel = "prefetch";
hint.as = "document";
hint.href = "/article/part3.html";
document.head.appendChild(hint);
```

注意：也可以改变页面中原有 Resource Hint 的 `href` 属性（或者 prefetch 时的 `as` 属性）时，会立即触发对新资源的 Resource Hint。

```js
var hint = document.querySelector('[rel="prefetch"]');
hint.href = './the.other.nextpage.js';
```

### 6.Preload

既然提到了 Resource Hint，那么不得不介绍一下与其类似的 Preload。`preload` 也是加载资源的功能，但区别于 `prefetch` 的是，`prefetch` 的目的在于针对即将访问的页面使用低优先级来提前预加载资源，而 `preload` 则会针对当前页面使用高优先级来加载资源，如字体文件，图片等，但值得注意的是，它不会阻塞 `onload` 事件的执行。

```html
<link rel=“preload” href=“./js/base.js” as=“script”>
```

* 参考：

1. [w3c Resource Hints](https://www.w3.org/TR/resource-hints/#resource-hints)
2. [cors-settings-attributes](https://html.spec.whatwg.org/multipage/urls-and-fetching.html#cors-settings-attributes)
3. [Resource Hints - What is Preload, Prefetch, and Preconnect?](https://www.keycdn.com/blog/resource-hints)