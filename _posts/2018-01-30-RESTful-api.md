---
layout: post
title:  理解 RESTful
date:   2018-01-29 19:58:00 +0800
categories: 开发者
tag: 开发者
---

* content
{:toc}

### 1.理解RESTful架构

RESTful架构风格最初由Roy T. Fielding（HTTP/1.1协议专家组负责人）在其2000年的博士学位论文中提出。HTTP就是该架构风格的一个典型应用。从其诞生之日开始，它就因其可扩展性和简单性受到越来越多的架构师和开发者们的青睐。

REST 即 `Representational State Transfer` 的缩写，可译为"表现层状态转化”。REST 最大的几个特点为：资源、统一接口、URI和无状态。

#### 1.1 资源

所谓"资源"，就是网络上的一个实体，或者说是网络上的一个具体信息。它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的实在。

资源总要通过某种载体反应其内容，文本可以用 txt 格式表现，也可以用 XML 格式表现，甚至可以采用二进制格式；图片可以用 JPG 格式表现，也可以用 PNG 格式表现；JSON 是现在最常用的资源表示格式。

数据(包括数据库)也是一种资源。

#### 1.2 统一接口

RESTful架构风格规定，数据的元操作，即 `CRUD`(create, read, update 和 delete，即数据的增删查改)操作，分别对应于 HTTP 方法，即：

- GET（SELECT）：从服务器取出资源（一项或多项）。
- POST（CREATE）：在服务器新建一个资源。
- PUT（UPDATE）：在服务器更新资源（客户端提供完整资源数据）。
- PATCH（UPDATE）：在服务器更新资源（客户端提供需要修改的资源数据）。
- DELETE（DELETE）：从服务器删除资源。

这样就统一了数据操作的接口，仅通过 HTTP 方法，就可以完成对数据的所有增删改查。

#### 1.3 URI

可以用一个 `URI`(统一资源定位符)指向资源，即每个URI都对应一个特定的资源。要获取这个资源，访问它的URI就可以，因此 URI 就成了每一个资源的地址或识别符。

一般的，每个资源至少有一个 URI 与之对应，最典型的 URI 即 URL。

#### 1.4 无状态

所谓无状态的，即所有的资源，都可以通过URI定位，而且这个定位与其他资源无关，也不会因为其他资源的变化而改变。

有状态和无状态的区别，举个简单的例子说明一下。如查询员工的工资，如果查询工资是需要登录系统，进入查询工资的页面，执行相关操作后，获取工资的多少，则这种情况是有状态的，因为查询工资的每一步操作都依赖于前一步操作，只要前置操作不成功，后续操作就无法执行；如果输入一个url即可得到指定员工的工资，则这种情况是无状态的，因为获取工资不依赖于其他资源或状态，且这种情况下，员工工资是一个资源，由一个url与之对应，可以通过 HTTP 中的 GET 方法得到资源，这是典型的 RESTful 风格。

### 2.注意点

1.协议

API与用户的通信协议，总是使用 HTTPS 协议，确保交互数据的传输安全。

2.域名

应该尽量将 API 部署在专用域名之下。

	https://api.example.com

如果确定 API 很简单，不会有进一步扩展，可以考虑放在主域名下。

	https://example.org/api/

3.api版本控制

可以将 API 的版本号放入 URL。

	https://api.example.com/v{n}/

另一种做法是，将版本号放在 HTTP 头信息中，但不如放入 URL 方便和直观。

采用多版本并存，增量发布的方式

`v{n}` n 代表版本号，分为整形和浮点型

整形的版本号: 大功能版本发布形式；具有当前版本状态下的所有API接口 ,例如：v1,v2

浮点型：为小版本号，只具备补充 api 的功能，其他 api 都默认调用对应大版本号的 api。例如：v1.1，v2.2。

4.API 路径规则

在 RESTful 架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能有名词，而且所用的名词往往与数据库的表格名对应。

### 3.axios封装RESTful标准接口

如下一个用户管理模块：

```javascript
import axios from 'axios'
import qs from 'query-string'

class UserManager {
  constructor() {
    this.$ajax = axios.create({
      baseUrl: 'https://api.example.com'
    });

    // 修改POST和PUT请求默认的Content-Type，根据需要而定
    this.dataMethodDefaults = {
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded'
      },
      transformRequest: [function (data) {
        return qs.stringify(data)
      }]
    }
  }
  
  getUsersPageableList (page = 0, size = 20) {
    return this.$ajax.get(`/users?page=${page}&size=${size}`);
  }
  
  getUsersFullList () {
    return this.$ajax.get('/users/all');
  }
  
  getUser (id) {
    if (!id) {
      return Promise.reject(new Error(`getUser：id(${id})无效`));
    }
    return this.$ajax.get(`/users/${id}`);
  }
  
  createUser (data = {}) {
    if (!data || !Object.keys(data).length) {
      return Promise.reject(new Error('createUser：提交的数据无效'));
    }
    return this.$ajax.post('/users', data, { ...this.dataMethodDefaults });
  }
}

export default new UserManager()
```

`Content-Type` Axios默认是 `application/json`，我根据后端接口的定义，将其调整成了表单类型 `application/x-www-form-urlencoded`。