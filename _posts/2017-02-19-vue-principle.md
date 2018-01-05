---
layout: post
title:  Vue 的响应式原理
date:   2017-02-19 20:58:00 +0800
categories: 2017
tag: Vue
---

* content
{:toc}

### 追踪变化

把一个普通 `Javascript` 对象传给 `Vue` 实例的 `data` 选项，`Vue` 将遍历此对象所有的属性，并使用 `Object.defineProperty` 把这些属性全部转为 `getter/setter`。用户看不到 `getter/setter`，但是在内部它们让 `Vue` 追踪依赖，在属性被访问和修改时通知变化。

每个组件实例都有相应的 `watcher` 实例对象，它会在组件渲染的过程中把属性记录为依赖，之后当依赖项的 `setter` 被调用时，会通知 `watcher` 重新计算，从而致使它关联的组件得以更新。

### Vue.set() 、vm.$set() 和 Vue.delete() 、vm.$delete()

Vue 不允许在已经创建的实例上动态添加新的根级响应式属性。然而它可以使用 `Vue.set(object, key, value)` 方法将响应属性添加到嵌套的对象上：

	Vue.set(vm.someObject, 'b', 2)

还可以使用 `vm.$set` 实例方法:

	this.$set(this.someObject,'b',2)

注意: 对象不能是 Vue 实例，或者 Vue 实例的根数据对象

有时你想向已有对象上添加一些属性，例如使用 `Object.assign()` 或 `_.extend()` 方法来添加属性。但是，添加到对象上的新属性不会触发更新。在这种情况下可以创建一个新的对象，让它包含原对象的属性和新的属性：

	this.someObject = Object.assign({}, this.someObject, { a: 1, b: 2 })

`Vue.delete(obj, key) 、vm.$delete(obj, key)`: 删除对象的属性。

### Vue 的生命周期和钩子

![image]({{ '/images/lifecycle.png' | prepend: site.baseurl }})

`beforeCreate`: 在实例初始化之后，数据观测(data observer) 和 event/watcher 事件配置之前被调用。

`create`: 实例已经创建完成之后被调用。在这一步，实例已完成以下的配置：数据观测(data observer)，属性和方法的运算， watch/event 事件回调。然而，挂载阶段还没开始，$el 属性目前不可见。

`beforeMount`: 在挂载开始之前被调用：相关的 render 函数首次被调用。

`mounted`: el 被新创建的 vm.$el 替换，并挂载到实例上去之后调用该钩子。如果 root 实例挂载了一个文档内元素，当 mounted 被调用时 vm.$el 也在文档内。

`beforeUpdate`: 数据更新时调用，发生在虚拟 DOM 重新渲染和打补丁之前。
你可以在这个钩子中进一步地更改状态，这不会触发附加的重渲染过程。

`update`: 由于数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子。
当这个钩子被调用时，组件 DOM 已经更新，所以你现在可以执行依赖于 DOM 的操作。然而在大多数情况下，你应该避免在此期间更改状态，因为这可能会导致更新无限循环。

`activated`: keep-alive 组件激活时调用。

`deactivated`: keep-alive 组件停用时调用。

`beforeDestroy`: 实例销毁之前调用。在这一步，实例仍然完全可用。

`destroy`: Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。

### 生命周期实例方法

`vm.$mount([elementOrSelector])`: 如果 Vue 实例在实例化时没有收到 el 选项，则它处于“未挂载”状态，没有关联的 DOM 元素。可以使用 vm.$mount() 手动地挂载一个未挂载的实例。

如果没有提供 `elementOrSelector` 参数，模板将被渲染为文档之外的的元素，并且你必须使用原生DOM API把它插入文档中。

`vm.$forceUpdate()`: 迫使Vue实例重新渲染。注意它仅仅影响实例本身和插入插槽内容的子组件，而不是所有子组件。

`vm.$nextTick([callback])`: 将回调延迟到下次 DOM 更新循环之后执行。在修改数据之后立即使用它，然后等待 DOM 更新。回调函数在 DOM 更新完成后就会调用。它跟全局方法 Vue.nextTick 一样，不同的是回调的 this 自动绑定到调用它的实例上。

`vm.$destroy()`: 完全销毁一个实例。清理它与其它实例的连接，解绑它的全部指令及事件监听器。