---
layout: post
title:  getDerivedStateFromProps派生状态
date:   2018-12-11 19:58:00 +0800
categories: React
tag: React
---

* content
{:toc}

### getDerivedStateFromProps派生状态

很长一段时间，`componentWillReceiveProps` 生命周期是在不进行额外 `render` 的前提下，响应 `props` 中的改变并更新 `state` 的唯一方式。在 16.3 版本中，一个新的生命周期函数 `getDerivedStateFromProps` 替代它去更安全地解决相同的问题。

在 16.4 版本前，`getDerivedStateFromProps` 只有当父组件重新渲染时，才会调用这个函数。同时，当这个组件执行 `setState` 时不会触发这个函数。在 16.4 版本时修复了这个 bug。

`getDerivedStateFromProps` 这个函数在组件的每一次渲染中都会被调用。

* `getDerivedStateFromProps` 函数的使用应避免副作用

副作用是在计算结果的过程中，系统状态的一种变化，或者与外部世界进行的可观察的交互。`getDerivedStateFromProps` 函数应该是一个纯净的 `props` 和 `state` 的函数。

`getDerivedStateFromeProps` 存在仅有一个目的。其能够让组件在 `prop` 变更时更新内部的状态。官方的博文提供了一些例子，例如[基于当前变更的偏移（offset）prop 记录当前的滚动方向](https://react.docschina.org/blog/2018/03/27/update-on-async-rendering.html#updating-state-based-on-props) 或者 [通过资源 prop 加载额外的特定资源](https://react.docschina.org/blog/2018/03/27/update-on-async-rendering.html#fetching-external-data-when-props-change)。

作为一个通用规则，派生状态应谨慎使用。由派生状态导致的问题最终都可归结为两点（1）无条件的通过 `props` 来更新状态或（2）无论 `props` 是否和状态匹配都更新状态。

* 若仅通过当前的 `props` 使用派生状态来缓存一些计算操作，则没必要使用派生状态。
* 若只是无条件的更新派生状态或无论 `props` 和状态是否匹配都进行更新，你的组件可能太过于频繁的重置它的内部状态。

### 使用派生状态的一些常见问题

重说一遍：`getDerivedStateFromProps` 这个函数在组件的每一次渲染中都会被调用。

一个普遍的误解是 `getDerivedStateFromProps` 和 `componentWillReceiveProps` 仅在 `props` 改变时被调用。这些生命周期会在父组件重新渲染时被调用，无论其 `props` 是否和之前有不同。由于这一原因，总是无条件地使用这些生命周期重载状态是不安全的。这么做可能会导致更新状态的丢失。例如：

```jsx
class EmailInput extends Component {
  state = { email: this.props.email };

  render() {
    return <input onChange={this.handleChange} value={this.state.email} />;
  }

  handleChange = event => {
    this.setState({ email: event.target.value });
  };

  componentWillReceiveProps(nextProps) {
    this.setState({ email: nextProps.email });
  }
}
```

如果我们组件的父元素重渲，任何我们在 `<input>` 中的输入都将丢失。使用 `shouldComponentUpdate` 方法保证当且仅当 `email prop` 发生变更时才进行重渲可能能修复该问题。然而在实践中，组件通常可以接受多个 `props`；另一个 `prop` 的变更仍会导致重渲并进行错误的重置。

修改下：

```jsx
componentWillReceiveProps(nextProps) {
  if (nextProps.email !== this.props.email) {
    this.setState({
      email: nextProps.email
    });
  }
}
```

这仍然存在一个潜在的问题。想象一个使用了之前输入框组件的密码管理应用。当定位到了使用相同邮箱的两个账户，输入框将无法进行重设。

以上例子同样的也对 `getDerivedStateFromProps` 适用。

### 替代方法

我认为下面这句话应该牢记：**对于数据的任何部分，你需要保证其是一个组件唯一数据源，并避免将其复制给其他组件。**

#### 1.完全受控组件

```jsx
function EmailInput(props) {
  return <input onChange={props.onChange} value={props.email} />;
}
```

将状态从我们的组件中完全移除，统一由父组件控制。

#### 2.带 key 的完全不受控组件

为了在切换不同内容时重置输入值，我们可以使用 React 的 `key` 属性。当 `key` 改变，React 会重新创建一个新的组件而不是更新它。

```jsx
class EmailInput extends Component {
  state = { email: this.props.defaultEmail };

  handleChange = event => {
    this.setState({ email: event.target.value });
  };

  render() {
    return <input onChange={this.handleChange} value={this.state.email} />;
  }
}

<EmailInput
  defaultEmail={this.props.user.email}
  key={this.props.user.id}
/>
```

每当 `id` 改变，`EmailInput` 会被重新创建然后 `state` 会被重置为最新的 `defaultEmail` 值。其实无需为每一个输入框添加一个 `key`。对整个表单添加一个 `key` 会显得更有用。

这一方是听上去可能比较慢，但性能上并没有明显的差异。如果该组件包含了繁重的逻辑如通过对比传递给子树的 `prop` 来进行更新等， 使用 `key` 甚至会更快。