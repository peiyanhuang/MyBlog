---
layout: post
title:  Vue 的过渡效果
date:   2017-02-20 20:58:00 +0800
categories: 2017
tag: Vue
---

* content
{:toc}

### 使用过渡的条件

Vue 提供了 `transition` 的封装组件，在下列情形中，可以给任何元素和组件添加 `entering/leaving` 过渡

- 条件渲染 （使用 v-if）
- 条件展示 （使用 v-show）
- 动态组件
- 组件根节点

### 过渡的类名

- `v-enter`: 定义进入过渡的开始状态。在元素被插入时生效，在下一个帧移除。
- `v-enter-active`: 定义进入过渡的结束状态。在元素被插入时生效，在 transition/animation 完成之后移除。
- `v-leave`: 定义离开过渡的开始状态。在离开过渡被触发时生效，在下一个帧移除。
- `v-leave-active`: 定义离开过渡的结束状态。在离开过渡被触发时生效，在 transition/animation 完成之后移除。

`v` 是这些类名的前缀，可以自定义。

```
<transition name="fade">
    <p v-if="show">hello</p>
</transition>

//css
.fade-enter-active, .fade-leave {
  transition: opacity .5s
}
.fade-enter, .fade-leave-active {
  opacity: 0
}
```

### 动画

CSS 动画用法同 CSS 过渡，区别是在动画中 `v-enter` 类名在节点插入 DOM 后不会立即删除，而是在 `animationend` 事件触发时删除。

### 自定义类名

- enter-class
- enter-active-class
- leave-class
- leave-active-class

```
<transition
    name="custom-classes-transition"
    enter-active-class="animated tada"
    leave-active-class="animated bounceOutRight">
    <p v-if="show">hello</p>
</transition>
```

### javascript钩子

```
<transition
  v-on:before-enter="beforeEnter"
  v-on:enter="enter"
  v-on:after-enter="afterEnter"
  v-on:enter-cancelled="enterCancelled"
  v-on:before-leave="beforeLeave"
  v-on:leave="leave"
  v-on:after-leave="afterLeave"
  v-on:leave-cancelled="leaveCancelled"
>
  <!-- ... -->
</transition>

//js
 ...
methods: {
  // --------
  // 进入中
  // --------
  beforeEnter: function (el) {
    // ...
  },
  // 此回调函数是可选项的设置
  // 与 CSS 结合时使用
  enter: function (el, done) {
    // ...
    done()
  },
  afterEnter: function (el) {
    // ...
  },
  enterCancelled: function (el) {
    // ...
  },
  // --------
  // 离开时
  // --------
  beforeLeave: function (el) {
    // ...
  },
  // 此回调函数是可选项的设置
  // 与 CSS 结合时使用
  leave: function (el, done) {
    // ...
    done()
  },
  afterLeave: function (el) {
    // ...
  },
  // leaveCancelled 只用于 v-show 中
  leaveCancelled: function (el) {
    // ...
  }
}
```

当只用 JavaScript 过渡的时候， 在 enter 和 leave 中，回调函数 done 是必须的 。 否则，它们会被同步调用，过渡会立即完成。

对于仅使用 JavaScript 过渡的元素添加 v-bind:css="false"，Vue 会跳过 CSS 的检测。这也可以避免过渡过程中 CSS 的影响。

### apper

`appear` 特性设置节点的在初始渲染的过渡。

```
<transition appear>
  <!-- ... -->
</transition>
```

同样也可以自定义 CSS 类名。

```
<transition
  appear
  appear-class="custom-appear-class"
  appear-active-class="custom-appear-active-class"
>
  <!-- ... -->
</transition>
```

自定义 JavaScript 钩子：

```
<transition
  appear
  v-on:before-appear="customBeforeAppearHook"
  v-on:appear="customAppearHook"
  v-on:after-appear="customAfterAppearHook"
>
  <!-- ... -->
</transition>
```

### 过渡模式

- `in-out`: 新元素先进行过渡，完成之后当前元素过渡离开。
- `out-in`: 当前元素先进行过渡，完成之后新元素过渡进入。

```
<transition name="fade" mode="out-in">
  <!-- ... the buttons ... -->
</transition>
```

### <transition-group>

如果使用 `v-for` 需要渲染整个列表，就要用到 `<transition-group>` 。

它会以一个真实元素呈现：默认为一个 `<span>`。也可以通过 `tag` 特性更换为其他元素。

元素 一定需要 指定唯一的 `key` 特性值

```
<transition-group name="list" tag="p">
    <span v-for="item in items" v-bind:key="item" class="list-item">
      	{{ item }}
    </span>
</transition-group>
```

### v-move

`v-move`: 它会在元素的改变定位的过程中应用。像之前的类名一样，可以通过 `name` 属性来自定义前缀，也可以通过 `move-class` 属性手动设置。

需要注意的是使用 FLIP 过渡的元素不能设置为 display: inline 。作为替代方案，可以设置为 display: inline-block 或者放置于 flex 中.