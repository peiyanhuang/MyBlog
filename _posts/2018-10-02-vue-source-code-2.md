---
layout: post
title:  Vue 源码学习-Vue实例
date:   2018-10-02 09:58:00 +0800
categories: Vue
tag: Vue
---

* content
{:toc}

### Vue 实例

比如有如下 Vue Code: 

```js
<!-- html -->
<div id="app">{{i}}</div>

<!-- .vue -->
let vm = new Vue({
    el: '#app',
    data: {
        i: 1
    }
})
```

当初始化 Vue 实例的过程执行了什么代码。找到 Vue 的构造函数：

```js
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}
```

创建 Vue 实例时，会执行 `this._init(options)` 方法，参数 `options` 即为：

```js
options = {
    el: '#app',
    data: {
        i: 1
    }
}
```

查下 `_init()` 这个方法，在 [initMixin](https://github.com/peiyanhuang/vue/blob/dev/src/core/instance/init.js)中定义。`_init` 方法的一开始，是这两句代码：

```js
const vm: Component = this
// a uid
vm._uid = uid++
```

首先声明了常量 `vm`，其值为 `this` 也就是当前这个 Vue 实例，然后在实例上添加了一个唯一标示：`_uid`，其值为 `uid`，初始化为 0，可以看到每次实例化一个 Vue 实例之后，uid 的值都会 `++`。之后：

```js
 let startTag, endTag
/* istanbul ignore if */
if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    startTag = `vue-perf-start:${vm._uid}`
    endTag = `vue-perf-end:${vm._uid}`
    mark(startTag)
}

// 中间省略....

if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    vm._name = formatComponentName(vm, false)
    mark(endTag)
    measure(`vue ${vm._name} init`, startTag, endTag)
}
```

首先声明两个变量 `startTag` 和 `endTag`，然后判断在非生产环境下，并且 `config.performance` 和 `mark` 都为真，那么才执行里面的代码。

Vue 提供了全局配置 `Vue.config.performance`，我们通过将其设置为 `true`，在浏览器开发工具的性能/时间线面板中启用对组件初始化、编译、渲染和打补丁的性能追踪。只适用于开发模式和支持 `performance.mark API` 的浏览器上。

其中组件初始化的性能追踪就是我们在 `_init` 方法中看到的那样去实现的，其实现的方式就是在初始化的代码的开头和结尾分别使用 `mark` 函数打上两个标记，然后通过 `measure` 函数对这两个标记点进行性能计算。[mark 和 measure](https://github.com/peiyanhuang/vue/blob/dev/src/core/util/perf.js)

然后是中间那段代码：

```js
// a flag to avoid this being observed
vm._isVue = true
// merge options
if (options && options._isComponent) {
    // optimize internal component instantiation
    // since dynamic options merging is pretty slow, and none of the
    // internal component options needs special treatment.
    initInternalComponent(vm, options)
} else {
    vm.$options = mergeOptions(
    resolveConstructorOptions(vm.constructor),
    options || {},
    vm
    )
}
/* istanbul ignore else */
if (process.env.NODE_ENV !== 'production') {
    initProxy(vm)
} else {
    vm._renderProxy = vm
}
// expose real self
vm._self = vm
initLifecycle(vm)
initEvents(vm)
initRender(vm)
callHook(vm, 'beforeCreate')
initInjections(vm) // resolve injections before data/props
initState(vm)
initProvide(vm) // resolve provide after data/props
callHook(vm, 'created')
```

上面的代码是那两段性能追踪的代码之间全部的内容，我们逐一分析，首先在 Vue 实例上添加 `_isVue` 属性，并设置其值为 true。目的是用来标识一个对象是 Vue 实例，即如果发现一个对象拥有 `_isVue` 属性并且其值为 true，那么就代表该对象是 Vue 实例。这样可以避免该对象被响应系统观测（其实在其他地方也有用到，但是宗旨都是一样的，这个属性就是用来告诉你：我不是普通的对象，我是Vue实例）。

再往下是这样一段代码：

```js
// merge options
if (options && options._isComponent) {
    // optimize internal component instantiation
    // since dynamic options merging is pretty slow, and none of the
    // internal component options needs special treatment.
    initInternalComponent(vm, options)
} else {
    vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
    )
}
```

上面的代码是一段 if 分支语句，条件是：`options && options._isComponent`，其中 `options` 就是我们调用 Vue 时传递的参数选项，但 `options._isComponent` 是什么鬼？我们知道在本节的例子中我们的 `options` 对象只有两个属性 `el` 和 `data`，并且在 Vue 的官方文档中你也找不到关于 `_isComponent` 这个选项的介绍，其实这是一个内部选项。

根据本节的例子，上面的代码必然会走 else 分支。这段代码在 Vue 实例上添加了 `$options` 属性。在后面一系列 init* 的方法，比如：

```js
initLifecycle(vm)
initEvents(vm)
initRender(vm)
callHook(vm, 'beforeCreate')
initInjections(vm) // resolve injections before data/props
initState(vm)
initProvide(vm) // resolve provide after data/props
callHook(vm, 'created')
```

这些方法才是真正起作用的一些初始化方法。在这些初始化方法中，无一例外的都使用到了实例的 `$options` 属性，即 `vm.$options`。所以 `$options` 这个属性的的确确是用于 Vue 实例初始化的，只不过在初始化之前，我们需要一些手段来产生 `$options` 属性，而这就是 `mergeOptions` 函数的作用。

#### 1. mergeOptions()

想要知道 `mergeOptions` 是干什么的，得先弄清楚他的参数。

##### 1.resolveConstructorOptions()

`resolveConstructorOptions()` 代码如下：

```js
export function resolveConstructorOptions (Ctor: Class<Component>) {
  let options = Ctor.options
  if (Ctor.super) {
    const superOptions = resolveConstructorOptions(Ctor.super)
    const cachedSuperOptions = Ctor.superOptions
    if (superOptions !== cachedSuperOptions) {
      // super option changed,
      // need to resolve new options.
      Ctor.superOptions = superOptions
      // check if there are any late-modified/attached options (#4976)
      const modifiedOptions = resolveModifiedOptions(Ctor)
      // update base extend options
      if (modifiedOptions) {
        extend(Ctor.extendOptions, modifiedOptions)
      }
      options = Ctor.options = mergeOptions(superOptions, Ctor.extendOptions)
      if (options.name) {
        options.components[options.name] = Ctor
      }
    }
  }
  return options
}
```

其中 `Ctor` 即传递进来的参数 `vm.constructor`，在我们的例子中他就是 Vue 构造函数。如果使用 `Vue.extend` 创造一个子类并使用子类创造实例时，那么 `vm.constructor` 就不是 Vue 构造函数，而是子类，比如：

```js
const Sub = Vue.extend()
const s = new Sub()
```

那么 `s.constructor` 自然就是 Sub 而非 Vue。

接下来看下 `if` 判断，`super` 这是子类才有的属性。这个属性是与 `Vue.extend` 有关。之后是

```js
const superOptions = resolveConstructorOptions(Ctor.super)
```

递归地调用了 `resolveConstructorOptions` 函数，只不过此时的参数是构造者的父类，之后的代码中，还有一些关于父类的 `options` 属性是否被改变过的判断和操作，并且大家注意这句代码：

```js
// check if there are any late-modified/attached options (#4976)
const modifiedOptions = resolveModifiedOptions(Ctor)
```

注意下注释，根据注释中括号内的 issue 索引去搜一下相关的问题，这句代码是用来解决使用 vue-hot-reload-api 或者 vue-loader 时产生的一个 bug 的。

按照例子，不会走判断语句。此时的 `mergeOptions` 函数的第一个参数就是 `Vue.options`:

```js
Vue.options = {
    components: {
        KeepAlive
        Transition,
        TransitionGroup
    },
    directives:{
        model,
        show
    },
    filters: Object.create(null),
    _base: Vue
}
```

第二个参数就是传进来的数据：

```js
{
  el: '#app',
  data: {
    test: 1
  }
}
```

#### 2.mergeOptions()

```js
export function mergeOptions (
  parent: Object,
  child: Object,
  vm?: Component
): Object {
  if (process.env.NODE_ENV !== 'production') {
    checkComponents(child)
  }

  if (typeof child === 'function') {
    child = child.options
  }

  normalizeProps(child, vm)
  normalizeInject(child, vm)
  normalizeDirectives(child)
  const extendsFrom = child.extends
  if (extendsFrom) {
    parent = mergeOptions(parent, extendsFrom, vm)
  }
  if (child.mixins) {
    for (let i = 0, l = child.mixins.length; i < l; i++) {
      parent = mergeOptions(parent, child.mixins[i], vm)
    }
  }
  const options = {}
  let key
  for (key in parent) {
    mergeField(key)
  }
  for (key in child) {
    if (!hasOwn(parent, key)) {
      mergeField(key)
    }
  }
  function mergeField (key) {
    const strat = strats[key] || defaultStrat
    options[key] = strat(parent[key], child[key], vm, key)
  }
  return options
}
```

在非生产环境下，会以 `child` 为参数调用 `checkComponents` 方法，这个方法是用来校验组件的名字是否符合要求的。

```js
/**
 * Validate component names
 */
function checkComponents (options: Object) {
  for (const key in options.components) {
    validateComponentName(key)
  }
}
```

命名规则 [validateComponentName](https://github.com/peiyanhuang/vue/blob/dev/src/core/util/options.js#L255)，Vue 限定组件的名字由普通的字符和中横线(-)组成，且必须以字母开头，且不能是内置及保留标签。

```js
export function validateComponentName (name: string) {
  if (!/^[a-zA-Z][\w-]*$/.test(name)) {
    warn(
      'Invalid component name: "' + name + '". Component names ' +
      'can only contain alphanumeric characters and the hyphen, ' +
      'and must start with a letter.'
    )
  }
  if (isBuiltInTag(name) || config.isReservedTag(name)) {
    warn(
      'Do not use built-in or reserved HTML elements as component ' +
      'id: ' + name
    )
  }
}
```

`isBuildInTag()`: 用来检测你所注册的组件是否是内置的标签(slot 和 component)

```js
export const isBuiltInTag = makeMap('slot,component', true)
```

`config.isReservedTag()`: 默认一直返回 false，但是在文件 [platforms/web/runtime/index.js](https://github.com/peiyanhuang/vue/blob/dev/src/platforms/web/runtime/index.js#L24) 中有这样一段代码：

```js
// install platform specific utils
Vue.config.mustUseProp = mustUseProp
Vue.config.isReservedTag = isReservedTag
Vue.config.isReservedAttr = isReservedAttr
Vue.config.getTagNamespace = getTagNamespace
Vue.config.isUnknownElement = isUnknownElement
```

`config.isReservedTag` 指向 [isReservedTag](https://github.com/peiyanhuang/vue/blob/dev/src/platforms/web/util/element.js#L36) 而不是 `no` 了。

接下来是另一个 if 语句：

```js
if (typeof child === 'function') {
  child = child.options
}
```

这说明 `child` 参数除了是普通的选项对象外，还可以是一个函数，如果是函数的话就取该函数的 `options` 静态属性作为新的 `child`，我们想一想什么样的函数具有 options 静态属性呢？现在我们知道 Vue 构造函数本身就拥有这个属性，其实通过 Vue.extend 创造出来的子类也是拥有这个属性的。所以这就允许我们在进行选项合并的时候，去合并一个 Vue 实例构造者的选项了。

接着看代码，接下来是三个用来规范化选项的函数调用 [normalize*](https://github.com/peiyanhuang/vue/blob/dev/src/core/util/options.js#L275)：

```js
normalizeProps(child, vm)
normalizeInject(child, vm)
normalizeDirectives(child)
```

如 `props` 有两种写法，无论开发者使用哪一种写法，在内部都将其规范成同一种方式，这样在选项合并的时候就能够统一处理，这就是上面三个函数的作用。