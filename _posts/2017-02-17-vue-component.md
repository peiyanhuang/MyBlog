---
layout: post
title:  Vue 组件
date:   2017-02-16 20:58:00 +0800
categories: JS
tag: Vue
---

* content
{:toc}

### 组件使用限制

```
<select>
	<my-component></my-component>
</select>
```

在这里自定义组件 `<my-component>` 无效，因为 Vue 只有在浏览器解析和标准化 HTML 后才能获取模版内容。像 `<ul> ， <ol>， <table> ， <select>` 这些元素限制了其内部的元素。

变通的方案是使用特殊的 is 属性：

```
<table>
  <tr is="my-row"></tr>
</table>
```

使用来自以下来源之一的字符串模板，这些限制将不适用:

- `<script type="text/x-template">`
- JavaScript内联模版字符串
- .vue 组件

### 构建组件

vuejs构建组件使用

	Vue.component('componentName',{ /*component*/ })；

这里注意一点，组件要先注册再使用，也就是说：

```
// 注册
Vue.component('my-component', {
  template: '<div>A custom component!</div>'
})

// 创建根实例
new Vue({
  el: '#example'
})
```

如果反过来会报错，因为反过来代表先使用了组件的，但是组件却没注册。

局部注册组件

```
//例一
var Child = {
  template: '<div>A custom component!</div>'
}
new Vue({
  // ...
  components: {
    'my-component': Child	// <my-component> 将只在父模板可用
  }
})

//例二
var Parent = Vue.extend({
    template: '<div>I\'m Parent, My children: <myComponent></myComponent><child></child></div>',
    components: {
        'myComponent': {
            template: '<div>child component!</div>',
        },
        'child': {
            template: '<div>child component!</div>',
        }
    }
});
```

使用组件时，大多数可以传入到 Vue 构造器中的选项可以在注册组件时使用，有一个例外： `data` 必须是函数。

```
<div id="example-2">
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
</div>

var data = { counter: 0 }
Vue.component('simple-counter', {
	template: '<button v-on:click="counter += 1">{{ counter }}</button>',
	// data 是一个函数，因此 Vue 不会警告，
	// 但是我们为每一个组件返回了同一个对象引用
	data: function () {
		return data
	}
})
new Vue({
  el: '#example-2'
})
```

由于这三个组件共享了同一个 data ， 因此增加一个 counter 会影响所有组件！我们可以通过为每个组件返回新的 data 对象来解决这个问题：

```
data: function () {
	return {
		counter: 0
	}
}
```

### prop

在 Vue.js 中，父组件通过 `props` 向下传递数据给子组件，子组件通过 `events` 给父组件发送消息。

props 可以是数组或对象，用于接收来自父组件的数据。

```
// 简单语法
Vue.component('props-demo-simple', {
  props: ['size', 'myMessage']
})

// 对象语法，提供校验
Vue.component('props-demo-advanced', {
  props: {
    // 只检测类型
    height: Number,
    // 检测类型 + 其他验证
    age: {
      type: Number,
      default: 0,
      required: true,
      //自定义验证
      validator: function (value) {
        return value >= 0
      }
    },
    // 数组／对象的默认值应当由一个工厂函数返回
    propE: {
      type: Object,
      default: function () {
        return { message: 'hello' }
      }
    }
  }
})
```

`prop` 是单向绑定的：当父组件的属性变化时，将传导给子组件，但是不会反过来。每次父组件更新时，子组件的所有 prop 都会更新为最新值。这意味着不应该在子组件内部改变 `prop` 。

`prop` 是一个对象而不是字符串数组时，它包含验证要求：

`type` 可以是下面原生构造器：

- String
- Number
- Boolean
- Function
- Object
- Array

### 字符串模板 和 非字符串模板

HTML 特性不区分大小写。当使用非字符串模版时，prop的名字形式会从 camelCase 转为 kebab-case（短横线隔开）

非字符串模板指：

```
Vue.component('child', {
  // camelCase in JavaScript
  props: ['myMessage'],
  template: '<span>{{ myMessage }}</span>'
})

<!-- kebab-case in HTML -->
<child my-message="hello!"></child>
```

字符串模板指单文件组件：如[单文件组件](http://cn.vuejs.org/v2/guide/single-file-components.html)

使用字符串模版，不用在意这些限制。

### 自定义事件

父组件是使用 `props` 传递数据给子组件，子组件可以使用自定义事件向父组件传递数据。

```
<div id="counter-event-example">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>

Vue.component('button-counter', {
  template: '<button v-on:click="crement">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    crement: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})
new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
```

上例中，`v-on:increment` 的 `increment` 是自定义事件，`incrementTotal` 是事件的方法。在组件 `button-counter` 中，绑定了 `click` 事件，子组件触发 `click` 事件， `crement` 函数通过 `$emit` 触发 `increment` 自定义事件。

### $on 、$emit 、$off 、$once

`vm.$on(event, callback)`: 监听当前实例上的自定义事件。事件可以由vm.$emit触发。回调函数会接收所有传入事件触发函数的额外参数。

```
vm.$on('test', function (msg) {
  console.log(msg)
})
vm.$emit('test', 'hi')     // $on 触发 -> "hi"
```

`vm.$emit(event, [...arg])`: 触发当前实例上的事件。附加参数都会传给监听器回调。

`vm.off(event, callback)`: 移除事件监听器。

- 如果没有提供参数，则移除所有的事件监听器；
- 如果只提供了事件，则移除该事件所有的监听器；
- 如果同时提供了事件与回调，则只移除这个回调的监听器。

`vm.once(event, callback)`: 监听一个自定义事件只触发一次，在第一次触发之后移除除监听器。

### slot

在使用组件时，常常要像这样组合它们：

```
<app>
  <app-header></app-header>
  <app-footer></app-footer>
</app>
```

- <app> 组件不知道它的挂载点会有什么内容。挂载点的内容是由<app>的父组件决定的。
- <app> 组件很可能有它自己的模版。

为了让组件可以组合，我们需要一种方式来混合父组件的内容与子组件自己的模板。这个过程被称为 内容分发

#### 单个slot

除非子组件模板包含至少一个 `<slot>` 插口，否则父组件的内容将会被丢弃。当子组件模板只有一个没有属性的 `slot` 时，父组件整个内容片段将插入到 `slot` 所在的 `DOM` 位置，并替换掉 `slot` 标签本身。

假定 my-component 组件有下面模板：

```
//组件模板
<div>
  <h2>我是子组件的标题</h2>
  <slot>
    只有在没有要分发的内容时才会显示。
  </slot>
</div>

//父组件模版：
<div>
  <h1>我是父组件的标题</h1>
  <my-component>
    <p>这是一些初始内容</p>
    <p>这是更多的初始内容</p>
  </my-component>
</div>

//渲染结果：
<div>
  <h1>我是父组件的标题</h1>
  <div>
    <h2>我是子组件的标题</h2>
    <p>这是一些初始内容</p>
    <p>这是更多的初始内容</p>
  </div>
</div>
```

#### 具名slot

`<slot>` 元素可以用一个特殊的属性 `name` 来配置如何分发内容。多个 `slot` 可以有不同的名字。具名 `slot` 将匹配内容片段中有对应 slot 特性的元素。

仍然可以有一个匿名 `slot` ，它是默认 `slot` ，作为找不到匹配的内容片段的备用插槽。如果没有默认的 `slot` ，这些找不到匹配的内容片段将被抛弃。

```
//app-layout 组件
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>

//父组件模版：
<app-layout>
  <h1 slot="header">这里可能是一个页面标题</h1>
  <p>主要内容的一个段落。</p>
  <p>另一个主要段落。</p>
  <p slot="footer">这里有一些联系信息</p>
</app-layout>

//渲染结果为：
<div class="container">
  <header>
    <h1>这里可能是一个页面标题</h1>
  </header>
  <main>
    <p>主要内容的一个段落。</p>
    <p>另一个主要段落。</p>
  </main>
  <footer>
    <p>这里有一些联系信息</p>
  </footer>
</div>
```

#### 作用域插槽

作用域插槽是一种特殊类型的插槽，用作使用一个（能够传递数据到）可重用模板替换已渲染元素。

在父级中，具有特殊属性 `scope` 的 `<template>` 元素，表示它是作用域插槽的模板。`scope` 的值对应一个临时变量名，此变量接收从子组件中传递的 `prop` 对象：

```
<my-awesome-list :items="items">
	<!-- 作用域插槽也可以在这里命名 -->
	<template slot="item" scope="props">
		<li class="my-fancy-item">{{ props.text }}</li>
	</template>
</my-awesome-list>

//my-awesome-list模板
<ul>
  	<slot name="item" v-for="item in items" :text="item.text">
    	<!-- fallback content here -->
  	</slot>
</ul>
```

### keep-alive

`<keep-alive>` 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。

```
<!-- 基本 -->
<keep-alive>
  <component :is="view"></component>
</keep-alive>

<!-- 多个条件判断的子组件 -->
<keep-alive>
  <comp-a v-if="a > 1"></comp-a>
  <comp-b v-else></comp-b>
</keep-alive>

<!-- 和 <transition> 一起使用 -->
<transition>
  <keep-alive>
    <component :is="view"></component>
  </keep-alive>
</transition>
```

### inline-template

```
<my-component inline-template>
  <div>
    <p>These are compiled as the component's own template.</p>
    <p>Not parent's transclusion content.</p>
  </div>
</my-component>
```

如果子组件有 `inline-template` 特性，组件将把它的内容当作它的模板，而不是把它当作分发内容。

### text/x-template

另一种定义模版的方式是在 JavaScript 标签里使用 text/x-template 类型，并且指定一个id。例如：

```
<script type="text/x-template" id="hello-world-template">
  <p>Hello hello hello</p>
</script>

Vue.component('hello-world', {
  template: '#hello-world-template'
})
```