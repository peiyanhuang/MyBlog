---
layout: post
title:  微前端的 6 种模式(译文)
date:   2019-12-10 19:00:00 +0800
categories: 译文
tag: 开发者
---

* content
{:toc}

[原文 6 Patterns for Microfrontends](https://blog.bitsrc.io/6-patterns-for-microfrontends-347ae0017ec0)
微前端模式概述：优点，缺点和实现。

微前端已经不是一个新鲜的事物了，但肯定是最近的潮流。在2016年，伴随着开发大型Web应用程序时开始出现的问题，这种模式逐渐开始流行起来。

在这片文章中，我们将介绍创建微前端的不同模式，它们的优缺点以及提出的每种方​​法的实现细节和示例。我也同意微前端会带来一些遗传的问题，这些问题可以通过更进一步来解决——进入一个可以称为 Modulith 或 Siteless UI 的区域，具体取决于场景。

但是，让我们一步一步来。我们以历史背景开始我们的旅程。

### 背景

当 Web（即以HTTP作为传输方式和以HTML作为表现形式）开始时，它们没有“设计(design)”或“布局(layout)”的概念。代替的是文本文档的交换。`<img>`标记的引入改变了一切，可以与`<table>`一起让设计师向优秀的体验宣战。但是，一个问题很快出现：如何在多个站点之间共享通用布局？为此，提出了两种解决方案：

1.使用程序动态生成 HTML（速度慢，但还可以-特别是具有CGI标准背后的强大功能）
2.使用已经集成到 Web 服务器中的机制将通用部件替换为其他部件

前者导致 C 和 Perl 网络服务器，后来变为 PHP 和 Java，然后转换为 C＃ 和 Ruby，直到最后 Elixir 和 Node.js 出现，后者实际上并不是在2002年之后的。Web 2.0 还要求使用更复杂的工具，这就是为什么使用成熟的应用程序进行服务端渲染在相当长的时间内一直占据主导地位。

直到 Netflix 出现并告诉所有人提供更小的服务以云服务商更富有。具有讽刺意味的是，尽管 Netflix 为自己的数据中心做好准备，但它们仍与 AWS 等云供应商建立了大规模的关系，也托管了包括Amazon Prime Video在内的大多数竞争对手。

### 模式

在下文中，我们将介绍一些实际上可以实现微前端架构的模式。我们将看到，当有人问“实现微前端的正确方法是什么？”时，“取决于”实际上是正确的答案。这在很大程度上取决于我们所追求的。

每个部分都包含一些示例代码和一个非常简单的代码片段（有时使用框架）以实现该模式，以进行概念验证。最后，我尝试提供一个小总结，向我的读者表现我的个人感受。

无论选择哪种模式，在集成单独的项目时，保持一致的UI始终是一个挑战。使用Bit（Github）之类的工具在您不同微前端UI组件上实现共享和协作。

### web 途径（The Web Approach）

实现微前端的最简单方法是部署一组小型网站（理想情况下只是一个页面），这些网站只是链接在一起。用户通过使用指向提供内容的不同服务器的链接来访问网站。

为了使布局保持一致，可以在服务器上使用模式库。每个团队都可以根据需要实现服务器端渲染。模式库还必须在不同平台上可用。

![image]({{ '/images/web/1_ajs5RXxReHCViUsJJZjQdA.png'|prepend:site.baseurl}})

使用Web方法就像将静态站点部署到服务器一样简单。可以使用Docker映像来完成，如下所示:

```shell
FROM nginx:stable
COPY ./dist/ /var/www
COPY ./nginx.conf /etc/nginx/conf.d/default.conf
CMD ["nginx -g 'daemon off;'"]
```

显然，我们不限于使用静态站点。我们也可以应用服务器端渲染。将 nginx 基本映像更改为例如 ASP.NET Core，使我们可以使用 ASP.NET Core 生成页面。但这与前端整体有何不同？例如，在这种情况下，我们将采用通过Web API公开的给定微服务（即返回类似JSON的东西）并将其更改为返回需要渲染的HTML代替。

从逻辑上讲，在这个世界，微前端无非是表示我们API的另一种方式。我们已经生成视图，而不是返回“裸”数据。

在这里，我们找到以下解决方案：

- nginx
- Docker

这种方法的优缺点是什么？

- 优点：完全隔离
- 优点：最灵活的方法
- 优点：最少复杂的方法
- 缺点：基础架构开销
- 缺点：用户体验不一致
- 缺点：内部网址暴露在外部

### 服务器端组成（Server-Side Composition）

这是真正的微前端方法。为什么？如我们所见，微前端被认为是在服务器端运行的。因此，整个方法肯定可以独立工作。当我们为每个小型前端代码段提供专用服务器时，我们实际上可以称其为微型前端。

以示意图的形式，我们可能会得到如下图所示的草图。

![image]({{ '/images/web/1_FZfXglk_E_boWZs3d_5qTQ.png'|prepend:site.baseurl}})

该解决方案的复杂性完全在于反向代理层。如何将不同的较小站点组合为一个站点可能很棘手。尤其是诸如缓存规则，跟踪和其他棘手的问题早晚会找上我们。

从某种意义上说，这为第一种方法增加了一种网关层。反向代理将不同的来源组合到一起。当然，那些棘手的问题需要（并且可以）以某种方式解决。

```
http {
  server {
    listen 80;
    server_name www.example.com;
    location /api/ {
      proxy_pass http://api-svc:8000/api;
    }
    location /web/admin {
      proxy_pass http://admin-svc:8080/web/admin;
    }
    location /web/notifications {
      proxy_pass http://public-svc:8080/web/notifications;
    }
    location / {
      proxy_pass /;
    }
  }
}
```

一个更强大一点的方法是使用像[Varnish Reverse Proxy](https://www.varnish-software.com/solutions/http-api-acceleration/)一样的方式。

此外，我们也发现这是ESI（Edge-Side Includes）的完美用例-它是历史悠久的Server-Side Includes（SSI）的（更加灵活）后继。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Demo</title>
  </head>
  <body>
    <esi:include src="http://header-service.example.com/" />
    <esi:include src="http://checkout-service.example.com/" />
    <esi:include
      src="http://navigator-service.example.com/"
      alt="http://backup-service.example.com/"
    />
    <esi:include src="http://footer-service.example.com/" />
  </body>
</html>
```

可以通过Tailor后端服务看到类似的设置，该服务是Project Mosaic的一部分。

在这个空间中，我们找到以下解决方案：

- Project Mosaic
- Podium

这种方法的优缺点是什么？

- 优点：完全隔离
- 优点：看起来对用户而言是嵌入式的
- 优点：非常灵活的方法
- 缺点：加强组件之间的耦合
- 缺点：基础架构复杂
- 缺点：用户体验不一致

### 客户端合成(Client-Side Composition)

在上一节，您可能会纳闷：我们需要反向代理吗？由于这是一个后端组件，我们可能希望完全避免这种情况。解决方案是客户端组合。以最简单的形式，可以使用`<iframe>`元素来实现。不同部分之间的通信是通过 postMessage 方法完成的。

![image]({{ '/images/web/1_vYmkhnQB7za8OB5yzdkppA.png'|prepend:site.baseurl}})

注意：如果是`<iframe>`，则JavaScript部分可以用“浏览器”代替。在这种情况下，潜在的交互性肯定是不同的。

顾名思义，此模式试图避免反向代理带来的基础架构开销。相反，由于微前端已经包含术语“前端”，因此整个渲染将留给客户端。优点是从此模式开始 serverless 是可能的。最后，可以将整个UI上传到GitHub页面存储库，并且一切正常。

如概述的那样，可以使用非常简单的方法（例如，仅`<iframe>`）完成合成。但是，主要的痛点之一是这种集成对于最终用户的外观。在重复资源需求方面，和模式1可以混合使用，可以将不同的部分放置在独立的Web服务器上。

但是，在这种模式下，再次需要认知-组件1已经知道组件2存在并且需要使用。潜在地，它甚至需要知道如何使用它。

考虑以下项（即已交付的应用程序或网站）：

(https://gist.github.com/dddbf80773ed8df03fcf20679f30c835)[https://gist.github.com/dddbf80773ed8df03fcf20679f30c835]

我们可以编写一个页面来启用直接通信：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Microfrontend</title>
  </head>
  <body>
    <h1>Child</h1>
    <p><button id="message_button">Send message to parent</button></p>
    <div id="results"></div>

    <script>
      const results = document.querySelector("#results");
      const messageButton = document.querySelector("#message_button");
      function sendMessage(msg) {
        window.parent.postMessage(msg, "*");
      }
      window.addEventListener("message", function(e) {
        results.innerHTML = e.data;
      });
      messageButton.addEventListener("click", function(e) {
        sendMessage(Math.random().toString());
      });
    </script>
  </body>
</html>
```

如果我们不考虑框架，那么我们也可以选择网络组件。在这里，可以使用自定义事件通过DOM进行通信。此时已经考虑客户端渲染而不是客户端合成是有意义的。因为渲染意味着需要JavaScript客户端（与网络组件方法保持一致）。

在这个部分中，我们找到以下解决方案：

- Web Components
- PostMate

这种方法的优缺点是什么？

- 优点：完全隔离
- 优点：看起来对用户而言是嵌入式的
- 优点：可能没有服务器
- 缺点：加强组件之间的耦合
- 缺点：用户体验不一致
- 缺点：可能需要JavaScript /没有无缝集成

### 客户端渲染(Client-Side Rendering)

虽然客户端合成可能不需要JavaScript（例如，仅使用不依赖于与父或彼此之间通信的 frame），但是如果没有JavaScript，客户端渲染将失败。在这个空间中，我们已经开始在组成应用程序中创建框架。引入的所有微前端都必须遵守此框架。至少他们需要使用它才能正确安装。

该模式如下所示:

![image]({{ '/images/web/1_fSIFnUAwpQ1O5JFTiHDHFA.png'|prepend:site.baseurl}})

非常接近客户端组成，对吗？在这种情况下，可能无法替换JavaScript部分。重要的区别在于服务器端渲染通常不在桌面上。取而代之的是交换数据，然后将其转换为视图。

根据设计或使用的框架，可以确定渲染的数据片段的位置，时间点和交互性。这种模式实现高度的交互性没有问题。

在这部分中，我们找到以下解决方案：

- Web Components
- Luigi

- 优点：加强关注点分离
- 优点：提供组件的松散耦合
- 优点：看起来对用户而言是嵌入式的
- 缺点：在客户端需要更多逻辑
- 缺点：用户体验不一致
- 缺点：需要JavaScript

### SPA

为什么我们应该停止使用单一技术进行客户端渲染？为什么不仅仅获取一个JavaScript文件并在所有其他JavaScript文件之外运行它呢？这样做的好处是可以并行使用多种技术。

![image]({{ '/images/web/1_cXaKz7k_bnySWSxlK1Mvqw.png'|prepend:site.baseurl}})

是否运用多种技术（独立的在后端还是在前端，在后端可能更“可以接受”）是一件好事还是应该避免这有带辩论，但是，在某些情况下，多种技术需要协同工作。

从我脑袋里冒出了：

- 迁移方案
- 支持特定的第三方技术
- 政治问题
- 团队约束

无论哪种方式，都可以如下绘制出的模式。

那么这是怎么回事？在这种情况下，仅提供一些带有应用程序外壳的JavaScript不再是可选的-相反，我们需要提供一个能够协调微前端的框架。

不同模块的编排归结为生命周期的管理：安装，运行，卸载。可以从独立运行的服务器中获取不同的模块，但是，它们的位置必须在应用程序外壳中已经知道。

实现这样的框架至少需要一些配置，例如，脚本映射包括：

```js
const scripts = [
  'https://example.com/script1.js',
  'https://example.com/script2.js',
];
const registrations = {};

function activityCheck(name) {
  const current = location.hash;
  const registration = registrations[name];

  if (registration) {
    if (registration.activity(current) !== registration.active) {
      if (registration.active) {
        registration.lifecycle.unmount();
      } else {
        registration.lifecycle.mount();
      }

      registration.active = !registration.active;
    }
  }
}

window.addEventListener('hashchange', function () {
  Object.keys(registrations).forEach(activityCheck);
});

window.registerApp = function(name, activity, lifecycle) {
  registrations[name] = {
    activity,
    lifecycle,
    active: false,
  };
  activityCheck(name);
}

scripts.forEach(src => {
  const script = document.createElement('script');
  script.src = src;
  document.body.appendChild(script);
});
```

生命周期管理可能比上面的脚本更复杂。因此，用于这种组合的模块需要带有一些应用类的结构-至少包含 `mount` 和 `unmount` 的功能。

在这部分内容中，我们找到以下解决方案：

- [Single SPA](https://single-spa.js.org/)
- [Frint.js](https://frint.js.org/)

这种方法的优缺点是什么？

- 优点：加强关注点分离
- 优点：给开发人员很大的自由度
- 优点：看起来对用户而言是嵌入式的
- 缺点：造成重复和开销
- 缺点：用户体验不一致
- 缺点：需要JavaScript

### Siteless UIs

这个主题应该值得我们单独一片文章，但是由于我们列出了所有模式，所以我不想在这里忽略它。采用SPA组合的方法，我们所缺少的只是将脚本源与服务解耦（或独立集中化），以及共享运行时。

这两件事都是有原因的：

- 解耦可确保UI和服务职责不混淆；这也使无服务器计算成为可能
- 共享运行时是解决先前模式所提供的资源密集型组合的方法

就像“serverless functions”对后端所做的那样，这两者都使前端获得了收益。它们也面临类似的挑战：

- 运行时不能只是更新-它必须与模块保留/保持一致
- 在本地调试或运行模块需要运行时的仿真器
- 并非所有技术都能得到同等支持

Siteless UIs 的示意图如下所示: 

![image]({{ '/images/web/1_FJBSCdCjLrNBay601l15wQ.png'|prepend:site.baseurl}})

此设计的主要优点是支持共享有用的或公共资源。共享模式库非常有意义。

总的来说，架构图看起来与前面提到的SPA组成非常相似。但是，`feed service` 以及与运行时的耦合带来了其他好处（以及该领域中任何框架都需要解决的挑战）。

最大的优势在于，一旦克服了这些挑战，就应该拥有出色的开发经验。可以完全自定义用户体验，将模块视为灵活的可选功能。