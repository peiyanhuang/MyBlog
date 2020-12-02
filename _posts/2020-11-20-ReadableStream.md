---
layout: post
title:  游览器中的 ReadableStream
date:   2020-11-19 19:00:00 +0800
categories: JS
tag: JS
---

* content
{:toc}

`ReadableStream` 是流操作API，呈现了一个可读取的二进制流操作。Fetch API 通过 Response 的属性 body 提供了一个具体的 `ReadableStream` 对象。

### 从 ReadableStream 中读取数据

`ReadableStream` 实例上提供了如下的方法和属性：

- `closed`
- `cancel()`
- `read()`
- `releaseLock()`

假设我们需要读取一个流中的的数据，可以循环调用 `reader` 的 `read()` 方法，它会返回一个 `Promise` 对象，在 `Promise` 中返回一个包含 `value` 参数和 `done` 参数的对象。

```js
const reader = stream.getReader();
let bytesReceived = 0;
const processData = (result) => {
  if (result.done) {
    console.log(`complete, total size: ${bytesReceived}`);
    return;
  }
  const value = result.value; // Uint8Array
  const length = value.length;
  console.log(`got ${length} bytes data:`, value);
  bytesReceived += length;
  // 读取下一个文件片段，重复处理步骤
  return reader.read().then(processData);
};
reader.read().then(processData);
```

其中 `result.value` 参数为这次读取得到的片段，它是一个 `Uint8Array`，通过循环调用 `reader.read()` 方法就能一点点地获取流的整个数据；而 `result.done` 参数负责表明这个流是否已经读取完毕，当 `result.done` 为 `true` 时表明流已经关闭，不会再有新的数据，此时 `result.value` 的值为 `undefined`.

* 所以获取 fetch 请求的传输进度：可以通过读取 `Response` 中的流得到正在接收的文件片段，累加各个片段的 `length` 就能得到类似 `XHR onprogress` 事件的 `loaded`，也就是已下载的字节数；通过从 `Response` 的 `headers` 中取出 `Content-Length` 就能得到类似 `XHR onprogress` 事件的 `total`，也就是总字节数。同时构造返回一个 `Response`，保证返回一致。

```js
const logProgress = (res) => {
  const total = res.headers.get("content-length");
  let loaded = 0;
  const reader = res.body.getReader();
  const stream = new ReadableStream({
    start(controller) {
      const push = () => {
        reader.read().then(({ value, done }) => {
          if (done) {
            controller.close();
            return;
          }
          loaded += value.length;
          if (total === null) {
            console.log(`Downloaded ${loaded}`);
          } else {
            console.log(
              `Downloaded ${loaded} of ${total} (${(loaded / total * 100).toFixed(2)}%)`
            );
          }
          controller.enqueue(value);
          push();
        });
      };
      push();
    },
  });
  return new Response(stream, { headers: res.headers });
};
fetch("/foo")
  .then(logProgress)
  .then((res) => res.json());
```

或者也可以使用 `tee()` 方法：

```js
const logProgress = (res) => {
  const total = res.headers.get("content-length");
  let loaded = 0;
  const [progressStream, returnStream] = res.body.tee();
  const reader = progressStream.getReader();
  const log = () => {
    reader.read().then(({ value, done }) => {
      if (done) return;
      // 省略输出进度
      log();
    });
  };
  log();
  return new Response(returnStream, { headers: res.headers });
};
fetch("/foo")
  .then(logProgress)
  .then((res) => res.json());
```

* 当然，`Response` 还有个 `clone()` 方法，可以克隆一个实例，我们可以克隆使用一个返回一个，也是一样的。

### 中断一个 ReadableStream

<!-- 更新 -->

JavaScript 规范中添加了新的 `AbortController`，允许开发人员使用信号中止一个或多个 fetch 调用。
以下是取消 fetch 调用的工作流程：

1. 创建一个 `AbortController` 实例
2. 该实例具有 `signal` 属性
3. 将 `signal` 传递给 `fetch option`  的 `signal`
4. 调用 `AbortController` 的 `abort` 属性来取消所有使用该信号的 fetch。

```js
const controller = new AbortController();
const { signal } = controller;

fetch("http://localhost:8000", { signal }).then(response => {
  console.log(`Request 1 is complete!`);
}).catch(e => {
  console.warn(`Fetch 1 error: ${e.message}`);
}).catch(e => {
  if(e.name === "AbortError") {
    // We know it's been canceled!
  }
});

// Abort request
controller.abort();
```

* 将相同的信号 `signal `传递给多个 fetch 调用将会取消该信号的所有请求.

<!-- 以前 -->

通过 `ReadableStream` 上的 `cancel()` 方法，我们可以关闭这个流。此外你可能也注意到 `reader` 上也有一个 `cancel()` 方法，这个方法的作用是关闭与这个 `reader` 相关联的流，所以从结果上来看，两者是一样的。而对于 `Fetch API` 来说，关闭返回的 `Response` 对象的流的结果就相当于中断了这个请求。

所以，我们可以像之前那样构造一个 `ReadableStream` 用于传递从 `res.body.getReader()` 中得到的数据，并对外暴露一个 `aborter()` 方法。调用这个 `aborter()` 方法时会调用 `reader.cancel()` 关闭 fetch 请求返回的流，然后调用 `controller.error()` 抛出错误，中断构造出来的传递给后续操作的流：

```js
let aborter = null;
const abortHandler = (res) => {
  const reader = res.body.getReader();
  const stream = new ReadableStream({
    start(controller) {
      let aborted = false;
      const push = () => {
        reader.read().then(({ value, done }) => {
          if (done) {
            if (!aborted) controller.close();
            return;
          }
          controller.enqueue(value);
          push();
        });
      };
      aborter = () => {
        reader.cancel();
        controller.error(new Error("Fetch aborted"));
        aborted = true;
      };
      push();
    },
  });
  return new Response(stream, { headers: res.headers });
};
fetch("/foo")
  .then(abortHandler)
  .then((res) => res.json());
aborter();
```

### 流的锁机制

既然流本身就有一个 `cancel()` 方法，为什么我们不直接暴露这个方法，反而要绕路构造一个新的 `ReadableStream` 呢？例如像下面这样：

```js
let aborter = null;
const abortHandler = (res) => {
  aborter = () => res.body.cancel();
  return res;
};
fetch("/foo")
  .then(abortHandler)
  .then((res) => res.json());
aborter();
```

可惜这样执行会得到下面的错误：这个流被锁了。

> TypeError: Failed to execute 'cancel' on 'ReadableStream': Cannot cancel a locked stream

一个流只能同时有一个处于活动状态的 `reader`，当一个流被一个 `reader` 使用时，这个流就被该 `reader` 锁定了，此时流的 `locked` 属性为 `true`。如果这个流需要被另一个 `reader` 读取，那么当前处于活动状态的 `reader` 可以调用 `reader`.`releaseLock()` 方法释放锁。此外 `reader` 的 `closed` 属性是一个 `Promise`，当 `reader` 被关闭或者释放锁时，这个 `Promise` 会被 `resolve`，可以在这里编写关闭 `reader` 的处理逻辑：

```js
reader.closed.then(() => {
  console.log('reader closed');
});
reader.releaseLock();
```

### 参考

- [ReadableStream MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/ReadableStream)
- [从 Fetch 到 Streams —— 以流的角度处理网络请求](https://copyfuture.com/blogs-details/20191223092053446yks8rifdoxsqpu4#%E5%8F%82%E8%80%83%E7%AD%94%E6%A1%88)
