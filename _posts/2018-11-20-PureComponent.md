---
layout: post
title:  组件性能优化--PureComponent
date:   2017-11-20 13:08:00 +0800
categories: React
tag: React
---

* content
{:toc}

[函数式编程](https://peiyanhuang.github.io/MyBlog/2018/04/27/adequate/)里面非常重要的概念 —— 纯函数（Pure Function）。它由三大原则构成：

1. 给定相同的输入，它总是返回相同的输出;
2. 过程没有副作用(side effect);
3. 没有额外的状态依赖。

`PureComponent` 也就是纯组件，取代其前身 `PureRenderMixin`, `PureComponent` 是优化 React 应用程序最重要的方法之一，易于实施，只要把继承类从 `Component` 换成 `PureComponent` 即可，可以减少不必要的 `render` 的次数，从而提高性能。

### 1. 默认渲染行为的问题

在 React Component 的生命周期中，有一个 `shouldComponentUpdate` 方法。当一个组件的 `props` 或者 `state` 改变时，React 通过比较新返回的元素和之前渲染的元素来决定是否有必要更新实际的 DOM。当他们不相等时，React 会更新 DOM。

在一些情况下，可以通过重写这个生命周期函数 `shouldComponentUpdate` 来提升速度，它是在重新渲染过程开始前触发的。 这个函数默认返回 `true`，可使 React 执行更新：

```js
shouldComponentUpdate(nextProps, nextState) {
  return true;
}
```

如果你知道在某些情况下你的组件不需要更新，你可以在 `shouldComponentUpdate` 内返回 `false` 来跳过整个渲染进程，该进程包括了对该组件和之后的内容调用 `render()` 指令。

### 2. React.PureComponent

React 提供了一个辅助对象来实现 `shouldComponentUpdate` 中属性和值的浅比较 - 继承自 `React.PureComponent`。

```js
class CounterButton extends React.PureComponent {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

React 自动帮我们做了一层浅比较：

```js
if (this._compositeType === CompositeTypes.PureClass) {
  shouldUpdate = !shallowEqual(prevProps, nextProps)
  || !shallowEqual(inst.state, nextState);
}
```

而 `shallowEqual` 又做了什么呢？会比较 `Object.keys(state | props)` 的长度是否一致，每一个 `key` 是否两者都有，并且是否是一个引用，也就是只比较了第一层的值，确实很浅，所以深层的嵌套数据是对比不出来的。

### 3. 使用注意

#### 3.1 易变数据不能使用一个引用

如下：

```js
class App extends React.PureComponent {
  state = {
    items: [1, 2, 3]
  }
  handleClick = () => {
    let { items } = this.state;
    items.push(4);
    this.setState({ items });
  }
  render() {
    return (<div>
      <ul>
        {this.state.items.map(i => <li key={i}>{i}</li>)}
      </ul>
      <button onClick={this.handleClick}>delete</button>
    </div>)
  }
}
```

会发现，无论怎么点 delete 按钮，li 都不会变少，因为 items 用的是一个引用，`shallowEqual` 的结果为 `true`。避免此类问题最简单的方式是避免使用原来的引用类型属性或状态。上面例子中的 `handleClick` 可以用 `concat` 重写成：

```js
handleClick = () => {
  let { items } = this.state;
  this.setState({ items: items.concat([4]) });
}
```

或者使用解构的方式：

```js
handleClick = () => {
  let { items } = this.state;
  this.setState({ items: [...this.state.items, '4'] });
}
```

但如果 items 里面是引用类型数据：

```js
items: [{a: 1}, {a: 2}, {a: 3}]
```

这个时候

```js
state.items[0] === nextState.items[0] // false
```

子组件里还是 re-render 了。这样就需要我们保证不变的子组件数据的引用不能改变。这个时候可以使用 immutable-js 函数库。

#### 3.2 直接为 props 设置对象或数组

每次调用 React 组件其实都会重新创建组件，就算传入的数组或对象的值没有改变，但它们引用的地址会发生改变。如下：

```jsx
<Account style={{color: 'red'}} />
```

这样设置 `prop`, 每次渲染时 `style` 都是新的对象。对于这样的复制操作，我们只需提前赋值成常量，不直接使用对象字面量即可。我们为 `style` 设置一个默认值也是一样的道理：

```jsx
<Account style={this.props.style || {}} />
```

我们只需要将默认值保存成同一份引用，就可以避免这个问题：

```jsx
const defaultStyle = {}
<Account style={this.props.style || defaultStyle} />
```

#### 3.3 设置 props 方法并通过事件绑定在元素上

```jsx
handleChange(e) {
  this.props.update(e.target.value)
}
render() {
  return <MyInput onChange={this.handleChange.bind(this)} />
}
```

由于每次 render 操作 MyInput 组件的 onChange 属性都会返回一个新的函数，由于引用不一样，所以父组件的 render 也会导致 MyInput 组件的 render ，即使没有任何改动，所以需要尽量避免这样的写法，最好这样写：

```jsx
handleChange = (e) => {
  this.props.update(e.target.value)
}
render() {
  return <MyInput onChange={this.handleChange} />
}
```

或者把绑定移到构造器内：

```jsx
constructor(props) {
  super(props)
  this.handleChange = this.handleChange.bind(this)
}

handleChange(e) {
  this.props.update(e.target.value)
}
render() {
  return <MyInput onChange={this.handleChange} />
}
```

### 4. immutable.js

什么是 `Immutable Data`?

`Immutable Data` 就是一旦创建，就不能再被更改的数据。对 `Immutable` 对象的任何修改或添加删除操作都会返回一个新的 `Immutable` 对象。`Immutable` 实现的原理是 `Persistent Data Structure`（持久化数据结构），也就是使用旧数据创建新数据时，要保证旧数据同时可用且不变。同时为了避免 `deepCopy` 把所有节点都复制一遍带来的性能损耗，`Immutable` 使用了 `Structural Sharing`（结构共享），即如果对象树中一个节点发生变化，只修改这个节点和受它影响的父节点，其它节点则进行共享。

#### 4.1 Immutable 的优点

* 降低了“可变”带来的复杂度

可变（Mutable）数据耦合了 Time 和 Value 的概念，造成了数据很难被回溯。比如下面一段代码：

```js
function touchAndLog(touchFn) {
  let data = { key: 'value' };
  touchFn(data);
  console.log(data.key); // 猜猜会打印什么？
}
```

在不查看 `touchFn` 的代码的情况下，因为不确定它对 `data` 做了什么，你是不可能知道会打印什么（这不是废话吗）。但如果 `data` 是 `Immutable` 的呢，你可以很肯定的知道打印的是 `value`。

* 节省内存

`Immutable.js` 使用了 `Structure Sharing` 会尽量复用内存，甚至以前使用的对象也可以再次被复用。没有被引用的对象会被垃圾回收。

* Undo/Redo，Copy/Paste，甚至时间旅行这些功能做起来小菜一碟

因为每次数据都是不一样的，只要把这些数据放到一个数组里储存起来，想回退到哪里就拿出对应数据即可，很容易开发出撤销重做这种功能。

* 并发安全

传统的并发非常难做，因为要处理各种数据不一致问题，因此『聪明人』发明了各种锁来解决。但使用了 Immutable 之后，数据天生是不可变的，并发锁就不需要了。然而现在并没什么卵用，因为 JavaScript 还是单线程运行的啊。但未来可能会加入，提前解决未来的问题不也挺好吗？

* 拥抱函数式编程

Immutable 本身就是函数式编程中的概念，纯函数式编程比面向对象更适用于前端开发。因为只要输入一致，输出必然一致，这样开发的组件更易于调试和组装。

像 ClojureScript，Elm 等函数式编程语言中的数据类型天生都是 Immutable 的，这也是为什么 ClojureScript 基于 React 的框架 --- Om 性能比 React 还要好的原因。

#### 4.2 Immutable 的缺点

1. 需要学习新的 API

2. 增加了资源文件大小

3. 容易与原生对象混淆

虽然 Immutable.js 尽量尝试把 API 设计的原生对象类似，有的时候还是很难区别到底是 Immutable 对象还是原生对象，容易混淆操作。

Immutable 中的 Map 和 List 虽对应原生 Object 和 Array，但操作非常不同，比如你要用 map.get('key')而不是 map.key，array.get(0) 而不是 array[0]。另外 Immutable 每次修改都会返回新对象，也很容易忘记赋值。

当使用外部库的时候，一般需要使用原生对象，也很容易忘记转换。

下面给出一些办法来避免类似问题发生：

* 使用 Flow 或 TypeScript 这类有静态类型检查的工具
* 约定变量命名规则：如所有 Immutable 类型对象以 $$ 开头。
* 使用 Immutable.fromJS 而不是 Immutable.Map 或 Immutable.List 来创建对象，这样可以避免 Immutable 和原生对象间的混用。

### immutable.js 文档

[immutable-js](https://github.com/facebook/immutable-js/)