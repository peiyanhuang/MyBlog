---
layout: post
title:  理解 Web Worker
date:   2018-07-20 19:58:00 +0800
categories: JS
tag: JS
---

* content
{:toc}

### 1.Web Worker是什么

Web Worker 是 HTML5 标准的一部分，这一规范定义了一套 API，它允许一段 JavaScript 程序运行在主线程之外的另外一个线程中。

Web Worker 规范中定义了两类 workes，分别是专用 Worker 和共享 Worker。在专用 workers 的情况下，`DedicatedWorkerGlobalScope` 对象代表了 worker 的上下文（专用 workers 是指标准 worker 仅在单一脚本中被使用；共享 worke r的上下文是 `SharedWorkerGlobalScope` 对象）。一个专用 worker 仅仅能被首次生成它的脚本使用，而共享 worker 可以同时被多个脚本使用。

### 2.专用 worker 的使用

#### 2.1 生成专用 worker

为了更好的错误处理控制以及向下兼容，对 Worker 进行特性检测：

```js
if (window.Worker) {
  ...
}
```

创建一个新的 worker 很简单。需要做的是调用 `Worker()` 的构造器，指定一个脚本的 URI 来执行 worker 线程（main.js）：

```js
var myWorker = new Worker('worker.js');
```

注意：作为参数传递给 worker 构造器的 URI 必须遵循‘同源策略’。


#### 2.2 专用 worker 中消息的接收和发送

workers 通过 `postMessage()` 方法和 `onmessage` 事件收发消息。向一个 worker 发送消息需要这样做（main.js）：

```js
// postMessage 发送消息
$name.addEventListener("change", e => {
  worker.postMessage({name: $name.value, address: $address.value});
  console.log('Post Message!');
});
$address.addEventListener("change", e => {
  worker.postMessage({name: $name.value, address: $address.value});
  console.log('Post Message!');
});
```

这段代码中变量 $name 和 $address 代表2个 `<input>` 元素；它们当中任意一个的值发生改变时，`myWorker.postMessage()` 会将这2个值组成数组发送给 worker。

在 worker 中接收到消息后，我们可以写这样一个事件处理函数代码作为响应（worker.js）：

```js
onmessage = e => {
  console.log("Get Client Message: ", e);
  let data = e.data;
  if (data.address === "stop") {
    console.log("Worker: Worker close!");
    close();
  } else {
    let result = `name: ${data.name}, address: ${data.address}`;
    postMessage(result);
  }
};
```

`onmessage` 处理函数允许我们在任何时刻，一旦接收到消息就可以执行一些代码，代码中消息本身作为事件的 data 属性进行使用。

回到主线程，我们再次使用 `onmessage` 以响应 worker 回传的消息：

```js
// 接受消息
worker.addEventListener("message", e => {
  $message.innerHTML = e.data;
});
```

注意：

1) 在主线程中使用时，`onmessage` 和 `postMessage()` 必须挂在 `worker` 对象上，而在 `worker.js` 中使用时不用这样做。原因是，在 `worker` 内部，`worker` 是有效的全局作用域。例如：

2) 当一个消息在主线程和 worker 之间传递时，它被复制或者转移了，而不是共享。

#### 2.3 终止 worker

如果需要从主线程中立刻终止一个运行中的 worker，可以调用 `terminate()` 方法：

```js
worker.terminate();
```

worker 线程会被立即杀死，不会有任何机会让它完成自己的操作或清理工作。

而在 worker 线程中，workers 也可以调用自己的 `close()` 方法进行关闭。

#### 2.4 处理错误

当 worker 出现运行中错误时，它的 `onerror` 事件处理函数会被调用。它会收到一个扩展了 `ErrorEvent` 接口的名为 `error` 的事件。

该事件不会冒泡并且可以被取消；为了防止触发默认动作，worker 可以调用错误事件的 `preventDefault()` 方法。

错误事件有以下三个用户关心的字段：

- message：可读性良好的错误消息
- filename：发生错误的脚本文件名
- lineno：发生错误时所在脚本文件的行号

#### 2.5 Worker 环境

就 Worker 来说，`self` 和 `this` 指的都是 Worker 的全局作用域。因此，也可以写成：

```js
self.addEventListener('message', function(e) {
  var data = e.data;
  switch (data.cmd) {
    case 'start':
      self.postMessage('WORKER STARTED: ' + data.msg);
      break;
    case 'stop':
      self.postMessage('WORKER STOPPED: ' + data.msg + '. (buttons will no longer work)');
      self.close(); // Terminates the worker.
      break;
    default:
      self.postMessage('Unknown command: ' + data.msg);
  };
});
<!-- 或 -->
addEventListener('message', function(e) {
  var data = e.data;
  switch (data.cmd) {
    case 'start':
      postMessage('WORKER STARTED: ' + data.msg);
      break;
    case 'stop':
  ...
});
```

或者，可以直接设置 `onmessage` 事件处理程序。

```js
onmessage = function(e) {
  var data = e.data;
  ...
};
```

#### 2.6 适用于 Worker 的功能

由于 Web Worker 的多线程行为，所以它们只能使用 JavaScript 功能的子集：

```
navigator 对象
location 对象（只读）
XMLHttpRequest
setTimeout()/clearTimeout() 和 setInterval()/clearInterval()
应用缓存(Application Cache)
使用 importScripts() 方法导入外部脚本
生成其他 Web Worker
```

Worker 无法使用：

```
DOM
window 对象
document 对象
parent 对象
```

#### 2.7 加载外部脚本

可以通过 `importScripts()` 函数将外部脚本文件或库加载到 Worker 中。该方法采用零个或多个字符串表示要导入的资源的文件名。

此示例将 script1.js 和 script2.js 加载到了 Worker 中(worker.js)：

```js
importScripts('script1.js');
importScripts('script2.js');
<!-- 或者写成单个导入语句 -->
importScripts('script1.js', 'script2.js');
```

浏览器加载并运行每一个列出的脚本。每个脚本中的全局对象都能够被 worker 使用。如果脚本无法加载，将抛出 `NETWORK_ERROR` 异常，接下来的代码也无法执行。而之前执行的代码(包括使用 window.setTimeout() 异步执行的代码)依然能够运行。`importScripts()` 之后的函数声明依然会被保留，因为它们始终会在其他代码之前运行。

注意： 脚本的下载顺序不固定，但执行时会按照传入 `importScripts()` 中的文件名顺序进行。这个过程是同步完成的；直到所有脚本都下载并运行完毕，`importScripts()` 才会返回。

#### 2.8 子 Worker (subworker)

如果需要的话 worker 能够生成更多的 worker。这就是所谓的 `subworker`，它们必须托管在同源的父页面内。而且，subworker 解析 URI 时会相对于父 worker 的地址而不是自身页面的地址。这使得 worker 更容易记录它们之间的依赖关系。

```js
var num_workers = 10;
var items_per_worker = 1000000;

// start the workers
var result = 0;
var pending_workers = num_workers;
for (var i = 0; i < num_workers; i += 1) {
  let workers = new Worker('./core.js');
  workers.postMessage(i * items_per_worker);
  workers.postMessage((i + 1) * items_per_worker);
  workers.onmessage = storeResult;
}

// handle the results
function storeResult(event) {
  result += event.data;
  pending_workers -= 1;
  if (pending_workers <= 0)
    postMessage(result); // finished!
}
```

注意：Chrome 有一个 bug 加载子 worker 会报错。可以用 Firefox 测试。

#### 2.9 内嵌 Worker

如果想即时创建 Worker 脚本，或者在不创建单独 Worker 文件的情况下创建独立网页，那该怎么做呢？在新 [BlobBuilder](https://dev.w3.org/2009/dap/file-system/file-writer.html#the-blobbuilder-interface) 中，可以创建 `BlobBuilder` 并以字符串形式附上 Worker 代码，从而在与主逻辑相同的 HTML 文件中“内嵌”Worker：

```js
// Prefixed in Webkit, Chrome 12, and FF6: window.WebKitBlobBuilder, window.MozBlobBuilder
var bb = new BlobBuilder();
bb.append("onmessage = function(e) { postMessage('msg from worker'); }");

// Obtain a blob URL reference to our worker 'file'.
// Note: window.webkitURL.createObjectURL() in Chrome 10+.
var blobURL = window.URL.createObjectURL(bb.getBlob());

var worker = new Worker(blobURL);
worker.onmessage = function(e) {
  // e.data == 'msg from worker'
};
worker.postMessage(); // Start the worker.
```

对于结束不在需要的 worker 可以通过 `window.URL.revokeObjectURL(blobURL)` 释放。

### 2. 共享worker

一个共享 worker 可以被多个脚本使用——即使这些脚本正在被不同的 window、iframe 或者 worker 访问。

生成一个新的共享 worker 与生成一个专用 worker 非常相似，只是构造器的名字不同:

```js
var myWorker = new SharedWorker('worker.js');
```

一个非常大的区别在于，与一个共享 worker 通信必须通过端口对象——一个确切的打开的端口供脚本与 worker 通信（在专用worker中这一部分是隐式进行的）。

在传递消息之前，端口连接必须被显式的打开，打开方式是使用 `onmessage` 事件处理函数或者 `start()` 方法。`start()` 方法的调用只在一种情况下需要，那就是消息事件被 `addEventListener()` 方法使用。

在使用 `start()` 方法打开端口连接时，如果父级线程和 worker 线程需要双向通信，那么它们都需要调用 `start()` 方法。

```js
myWorker.port.start();  // 父级线程中的调用
port.start(); // worker线程中的调用, 假设port变量代表一个端口
```

现在，消息可以像之前那样发送到 worker 了，但是 `postMessage()` 方法必须被端口对象调用：

```js
squareNumber.onchange = function() {
  myWorker.port.postMessage([squareNumber.value,squareNumber.value]);
  console.log('Message posted to worker');
}
```

回到 worker 中，这里也有一些些复杂（worker.js）:

```js
onconnect = function(e) {
  var port = e.ports[0];

  port.onmessage = function(e) {
    var workerResult = 'Result: ' + (e.data[0] * e.data[1]);
    port.postMessage(workerResult);
  }
}
```

首先，当一个端口连接被创建时（例如：在父级线程中，设置 `onmessage` 事件处理函数，或者显式调用 `start()` 方法时），使用 `onconnect` 事件处理函数来执行代码。

使用事件的 `ports` 属性来获取端口并存储在变量中。

然后，为端口添加一个消息处理函数用来做运算并回传结果给主线程。在worker线程中设置此消息处理函数也会隐式的打开与主线程的端口连接，因此这里跟前文一样，对 `port.start()` 的调用也是不必要的。

最后，回到主脚本，我们处理消息:

```js
myWorker.port.onmessage = function(e) {
  result2.textContent = e.data;
  console.log('Message received from worker');
}
```

当一条消息通过端口回到 worker，我们检查结果的类型，然后将运算结果放入结果段落中合适的地方。

### 参考资料

[Web Workers 的基本信息](https://www.html5rocks.com/zh/tutorials/workers/basics/)

[MDN 使用 Web Workers](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers)