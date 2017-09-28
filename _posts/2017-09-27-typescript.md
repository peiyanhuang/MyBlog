---
layout: post
title:  typeScript
date:   2017-09-27 19:58:00 +0800
categories: Python
tag: Python
---

* content
{:toc}

### 1. 安装使用

npm 安装:

```
> npm install -g typescript
```

编译方式:

```
tsc ***.ts
```

输出结果为一个`***.js`文件，它包含了和输入文件中相同的JavsScript代码。

### 2. 类型注解

可以为函数变量提供类型约束。

```
function greeter(person: string) {
    return "Hello, " + person;
}

var user = [0, 1, 2];

document.body.innerHTML = greeter(user);
```

### 3. 接口

