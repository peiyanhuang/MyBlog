---
layout: post
title:  Vue 源码学习-Vue实例(2)
date:   2018-10-24 19:58:00 +0800
categories: Vue
tag: Vue
---

* content
{:toc}

### 1. 用于初始化的最终选项 $options

上一篇那么多讲的都是 `mergeOptions` 函数是如何对父子选项进行合并处理的。在 `_init` 方法中被调用：

```js
vm.$options = mergeOptions(
    resolveConstructorOptions(vm.constructor),
    options || {},
    vm
)
```

`mergeOptions` 函数最终将合并处理后的选项返回，并以该返回值作为 `vm.$options` 的值。`vm.$options` 在 Vue 的官方文档中是可以找到的，它作为实例属性暴露给开发者，那么现在你应该知道 `vm.$options` 到底是什么了。并且看文档的时候你应该更能够理解其作用，比如官方文档是这样介绍 `$options` 实例属性的：

用于当前 Vue 实例的初始化选项。需要在选项中包含自定义属性时会有用处

并且给了一个例子，如下：

```js
new Vue({
  customOption: 'foo',
  created: function () {
    console.log(this.$options.customOption) // => 'foo'
  }
})
```

上面的例子中，在创建 Vue 实例的时候传递了一个自定义选项：`customOption`，在之后的代码中我们可以通过 `this.$options.customOption` 进行访问。那原理其实就是使用 `mergeOptions` 函数对自定义选项进行合并处理，由于没有指定 `customOption` 选项的合并策略，所以将会使用默认的策略函数 `defaultStrat`。最终效果就是你初始化的值是什么，得到的就是什么。

Vue 也提供了 `Vue.config.optionMergeStrategies` 全局配置，大家也可以在官方文档中找到，我们知道这个对象其实就是选项合并中的策略对象，所以我们可以通过他指定某一个选项的合并策略，常用于指定自定义选项的合并策略，比如我们给 `customOption` 选项指定一个合并策略，只需要在 `Vue.config.optionMergeStrategies` 上添加与选项同名的策略函数即可：

```js
Vue.config.optionMergeStrategies.customOption = function (parentVal, childVal) {
  return parentVal ? (parentVal + childVal) : childVal
}
```

如上代码中，我们添加了自定义选项 `customOption` 的合并策略，其策略为：如果没有 `parentVal` 则直接返回 `childVal`，否则返回两者的和。

### 2. 渲染函数的作用域代理

`_init` 方法中，在经过 `mergeOptions` 合并处理选项之后，要执行的是下面这段代码：

```js
/* istanbul ignore else */
if (process.env.NODE_ENV !== 'production') {
  initProxy(vm)
} else {
  vm._renderProxy = vm
}
```

`initProxy()` 如下：

```js
import config from 'core/config'
import { warn, makeMap } from '../util/index'

// 声明 initProxy 变量
let initProxy

if (process.env.NODE_ENV !== 'production') {
  // ... 其他代码
  
  // 在这里初始化 initProxy
  initProxy = function initProxy (vm) {
    if (hasProxy) {
      // determine which proxy handler to use
      const options = vm.$options
      const handlers = options.render && options.render._withStripped
        ? getHandler
        : hasHandler
      vm._renderProxy = new Proxy(vm, handlers)
    } else {
      vm._renderProxy = vm
    }
  }
}

// 导出
export { initProxy }
```

这个函数的主要作用其实就是在实例对象 `vm` 上添加 `_renderProxy` 属性。对于 `hasProxy` 顾名思义，这是用来判断宿主环境是否支持 js 原生的 `Proxy` 特性的。

`options.render._withStripped` 这个属性只在测试代码中出现过，所以一般情况下这个条件都会为假。

```js
const hasHandler = {
  has (target, key) {
    // has 常量是真实经过 in 运算符得来的结果
    const has = key in target
    // 如果 key 在 allowedGlobals 之内，或者 key 是以下划线 _ 开头的字符串，则为真
    const isAllowed = allowedGlobals(key) || (typeof key === 'string' && key.charAt(0) === '_')
    // 如果 has 和 isAllowed 都为假，使用 warnNonPresent 函数打印错误
    if (!has && !isAllowed) {
        warnNonPresent(target, key)
    }
    return has || !isAllowed
  }
}
```

### 3. initLifecycle

`_init` 函数在执行完 `initProxy` 之后，执行的就是 `initLifecycle` 函数:

```js
export function initLifecycle (vm: Component) {
  // 定义 options，它是 vm.$options 的引用，后面的代码使用的都是 options 常量
  const options = vm.$options

  // locate first non-abstract parent
  // 定义 parent，它引用当前实例的父实例
  let parent = options.parent
  // 如果当前实例有父组件，且当前实例不是抽象的
  if (parent && !options.abstract) {
    // 使用 while 循环查找第一个非抽象的父组件
    while (parent.$options.abstract && parent.$parent) {
      parent = parent.$parent
    }
    parent.$children.push(vm)
  }

  vm.$parent = parent
  vm.$root = parent ? parent.$root : vm

  vm.$children = []
  vm.$refs = {}

  vm._watcher = null
  vm._inactive = null
  vm._directInactive = false
  vm._isMounted = false
  vm._isDestroyed = false
  vm._isBeingDestroyed = false
}
```

首先定义 `options` 常量，它是 `vm.$options` 的引用。将当前实例添加到父实例的 `$children` 属性里，并设置当前实例的 `$parent` 指向父实例。

什么是抽象的实例？实际上 Vue 内部有一些选项是没有暴露给我们的，就比如这里的 `abstract`，通过设置这个选项为 `true`，可以指定该组件是抽象的，那么通过该组件创建的实例也都是抽象的，比如：

```js
AbsComponents = {
  abstract: true,
  created () {
    console.log('我是一个抽象的组件')
  }
}
```

抽象的组件有什么特点呢？一个最显著的特点就是它们一般不渲染真实DOM，这么说大家可能不理解，我举个例子大家就明白了，我们知道 Vue 内置了一些全局组件比如 `keep-alive` 或者 `transition`，我们知道这两个组件它是不会渲染DOM至页面的，但他们依然给我提供了很有用的功能。所以他们就是抽象的组件，我们可以查看一下它的源码，打开 `core/components/keep-alive.js` 文件，你能看到这样的代码：

```js
export default {
  name: 'keep-alive',
  abstract: true,
  ...
}
```

可以发现，它使用 `abstract` 选项来声明这是一个抽象组件。除了不渲染真实DOM，抽象组件还有一个特点，就是它们不会出现在父子关系的路径上。这么设计也是合理的，这是由它们的性质所决定的。

### 4. initEvents

```js
export function initEvents (vm: Component) {
  vm._events = Object.create(null)
  vm._hasHookEvent = false
  // init parent attached events
  const listeners = vm.$options._parentListeners
  if (listeners) {
    updateComponentListeners(vm, listeners)
  }
}
```

首先在 vm 实例对象上添加两个实例属性 _events 和 _hasHookEvent，其中 _events 被初始化为一个空对象，_hasHookEvent 的初始值为 false。

`vm.$options._parentListeners` 这个 `_parentListeners` 是哪里来的？我们之前看过一个函数叫做 `createComponentInstanceForVnode`，他在 `core/vdom/create-component.js` 文件中，在创建子组件实例的时候才会有这个参数选项。

### 5. initRender

```js
export function initRender (vm: Component) {
  vm._vnode = null // the root of the child tree
  vm._staticTrees = null // v-once cached trees
  const options = vm.$options
  const parentVnode = vm.$vnode = options._parentVnode // the placeholder node in parent tree
  const renderContext = parentVnode && parentVnode.context
  vm.$slots = resolveSlots(options._renderChildren, renderContext)
  vm.$scopedSlots = emptyObject
  // bind the createElement fn to this instance
  // so that we get proper render context inside it.
  // args order: tag, data, children, normalizationType, alwaysNormalize
  // internal version is used by render functions compiled from templates
  vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
  // normalization is always applied for the public version, used in
  // user-written render functions.
  vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)

  // $attrs & $listeners are exposed for easier HOC creation.
  // they need to be reactive so that HOCs using them are always updated
  const parentData = parentVnode && parentVnode.data

  /* istanbul ignore else */
  if (process.env.NODE_ENV !== 'production') {
    defineReactive(vm, '$attrs', parentData && parentData.attrs || emptyObject, () => {
      !isUpdatingChildComponent && warn(`$attrs is readonly.`, vm)
    }, true)
    defineReactive(vm, '$listeners', options._parentListeners || emptyObject, () => {
      !isUpdatingChildComponent && warn(`$listeners is readonly.`, vm)
    }, true)
  } else {
    defineReactive(vm, '$attrs', parentData && parentData.attrs || emptyObject, null, true)
    defineReactive(vm, '$listeners', options._parentListeners || emptyObject, null, true)
  }
}
```

### 6. 生命周期钩子的实现方式

在 `initRender` 函数执行完毕后，是这样一段代码：

```js
callHook(vm, 'beforeCreate')
initInjections(vm) // resolve injections before data/props
initState(vm)
initProvide(vm) // resolve provide after data/props
callHook(vm, 'created')
```

可以发现，`initInjections(vm)`、`initState(vm)` 以及 `initProvide(vm)` 被包裹在两个 `callHook` 函数调用的语句中。那么 `callHook` 函数的作用是什么呢？正如它的名字一样，`callHook` 函数的作用是调用生命周期钩子函数。根据引用关系可知 `callHook` 函数来自于 `lifecycle.js` 文件，打开该文件找到 `callHook` 函数如下：

```js
export function callHook (vm: Component, hook: string) {
  // #7573 disable dep collection when invoking lifecycle hooks
  pushTarget()
  const handlers = vm.$options[hook]
  if (handlers) {
    for (let i = 0, j = handlers.length; i < j; i++) {
      try {
        handlers[i].call(vm)
      } catch (e) {
        handleError(e, vm, `${hook} hook`)
      }
    }
  }
  if (vm._hasHookEvent) {
    vm.$emit('hook:' + hook)
  }
  popTarget()
}
```

以上是 `callHook` 函数的全部代码，它接收两个参数：实例对象和要调用的生命周期钩子的名称。接下来我们就看看 `callHook` 是如何实现的。

大家可能注意到了 `callHook` 函数体的代码以 `pushTarget()` 开头，并以 `popTarget()` 结尾，这里我们暂且不讲这么做的目的，这其实是为了避免在某些生命周期钩子中使用 `props` 数据导致收集冗余的依赖，我们在 `Vue` 响应系统的章节会回过头来仔细给大家讲解。下面我们开始分析 `callHook` 函数的代码的中间部分，首先获取要调用的生命周期钩子：

```js
const handlers = vm.$options[hook]
```

比如 `callHook(vm, created)`，那么上面的代码就相当于：

```js
const handlers = vm.$options.created
```

对于生命周期钩子选项最终会被合并处理成一个数组，所以得到的 `handlers` 就是对应生命周期钩子的数组。接着执行的是这段代码：

```js
if (handlers) {
  for (let i = 0, j = handlers.length; i < j; i++) {
    try {
      handlers[i].call(vm)
    } catch (e) {
      handleError(e, vm, `${hook} hook`)
    }
  }
}
```

由于开发者在编写组件时未必会写生命周期钩子，所以获取到的 `handlers` 可能不存在，所以使用 `if` 语句进行判断，只有当 `handlers` 存在的时候才对 `handlers` 进行遍历，`handlers` 数组的元素就是生命周期钩子函数，所以直接执行即可：

```js
handlers[i].call(vm)
```

为了保证生命周期钩子函数内可以通过 `this` 访问实例对象，所以使用 `.call(vm)` 执行这些函数。另外由于生命周期钩子函数的函数体是开发者编写的，为了捕获可能出现的错误，使用 `try...catch` 语句块，并在 `catch` 语句块内使用 `handleError` 处理错误信息。其中 `handleError` 来自于 `core/util/error.js` 文件，大家可以在附录 [core/util 目录下的工具方法全解](../appendix/core-util.md) 中查看关于 `handleError` 的讲解。

所以我们发现，对于生命周期钩子的调用，其实就是通过 `this.$options` 访问处理过的对应的生命周期钩子函数数组，遍历并执行它们。原理还是很简单的。

我们回过头来再看一下这段代码：

```js
callHook(vm, 'beforeCreate')
initInjections(vm) // resolve injections before data/props
initState(vm)
initProvide(vm) // resolve provide after data/props
callHook(vm, 'created')
```

现在大家应该知道，`beforeCreate` 以及 `created` 这两个生命周期钩子的调用时机了。其中 `initState` 包括了：`initProps`、`initMethods`、`initData`、`initComputed` 以及 `initWatch`。所以当 `beforeCreate` 钩子被调用时，所有与 `props`、`methods`、`data`、`computed` 以及 `watch` 相关的内容都不能使用，当然了 `inject/provide` 也是不可用的。

作为对立面，`created` 生命周期钩子则恰恰是等待 `initInjections`、`initState` 以及 `initProvide` 执行完毕之后才被调用，所以在 `created` 钩子中，是完全能够使用以上提到的内容的。但由于此时还没有任何挂载的操作，所以在 `created` 中是不能访问DOM的，即不能访问 `$el`。

最后我们注意到 `callHook` 函数的最后有这样一段代码：

```js
if (vm._hasHookEvent) {
  vm.$emit('hook:' + hook)
}
```

其中 `vm._hasHookEvent` 是在 `initEvents` 函数中定义的，它的作用是判断是否存在**生命周期钩子的事件侦听器**，初始化值为 `false` 代表没有，当组件检测到存在**生命周期钩子的事件侦听器**时，会将 `vm._hasHookEvent` 设置为 `true`。那么问题来了，什么叫做**生命周期钩子的事件侦听器**呢？大家可能不知道，其实 `Vue` 是可以这么玩儿的：

```html
<child
  @hook:beforeCreate="handleChildBeforeCreate"
  @hook:created="handleChildCreated"
  @hook:mounted="handleChildMounted"
  @hook:生命周期钩子
 />
```

如上代码可以使用 `hook:` 加 `生命周期钩子名称` 的方式来监听组件相应的生命周期事件。这是 `Vue` 官方文档上没有体现的，但你确实可以这么用，不过除非你对 `Vue` 非常了解，否则不建议使用。

正是为了实现这个功能，才有了这段代码：

```js
if (vm._hasHookEvent) {
  vm.$emit('hook:' + hook)
}
```

另外大家可能会疑惑，`vm._hasHookEvent` 是在什么时候被设置为 `true` 的呢？或者换句话说，`Vue` 是如何检测是否存在生命周期事件侦听器的呢？对于这个问题等我们在讲解 `Vue` 事件系统时自然会知道。

### 7. initState

实际上根据如下代码所示：

```js
callHook(vm, 'beforeCreate')
initInjections(vm) // resolve injections before data/props
initState(vm)
initProvide(vm) // resolve provide after data/props
callHook(vm, 'created')
```

可以看到在 `initState` 函数执行之前，先执行了 `initInjections` 函数，也就是说 `inject` 选项要更早被初始化，不过由于初始化 `inject` 选项的时候涉及到 `defineReactive` 函数，并且调用了 `toggleObserving` 函数操作了用于控制是否应该转换为响应式属性的状态标识 `observerState.shouldConvert`，所以我们决定先讲解 `initState`，之后再来讲解 `initInjections` 和 `initProvide`，这才是一个合理的顺序，并且从 `Vue` 的时间线上来看 `inject/provide` 选项确实是后来才添加的。

所以我们打开 `core/instance/state.js` 文件，找到 `initState` 函数，如下：

```js
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```

以上是 `initState` 函数的全部代码，我们慢慢来看，首先在 `Vue` 实例对象添加一个属性 `vm._watchers = []`，其初始值是一个数组，这个数组将用来存储所有该组件实例的 `watcher` 对象。随后定义了常量 `opts`，它是 `vm.$options` 的引用。接着执行了如下两句代码：

```js
if (opts.props) initProps(vm, opts.props)
if (opts.methods) initMethods(vm, opts.methods)
```

如果 `opts.props` 存在，即选项中有 `props`，那么就调用 `initProps` 初始化 `props` 选项。同样的，如果 `opts.methods` 存在，则调用 `initMethods` 初始化 `methods` 选项。

再往下执行的是这段代码：

```js
if (opts.data) {
  initData(vm)
} else {
  observe(vm._data = {}, true /* asRootData */)
}
```

首先判断 `data` 选项是否存在，如果存在则调用 `initData` 初始化 `data` 选项，如果不存在则直接调用 `observe` 函数观测一个空对象：`{}`。

最后执行的是如下这段代码：

```js
if (opts.computed) initComputed(vm, opts.computed)
if (opts.watch && opts.watch !== nativeWatch) {
  initWatch(vm, opts.watch)
}
```

采用同样的方式初始化 `computed` 选项，但是对于 `watch` 选项仅仅判断 `opts.watch` 是否存在是不够的，还要判断 `opts.watch` 是不是原生的 `watch` 对象。前面的章节中我们提到过，这是因为在 `Firefox` 中原生提供了 `Object.prototype.watch` 函数，所以即使没有 `opts.watch` 选项，如果在火狐浏览器中依然能够通过原型链访问到原生的 `Object.prototype.watch`。但这其实不是我们想要的结果，所以这里加了一层判断避免把原生 `watch` 函数误认为是我们预期的 `opts.watch` 选项。之后才会调用 `initWatch` 函数初始化 `opts.watch` 选项。

通过阅读 `initState` 函数，我们可以发现 `initState` 其实是很多选项初始化的汇总，包括：`props`、`methods`、`data`、`computed` 和 `watch` 等。并且我们注意到 `props` 选项的初始化要早于 `data` 选项的初始化，那么这是不是可以使用 `props` 初始化 `data` 数据的原因呢？答案是：“是的”。接下来我们就深入讲解这些初始化工作都做了什么事情。

[Vue技术内幕](http://hcysun.me/vue-design/art/3vue-example.html)