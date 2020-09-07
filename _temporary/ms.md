---
layout: post
title: 面试相关
permalink: /ms/
---

* content
{:toc}

1. [Promise的实现](https://peiyanhuang.github.io/MyBlog/2017/03/10/promise/)[Git 代码](https://github.com/peiyanhuang/learn/blob/master/js/myPromise.js)
2. [JavaScript防抖（Debouncing）和节流（Throttling）](https://peiyanhuang.github.io/MyBlog/2017/02/06/%E5%87%BD%E6%95%B0%E9%98%B2%E6%8A%96%E4%B8%8E%E8%8A%82%E6%B5%81/)
3. apply、call、bind 的区别
  - 三者都可以改变函数的this对象指向。
  - 三者第一个参数都是this要指向的对象，如果如果没有这个参数或参数为undefined或null，则默认指向全局window。
  - 三者都可以传参，但是apply是数组，而call是参数列表，且apply和call是一次性传入参数，而bind可以分为多次传入。
  - bind 是返回绑定this之后的函数，便于稍后调用；apply 、call 则是立即执行 。
```js
Function.prototype.bind = function () {
  var _this = this;
  var context = arguments[0];
  var arg = [].slice.call(arguments, 1);
  return function () {
    arg = [].concat.apply(arg, arguments);
    _this.apply(context, arg);
  }
};
```