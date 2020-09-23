---
layout: post
title: 面试相关
permalink: /books/
---

* content
{:toc}

### JS

- [Promise的实现](https://peiyanhuang.github.io/MyBlog/2017/03/10/promise/)[Git 代码](https://github.com/peiyanhuang/learn/blob/master/js/myPromise.js)
- [JavaScript防抖（Debouncing）和节流（Throttling）](https://peiyanhuang.github.io/MyBlog/2017/02/06/%E5%87%BD%E6%95%B0%E9%98%B2%E6%8A%96%E4%B8%8E%E8%8A%82%E6%B5%81/)
- apply、call、bind 的区别
  1. 三者都可以改变函数的this对象指向。
  2. 三者第一个参数都是this要指向的对象，如果如果没有这个参数或参数为undefined或null，则默认指向全局window。
  3. 三者都可以传参，但是apply是数组，而call是参数列表，且apply和call是一次性传入参数，而bind可以分为多次传入。
  4. bind 是返回绑定this之后的函数，便于稍后调用；apply 、call 则是立即执行。
```js
// 先实现一个apply
Function.prototype.apply = function (context) {
  context = context || window
  context.fn = this;
  const params = arguments[1] || []
  const result = context.fn(...params);
  delete context.fn;
  return result;
}
Function.prototype.bind = function (context) {
  const _this = this;
  context = context || window;
  let arg = [].slice.call(arguments, 1);
  return function () {
    arg = [].concat.apply(arg, arguments);
    _this.apply(context, arg);
  }
};
```
- 实现 new 功能
```js
function myNew (func, ...args) {
  const instance = Object.create({})
  if (func.prototype) {
    Object.setPrototypeOf(instance, func.prototype)
  }
  const res = func.apply(instance, args)
  if (typeof res === "function" || (typeof res === "object" && res !== null)) {
    return res
  }
  return instance
}
```
- [Event Loop](https://peiyanhuang.github.io/MyBlog/2019/03/20/event-loop/)
- [浏览器缓存-强缓存和协商缓存](https://peiyanhuang.github.io/MyBlog/2018/11/21/%E6%B5%8F%E8%A7%88%E5%99%A8%E7%BC%93%E5%AD%98/#%E5%BC%BA%E7%BC%93%E5%AD%98%E5%8D%8F%E5%95%86%E7%BC%93%E5%AD%98)
- 深拷贝的实现
```js
function deepCopy(value, targets) {
  let target = targets || {};
  if (typeof value !== "object" && !(value instanceof Function)) {
    target = {};
  }
  if (value !== null) {
    for (var i in value) {
      if (value[i] === target[i]) {
        continue;
      }
      if (typeof value[i] === 'object') {
        if (value[i] instanceof Array) {    //is array
          target[i] = [];
        } else {    // is object
          target[i] = {};
        }
        target[i] = deepCopy(value[i], target[i]);
      } else {
        target[i] = value[i]
      }
    }
  }
  return target;
}
```
- [柯里化](https://peiyanhuang.github.io/MyBlog/2018/04/27/adequate/#3%E6%9F%AF%E9%87%8C%E5%8C%96curry)
```js
function curry(func) {
  return function curried(...args) {
    // 关键知识点：function.length 用来获取函数的形参个数
    if (args.length >= func.length) {
      return func.apply(this, args)
    }
    return function (...args2) {
      return curried.apply(this, args.concat(args2))
    }
  }
}
// 测试
function sum (a, b, c) {
  return a + b + c
}
const curriedSum = curry(sum)
console.log(curriedSum(1, 2, 3))
console.log(curriedSum(1)(2,3))
console.log(curriedSum(1)(2)(3))
```
- 异步并发数限制
```js
/**
 * 关键点
 * 1. new promise 一经创建，立即执行
 * 2. 使用 Promise.resolve().then 可以把任务加到微任务队列，防止立即执行迭代方法
 * 3. 微任务处理过程中，产生的新的微任务，会在同一事件循环内，追加到微任务队列里
 * 4. 使用 race 在某个任务完成时，继续添加任务，保持任务按照最大并发数进行执行
 * 5. 任务完成后，需要从 doingTasks 中移出
 */
function limit(count, array, iterateFunc) {
  const tasks = []
  const doingTasks = []
  let i = 0
  const enqueue = () => {
    if (i === array.length) {
      return Promise.resolve()
    }
    const task = Promise.resolve().then(() => iterateFunc(array[i++]))
    tasks.push(task)
    const doing = task.then(() => doingTasks.splice(doingTasks.indexOf(doing), 1))
    doingTasks.push(doing)
    const res = doingTasks.length >= count ? Promise.race(doingTasks) : Promise.resolve()
    return res.then(enqueue)
  };
  return enqueue().then(() => Promise.all(tasks))
}
```
- 异步串行 | 异步并行
```js
// 字节面试题，实现一个异步加法
function asyncAdd(a, b, callback) {
  setTimeout(function () {
    callback(null, a + b);
  }, 500);
}

// 解决方案
// 1. promisify
const promiseAdd = (a, b) => new Promise((resolve, reject) => {
  asyncAdd(a, b, (err, res) => {
    if (err) {
      reject(err)
    } else {
      resolve(res)
    }
  })
})

// 2. 串行处理
async function serialSum(...args) {
  return args.reduce((task, now) => task.then(res => promiseAdd(res, now)), Promise.resolve(0))
}

// 3. 并行处理
async function parallelSum(...args) {
  if (args.length === 1) return args[0]
  const tasks = []
  for (let i = 0; i < args.length; i += 2) {
    tasks.push(promiseAdd(args[i], args[i + 1] || 0))
  }
  const results = await Promise.all(tasks)
  return parallelSum(...results)
}

// 测试
(async () => {
  console.log('Running...');
  const res1 = await serialSum(1, 2, 3, 4, 5, 8, 9, 10, 11, 12)
  console.log(res1)
  const res2 = await parallelSum(1, 2, 3, 4, 5, 8, 9, 10, 11, 12)
  console.log(res2)
  console.log('Done');
})()
```
- [JS模块规范：AMD，CMD，CommonJS 与 import](https://peiyanhuang.github.io/MyBlog/2017/11/27/js-module/)
- [游览器工作原理](https://peiyanhuang.github.io/MyBlog/2017/01/15/Browser-Work/)

### Vue

- [Vue 数据的双向绑定？](https://peiyanhuang.github.io/MyBlog/2018/10/28/vue-source-code-4/)
- [$nextTick 的实现？](https://peiyanhuang.github.io/MyBlog/2018/11/02/vue-source-code-5/#nexttick-%E7%9A%84%E5%AE%9E%E7%8E%B0)
- [$emit 实现](https://github.com/vuejs/vue/blob/11b7d5dff276caa54da3ef5b52444c0e2c5bbf41/src/core/instance/events.js#L111)
- 你知道 Vue3 响应式数据原理吗？
  - Vue3 改用 Proxy 替代 Object.defineProperty。
  - 因为Proxy可以直接监听对象和数组的变化，并且有多达13种拦截方法。并且作为新标准将受到浏览器厂商重点持续的性能优化。
  - Proxy只会代理对象的第一层，Vue3是怎样处理这个问题的呢？
    - 判断当前Reflect.get的返回值是否为Object，如果是则再通过 reactive 方法做代理， 这样就实现了深度观测。
  - 监测数组的时候可能触发多次get/set，那么如何防止触发多次呢？
    - 我们可以判断key是否为当前被代理对象target自身属性，也可以判断旧值与新值是否相等，只有满足以上两个条件之一时，才有可能执行trigger。[源码](https://github.com/vuejs/vue-next/blob/110e96d152/packages/reactivity/src/baseHandlers.ts#L127)
- [diff算法说一下](https://peiyanhuang.github.io/MyBlog/2018/05/01/react-diff/)

### 网络

- [HTTP2](https://peiyanhuang.github.io/MyBlog/2017/11/24/http2/)
- TCP与UDP总结
1. TCP面向连接；UDP是无连接的，即发送数据之前不需要建立连接；
2. TCP提供可靠数据传输，通过使用流量控制、序号、确认和定时器，TCP确保正确的、按序的将数据从发送进程交付给接收进程；UDP尽最大努力交付，即不保证可靠交付；
3. UDP具有较好的实时性，工作效率比TCP高，适用于对高速传输和实时性比较高的通信或广播通信；
4. 每一条TCP连接只能是点对点的，UDP支持一对一，一对多，多对一和多对多的交互通信
5. TCP对系统资源要求较多，UDP对系统资源要求较少。

- UDP应用场景
1. 面向数据报方式
2. 网络数据大多为短消息
3. 拥有大量Client
4. 对数据安全性无特殊要求
5. 网络负担非常重，但对响应速度要求高

- TCP如何提供可靠数据传输
1. 通过使用流量控制、序号、确认和定时器，TCP确保正确的、按序的将数据从发送进程交付给接收进程。