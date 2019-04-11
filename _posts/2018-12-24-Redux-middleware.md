---
layout: post
title:  Redux -- Middleware
date:   2018-12-24 19:58:00 +0800
categories: React
tag: React
---

* content
{:toc}

一个 Redux 中简单的同步数据流动场景：点击 button，在回调中分发一个 `action`，`reducer` 收到 `action` 后更新 `state` 并通知 `view` 重新渲染。单向数据流，看着很简单，但如果需要打印每一个 `action` 信息来调试，就得去修改 `dispatch` 或者 `reducer` 类似实现；又比如想要 `dispatch` 或 `reducer` 拥有异步请求的功能等等。

Redux middleware 与 Express 或者 Koa 中的概念是类似的。它提供的是位于 `action` 被发起之后，到达 `reducer` 之前的扩展点。 可以利用 Redux middleware 来进行日志记录、创建崩溃报告、调用异步接口或者路由等等。

### 1. 理解 Middleware

Redux 提供了 `applyMiddleware` 来加载 Middleware，源码如下：

```js
import compose from './compose'

/**
 * @param {...Function} middlewares The middleware chain to be applied.
 * @returns {Function} A store enhancer applying the middleware.
 */
export default function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStore(...args)
    let dispatch = () => {
      throw new Error(
        `Dispatching while constructing your middleware is not allowed. ` +
          `Other middleware would not be applied to this dispatch.`
      )
    }

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```

middleware 设计包含了函数式编程中的[柯里化(Currying)](https://peiyanhuang.github.io/MyBlog/2018/04/27/adequate/#3%E6%9F%AF%E9%87%8C%E5%8C%96curry)。与闭包类似，将多参数函数转化为单参数函数并延迟执行函数。

* `middlewares`：middleware 数组;
* `createStore`：Redux 原生的 createStore;

**说明**：`middlewareAPI` 中 `dispatch` 为什么要用匿名函数包裹？`applyMiddleware` 执行完后，`dispatch` `是变化的，middlewareAPI` 要分发到每一个 `middleware` 中，用匿名函数包裹 `dispatch`，这样只要 `dispatch` 更新了，`middlewareAPI` 中 `dispatch` 也会发生改变。

来看看 `logger middleware` 的实现：

```js
export default store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}
```

`const chain = middlewares.map(middleware => middleware(middlewareAPI))` 让每个 middleware 以 `middlewarePI` 为参数都执行一遍。`chain` 数组保存 middleware 的第二个（next）箭头函数返回的匿名函数，它们都含有相同的 `store`，即 `middlewareAPI`。

在 middleware 中调用 `next()` 而不是调用 `store.dispatch()`，是因为调用 `next()` 会进入下一个 middleware，而某个 middleware 使用 `store.dispatch()` 来分发 `action` 会重来一遍整个过程，如果不做判断，可能形成无限循环。

`dispatch = compose(...chain)(store.dispatch)` 其中 `compose` 是函数式编程中的组合，Redux 中定义如下：

```js
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

例如 `chain = [f1, f2, f3]`，则 `compose(...chain)` 返回如下：

```js
(...args) => {
    return f1(f2(f3(...args)))
}
```

即 `dispatch` 为 `f1(f2(f3(store.dispatch)))`。

### 2. Redux 异步流

默认情况下，`createStore()` 所创建的 Redux store 没有使用 middleware，所以只支持同步数据流。异步的数据流需要 middleware。

middleware 如何使用呢？

```js
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from './reducers';

const store = createStore(
  rootReducer,
  applyMiddleware(thunk)
);
```

#### 2.1 redux-thunk

`action` 创建函数除了返回 `action` 对象外还可以返回函数。这时，这个 `action` 创建函数就成为了 `thunk`。当 `action` 创建函数返回函数时，这个函数会被 `Redux Thunk middleware` 执行。这个函数并不需要保持纯净；它还可以带有副作用，包括执行异步 API 请求。这个函数还可以 `dispatch action`，就像 `dispatch` 前面定义的同步 `action` 一样。

redux-thunk 源码很简单：

```js
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```

当 `action` 为函数时，返回 `action` 的调用。这里的 `action` 即为一个 Trunk。什么是 Trunk？

Trunk 函数实现上就是针对多参数的 currying 以实现对函数的惰性求值。例如：

```js
let trunk = function(fileName) {
  return function(callback) {
    return fs.readFile(fileName, callback);
  }
}
let readFileTrunk = trunk(fileName);
readFileTrunk(callback);
```

任何函数，只要参数有回调函数，都能写成 Trunk 函数的形式。

那 `action create` 该如何写：

```js
function makeASandwichWithSecretSauce(forPerson) {
  return function (dispatch) {
    return fetch('https://www.google.com/search?q=secret+sauce').then(
      sauce => dispatch(makeASandwich(forPerson, sauce)),
      error => dispatch(apologize('The Sandwich Shop', forPerson, error))
    );
  };
}

store.dispatch(makeASandwichWithSecretSauce('Me'));
```

#### 2.2 redux-promise

异步请求都是利用 promise 来完成的，通过抽象 promise 的方法也可以解决异步流的问题。

```js
import isPromise from 'is-promise';
import { isFSA } from 'flux-standard-action';

export default function promiseMiddleware({ dispatch }) {
  return next => action => {
    if (!isFSA(action)) {
      return isPromise(action) ? action.then(dispatch) : next(action);
    }

    return isPromise(action.payload)
      ? action.payload
          .then(result => dispatch({ ...action, payload: result }))
          .catch(error => {
            dispatch({ ...action, payload: error, error: true });
            return Promise.reject(error);
          })
      : next(action);
  };
}
```

实现很简单，就不多说了。