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

