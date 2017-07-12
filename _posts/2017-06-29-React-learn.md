---
layout: post
title:  React 学习入门
date:   2017-06-29 19:58:00 +0800
categories: React
tag: React
---

* content
{:toc}

### JSX

React DOM 使用驼峰(camelCase)属性命名约定, 而不是HTML属性名称。

```
class -> className
for -> htmlFor
onclick -> onClick ...
tabindex -> tabIndex
```

组件名称总是以大写字母开始。组件必须返回一个单独的根元素。

如果我们对 Button 绑定了一个点击事件，那么其子元素中也会绑定该点击事件。因此，如果 Button 中存在子元素，那么该子元素仍然是可以点击并触发点击事件的

```
&[disabled] > * {
    pointer-events: none;
}
```


`dangerouslySetInnerHTML`: 避免 React 转义字符。

*constructor / getInitialState*:

初始化 `this.state` 的值，只在组件装载之前调用一次。
如果是使用 ES6 的语法，你也可以在构造函数中初始化状态，比如：

```
class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = { count: props.initialCount };
  }

  render() {
    // ...
  }
}
```

### 生命周期

*组件挂载*：

`componentWillMount()`: 钩子在组件输出被渲染到 DOM 之前(render 方法执行前)调用。

`componentDidMount()`: 钩子在组件输出被渲染到 DOM 之后(render 方法执行后)调用。

*组件卸载*：

`componentWillUnmount()`: 卸载组件触发。

*更新组件触发*：

这些方法不会在首次 render 组件的周期调用

`componentWillReceiveProps(nextProps)`: 组件的 props 属性可以通过父组件来更改，这时，componentWillReceiveProps 将来被调用。可以在这个方法里更新 state,以触发 render 方法重新渲染组件。

`shouldComponentUpdate(nextProps, nextState)`: 接受需要更新的 props 和 state。可以通过判断让其在需要时更新，不需要时不更新。当返回 false 时，组件不在向下执行生命周期方法。

`componentWillUpdate(nextProps, nextState)`: 接受需要更新的 props 和 state。在 render 前执行，此时不能执行 setState()，

`componentDidUpdate(prveProps, prevState)`: 接受更新前的 props 和 state。在 render 后执行。

(未完)