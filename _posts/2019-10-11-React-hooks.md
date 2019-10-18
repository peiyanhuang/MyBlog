---
layout: post
title:  React hooks 的使用
date:   2019-10-11 19:00:00 +0800
categories: React
tag: React
---

* content
{:toc}

React 在 v16.8 版本引入了全新的 API，叫做 React Hooks，可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。

Redux 的作者 Dan Abramov 总结了组件类的几个缺点。

- 大型组件很难拆分和重构，也很难测试;
- 业务逻辑分散在组件的各个方法之中，导致重复逻辑或关联逻辑;
- 组件类引入了复杂的编程模式，比如 render props 和高阶组件。

React 团队希望，组件不要变成复杂的容器，最好只是数据流的管道。开发者根据需要，组合管道即可。 组件的最佳写法应该是函数，而不是类。

但是，这种写法有重大限制，必须是纯函数，不能包含状态，也不支持生命周期方法，因此无法取代类。

React Hooks 的设计目的，就是加强版函数组件，完全不使用"类"，就能写出一个全功能的组件。

**那么，什么是 Hook?**Hook 是一些可以让你在函数组件里“钩入” React state 及生命周期等特性的函数。Hook 不能在 class 组件中使用 —— 这使得你不使用 class 也能使用 React。

React 默认提供的四个最常用的钩子:

```js
useState()
useEffect()
useContext()
useReducer()
```

### State Hook - useState()

`useState()` 用于为函数组件引入状态（state），这个用法比较简单，这里不多做介绍。Example：

```jsx
import React, { useState } from "react";

export default function Button()  {
  const [buttonText, setButtonText] =  useState("Click Me!");
  const [list, setList] = useState([{id: 1, label: "0"}]);

  function handleClick()  {
  	let listCopy = Array.from(list);
  	listCopy.push({id: list.length + 1, label: Math.floor(Math.random() * 100)})
  	setList(listCopy);
    return setButtonText("Thanks, been clicked!");
  }

  return  (<div>
		<button onClick={handleClick}>{buttonText}</button>
		{
			list.map(item => {
				return <p key={item.id}>{item.id}: {item.label}</p>
			})
		}
	</div>);
}
```

### Effect Hook - useEffect()

`useEffect()` 用来引入具有**副作用**的操作，最常见的就是向服务器请求数据、订阅或者手动修改过 DOM。以前，放在 `componentDidMount` 里面的代码，现在可以放在 `useEffect()`。用法如下：

```js
useEffect(()  =>  {
  // Async Action
}, [dependencies])
```

`useEffect()` 接受两个参数。第一个参数是一个函数，副作用代码放在里面。第二个参数是一个数组，用于给出 Effect 的依赖项，只要这个数组发生变化，`useEffect()` 就会执行。第二个参数可以省略，这时每次组件渲染时，就会执行 `useEffect()`。

如果想执行只运行一次的 effect（仅在组件挂载和卸载时执行），可以传递一个空数组（[]）作为第二个参数。这就告诉 React 你的 effect 不依赖于 props 或 state 中的任何值，所以它永远都不需要重复执行。

跟 `useState` 一样，`useEffect` 也可以在组件中多次使用。

```jsx
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // 相当于 componentDidMount 和 componentDidUpdate:
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  }, [count]); // 仅在 count 更改时更新

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

默认情况下，它在第一次渲染之后和每次更新之后都会执行。副作用函数还可以通过返回一个函数来指定如何“清除”副作用。 React 会在组件卸载的时候执行清除操作。

### useReducer()

`useState` 的替代方案。它接收一个形如 `(state, action) => newState` 的 reducer，并返回当前的 state 以及与其配套的 dispatch 方法。

```js
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

当你涉及多个子值的复杂 state(状态) 逻辑时，`useReducer` 通常优于 `useState`。


有两种不同初始化 useReducer state 的方式。将初始 state 作为第二个参数传入 `useReducer` 是最简单的方法:

```jsx
/* reducer */
const myReducer = (state, action) => {
  switch(action.type)  {
    case('countUp'):
      return  {
        ...state,
        count: state.count + 1
      }
    default:
      return  state;
  }
}

/* component */
function App() {
  const [state, dispatch] = useReducer(myReducer, { count: 0 });
  return  (
    <div className="App">
      <button onClick={() => dispatch({ type: 'countUp' })}>
        +1
      </button>
      <p>Count: {state.count}</p>
    </div>
  );
}
```

第二种可以选择惰性地创建初始 state。需要将 init 函数作为 `useReducer` 的第三个参数传入，这样初始 state 将被设置为 `init(initialArg)`:

```jsx
function init(initialCount) {
  return {count: initialCount};
}

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    case 'reset':
      return init(action.payload);
    default:
      throw new Error();
  }
}

function Counter({initialCount}) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  return (
    <>
      Count: {state.count}
      <button
        onClick={() => dispatch({type: 'reset', payload: initialCount})}>
        Reset
      </button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
    </>
  );
}
```

**与 useState 的区别**

- 当 state 状态值结构比较复杂时，使用 `useReducer` 更有优势。
- 使用 `useState` 获取的 `setState` 方法更新数据时是异步的；而使用 `useReducer` 获取的 `dispatch` 方法更新数据是同步的。

### 自定义 Hook

通过自定义 Hook，可以将组件逻辑提取到可重用的函数中。

自定义 Hook 是一个函数，其名称以 “use” 开头，函数内部可以调用其他的 Hook。

```jsx
<!-- usePerson 就是一个自定义的 Hook -->
const usePerson = (personId) => {
  const [loading, setLoading] = useState(true);
  const [person, setPerson] = useState({});
  useEffect(() => {
    setLoading(true);
    fetch(`https://swapi.co/api/people/${personId}/`)
      .then(response => response.json())
      .then(data => {
        setPerson(data);
        setLoading(false);
      });
  }, [personId]);  
  return [loading, person];
};

const Person = ({ personId }) => {
  const [loading, person] = usePerson(personId);

  if (loading === true) {
    return <p>Loading ...</p>;
  }

  return (
    <div>
      <p>You're viewing: {person.name}</p>
      <p>Height: {person.height}</p>
      <p>Mass: {person.mass}</p>
    </div>
  );
};
```

### 其他 hooks

#### useContext

`useContext` 让你不使用组件嵌套就可以订阅 React 的 Context。

```jsx
import React, { useContext } from "react";
import ReactDOM from "react-dom";

const AppContext = React.createContext({});

const Navbar = () => {
  const { username } = useContext(AppContext)

  return (
    <div className="navbar">
      <p>AwesomeSite</p>
      <p>{username}</p>
    </div>
  )
}

const Messages = () => {
  const { username } = useContext(AppContext)

  return (
    <div className="messages">
      <h1>Messages</h1>
      <p>1 message for {username}</p>
      <p className="message">useContext is awesome!</p>
    </div>
  )
}

function App() {
  return (
    <AppContext.Provider value={{
      username: 'superawesome'
    }}>
      <div className="App">
        <Navbar />
        <Messages />
      </div>
    </AppContext.Provider>
  );
}
```

#### useCallback

```js
const memoizedCallback = useCallback(() => { doSomething(a, b); }, [a, b]);
````

返回一个 memoized 回调函数。把内联回调函数及依赖项数组作为参数传入 useCallback，它将返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（例如 shouldComponentUpdate）的子组件时，它将非常有用。

#### useMemo

```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

返回一个 memoized 值。传递“创建”函数和依赖项数组。`useMemo` 只会在其中一个依赖项发生更改时重新计算 memoized 值。此优化有助于避免在每个渲染上进行昂贵的计算。

传入 `useMemo` 的函数会在渲染期间执行。请不要在这个函数内部执行与渲染无关的操作，诸如副作用这类的操作属于 `useEffect` 的适用范畴，而不是 `useMemo`。

#### useLayoutEffect

```js
useLayoutEffect(() => { doSomething });
````

与 `useEffect Hooks` 类似，都是执行副作用操作。但是它是在所有 DOM 更新完成后触发。可以用来执行一些与布局相关的副作用，比如获取 DOM 元素宽高，窗口滚动距离等等。

进行副作用操作时尽量优先选择 `useEffect`，以免阻止视觉更新。与 DOM 无关的副作用操作请使用 `useEffect`。

### Hook 规则

- 不要从常规 JavaScript 函数调用 Hooks;
- 不要在循环，条件或嵌套函数中调用 Hooks;
- 必须在组件的顶层调用 Hooks;
- 可以从 React 功能组件调用 Hooks;
- 可以从自定义 Hooks 中调用 Hooks;
- 自定义 Hooks 必须使用 use 开头，这是一种约定。

ESLint 插件: eslint-plugin-react-hooks

```js
// 你的 ESLint 配置
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error", // 检查 Hook 的规则
    "react-hooks/exhaustive-deps": "warn" // 检查 effect 的依赖
  }
}
```

### 参考

[React 官网](https://reactjs.org/docs/hooks-intro.html)