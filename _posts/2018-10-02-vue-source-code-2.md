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

### mergeOptions()

想要知道 `mergeOptions` 是干什么的，得先弄清楚他的参数。

#### 2.1 resolveConstructorOptions()

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

#### 2.2 mergeOptions()

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

这说明 `child` 参数除了是普通的选项对象外，还可以是一个函数，如果是函数的话就取该函数的 `options` 静态属性作为新的 `child`，我们想一想什么样的函数具有 `options` 静态属性呢？现在我们知道 Vue 构造函数本身就拥有这个属性，其实通过 `Vue.extend` 创造出来的子类也是拥有这个属性的。所以这就允许我们在进行选项合并的时候，去合并一个 Vue 实例构造者的选项了。

接着看代码，接下来是三个用来规范化选项的函数调用 [normalize*](https://github.com/peiyanhuang/vue/blob/dev/src/core/util/options.js#L275)：

```js
normalizeProps(child, vm)
normalizeInject(child, vm)
normalizeDirectives(child)
```

#### 2.3 规范化 props（normalizeProps）

如 `props` 有两种写法，无论开发者使用哪一种写法，在内部都将其规范成同一种方式，这样在选项合并的时候就能够统一处理，这就是上面三个函数的作用。

```js
/**
 * Ensure all props option syntax are normalized into the
 * Object-based format.
 */
function normalizeProps (options: Object, vm: ?Component) {
  const props = options.props
  if (!props) return
  const res = {}
  let i, val, name
  if (Array.isArray(props)) {
    i = props.length
    while (i--) {
      val = props[i]
      if (typeof val === 'string') {
        name = camelize(val)
        res[name] = { type: null }
      } else if (process.env.NODE_ENV !== 'production') {
        warn('props must be strings when using array syntax.')
      }
    }
  } else if (isPlainObject(props)) {
    for (const key in props) {
      val = props[key]
      name = camelize(key)
      res[name] = isPlainObject(val)
        ? val
        : { type: val }
    }
  } else if (process.env.NODE_ENV !== 'production') {
    warn(
      `Invalid value for option "props": expected an Array or an Object, ` +
      `but got ${toRawType(props)}.`,
      vm
    )
  }
  options.props = res
}
```

[camelize](https://github.com/peiyanhuang/vue/blob/dev/src/shared/util.js#L153) 函数作用是将中横线转驼峰。

#### 2.4 规范化 inject（normalizeInject）

```js
/**
 * Normalize all injections into Object-based format
 */
function normalizeInject (options: Object, vm: ?Component) {
  const inject = options.inject
  if (!inject) return
  const normalized = options.inject = {}
  if (Array.isArray(inject)) {
    for (let i = 0; i < inject.length; i++) {
      normalized[inject[i]] = { from: inject[i] }
    }
  } else if (isPlainObject(inject)) {
    for (const key in inject) {
      const val = inject[key]
      normalized[key] = isPlainObject(val)
        ? extend({ from: key }, val)
        : { from: val }
    }
  } else if (process.env.NODE_ENV !== 'production') {
    warn(
      `Invalid value for option "inject": expected an Array or an Object, ` +
      `but got ${toRawType(inject)}.`,
      vm
    )
  }
}
```

`inject` 选项的使用方法和写法与 `props` 一样拥有两种，一种是字符串数组，一种是对象语法。`normalizeInject` 的作用也是规范化 `inject`。

#### 2.5 规范化 directives（normalizeDirectives）

```js
/**
 * Normalize raw function directives into object format.
 */
function normalizeDirectives (options: Object) {
  const dirs = options.directives
  if (dirs) {
    for (const key in dirs) {
      const def = dirs[key]
      if (typeof def === 'function') {
        dirs[key] = { bind: def, update: def }
      }
    }
  }
}
```

#### 2.6 extends 和 mixins

看完了 `mergeOptions` 函数里的三个规范化函数之后，我们继续看后面的代码，接下来是这样一段代码：

```js
const extendsFrom = child.extends
if (extendsFrom) {
  parent = mergeOptions(parent, extendsFrom, vm)
}
if (child.mixins) {
  for (let i = 0, l = child.mixins.length; i < l; i++) {
    parent = mergeOptions(parent, child.mixins[i], vm)
  }
}
```

这段代码是处理 `extends` 选项和 `mixins` 选项的，首先使用变量 `extendsFrom` 保存了对 `child.extends` 的引用，之后的处理都是用 `extendsFrom` 来做，然后判断 `extendsFrom` 是否为真，即 `child.extends` 是否存在，如果存在的话就递归调用 `mergeOptions` 函数将 `parent` 与 `extendsFrom` 进行合并，并将结果作为新的 `parent`。这里要注意，我们之前说过 `mergeOptions` 函数将会产生一个新的对象，所以此时的 `parent` 已经被新的对象重新赋值了。

接着检测 `child.mixins` 选项是否存在，如果存在则使用同样的方式进行操作，不同的是，由于 `mixins` 是一个数组所以要遍历一下。

### Vue 选项的合并

继续看 `mergeOptions` 函数的代码，接下来的一段代码如下：

```js
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
```

`satrts` 是什么？

```js
/**
 * Option overwriting strategies are functions that handle
 * how to merge a parent option value and a child option
 * value into the final value.
 */
const strats = config.optionMergeStrategies
```

它是一个常量，这个常量的值为 `config.optionMergeStrategies`，这个 `config` 对象是全局配置对象，来自于 `core/config.js` 文件，此时 `config.optionMergeStrategies` 还只是一个空的对象。[Option merge strategies (used in core/util/options)](https://github.com/peiyanhuang/vue/blob/dev/src/core/util/options.js#L28)

所以，当 `starts[key]` 不存在时，`start` 就等于 `defaultStrat` 这个函数。

```js
/**
 * Default strategy.
 */
const defaultStrat = function (parentVal: any, childVal: any): any {
  return childVal === undefined
    ? parentVal
    : childVal
}
```

`starts[key]` 的方法在下面。

#### 3.1 选项 el、propsData 的合并策略

```js
/**
 * Options with restrictions
 */
if (process.env.NODE_ENV !== 'production') {
  strats.el = strats.propsData = function (parent, child, vm, key) {
    if (!vm) {
      warn(
        `option "${key}" can only be used during instance ` +
        'creation with the `new` keyword.'
      )
    }
    return defaultStrat(parent, child)
  }
}
```

函数中的 `vm` 来自于 `mergeOptions` 函数的第三个参数。所以当调用 `mergeOptions` 函数且不传递第三个参数的时候，那么在策略函数中就拿不到 vm 参数。所以我们可以猜测到一件事，那就是 `mergeOptions` 函数除了在 `_init` 方法中被调用之外，还在其他地方被调用，且没有传递第三个参数。那么到底是在哪里被调用的呢？这里可以先明确地告诉大家，就是在 `Vue.extend` 方法中被调用的，大家可以打开 `core/global-api/extend.js` 文件找到 `Vue.extend` 方法，其中有这么一段代码：

```js
Sub.options = mergeOptions(
  Super.options,
  extendOptions
)
```

可以发现，此时调用 `mergeOptions` 函数就没有传递第三个参数，也就是说通过 `Vue.extend` 创建子类的时候 `mergeOptions` 会被调用，此时策略函数就拿不到第三个参数。

所以现在就比较明朗了，在策略函数中通过判断是否存在 `vm` 就能够得知 `mergeOptions` 是在实例化时调用(使用 new 操作符走 _init 方法)还是在继承时调用(Vue.extend)，而子组件的实现方式就是通过实例化子类完成的，子类又是通过 Vue.extend 创造出来的，所以我们就能通过对 `vm` 的判断而得知是否是子组件了。

接着看 `strats.el` 和 `strats.propsData` 策略函数的代码，返回了 `defaultStrat(parent, child)`。这个函数很简单，前面已经展示过了。

#### 3.2 data 的合并策略

```js
strats.data = function (
  parentVal: any,
  childVal: any,
  vm?: Component
): ?Function {
  if (!vm) {
    if (childVal && typeof childVal !== 'function') {
      process.env.NODE_ENV !== 'production' && warn(
        'The "data" option should be a function ' +
        'that returns a per-instance value in component ' +
        'definitions.',
        vm
      )

      return parentVal
    }
    return mergeDataOrFn(parentVal, childVal)
  }

  return mergeDataOrFn(parentVal, childVal, vm)
}
```

[mergeDataOrFn/mergeData](https://github.com/peiyanhuang/vue/blob/dev/src/core/util/options.js#L47)

先通过 `vm` 判断是否是子组件，是的话，子组件中的 `data` 必须是一个返回对象的函数。如果不是函数，除了给你一段警告之外，会直接返回 `parentVal`。如果 `childVal` 是函数类型，那说明满足了子组件的 `data` 选项需要是一个函数的要求，那么就直接返回 `mergeDataOrFn` 函数的执行结果。

如果不是子组件，那也直接返回 `mergeDataOrFn` 函数的执行结果，不同的是参数多了个 `vm`。

`mergeDataOrFn` 函数永远返回一个函数。

1. 为什么最终 `strats.data` 会被处理成一个函数？

这是因为，通过函数返回数据对象，保证了每个组件实例都有一个唯一的数据副本，避免了组件间数据互相影响。后面讲到 Vue 的初始化的时候大家会看到，在初始化数据状态的时候，就是通过执行 `strats.data` 函数来获取数据并对其进行处理的。

2. 为什么不在合并阶段就把数据合并好，而是要等到初始化的时候再合并数据？

这个问题是什么意思呢？我们知道在合并阶段 `strats.data` 将被处理成一个函数，但是这个函数并没有被执行，而是到了后面初始化的阶段才执行的，这个时候才会调用 `mergeData` 对数据进行合并处理，那这么做的目的是什么呢？

其实这么做是有原因的，后面讲到 Vue 的初始化的时候，大家就会发现 `inject` 和 `props` 这两个选项的初始化是先于 `data` 选项的，这就保证了我们能够使用 `props` 初始化 `data` 中的数据。

#### 3.3 生命周期钩子选项的合并策略

```js
/**
 * Hooks and props are merged as arrays.
 */
function mergeHook (
  parentVal: ?Array<Function>,
  childVal: ?Function | ?Array<Function>
): ?Array<Function> {
  return childVal
    ? parentVal
      ? parentVal.concat(childVal)
      : Array.isArray(childVal)
        ? childVal
        : [childVal]
    : parentVal
}

LIFECYCLE_HOOKS.forEach(hook => {
  strats[hook] = mergeHook
})

<!-- shared/constants.js -->
export const LIFECYCLE_HOOKS = [
  'beforeCreate',
  'created',
  'beforeMount',
  'mounted',
  'beforeUpdate',
  'updated',
  'beforeDestroy',
  'destroyed',
  'activated',
  'deactivated',
  'errorCaptured'
]
```

`mergeHook` 返回的总是数组。`parentVal` 也必定是数组。因为如下：

```js
const Parent = Vue.extend({
  created: function () {
    console.log('parentVal')
  }
})

const Child = new Parent({
  created: function () {
    console.log('childVal')
  }
})
```

其中 `Child` 是使用 `new Parent` 生成的，所以对于 `Child` 来讲，`childVal` 是：

```js
created: function () {
  console.log('childVal')
}
```

而 `parentVal` 已经不是 `Vue.options.created` 了，而是 `Parent.options.created`，那么 `Parent.options.created` 是什么呢？它其实是通过 `Vue.extend` 函数内部的 `mergeOptions` 处理过的，所以它应该是这样的：

```js
Parent.options.created = [
  created: function () {
    console.log('parentVal')
  }
]
```

生命周期钩子是可以写成数组的，虽然 Vue 的文档里没有，`: Array.isArray(childVal) ? childVal : [childVal]`。

#### 3.4 资源(assets)选项的合并策略

在 Vue 中 directives、filters 以及 components 被认为是资源。

```js
/**
 * Assets
 *
 * When a vm is present (instance creation), we need to do
 * a three-way merge between constructor options, instance
 * options and parent options.
 */
function mergeAssets (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): Object {
  const res = Object.create(parentVal || null)
  if (childVal) {
    process.env.NODE_ENV !== 'production' && assertObjectType(key, childVal, vm)
    return extend(res, childVal)
  } else {
    return res
  }
}

ASSET_TYPES.forEach(function (type) {
  strats[type + 's'] = mergeAssets
})

<!-- shared/constants.js -->
export const ASSET_TYPES = [
  'component',
  'directive',
  'filter'
]
```

上面的代码本身逻辑很简单，首先以 `parentVal` 为原型创建对象 `res`，然后判断是否有 `childVal`，如果有的话使用 `extend` 函数将 `childVal` 上的属性混合到 `res` 对象上并返回。如果没有 `childVal` 则直接返回 `res`。

举个例子，大家知道任何组件的模板中我们都可以直接使用 `<transition/>` 组件或者 `<keep-alive/>` 等，但是我们并没有在我们自己的组件实例的 `components` 选项中显式地声明这些组件。那么这是怎么做到的呢？其实答案就在 `mergeAssets` 函数中。以下面的代码为例：

```js
var v = new Vue({
  el: '#app',
  components: {
    ChildComponent: ChildComponent
  }
})
```

上面的代码中，我们创建了一个 Vue 实例，并注册了一个子组件 `ChildComponent`，此时 `mergeAssets` 方法内的 `childVal` 就是例子中的 `components` 选项：

```js
components: {
  ChildComponent: ChildComponent
}
```

而 `parentVal` 就是 `Vue.options.components`，我们知道 `Vue.options` 如下：

```js
Vue.options = {
  components: {
    KeepAlive,
    Transition,
    TransitionGroup
  },
  directives: Object.create(null),
  directives:{
    model,
    show
  },
  filters: Object.create(null),
  _base: Vue
}
```

所以 `Vue.options.components` 就应该是一个对象：

```js
{
  KeepAlive,
  Transition,
  TransitionGroup
}
```

也就是说 `parentVal` 就是如上包含三个内置组件的对象，所以经过如下这句话之后：

```js
const res = Object.create(parentVal || null)
```

你可以通过 `res.KeepAlive` 访问到 `KeepAlive` 对象，因为虽然 res 对象自身属性没有 `KeepAlive`，但是它的原型上有。

然后再经过 `return extend(res, childVal)` 这句话之后，res 变量将被添加 `ChildComponent` 属性，最终 res 如下：

```js
res = {
  ChildComponent
  // 原型
  __proto__: {
    KeepAlive,
    Transition,
    TransitionGroup
  }
}
```

#### 3.5 选项 watch 的合并策略

```js
/**
 * Watchers.
 *
 * Watchers hashes should not overwrite one
 * another, so we merge them as arrays.
 */
strats.watch = function (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): ?Object {
  // work around Firefox's Object.prototype.watch...
  if (parentVal === nativeWatch) parentVal = undefined
  if (childVal === nativeWatch) childVal = undefined
  /* istanbul ignore if */
  if (!childVal) return Object.create(parentVal || null)
  if (process.env.NODE_ENV !== 'production') {
    assertObjectType(key, childVal, vm)
  }
  if (!parentVal) return childVal
  const ret = {}
  extend(ret, parentVal)
  for (const key in childVal) {
    let parent = ret[key]
    const child = childVal[key]
    if (parent && !Array.isArray(parent)) {
      parent = [parent]
    }
    ret[key] = parent
      ? parent.concat(child)
      : Array.isArray(child) ? child : [child]
  }
  return ret
}
```

在 Firefox 浏览器中 `Object.prototype` 拥有原生的 `watch` 函数，所以即便一个普通的对象你没有定义 `watch` 属性，但是依然可以通过原型链访问到原生的 `watch` 属性，这就会给 Vue 在处理选项的时候造成迷惑，因为 Vue 也提供了一个叫做 `watch` 的选项，即使你的组件选项中没有写 `watch` 选项，但是 Vue 通过原型访问到了原生的 `watch`。这不是我们想要的，所以上面两句代码的目的是一个变通方案，当发现组件选项是浏览器原生的 `watch` 时，那说明用户并没有提供 Vue 的 `watch` 选项，直接重置为 `undefined。`

在非生产环境下使用 `assertObjectType` 函数对 `childVal` 进行类型检测，检测其是否是一个纯对象，我们知道 Vue 的 `watch` 选项需要是一个纯对象。

#### 3.6 props、methods、inject、computed 的合并策略

```js
/**
 * Other object hashes.
 */
strats.props =
strats.methods =
strats.inject =
strats.computed = function (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): ?Object {
  if (childVal && process.env.NODE_ENV !== 'production') {
    assertObjectType(key, childVal, vm)
  }
  if (!parentVal) return childVal
  const ret = Object.create(null)
  extend(ret, parentVal)
  if (childVal) extend(ret, childVal)
  return ret
}
```

#### 3.7 provide 的合并策略

最后一个选项的合并策略，就是 `provide` 选项的合并策略，只有一句代码，如下：

```js
strats.provide = mergeDataOrFn
```

也就是说 `provide` 选项的合并策略与 `data` 选项的合并策略相同，都是使用 `mergeDataOrFn` 函数。

#### 3.8 总结

现在我们了解了 Vue 中是如何合并处理选项的，接下来我们稍微做一个总结：

- 对于 el、propsData 选项使用默认的合并策略 defaultStrat。
- 对于 data 选项，使用 mergeDataOrFn 函数进行处理，最终结果是 data 选项将变成一个函数，且该函数的执行结果为真正的数据对象。
- 对于 生命周期钩子 选项，将合并成数组，使得父子选项中的钩子函数都能够被执行
- 对于 directives、filters 以及 components 等资源选项，父子选项将以原型链的形式被处理，正是因为这样我们才能够在任何地方都使用内置组件、指令等。
- 对于 watch 选项的合并处理，类似于生命周期钩子，如果父子选项都有相同的观测字段，将被合并为数组，这样观察者都将被执行。
- 对于 props、methods、inject、computed 选项，父选项始终可用，但是子选项会覆盖同名的父选项字段。
- 对于 provide 选项，其合并策略使用与 data 选项相同的 mergeDataOrFn 函数。
- 最后，以上没有提及到的选项都将使默认选项 defaultStrat。
- 最最后，默认合并策略函数 defaultStrat 的策略是：只要子选项不是 undefined 就使用子选项，否则使用父选项。