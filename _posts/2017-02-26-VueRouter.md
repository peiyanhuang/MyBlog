---
layout: post
title:  Vue Router学习（一）
date:   2017-02-27 20:58:00 +0800
categories: JS
tag: Vue
---

* content
{:toc}

### 开始

一个简单的例子

```
<div id="app">
	<h1>hello</h1>
	<p>
		<router-link to="/foo">going to foo</router-link>
		<br>
		<router-link to="/bar">go to bar</router-link>
	</p>
	
	<router-view></router-view>
</div>

//index.js
import Vue from 'vue';
import VueRouter from 'vue-router';

Vue.use(VueRouter);		//如果使用模块化机制编程，導入Vue和VueRouter，要调用 Vue.use(VueRouter)

const Foo = { template: '<div>foo</div>' };
const Bar = { template: '<div>bar</div>' };

// 创建 router 实例，然后传 `routes` 配置
const router = new VueRouter({
  	routes: [
  		{ path: '/foo', component: Foo },
  		{ path: '/bar', component: Bar }
  	] 
});

// 创建和挂载根实例。
const apps = new Vue({
	el: "#app",
  	router
});
```

### HTML5 History 模式

`vue-router` 默认 `hash` 模式 —— 使用 `URL` 的 `hash` 来模拟一个完整的 URL

```
const router = new VueRouter({
  mode: 'history',  //默认 hash
  routes: [...]
})
```

### router-link

`<router-link>` 组件支持用户在具有路由功能的应用中（点击）导航。通过 `to` 属性指定目标地址，默认渲染成带有正确链接的 `<a>` 标签，可以通过配置 `tag` 属性生成别的标签.。

有时候，同一个路径可以匹配多个路由，此时，匹配的优先级就按照路由的定义顺序：谁先定义的，谁的优先级就最高。

- `to`：表示目标路由的链接。当被点击后，内部会立刻把 `to` 的值传到 `router.push()`

```
<!-- 字符串 -->
<router-link to="home">Home</router-link>
<!-- 渲染结果 -->
<a href="home">Home</a>

<!-- 使用 v-bind 的 JS 表达式 -->
<router-link v-bind:to="'home'">Home</router-link>

<!-- 不写 v-bind 也可以，就像绑定别的属性一样 -->
<router-link :to="'home'">Home</router-link>

<!-- 同上 -->
<router-link :to="{ path: 'home' }">Home</router-link>

<!-- 命名的路由 -->
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>

<!-- 带查询参数，下面的结果为 /register?plan=private -->
<router-link :to="{ path: 'register', query: { plan: 'private' }}">Register</router-link>
```

- `replace`: 设置 replace 属性的话，当点击时，会调用 router.replace() 而不是 router.push()，于是导航后不会留下 history 记录。

- `append`: 设置 append 属性后，则在当前（相对）路径前添加基路径。例如，我们从 /a 导航到一个相对路径 b，如果没有配置 append，则路径为 /b，如果配了，则为 /a/b

- `tag`: 有时候想要 <router-link> 渲染成某种标签，例如 <li>。 于是我们使用 tag prop 类指定何种标签，同样它还是会监听点击，触发导航。

- `active-class`: 设置 链接激活时使用的 CSS 类名。默认值可以通过路由的构造选项 linkActiveClass 来全局配置。

#### 动态路由

```
const router = new VueRouter({
  routes: [
    // 动态路径参数 以冒号开头
    { path: '/user/:id', component: User }
  ]
})

//$route.params.id => index
```

`/user/index`路径可以匹配上述路由。也可以在一个路由中设置多段『路径参数』，对应的值都会设置到 `$route.params` 中。

- `$route.path`: 对应当前路由的路径，总是解析为绝对路径
- `$route.params`: 一个 `key/value` 对象，包含了 动态片段 和 全匹配片段，如果没有路由参数，就是一个空对象。
- `$route.query`: 一个 `key/value` 对象，表示 `URL` 查询参数。例如，对于路径 `/foo?user=1`，则有 `$route.query.user == 1`，如果没有查询参数，则是个空对象。
- `$route.hash`: 当前路由的 hash 值 (带 #) ，如果没有 hash 值，则为空字符串。
- `$route.fullPath`: 完成解析后的 URL，包含查询参数和 hash 的完整路径。
- `$route.matched`: 一个数组，包含当前路由的所有嵌套路径片段的 路由记录 。路由记录就是 routes 配置数组中的对象副本（还有在 children 数组）。
- `$route.name`: 当前路由的名称，如果有的话。

### 嵌套路由

```
//html
<div id="app">
  <router-view></router-view>
</div>

//user.vue
<template>
	<div>
		<p>{{ $route.path }}</p>
		<p>{{ $route.params.animal }}</p>
		<router-view></router-view>
	</div>
</template>

//index.js
const router = new VueRouter({
  	routes: [
  		{ path: '/foo', component: Foo },
	    { path: '/user/:animal', component: User,
			children: [
				{ path: 'post', component: Post}
			]
		}
  	]
})
```

`#app` 下的 `<router-view>` 是最顶层的出口，渲染最高级路由匹配到的组件。
一个被渲染的组件同样可以包含自己的嵌套 `<router-view>`。

要在嵌套的出口中渲染组件，需要在 `VueRouter` 的参数中使用 `children` 配置：

要注意，以 `/` 开头的嵌套路径会被当作根路径。 

### 编程式的导航

- `router.push(location)`: 想要导航到不同的 `URL`，则使用 `router.push` 方法。这个方法会向 `history` 栈添加一个新的记录，点击 `<router-link :to="...">` 等同于调用 `router.push(...)`。

```
// 字符串
router.push('home')

// 对象
router.push({ path: 'home' })

// 命名的路由
router.push({ name: 'user', params: { userId: 123 }})

// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }})
```

- `router.replace(location)`: 跟 `router.push` 很像，唯一的不同就是，它不会向 `history` 添加新记录，而是跟它的方法名一样 —— 替换掉当前的 `history` 记录。

- `router.go(n)`: 这个方法的参数是一个整数，意思是在 `history` 记录中向前或者后退多少步，类似 `window.history.go(n)`。

### 命名路由

```
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})
```

要链接到一个命名路由，可以给 router-link 的 to 属性传一个对象：

```
<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
```

### 命名视图

```
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

一个视图使用一个组件渲染，因此对于同个路由，多个视图就需要多个组件。确保正确使用 `components` 配置：

```
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```

### 重命名和别名

重定向也是通过 routes 配置来完成，下面例子是从 /a 重定向到 /b：

```
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: '/b' }
  ]
})

//或者
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: { name: 'foo' }}
  ]
})

//或者
const router = new VueRouter({
  routes: [
    { path: '/a', redirect: to => {
      // 方法接收 目标路由 作为参数
      // return 重定向的 字符串路径/路径对象
    }}
  ]
})
```

`/a` 的别名是 `/b`，意味着，当用户访问 `/b` 时，`URL` 会保持为 `/b`，但是路由匹配则为 `/a`，就像用户访问 `/a` 一样。

上面对应的路由配置为：

```
const router = new VueRouter({
  routes: [
    { path: '/a', component: A, alias: '/b' }
  ]
})
```

『别名』的功能让你可以自由地将 UI 结构映射到任意的 URL，而不是受限于配置的嵌套路由结构。