---
layout: post
title:  理解 React 高阶组件
date:   2018-04-27 13:08:00 +0800
categories: React
tag: React
---

* content
{:toc}

### 1. 什么是高阶组件？

`higher-order function`(高阶函数) 这在函数式编程中是一个基本的概念。他描述的是这样一个函数：这种函数接受函数作为输入，或是输出一个函数。常用的 `map reduce sort` 等都是高阶函数。

`higher-order component`(高阶组件) 类似于高阶函数，他接受 React 组件作为输入，输出一个新的 React 组件。

### 2. 高阶组件

比如说有两个组件如下：

```javascript
class Goodbye extends Component {
    constructor(props) {
        super(props);
        this.state = {
            username: ''
        }
    }

    componentDidMount() {
        setTimeout(() => {
            this.setState({
	            username: username
	        });
        }, 100);
    }

    render() {
        return (
            <div>goodbye {this.state.username}</div>
        )
    }
}

class Welcome extends Component {
    constructor(props) {
        super(props);
        this.state = {
            username: ''
        }
    }

    componentDidMount() {
        setTimeout(() => {
            this.setState({
	            username: username
	        });
        }, 100);
    }

    render() {
        return (
            <div>welcome {this.state.username}</div>
        )
    }
}
```

上面这两个组件大部分代码都是重复的，可以用高阶组件改写下：

```javascript
const MyContainer = (MyComponent) => {
    return class extends React.Component {
        constructor(props) {
	        super(props);
	        this.state = {
	            username: ''
	        }
	    }

	    componentDidMount() {
	        setTimeout(() => {
	            this.setState({
		            username: username
		        });
	        }, 100);
	    }
        
        render() {
            return (
                <MyComponent { ...this.state }></MyComponent>
            );
        }
    }
}

class Welcome extends Component {
    constructor(props) {
        super(props);
    }

    render() {
        return (
            <div>Welcome {this.props.username}</div>
        )
    }
}

class Goodbye extends Component {
    constructor(props) {
        super(props);
    }

    render() {
        return (
            <div>Goodbye {this.props.username}</div>
        )
    }
} 
```

这里高阶组件就是把 username 通过 props 传递给目标组件了。目标组件只管从 props 里面拿来用就好了。


#### 抽象 state

我们可以通过 MyContainer 提供的 props 和回调函数抽象 state，将原组件抽象为展示型组件，分离内部状态。例如：

```javascript
import React from "react";

const MyContainer = (MyComponent) => {
    return class extends React.Component {
        constructor(props) {
            super(props);
            this.state = {
                name: '',
                disabled: false,
            };

            this.onNameChange = this.onNameChange.bind(this);
        }
        
        onNameChange(event) {
            this.setState({
                name: event.target.value
            });
        }

        render() {
            const newProps = {
                name: {
                    value: this.state.name,
                    onChange: this.onNameChange
                }
            }
            return (
                <MyComponent { ...newProps } { ...this.props }></MyComponent>
            );
        }
    }
}

class MyComponent extends React.Component {
    constructor(props) {
        super(props);
    }

    render() {
        return (
            <div>
                <p>input: {this.props.name.value}</p>
                <input type="text" { ...this.props.name } />
            </div>
        );
    }
}

const Container = MyContainer(MyComponent);

export default Container;
```

这里，我把 `input` 组件中对 `name prop` 的 `onChange` 方法提取到高阶组件中，这样就有效的抽象了同样的 `state` 操作。

#### 通过 refs 使用引用

在高阶组件中，我们可以接受 MyComponent 的引用。把上述 MyContainer 稍作修改

```javascript
const MyContainer = (MyComponent) => {
    return class extends React.Component {
        constructor(props) {
            super(props);
            this.state = {
                name: '',
                disabled: false,
            };

            this.onNameChange = this.onNameChange.bind(this);
            this.refCallback = this.refCallback.bind(this);
        }
        
        onNameChange(event) {
            this.setState({
                name: event.target.value
            });
        }

        refCallback(node) {
            this.node = node;
            this.node.style.background = "red";
        }

        render() {
            const newProps = {
                name: {
                    value: this.state.name,
                    onChange: this.onNameChange,
                    ref: this.refCallback,
                }
            }
            return (
                <MyComponent { ...newProps } { ...this.props }></MyComponent>
            );
        }
    }
}
```

以上，组件 MyComponent 在渲染是 input 元素的 background 会被渲染为 red。