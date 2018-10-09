---
layout: post
title:  Vue 源码学习（1）
date:   2018-10-01 09:58:00 +0800
categories: Vue
tag: Vue
---

* content
{:toc}

### Vue 构造函数

首先看下 package.json 文件，以 `npm run dev` 为切入点：

```js
"dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev"
```

之后找到 `scripts/config.js` 文件中的配置:

```js
// Runtime+compiler development build (Browser)
'web-full-dev': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.js'),
    format: 'umd',
    env: 'development',
    alias: { he: './entity-decoder' },
    banner
},
```

入口文件为 `web/entry-runtime-with-compiler.js`，最终输出 `dist/vue.js`。这个 `web` 指的是哪个目录呢？

在 `config.js` 同目录下，有个 `alias.js` 文件：

```js
const path = require('path')

const resolve = p => path.resolve(__dirname, '../', p)

module.exports = {
  vue: resolve('src/platforms/web/entry-runtime-with-compiler'),
  compiler: resolve('src/compiler'),
  core: resolve('src/core'),
  shared: resolve('src/shared'),
  web: resolve('src/platforms/web'),
  weex: resolve('src/platforms/weex'),
  server: resolve('src/server'),
  entries: resolve('src/entries'),
  sfc: resolve('src/sfc')
}
```

所以 `web` 指向的应该是 `src/platforms/web`，除了 `web` 之外，`alias.js` 文件中还配置了其他的别名，在找对应目录的时候，可以来这里查阅，后面就不做这种目录寻找的说明了。

打开 `src/platforms/web/entry-runtime-with-compiler.js` 文件，你可以看到这样一句话：

```js
import Vue from './runtime/index'
```

`Vue` 构造函数不是在这里定义的，继续查找。直到在 `src/core/instance/index.js` 文件中：

```js
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```

可以看到 `Vue` 函数在这里定义。首先分别从 `./init.js、./state.js、./render.js、./events.js、./lifecycle.js` 这五个文件中导入五个方法，分别是：`initMixin、stateMixin、renderMixin、eventsMixin` 以及 `lifecycleMixin`，然后定义了 `Vue` 构造函数，其中使用了安全模式来提醒你要使用 `new` 操作符来调用 `Vue`，接着将 `Vue` 构造函数作为参数，分别传递给了导入进来的这五个方法，最后导出 `Vue`。

#### initMixin()

找到 `initMixin` 方法，如下：

```js
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    // ... _init 方法的函数体，此处省略
  }
}
```

这个方法的作用就是在 Vue 的原型上添加了 `_init` 方法，这个 `_init` 方法看上去应该是内部初始化的一个方法，其实在 `instance/index.js` 文件中我们是见过这个方法的。

#### stateMixin()

找到 `stateMixin` 方法，这个方法的一开始，是这样一段代码：

```js
export function stateMixin (Vue: Class<Component>) {
  // flow somehow has problems with directly declared definition object
  // when using Object.defineProperty, so we have to procedurally build up
  // the object here.
  const dataDef = {}
  dataDef.get = function () { return this._data }
  const propsDef = {}
  propsDef.get = function () { return this._props }
  if (process.env.NODE_ENV !== 'production') {
    dataDef.set = function (newData: Object) {
      warn(
        'Avoid replacing instance root $data. ' +
        'Use nested data properties instead.',
        this
      )
    }
    propsDef.set = function () {
      warn(`$props is readonly.`, this)
    }
  }
  Object.defineProperty(Vue.prototype, '$data', dataDef)
  Object.defineProperty(Vue.prototype, '$props', propsDef)
  /* 
    省略...
  */
}
```

最后两句，使用 `Object.defineProperty` 在 `Vue.prototype` 上定义了两个属性，就是大家熟悉的：`$data` 和 `$props`。这两个属性的定义分别写在了 `dataDef` 以及 `propsDef` 这两个对象里。

`$data` 属性实际上代理的是 `_data` 这个实例属性，而 `$props` 代理的是 `_props` 这个实例属性。然后有一个是否为生产环境的判断，如果不是生产环境的话，就为 `$data` 和 `$props` 这两个属性设置一下 `set`，实际上就是提示你一下：别想修改我。

也就是说，`$data` 和 `$props` 是两个只读的属性。

接下来 `stateMixin` 又在 `Vue.prototype` 上定义了三个方法：

```js
Vue.prototype.$set = set
Vue.prototype.$delete = del

Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: any,
    options?: Object
): Function {}
```

这三个方法分别是：`$set`、`$delete` 以及 `$watch`。

#### eventsMixin()

```js
export function eventsMixin (Vue: Class<Component>) {
  const hookRE = /^hook:/
  Vue.prototype.$on = function (event: string | Array<string>, fn: Function): Component {}

  Vue.prototype.$once = function (event: string, fn: Function): Component {}

  Vue.prototype.$off = function (event?: string | Array<string>, fn?: Function): Component {}

  Vue.prototype.$emit = function (event: string): Component {}
}
```

找到 `eventsMixin` 方法，这个方法在 `Vue.prototype` 上添加了四个方法。

#### lifecycleMixin()

下一个是 `lifecycleMixin`，这个方法在 `Vue.prototype` 上添加了三个方法：

```js
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {}
Vue.prototype.$forceUpdate = function () {}
Vue.prototype.$destroy = function () {}
```

#### renderMixin()

最后一个就是 `renderMixin` 方法了，这个方法的一开始以 `Vue.prototype` 为参数调用了 `installRenderHelpers` 函数，这个函数来自于与 `render.js` 文件相同目录下的 `render-helpers/index.js` 文件，打开这个文件找到 `installRenderHelpers` 函数：

```js
export function installRenderHelpers (target: any) {
  target._o = markOnce
  target._n = toNumber
  target._s = toString
  target._l = renderList
  target._t = renderSlot
  target._q = looseEqual
  target._i = looseIndexOf
  target._m = renderStatic
  target._f = resolveFilter
  target._k = checkKeyCodes
  target._b = bindObjectProps
  target._v = createTextVNode
  target._e = createEmptyVNode
  target._u = resolveScopedSlots
  target._g = bindObjectListeners
}
```

这个函数的作用就是在 `Vue.prototype` 上添加一系列方法，这些方法的作用后面会讲解到。

`renderMixin` 方法在执行完 `installRenderHelpers` 函数之后，又在 `Vue.prototype` 上添加了两个方法，分别是：`$nextTick` 和 `_render`。

```js
Vue.prototype.$nextTick = function (fn: Function) {}
Vue.prototype._render = function (): VNode {}
```

至此，执行命令 `npm run dev` 后 `instance/index.js` 文件中的代码就运行完毕了。大概了解了每个 *Mixin 方法的作用其实就是包装 `Vue.prototype`，在其上挂载一些属性和方法。

### Vue 构造函数的静态属性和方法(global api)

之前查看到的是 `src/core/instance/index.js` 文件，它被 `src/core/index.js` 文件所引用。看下：

```js
import Vue from './instance/index'
import { initGlobalAPI } from './global-api/index'
import { isServerRendering } from 'core/util/env'
import { FunctionalRenderContext } from 'core/vdom/create-functional-component'

initGlobalAPI(Vue)

Object.defineProperty(Vue.prototype, '$isServer', {
  get: isServerRendering
})

Object.defineProperty(Vue.prototype, '$ssrContext', {
  get () {
    /* istanbul ignore next */
    return this.$vnode && this.$vnode.ssrContext
  }
})

// expose FunctionalRenderContext for ssr runtime helper installation
Object.defineProperty(Vue, 'FunctionalRenderContext', {
  value: FunctionalRenderContext
})

Vue.version = '__VERSION__'

export default Vue
```

在 `Vue.prototype` 上分别添加了两个只读的属性，分别是：`$isServer` 和 `$ssrContext`。接着又在 Vue 构造函数上定义了 `FunctionalRenderContext` 静态属性。

最后，在 Vue 构造函数上添加了一个静态属性 `version`，存储了当前 Vue 的版本值，但是这里的 `__VERSION__` 是什么鬼？打开 `scripts/config.js` 文件，找到 `genConfig` 方法，其中有这么一句话：`__VERSION__: version`。这句话被写在了 `rollup` 的 `replace` 插件中，也就是说，`__VERSION__` 将被 `version` 的值替换，而 `version` 的值就是 Vue 的版本号。

接下来回头看这句 `initGlobalAPI(Vue)`:

```js
export function initGlobalAPI (Vue: GlobalAPI) {
  // config
  const configDef = {}
  configDef.get = () => config
  if (process.env.NODE_ENV !== 'production') {
    configDef.set = () => {
      warn(
        'Do not replace the Vue.config object, set individual fields instead.'
      )
    }
  }
  Object.defineProperty(Vue, 'config', configDef)

  // exposed util methods.
  // NOTE: these are not considered part of the public API - avoid relying on
  // them unless you are aware of the risk.
  Vue.util = {
    warn,
    extend,
    mergeOptions,
    defineReactive
  }

  Vue.set = set
  Vue.delete = del
  Vue.nextTick = nextTick

  Vue.options = Object.create(null)
  ASSET_TYPES.forEach(type => {
    Vue.options[type + 's'] = Object.create(null)
  })

  // this is used to identify the "base" constructor to extend all plain-object
  // components with in Weex's multi-instance scenarios.
  Vue.options._base = Vue

  extend(Vue.options.components, builtInComponents)

  initUse(Vue)
  initMixin(Vue)
  initExtend(Vue)
  initAssetRegisters(Vue)
}
```

这段代码也比较容易理解：在 Vue 构造函数上添加了 `config、util、set、delete、nextTick、options` 等属性和方法。

其 `util` 属性拥有 `warn、extend、mergeOptions、defineReactive` 四个属性。注释说这四个方法都不被认为是公共API的一部分，要避免依赖他们，但是依然可以使用，只不过风险要自己控制。

`options` 属性通过 `Object.create(null)` 创建，通过 `ASSET_TYPES.forEach` 赋予了三个属性：

```js
export const ASSET_TYPES = [
  'component',
  'directive',
  'filter'
]
```

之后是这句 `extend(Vue.options.components, builtInComponents)`。extend 来自于 src/shared/util.js 文件。查看下作用：就是将 `builtInComponents` 的属性混合到 `Vue.options.components` 中。`builtInComponents` 如下：

```js
import KeepAlive from './keep-alive'

export default {
  KeepAlive
}
```

到现在为止，`Vue.options` 已经变成了这样：

```js
Vue.options = {
  components: {
    KeepAlive
  },
  directives: Object.create(null),
  filters: Object.create(null),
  _base: Vue
}
```

在 `initGlobalAPI` 方法的最后部分，以 Vue 为参数调用了四个 `init*` 方法：

```js
initUse(Vue)
initMixin(Vue)
initExtend(Vue)
initAssetRegisters(Vue)
```

`initUse` 的作用是在 Vue 构造函数上添加 `use` 方法，也就是传说中的 `Vue.use` 这个全局 API。

`initMixin` 的作用是，在 Vue 上添加 `mixin` 这个全局 API。

`initExtend` 方法在 Vue 上添加了 `Vue.cid` 静态属性，和 `Vue.extend` 静态方法。

`initAssetRegisters` 为 Vue 添加了三个方法：`Vue.component、Vue.directive、Vue.filter`。

### 

之前是在 `src/core/index.js` 文件，我们在往上一层文件是在 `src/platforms/web/runtime/index.js`。

```js
// install platform specific utils
Vue.config.mustUseProp = mustUseProp
Vue.config.isReservedTag = isReservedTag
Vue.config.isReservedAttr = isReservedAttr
Vue.config.getTagNamespace = getTagNamespace
Vue.config.isUnknownElement = isUnknownElement

// install platform runtime directives & components
extend(Vue.options.directives, platformDirectives)
extend(Vue.options.components, platformComponents)

// install platform patch function
Vue.prototype.__patch__ = inBrowser ? patch : noop

// public mount method
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```

记得上一个文件 `src/core/index.js` 中就通过 `initGlobalAPI` 定义过 `config` 属性。其代理的值是从 `core/config.js` 文件导出的对象，这个对象最开始长成这样：

```js
Vue.config = {
  optionMergeStrategies: Object.create(null),
  silent: false,
  productionTip: process.env.NODE_ENV !== 'production',
  devtools: process.env.NODE_ENV !== 'production',
  performance: false,
  errorHandler: null,
  warnHandler: null,
  ignoredElements: [],
  keyCodes: Object.create(null),
  isReservedTag: no,
  isReservedAttr: no,
  isUnknownElement: no,
  getTagNamespace: noop,
  parsePlatformTagName: identity,
  mustUseProp: no,
  _lifecycleHooks: LIFECYCLE_HOOKS
}
```

看下注释，这是与平台有关的配置。

```js
Vue.config.mustUseProp = mustUseProp
Vue.config.isReservedTag = isReservedTag
Vue.config.isReservedAttr = isReservedAttr
Vue.config.getTagNamespace = getTagNamespace
Vue.config.isUnknownElement = isUnknownElement
```

这里的这四句就是在覆盖默认导出的 `config` 对象的属性。

接着是这两句代码：

```js
// install platform runtime directives & components
extend(Vue.options.directives, platformDirectives)
extend(Vue.options.components, platformComponents)
```

`Vue.options` 在前面也已经定义过了，`extend` 方法也见过。

看下 `platformDirectives` 和 `platformComponents`:

```js
<!-- platformDirectives -->
import model from './model'
import show from './show'

export default {
  model,
  show
}

<!-- platformComponents -->
import Transition from './transition'
import TransitionGroup from './transition-group'

export default {
  Transition,
  TransitionGroup
}
```