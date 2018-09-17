---
layout: post
title:  理解 Virtual DOM
date:   2018-05-15 19:58:00 +0800
categories: 开发者
tag: 开发者
---

* content
{:toc}

React 好像已经火了很久很久，以致于我们对于 Virtual DOM 这个词都已经很熟悉了，网上也有非常多的介绍 React、Virtual DOM 的文章。但是直到前不久我专门花时间去学习 Virtual DOM，才让我对 Virtual DOM 有了一定的理解。在此分享下。

### 1. Virtual DOM

Virtual DOM 概况来讲，实际上实在游览器端用 JavaScript 实现了一套 DOM Api，就是在数据和真实 DOM 之间建立了一层缓冲。对于开发者而言，数据变化了就调用 React 的渲染方法，而 React 并不是直接得到新的 DOM 进行替换，而是先生成 Virtual DOM，与上一次渲染得到的 Virtual DOM 进行比对，在渲染得到的 Virtual DOM 上发现变化，然后将变化的地方更新到真实 DOM 上。

[virtual-dom](https://github.com/Matt-Esch/virtual-dom)

构建一个简易的 Virtual DOM 模型并不复杂，只要使它具备真实 DOM 标签所具有的基本属性就行，包括：

- tagName(标签名)
- properties(节点属性，包含样式、属性、事件等)
- children(子节点)
- key(标识id)

```js
{
    tagName: 'div',
    properties: {
        style: {}
    },
    children: [],
    key: 1
}
```

Virtual DOM 模型当然更加复杂，不止这些属性。

### 2. 创建 Virtual DOM

Virtual DOM 中的节点称为 ReactElement。React 通过 [createElement](https://github.com/facebook/react/blob/master/packages/react/src/ReactElement.js#L171) 方法创建虚拟元素：

Babel 将 JSX 编译成 `React.createElement()` 调用。下面的两个例子是是完全相同的：

```js
const element = (
    <h1 className="greeting">
        Hello, world!
    </h1>
);

const element = React.createElement(
    'h1',
    {className: 'greeting'},
    'Hello, world!'
);
```

`createElement` 会返回一个 [ReactElement](https://github.com/facebook/react/blob/master/packages/react/src/ReactElement.js#L111) 实例对象

```js
export function createElement(type, config, children) {
    let propName;

    // Reserved names are extracted
    const props = {};

    let key = null;
    let ref = null;
    let self = null;
    let source = null;

    // 如果 config 存在，则提取里面的内容
    if (config != null) {
        if (hasValidRef(config)) {
            ref = config.ref;
        }
        if (hasValidKey(config)) {
            key = '' + config.key;
        }

        self = config.__self === undefined ? null : config.__self;
        source = config.__source === undefined ? null : config.__source;
        // 复制 config 中的内容到 props
        for (propName in config) {
            if (
                hasOwnProperty.call(config, propName) &&
                !RESERVED_PROPS.hasOwnProperty(propName)
            ) {
                props[propName] = config[propName];
            }
        }
    }

    // 处理 children，挂到 props.children 属性上
    // 如果只有一个，则直接赋值；若有多个，则合并为 childArray 再赋值
    const childrenLength = arguments.length - 2;
    if (childrenLength === 1) {
        props.children = children;
    } else if (childrenLength > 1) {
        const childArray = Array(childrenLength);
        for (let i = 0; i < childrenLength; i++) {
            childArray[i] = arguments[i + 2];
        }
        if (__DEV__) {
            if (Object.freeze) {
                Object.freeze(childArray);
            }
        }
        props.children = childArray;
    }

    // 若某个 prop 为空但存在默认的 defaultProp，则将默认值赋值给当前的 prop
    if (type && type.defaultProps) {
        const defaultProps = type.defaultProps;
        for (propName in defaultProps) {
            if (props[propName] === undefined) {
                props[propName] = defaultProps[propName];
            }
        }
    }
    if (__DEV__) {
        if (key || ref) {
            if (
                typeof props.$$typeof === 'undefined' ||
                props.$$typeof !== REACT_ELEMENT_TYPE
            ) {
                const displayName =
                    typeof type === 'function' ?
                    type.displayName || type.name || 'Unknown' :
                    type;
                if (key) {
                    defineKeyPropWarningGetter(props, displayName);
                }
                if (ref) {
                    defineRefPropWarningGetter(props, displayName);
                }
            }
        }
    }
    // 返回一个 ReactElement 对象
    return ReactElement(
        type,
        key,
        ref,
        self,
        source,
        ReactCurrentOwner.current,
        props,
    );
}
```

看了下源码，再打印下 element 看下返回的 ReactElement 实例对象具体是什么：

![ReactElement 实例对象]({{ '/images/react/ReactElement.png' | prepend: site.baseurl }})

接下来看下 [React.Component](https://github.com/facebook/react/blob/master/packages/react/src/ReactBaseClasses.js#L17)

```js
function Component(props, context, updater) {
    this.props = props;
    this.context = context;
    this.refs = emptyObject;

    this.updater = updater || ReactNoopUpdateQueue;
}

Component.prototype.isReactComponent = {};

Component.prototype.setState = function (partialState, callback) {
    invariant(
        typeof partialState === "object" ||
        typeof partialState === "function" ||
        partialState == null,
        "setState(...): takes an object of state variables to update or a " +
        "function which returns an object of state variables."
    );
    this.updater.enqueueSetState(this, partialState, callback, "setState");
};

Component.prototype.forceUpdate = function(callback) {
    this.updater.enqueueForceUpdate(this, callback, 'forceUpdate');
};
```

### 3. 初始化组件

我们用 babel 转换一下代码：

```js
class A extends React.Component {
    constructor(props) {
        super(props)
    }

    render() {
        <div>
            <p>A</p>
        </div>
    }
}

<!-- 转化为 -->
var A = _wrapComponent("A")(
    (function (_React$Component) {
        _inherits(A, _React$Component);

        function A(props) {
            _classCallCheck(this, A);

            return _possibleConstructorReturn(
                this,
                (A.__proto__ || Object.getPrototypeOf(A)).call(this, props)
            );
        }

        _createClass(A, [{
            key: "render",
            value: function render() {
                _react3.default.createElement(
                    "div",
                    null,
                    _react3.default.createElement("p", null, "A")
                );
            }
        }]);

        return A;
    })(_react3.default.Component)
);
```

我们发现 render 方法实际上是调用了 `React.createElement` 方法。这前面已经介绍过了。

### 4. 组件的挂载

我们知道可以通过 `ReactDOM.render(component,mountNode)` 的形式对组件挂载，那么挂载的过程又是如何实现的呢？

[ReactDom](https://github.com/facebook/react/blob/master/packages/react-dom/src/client/ReactDOM.js#L581)在此处定义。

```js
const ReactDOM: Object = {
    // ...
    hydrate(element: React$Node, container: DOMContainer, callback: ? Function) {
        return legacyRenderSubtreeIntoContainer(
            null,
            element,
            container,
            true,
            callback
        );
    },

    render(
        element: React$Element < any > ,
        container: DOMContainer,
        callback: ? Function
    ) {
        return legacyRenderSubtreeIntoContainer(
            null,
            element,
            container,
            false,
            callback
        );
    },

    unstable_renderSubtreeIntoContainer(
        parentComponent: React$Component < any, any > ,
        element: React$Element < any > ,
        containerNode: DOMContainer,
        callback: ? Function
    ) {
        invariant(
            parentComponent != null && ReactInstanceMap.has(parentComponent),
            "parentComponent must be a valid React Component"
        );
        return legacyRenderSubtreeIntoContainer(
            parentComponent,
            element,
            containerNode,
            false,
            callback
        );
    },
    // ...
}
```

我们可以看到，`render` 方法其实调用的是 `legacyRenderSubtreeIntoContainer` 方法。

```js
function legacyRenderSubtreeIntoContainer(
    parentComponent: ? React$Component < any, any >, // 当前组件的父组件，第一次渲染时为null
    children : ReactNodeList, // 要插入DOM中的组件
    container: DOMContainer, // 要插入的容器节点
    forceHydrate: boolean,
    callback: ? Function  // 完成后的回调函数
) {
    invariant(
        isValidContainer(container),
        "Target container is not a DOM element."
    );

    if (__DEV__) {
        topLevelUpdateWarnings(container);
    }

    let root: Root = (container._reactRootContainer: any);
    if (!root) {
        // 如果 root 不存在，则通过 legacyCreateRootFromDOMContainer 初始化一个 ReactRoot 实例
        // 其是一个 Root.fiber，是一个特殊的 fiber 对象
        root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
            container,
            forceHydrate
        );
        if (typeof callback === "function") {
            const originalCallback = callback;
            callback = function () {
                const instance = DOMRenderer.getPublicRootInstance(root._internalRoot);
                originalCallback.call(instance);
            };
        }
        // Initial mount should not be batched.
        DOMRenderer.unbatchedUpdates(() => {
            // Update ReactRoot
            if (parentComponent != null) {
                root.legacy_renderSubtreeIntoContainer(
                    parentComponent,
                    children,
                    callback
                );
            } else {
                root.render(children, callback);
            }
        });
    } else {
        if (typeof callback === "function") {
            const originalCallback = callback;
            callback = function () {
                const instance = DOMRenderer.getPublicRootInstance(root._internalRoot);
                originalCallback.call(instance);
            };
        }
        // Update
        if (parentComponent != null) {
            root.legacy_renderSubtreeIntoContainer(
                parentComponent,
                children,
                callback
            );
        } else {
            root.render(children, callback);
        }
    }
    return DOMRenderer.getPublicRootInstance(root._internalRoot);
}
```

1. `legacyCreateRootFromDOMContainer` 方法：

```js
function legacyCreateRootFromDOMContainer(
  container: DOMContainer,
  forceHydrate: boolean,
): Root {
  // ...

  // Legacy roots are not async by default.
  const isAsync = false;
  return new ReactRoot(container, isAsync, shouldHydrate);
}
```

2. `legacy_renderSubtreeIntoContainer` 方法：

```ts
ReactRoot.prototype.legacy_renderSubtreeIntoContainer = function(
  parentComponent: ?React$Component<any, any>,
  children: ReactNodeList,
  callback: ?() => mixed,
): Work {
  const root = this._internalRoot;
  const work = new ReactWork();
  callback = callback === undefined ? null : callback;
  if (__DEV__) {
    warnOnInvalidCallback(callback, 'render');
  }
  if (callback !== null) {
    work.then(callback);
  }
  DOMRenderer.updateContainer(children, root, parentComponent, work._onCommit);
  return work;
};
```

这里的 Async 模式是写死的，不能进行，如果想试试，直接手动修改为 True。

3. `ReactRoot.render` 方法:

```ts
ReactRoot.prototype.render = function(
  children: ReactNodeList,
  callback: ?() => mixed,
): Work {
  const root = this._internalRoot;
  const work = new ReactWork();
  callback = callback === undefined ? null : callback;
  if (__DEV__) {
    warnOnInvalidCallback(callback, 'render');
  }
  if (callback !== null) {
    work.then(callback);
  }
  DOMRenderer.updateContainer(children, root, null, work._onCommit);
  return work;
};
```

4. `getPublicRootInstance`

```js
getPublicRootInstance(
    container: OpaqueRoot,
): React$Component<any, any> | PI | null {
    const containerFiber = container.current;
    if (!containerFiber.child) {
    return null;
    }
    switch (containerFiber.child.tag) {
    case HostComponent:
        return getPublicInstance(containerFiber.child.stateNode);
    default:
        return containerFiber.child.stateNode;
    }
},
```

- [DOMRenderer](https://github.com/facebook/react/blob/master/packages/react-dom/src/client/ReactDOM.js#L450)
- [ReactRoot](https://github.com/facebook/react/blob/master/packages/react-dom/src/client/ReactDOM.js#L332)
- [ReactWork](https://github.com/facebook/react/blob/master/packages/react-dom/src/client/ReactDOM.js#L279)
- [updateContainer](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberReconciler.js#L429)

### 5. 渲染更新

我们知道浏览器渲染引擎是单线程的，在 React 15.x 版本及之前版本，计算组件树变更时将会阻塞整个线程，整个渲染过程是连续不中断完成的，而这时的其他任务都会被阻塞，如动画等，这可能会使用户感觉到明显卡顿。

这个版本的调和器可以称为栈调和器（Stack Reconciler），其调和算法大致过程见[React Diff](http://blog.codingplayboy.com/2016/10/27/react_diff/) 算法和 [React Stack Reconciler](https://reactjs.org/docs/implementation-notes.html) 实现。

React 16.x 版本提出了一个更先进的调和器，它允许渲染进程分段完成，而不必须一次性完成，中间可以返回至主进程控制执行其他任务。而这是通过计算部分组件树的变更，并暂停渲染更新，询问主进程是否有更高需求的绘制或者更新任务需要执行，这些高需求的任务完成后才开始渲染。这一切的实现是在代码层引入了一个新的数据结构- Fiber 对象，每一个组件实例对应有一个 fiber 实例，此 fiber 实例负责管理组件实例的更新，渲染任务及与其他 fiber 实例的联系。

这个新推出的调和器就叫做纤维调和器（Fiber Reconciler），它提供的新功能主要有：

1. 可切分，可中断任务；
2. 可重用各分阶段任务，且可以设置优先级；
3. 可以在父子组件任务间前进后退切换任务；
4. render 方法可以返回多元素（即可以返回数组）；
5. 支持异常边界处理异常；

我们看下 [Fiber](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiber.js#L71) 定义了哪些东西：

```js
function FiberNode(
  tag: TypeOfWork,  // 识别 fiber 类型的标志
  pendingProps: mixed, // Input is the data coming into process this fiber. Arguments. Props.
  key: null | string, // 唯一标识符
  mode: TypeOfMode, // 其以及其 subtree 的模式属性
) {
  // Instance
  this.tag = tag;
  this.key = key;
  this.type = null;
  this.stateNode = null;

  // Fiber
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;

  this.ref = null;

  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;

  this.mode = mode;

  // Effects
  this.effectTag = NoEffect;
  this.nextEffect = null;

  this.firstEffect = null;
  this.lastEffect = null;

  this.expirationTime = NoWork;

  this.alternate = null;

  if (enableProfilerTimer) {
    this.selfBaseTime = 0;
    this.treeBaseTime = 0;
  }

  if (__DEV__) {
    this._debugID = debugCounter++;
    this._debugSource = null;
    this._debugOwner = null;
    this._debugIsCurrentlyTiming = false;
    if (!hasBadMapPolyfill && typeof Object.preventExtensions === 'function') {
      Object.preventExtensions(this);
    }
  }
}

const createFiber = function(
  tag: TypeOfWork,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
): Fiber {
  return new FiberNode(tag, pendingProps, key, mode);
};
```

Fiber 可以异步实现不同优先级任务的协调执行，目前新版本主流浏览器已经提供了可用API：`requestIdleCallback` 和`requestAnimationFrame`:

- [requestIdleCallback](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback)
- [requestAnimationFrame](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame)

继续看下去：

- [scheduleWork](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberScheduler.js#L1301)
- [enqueueUpdate](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactUpdateQueue.js#L232)

render 流程如下：

- [ReactRoot](https://github.com/facebook/react/blob/master/packages/react-dom/src/client/ReactDOM.js#L335) 其实也就是制造一个 ReactRoot Fiber 节点，这是一个特殊的节点，目前研究还没搞明白为什么要这么做;
- updateContainer/updateContainerAtExpirationTime
- [scheduleRootUpdate](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberReconciler.js#L99) 这个函数就开始比较重要，当我们调用 render 和 setState 以后，我们都会让 React 进入 「更新」的流程，无论你是首次渲染;
- enqueueUpdate：这个函数做了一件事情，setState 和 render 的时候，都会创建一个 update，这个方法就是把这个 update 放入队列中，这个队列是一个链表;
- [scheduleWork]()https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberScheduler.js#L1301 是执行虚拟DOM（fiber树）的更新;
- [requestWork(root, nextExpirationTimeToWorkOn)](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberScheduler.js#L1477) 请求任务。每当 root 接收到更新时，调度程序就会调用 requestWork。渲染器在将来的某个时刻调用renderRoot;
- [performWorkOnRoot](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberScheduler.js#L1731)
- [renderRoot](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberScheduler.js#L980)
- [workLoop](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberScheduler.js#L960) 异步与否在这里走分支;

```js
function workLoop(isAsync) {
  if (!isAsync) {
    // Flush all expired work.
    while (nextUnitOfWork !== null) {
      nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    }
  } else {
    // Flush asynchronous work until the deadline runs out of time.
    while (nextUnitOfWork !== null && !shouldYield()) {
      nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    }

    if (enableProfilerTimer) {
      // If we didn't finish, pause the "actual" render timer.
      // We'll restart it when we resume work.
      pauseActualRenderTimerIfRunning();
    }
  }
}
```

- [performUnitOfWork](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberScheduler.js#L899) 这个函数会循环的返回树中的下一个节点，其中的逻辑比较复杂，其做的就是拿到当前的节点，初始化，构建 Fiber 信息（sibling return 等），然后返回节点的子元素出去，子元素作为 next 继续进来找子元素，层层递进，遍历，直到树走完为止;
- [beginWork](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberBeginWork.js#L1130) 这个函数其实是一个 switch 函数，根据节点 Type 选择不同的策略进行 Mount/update 操作;

里面的 [workInProgress.tag](https://github.com/facebook/react/blob/master/packages/shared/ReactTypeOfWork.js) 标识。

- [reconcileChildren(current, workInProgress, nextChildren)](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberBeginWork.js#L113) 在这里就是处理 React child 的核心逻辑了
- mountChildFibers/reconcileChildFibers
- [reconcileChildFibers](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactChildFiber.js#L1199)

1. 统一首次渲染和更新阶段：将首次渲染同样当成更新来处理，因为其实从没有节点，到有节点实际上就是一个更新过程。
2. 构建流程从递归改为大循环，每一次循环只对本层的节点进行操作，将孩子返回给顶层，让顶层来做循环，在 Async 模式下可以中断等操作
3. 构建的同时，生成 Fiber 信息，这些 Fiber 信息实际上就是为了之后在更新时，能够在任意的地方使得树遍历都可以用大循环来搞

最后整理一张导图供参考：
![流程图]({{ '/images/react/React.png' | prepend: site.baseurl }})

### 参考资料

- [React Fiber Architecture](https://github.com/Foveluy/react-fiber-architecture)翻译[地址](https://github.com/xxn520/react-fiber-architecture-cn)
- [React Fiber初探](https://zhuanlan.zhihu.com/p/31634312)
- [完全理解React Fiber](http://www.ayqy.net/blog/dive-into-react-fiber/)
- [React 16 架构研究记录](https://zhuanlan.zhihu.com/p/36926155)