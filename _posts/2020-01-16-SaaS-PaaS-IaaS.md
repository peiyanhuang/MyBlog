---
layout: post
title:  SaaS、PaaS、IaaS是什么及它们的区别
date:   2020-01-16 19:00:00 +0800
categories: 开发者
tag: 开发者
---

* content
{:toc}

从小型企业到全球企业，云都是一个非常热门的话题，它是一个非常广泛的概念，涵盖了很多在线领域。 无论是应用程序还是基础架构部署，当您开始考虑将业务转移到云时，了解各种云服务的差异和优势比以往任何时候都更加重要。

通常有三种云服务模型：

- IaaS：基础设施服务，Infrastructure-as-a-service
- PaaS：平台服务，Platform-as-a-service
- SaaS：软件服务，Software-as-a-service

![image]({{'/images/saas-paas-iaas.png'|prepend:site.baseurl}})

### SaaS

SaaS 是软件的开发、管理、部署都交给第三方，不需要关心技术问题，可以拿来即用。普通用户接触到的互联网服务，几乎都是 SaaS。

- 客户管理服务 Salesforce
- 团队协同服务 Google Apps
- 储存服务 Dropbox
- 社交服务 Facebook / Twitter / Instagram

### PaaS

PaaS 提供软件部署平台（runtime），抽象掉了硬件和操作系统细节，可以无缝地扩展（scaling）。开发者只需要关注自己的业务逻辑，不需要关注底层。包括不需要管理或控制底层的云基础设施，包括网络、服务器、操作系统、存储等，但客户能控制部署的应用程序，也可能控制运行应用程序的托管环境配置。

### IaaS

IaaS 是云服务的最底层，主要提供一些基础资源。它与 PaaS 的区别是，用户需要自己控制底层，实现基础设施的使用逻辑。包括控制操作系统的选择、存储空间、部署的应用，也有可能获得有限制的网络组件（例如路由器、，防火墙，、负载均衡器等）的控制。