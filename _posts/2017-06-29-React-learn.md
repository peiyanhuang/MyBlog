---
layout: post
title:  React 学习入门
date:   2017-06-29 19:58:00 +0800
categories: React
tag: React
---

* content
{:toc}

### JSX

React DOM 使用驼峰(camelCase)属性命名约定, 而不是HTML属性名称。

```
class -> className
tabindex -> tabIndex
```


如果我们对 Button 绑定了一个点击事件，那么其子元素中也会绑定该点击事件。因此，如果 Button 中存在子元素，那么该子元素仍然是可以点击并触发点击事件的

```
&[disabled] > * {
    pointer-events: none;
}
```


