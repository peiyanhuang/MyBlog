---
layout: post
title:  JavaScript中的 Event Loop（事件循环）机制
date:   2019-03-18 19:00:00 +0800
categories: 开发者
tag: 开发者
---

* content
{:toc}

我们都知道，Javascript 从诞生之日起就是一门单线程的非阻塞的脚本语言。这是由其最初的用途来决定的：与浏览器交互。

单线程意味着，Javascript 代码在执行的任何时候，都只有一个主线程来处理所有的任务。单线程是必要的，也是 Javascript 这门语言的基石，原因之一在其最主要的执行环境——浏览器中，我们需要进行各种各样的 dom 操作。试想一下 如果 Javascript 是多线程的，那么当两个线程同时对同一 dom 进行操作，例如一个向其添加事件，而另一个删除了这个 dom，此时该如何处理？

而非阻塞则是当代码需要进行一项异步任务（无法立刻返回结果，需要花一定时间才能返回的任务，如I/O事件）的时候，主线程会挂起（pending）这个任务，然后在异步任务返回结果的时候再根据一定规则去执行相应的回调。

### 1.执行栈和任务队列

当 Javascript 代码执行的时候会将不同的变量存于内存中的不同位置：堆（heap）和栈（stack）中来加以区分。其中，堆里存放着一些对象。而栈中则存放着一些基础类型变量以及对象的指针。 但是我们这里说的执行栈和上面这个栈的意义却有些不同。

我们知道，当我们调用一个方法的时候，js会生成一个与这个方法对应的执行环境（context），又叫执行上下文。这个执行环境中存在着这个方法的私有作用域、上层作用域的指向、方法的参数、这个作用域中定义的变量以及这个作用域的 this 对象。 而当一系列方法被依次调用的时候，因为 js 是单线程的，同一时间只能执行一个方法，于是这些方法被排队在一个单独的地方。这个地方被称为`执行栈`(execution context stack)。

当一个脚本第一次执行的时候，js 引擎会解析这段代码，并将其中的同步代码按照执行顺序加入执行栈中，然后从头开始执行。如果当前执行的是一个方法，那么 js 会向执行栈中添加这个方法的执行环境，然后进入这个执行环境继续执行其中的代码。当这个执行环境中的代码执行完毕并返回结果后，js 会退出这个执行环境并把这个执行环境销毁，回到上一个方法的执行环境。这个过程反复进行，直到执行栈中的代码全部执行完毕。

以上的过程说的都是同步代码的执行。那么当一个异步代码（如发送 ajax 请求数据）执行后会如何呢？这就需要任务队列（Task Queue）了。

具体来说，异步执行的运行机制如下。（同步执行也是如此，因为它可以被视为没有异步任务的异步执行。）

1. 所有同步任务都在主线程上执行，形成一个执行栈；
2. 主线程之外，还存在一个"任务队列"。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件；
3. 一旦"执行栈"中的所有同步任务执行完毕，主线程处于闲置状态时，主线程会去查找任务队列是否有任务；如果有，那么主线程会从中取出排在第一位的事件，并把这个事件对应的回调放入执行栈中，然后执行其中的同步代码；
4. 主线程不断重复上面的第三步；

如此反复，这样就形成了一个无限的循环。这就是这个过程被称为`事件循环`（Event Loop）的原因。

### macro task 与 micro task

以上的事件循环过程是一个宏观的表述，实际上因为异步任务之间并不相同，因此他们的执行优先级也有区别。不同的异步任务被分为两类：微任务（micro task）和宏任务（macro task）。

以下事件属于宏任务：

* setInterval()
* setTimeout()
* setImmediate (Node独有)
* requestAnimationFrame (浏览器独有)
* I/O
* UI rendering (浏览器独有)

以下事件属于微任务，也叫 jobs:

* new Promise()
* new MutaionObserver()
* Object.observe
* process.nextTick (Node独有)

在一个事件循环中，异步事件返回结果后会被放到一个任务队列中。然而，根据这个异步事件的类型，这个事件实际上会被对应的宏任务队列或者微任务队列中去。并且在当前执行栈为空的时候，主线程会查看微任务队列是否有事件存在。如果不存在，那么再去宏任务队列中取出一个事件并把对应的回到加入当前执行栈；如果存在，则会依次执行队列中事件对应的回调，直到微任务队列为空，然后去宏任务队列中取出最前面的一个事件，把对应的回调加入当前执行栈...如此反复，进入循环。具体流程如下：

1. 执行全局 Script 同步代码，这些同步代码有一些是同步语句，有一些是异步语句（比如setTimeout等）；
2. 全局 Script 代码执行完毕后，调用栈 Stack 会清空；
3. 从微队列 microtask queue 中取出位于队首的回调任务，放入调用栈 Stack 中执行，执行完后 microtask queue 长度减1；
4. 继续取出位于队首的任务，放入调用栈Stack中执行，以此类推，直到直到把microtask queue中的所有任务都执行完毕。**注意，如果在执行 microtask 的过程中，又产生了 microtask，那么会加入到队列的末尾，也会在这个周期被调用执行**；
5. microtask queue 中的所有任务都执行完毕，此时 microtask queue 为空队列，调用栈 Stack 也为空；
6. 取出宏队列 macrotask queue 中位于队首的任务，放入 Stack 中执行；
7. 执行完毕后，调用栈 Stack 为空；
8. 重复第3-7个步骤；
9. ...

### 3.Node环境下的事件循环机制

在 NodeJS 中，事件循环表现出的状态与浏览器中大致相同。不同的是 Node 中有一套自己的模型。Node 中事件循环的实现是依靠的 libuv。

Node 的 Event Loop 中，执行宏队列的回调任务有 6 个阶段，如下图：

![image]({{ '/images/web/node_event_loop.png' | prepend: site.baseurl }})

各个阶段执行的任务如下：

* timers阶段：这个阶段执行 `setTimeout` 和 `setInterval` 预定的 callback
* I/O callback阶段：执行除了 `close` 事件的 callbacks、被 timers 设定的 callbacks、setImmediate() 设定的 callbacks 这些之外的 callbacks
* idle, prepare阶段：仅 node 内部使用
* poll阶段：获取新的 I/O 事件，适当的条件下 node 将阻塞在这里
* check阶段：执行 `setImmediate()` 设定的 callbacks
* close callbacks阶段：执行 `socket.on('close', ....)` 这些 callbacks

NodeJS 中宏队列主要有4个，由上面的介绍可以看到，回调事件主要位于4个 macrotask queue 中：

* Timers Queue
* IO Callbacks Queue
* Check Queue
* Close Callbacks Queue

这4个都属于宏队列，但是在浏览器中，可以认为只有一个宏队列，所有的 macrotask 都会被加到这一个宏队列中，但是在 NodeJS 中，不同的 macrotask 会被放置在不同的宏队列中。

NodeJS 中微队列主要有2个：

* Next Tick Queue：是放置 process.nextTick(callback) 的回调任务的
* Other Micro Queue：放置其他 microtask，比如 Promise 等

大体解释一下NodeJS的Event Loop过程：

1. 执行全局Script的同步代码
2. 执行microtask微任务，先执行所有Next Tick Queue中的所有任务，再执行Other Microtask Queue中的所有任务
3. 开始执行macrotask宏任务，共6个阶段，从第1个阶段开始执行相应每一个阶段macrotask中的所有任务，注意，这里是所有每个阶段宏任务队列的所有任务
4. Timers Queue -> 步骤2 -> I/O Queue -> 步骤2 -> Check Queue -> 步骤2 -> Close Callback Queue -> 步骤2 -> Timers Queue
5. ......