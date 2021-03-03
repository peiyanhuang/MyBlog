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
- [浏览器地址栏里输入URL后的全过程](https://juejin.cn/post/6844903757877084174)
- [webpack模块化原理-异步加载模块](https://zhuanlan.zhihu.com/p/88332125?utm_source=wechat_session)

### Vue

- [Vue 数据的双向绑定？](https://peiyanhuang.github.io/MyBlog/2018/10/28/vue-source-code-4/)

observer walk,observeArray -> defineReactive

initLifecycle(vm)
initEvents(vm)
initRender(vm)
callHook(vm, 'beforeCreate')
initInjections(vm) // resolve injections before data/props
initState(vm)
initProvide(vm) // resolve provide after data/props
callHook(vm, 'created')

getter 中 dep 的 depend() 方法收集依赖（调用 dep.target 即 watcher 的 addDep()方法。）dep.addSub() 把watcher push 到 dep.subs 里。

setter 中 dep.notify() 触发依赖。dep.subs 中的 watcher 的 update() 方法重新求值。

整个 Vue 渲染过程，前面我们说了 complie 的过程，在做完 parse、optimize 和 generate 之后，我们得到了一个 render 函数字符串。
那么接下来 Vue 做的事情就是 new watcher，这个时候会对绑定的数据执行监听，render 函数就是数据监听的回调所调用的，其结果便是重新生成 vnode。当这个 render 函数字符串在第一次 mount、或者绑定的数据更新的时候，都会被调用，生成 Vnode。如果是数据的更新，那么 Vnode 会与数据改变之前的 Vnode 做 diff，对内容做改动之后，就会更新到我们真正的 DOM 上啦~

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

- [7层网络和5层网络](https://www.cnblogs.com/qishui/p/5428938.html)

- get post 区别
GET在浏览器回退时是无害的，而POST会再次提交请求。
GET产生的URL地址可以被Bookmark，而POST不可以。
GET请求会被浏览器主动cache，而POST不会，除非手动设置。
GET请求只能进行url编码，而POST支持多种编码方式。
GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。
GET请求在URL中传送的参数是有长度限制的，而POST么有。
对参数的数据类型，GET只接受ASCII字符，而POST没有限制。
GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。
GET参数通过URL传递，POST放在Request body中。


- 进程线程区别
根本区别：进程是操作系统资源分配的基本单位，而线程是任务调度和执行的基本单位

在开销方面：每个进程都有独立的代码和数据空间（程序上下文），程序之间的切换会有较大的开销；线程可以看做轻量级的进程，同一类线程共享代码和数据空间，每个线程都有自己独立的运行栈和程序计数器（PC），线程之间切换的开销小。

所处环境：在操作系统中能同时运行多个进程（程序）；而在同一个进程（程序）中有多个线程同时执行（通过CPU调度，在每个时间片中只有一个线程执行）

内存分配方面：系统在运行的时候会为每个进程分配不同的内存空间；而对线程而言，除了CPU外，系统不会为线程分配内存（线程所使用的资源来自其所属进程的资源），线程组之间只能共享资源。

包含关系：没有线程的进程可以看做是单线程的，如果一个进程内有多个线程，则执行过程不是一条线的，而是多条线（线程）共同完成的；线程是进程的一部分，所以线程也被称为轻权进程或者轻量级进程。
