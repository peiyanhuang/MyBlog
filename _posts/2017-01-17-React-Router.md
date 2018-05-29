---
layout: post
title:  学习React Router
date:   2018-02-26 19:29:00 +0800
categories: React
tag: React
---

* content
{:toc}

### React Router V4 更新

React Router V4 相较于前面三个版本有根本性变化，现于此更新下。

首先是遵循 `Just Component` 的 API 设计理念，其次 API 方面也精简了不少。其基于 Lerna 管理多个 Repository。在此代码库包括：

- `react-router`: React Router 核心
- `react-router-dom`: 用于 DOM 绑定的 React Router
- `react-router-native`: 用于 React Native 的 React Router
- `react-router-redux`: React Router 和 Redux 的集成
- `react-router-config`: 静态路由配置帮助助手

#### 1. 插件初引入

注意，入门第一坑就在这里。`react-router` 和 `react-router-dom` 只要引用一个就行了，不同之处就是后者比前者多出了 `<Link>、<BrowserRouter>` 这样的 DOM 类组件。因此我们只需引用 `react-router-dom` 这个包就OK了。当然，如果搭配 `redux`，你还需要使用 `react-router-redux`。

```jsx
import React from "react";
import { BrowserRouter as Router, Route, Link } from "react-router-dom";

const BasicExample = () => (
  <Router>
    <div>
      <ul>
        <li>
          <Link to="/">Home</Link>
        </li>
        <li>
          <Link to="/about">About</Link>
        </li>
        <li>
          <Link to="/topics">Topics</Link>
        </li>
      </ul>

      <hr />

      <Route exact path="/" component={Home} />
      <Route path="/about" component={About} />
      <Route path="/topics" component={Topics} />
    </div>
  </Router>
);
```

#### `<Router>`

在4.0之前版本的 API 中，[`<Router>`](https://reacttraining.com/react-router/web/api/Router) 组件的 `children` 只能是 `React Router` 提供的各种组件，如 `<Route>、<IndexRoute>、<Redirect>` 等。而在 React Router 4 中，你可以将各种组件及标签放进 `<Router>` 组件中，他的角色也更像是 `Redux` 中的 `<Provider>`。不同的是 `<Provider>` 是用来保持与 `store` 的更新，而 `<Router>` 是用来保持与 `location` 的同步。

`<Router>` 是所有路由组件共用的底层接口，一般我们的应用并不会使用这个接口，而是使用高级的路由：

- [`<BrowserRouter>`](https://reacttraining.com/react-router/web/api/BrowserRouter)：使用 HTML5 提供的 history API 来保持 UI 和 URL 的同步；
- [`<HashRouter>`](https://reacttraining.com/react-router/web/api/HashRouter)：使用 URL 的 hash (例如：window.location.hash) 来保持 UI 和 URL 的同步；
- `<MemoryRouter>`：能在内存保存你 “URL” 的历史纪录(并没有对地址栏读写)；
- `<NativeRouter>`：为使用React Native提供路由支持；
- `<StaticRouter>`：从不会改变地址；

注意，和之前的 `Router` 不一样，这里 `<Router>` 组件下只允许存在一个子元素，如存在多个则会报错。

#### `<Route>`

我们知道，[`<Route>`](https://reacttraining.com/react-router/web/api/Route) 组件主要的作用就是当一个 `location` 匹配路由的 `path` 时，渲染某些UI。示例如下：`

```jsx
<Router>
  <div>
    <Route exact path="/" component={Home}/>
    <Route path="/news" component={NewsFeed}/>
  </div>
</Router>

// 如果应用的地址是/,那么相应的UI会类似这个样子：
<div>
  <Home/>
</div>

// 如果应用的地址是/news,那么相应的UI就会成为这个样子：
<div>
  <NewsFeed/>
</div>
```

`<Route>` 组件有如下属性：

- `path`（string）: 路由匹配路径。（没有 path 属性的 Route 总是会匹配）；
- `exact`（bool）：为 `true` 时，则要求路径与 `location.pathname` 必须完全匹配；
- `strict`（bool）：为 `true` 时，有结尾斜线的路径只能匹配有斜线的 `location.pathname`；

同时，新版的路由为 `<Route>` 提供了三种渲染内容的方法：

- `<Route component>`：在地址匹配的时候React的组件才会被渲染，route props 也会随着一起被渲染；
- `<Route render>`：这种方式对于内联渲染和包装组件却不引起意料之外的重新挂载特别方便；
- `<Route children>`：与 render 属性的工作方式基本一样，除了它是不管地址匹配与否都会被调用；

第一种方式没啥可说的，和之前一样，这里我们重点看下 `<Route render>` 的渲染方式：

```jsx
// 行内渲染示例
<Route path="/home" render={() => <div>Home</div>}/>

// 包装/合成
const FadingRoute = ({ component: Component, ...rest }) => (
  <Route {...rest} render={props => (
    <FadeIn>
      <Component {...props}/>
    </FadeIn>
  )}/>
)

<FadingRoute path="/cool" component={Something}/>
```

注意：`<Route component>` 的优先级要比 `<Route render>` 高，所以不要在同一个 `<Route>` 中同时使用这两个属性。

#### `<Link>`

[`Link`](https://reacttraining.com/react-router/web/api/Link)和之前版本没太大区别，重点看下组件属性：

- `to`（string/object）：要跳转的路径或地址；
- `replace`（bool）：为 `true` 时，点击链接后将使用新地址替换掉访问历史记录里面的原地址；为 `false` 时，点击链接后将在原有访问历史记录的基础上添加一个新的纪录。默认为 `false`；

```jsx
<Link to='/courses?sort=name' replace/>

<Link to={{
  pathname: '/courses',
  search: '?sort=name',
  hash: '#the-hash',
  state: { fromDashboard: true }
}}/>
```

#### `<NavLink>`

[`<NavLink>`](https://reacttraining.com/react-router/web/api/NavLink) 是 `<Link>` 的一个特定版本, 会在匹配上当前 URL 的时候会给已经渲染的元素添加样式参数，组件属性：

- `activeClassName`（string）：设置选中样式，默认值为 active；
- `activeStyle`（object）：当元素被选中时, 为此元素添加样式；
- `exact`（bool）：为 true 时, 只有当地址完全匹配 class 和 style 才会应用；
- `strict`（bool）：为 true 时，在确定位置是否与当前 URL 匹配时，将考虑位置 pathname 后的斜线；
- `isActive`（func）：判断链接是否激活的额外逻辑的功能；

#### `<Switch>`

[`<Switch>`](https://reacttraining.com/react-router/web/api/Switch) 组件用来渲染匹配地址的第一个 `<Route>` 或者 `<Redirect>`。那么它与使用一堆 `route` 又有什么区别呢？

`<Switch>` 的独特之处是独它仅仅渲染一个路由。相反地，每一个匹配地址(location)的 `<Route>` 都会被渲染。

```jsx
import { Switch, Route } from 'react-router'

<Switch>
  <Route exact path="/" component={Home}/>
  <Route path="/about" component={About}/>
  <Route path="/:user" component={User}/>
  <Route component={NoMatch}/>
</Switch>
```

### 安装

[官方 React-Router 路由库](https://github.com/ReactTraining/react-router)

React Router 安装命令如下。

  $ npm install --save react-router

使用时，路由器 Router 就是 React 的一个组件。

Router组件本身只是一个容器，真正的路由要通过Route组件定义。

```jsx
import { Router, Route, hashHistory } from 'react-router';

render((
  <Router history={hashHistory}>
    <Route path="/" component={App}/>
  </Router>
), document.getElementById('app'));
```

上面代码中，用户访问根路由 `/`（比如http://www.example.com/），组件APP就会加载到 `document.getElementById('app')`。
你可能还注意到，Router组件有一个参数`history`，它的值`hashHistory`表示，路由的切换由URL的hash变化决定，即URL的#部分发生变化。举例来说，用户访问 http://www.example.com/ ，实际会看到的是 http://www.example.com/#/。

Route组件定义了URL路径与组件的对应关系。你可以同时使用多个Route组件。

```jsx
<Router history={hashHistory}>
  <Route path="/" component={App}/>
  <Route path="/repos" component={Repos}/>
  <Route path="/about" component={About}/>
</Router>
```

上面代码中，用户访问`/repos`（比如http://localhost:8080/#/repos）时，加载Repos组件；访问`/about`（http://localhost:8080/#/about）时，加载About组件。

### Route 的 path 属性

#### path

Route 组件的 `path` 属性指定路由的匹配规则。这个属性是可以省略的，这样的话，不管路径是否匹配，总是会加载指定组件。

请看下面的例子。

```jsx
<Route path="inbox" component={Inbox}>
    <Route path="messages/:id" component={Message} />
</Route>
```

上面代码中，当用户访问/inbox/messages/:id时，会加载下面的组件。

```jsx
<Inbox>
  <Message/>
</Inbox>
```

如果省略外层Route的path参数，写成下面的样子。

```jsx
<Route component={Inbox}>
  <Route path="inbox/messages/:id" component={Message} />
</Route>
```

现在用户访问/inbox/messages/:id时，组件加载还是原来的样子。

```
<Inbox>
  <Message/>
</Inbox>
```

#### path 的通配符

```
（1）:paramName
:paramName匹配URL的一个部分，直到遇到下一个/、?、#为止。这个路径参数可以通过this.props.params.paramName取出。

（2）()
()表示URL的这个部分是可选的。

（3）*
*匹配任意字符，直到模式里面的下一个字符为止。匹配方式是非贪婪模式。

（4） **
** 匹配任意字符，直到下一个/、?、#为止。匹配方式是贪婪模式。
```

`path`属性也可以使用相对路径（不以/开头），匹配时就会相对于父组件的路径。嵌套路由如果想摆脱这个规则，可以使用绝对路由。

路由匹配规则是从上到下执行，一旦发现匹配，就不再其余的规则了。

此外，URL的查询字符串`/foo?bar=baz`，可以用`this.props.location.query.bar`获取。

### Link

Link 组件用于取代`<a>`元素，生成一个链接，允许用户点击后跳转到另一个路由。它基本上就是`<a>`元素的 React 版本，可以接收 Router 的状态。

```jsx
// modules/App.js
import React from 'react'
import { Link } from 'react-router'

export default React.createClass({
  render() {
  return (
    <div>
    <h1>React Router Tutorial</h1>
    <ul role="nav">
      <li><Link to="/about">About</Link></li>
      <li><Link to="/repos">Repos</Link></li>
    </ul>
    </div>
  )
  }
})
```

`to` 属性要使用 `/` 开始的绝对路径，相对路径会加在当前地址后。

如果希望当前的路由与其他路由有不同样式，这时可以使用Link组件的`activeStyle`属性。

```jsx
<Link to="/about" activeStyle={{color: 'red'}}>About</Link>
<Link to="/repos" activeStyle={{color: 'red'}}>Repos</Link>
```

上面代码中，当前页面的链接会红色显示。
另一种做法是，使用`activeClassName`指定当前路由的`Class`。

```jsx
<Link to="/about" activeClassName="active">About</Link>
<Link to="/repos" activeClassName="active">Repos</Link>
```

上面代码中，当前页面的链接的class会包含active类。

在Router组件之外，导航到路由页面，可以使用浏览器的`History API`，像下面这样写。

```jsx
import { browserHistory } from 'react-router';
browserHistory.push('/some/path');
```

如果你不希望在每一处的`Link`都写下`activeClassName`和`activeStyle`，就可以像下面这样，

```jsx
// modules/NavLink.js
import React from 'react'
import { Link } from 'react-router'

export default React.createClass({
  render() {
    return <Link {...this.props} activeClassName="active"/>
  }
})

// modules/App.js
import NavLink from './NavLink'

// ...

<li><NavLink to="/about">About</NavLink></li>
<li><NavLink to="/repos">Repos</NavLink></li>
```

### 嵌套路由

Route 组件还可以嵌套。

```jsx
<Router history={hashHistory}>
  <Route path="/" component={App}/>
  <Route path="/repos" component={Repos}/>
  <Route path="/about" component={About}/>
</Router>
```

上面代码中，用户访问`/repos`时，会先加载 App 组件，然后在它的内部再加载 Repos 组件。

```
<App>
  <Repos/>
</App>
```

App 组件要写成下面的样子。

```jsx
export default React.createClass({
  render() {
    return (
      <div>
        {this.props.children}
      </div>
    )
  }
})
```

上面代码中，App 组件的`this.props.children`属性就是子组件。

子路由也可以不写在 Router 组件里面，单独传入`Router`组件的`routes`属性。

```jsx
var routes = <Route path="/" component={App}>
  <Route path="/repos" component={Repos}/>
  <Route path="/about" component={About}/>
</Route>;

<Router routes={routes} history={browserHistory}/>
```

### IndexRoute 

```
<Router>
  <Route path="/" component={App}>
    <Route path="accounts" component={Accounts}/>
    <Route path="statements" component={Statements}/>
  </Route>
</Router>
```

上面代码中，访问根路径/，不会加载任何子组件。也就是说，App组件的`this.props.children`，这时是 `undefined`。

```jsx
// modules/App.js
import Home from './Home'

// ...
<div>
  {/* ... */}
  {this.props.children || <Home/>}
</div>
//...
```

因此，通常会采用`{this.props.children || <Home/>}`这样的写法。这时，Home 明明是 Accounts 和 Statements 的同级组件，却没有写在 Route 中。

`IndexRoute`就是解决这个问题，显式指定Home是根路由的子组件，即指定默认情况下加载的子组件。你可以把`IndexRoute`想象成某个路径的 index.html。

```jsx
// index.js

import { Router, Route, hashHistory, IndexRoute } from 'react-router'
import Home from './modules/Home'

// ...

render((
  <Router history={hashHistory}>
    <Route path="/" component={App}>

      {/* add it here, as a child of `/` */}
      <IndexRoute component={Home}/>

      <Route path="/repos" component={Repos}>
        <Route path="/repos/:userName/:repoName" component={Repo}/>
      </Route>
      <Route path="/about" component={About}/>
    </Route>
  </Router>
), document.getElementById('app'))
```

现在，用户访问/的时候，加载的组件结构如下。

```
<App>
  <Home/>
</App>
```

这种组件结构就很清晰了：App 只包含下级组件的共有元素，本身的展示内容则由 Home 组件定义。这样有利于代码分离，也有利于使用 React Router 提供的各种 API。

注意，IndexRoute 组件没有路径参数 `path`。

### IndexLink

如果链接到根路由 `/`，不要使用 Link 组件，而要使用 IndexLink 组件。
这是因为对于根路由来说，`activeStyle` 和 `activeClassName` 会失效，或者说总是生效，因为 `/` 会匹配任何子路由。而 IndexLink 组件会使用路径的精确匹配。

```jsx
<IndexLink to="/" activeClassName="active">
  Home
</IndexLink>
```

上面代码中，根路由只会在精确匹配时，才具有 `activeClassName`。另一种方法是使用 Link 组件的`onlyActiveOnIndex` 属性，也能达到同样效果。

```jsx
<Link to="/" activeClassName="active" onlyActiveOnIndex={true}>
  Home
</Link>
```

实际上，IndexLink 就是对 Link 组件的 `onlyActiveOnIndex` 属性的包装。

### histroy 属性 

Router 组件的 `history` 属性，用来监听浏览器地址栏的变化，并将 URL 解析成一个地址对象，供 React Router 匹配。`history` 属性，一共可以设置三种值。

  browserHistory
  hashHistory
  createMemoryHistory

如果设为`hashHistory`，路由将通过 URL 的 `hash` 部分（#）切换，URL 的形式类似 example.com/#/some/path。

```jsx
import { hashHistory } from 'react-router'

render(
  <Router history={hashHistory} routes={routes} />,
  document.getElementById('app')
)
```

如果设为`browserHistory`，浏览器的路由就不再通过 `Hash` 完成了，而显示正常的路径 example.com/some/path，背后调用的是浏览器的 History API。

```jsx
import { browserHistory } from 'react-router'

render(
  <Router history={browserHistory} routes={routes} />,
  document.getElementById('app')
)
```

但是，这种情况需要对服务器改造。否则用户直接向服务器请求某个子路由，会显示网页找不到的404错误。
如果开发服务器使用的是 webpack-dev-server，加上`--history-api-fallback`参数就可以了。

```
"start": "webpack-dev-server --inline --content-base . --history-api-fallback"
```

`createMemoryHistory`主要用于服务器渲染。它创建一个内存中的 `history` 对象，不与浏览器 URL 互动。

```
const history = createMemoryHistory(location)
```

### Redirect 组件

`<Redirect>`组件用于路由的跳转，即用户访问一个路由，会自动跳转到另一个路由。

```jsx
<Route path="inbox" component={Inbox}>
  {/* 从 /inbox/messages/:id 跳转到 /messages/:id */}
  ＜Redirect from="messages/:id" to="/messages/:id" />
</Route>
```

现在访问`/inbox/messages/5`，会自动跳转到`/messages/5`。

### IndexRedirect 组件

`IndexRedirect`组件用于访问根路由的时候，将用户重定向到某个子组件。

```jsx
<Route path="/" component={App}>
    <IndexRedirect to="/welcome" />
    <Route path="welcome" component={Welcome} />
    <Route path="about" component={About} />
</Route>
```

上面代码中，用户访问根路径时，将自动重定向到子组件 welcome。

### 表单处理

`Link`组件用于正常的用户点击跳转，但是有时还需要表单跳转、点击按钮跳转等操作。这些情况怎么跟React Router对接呢？
下面是一个表单。

```jsx
<form onSubmit={this.handleSubmit}>
    <input type="text" placeholder="userName"/>
    <input type="text" placeholder="repo"/>
    <button type="submit">Go</button>
</form>
```

第一种方法是使用browserHistory.push

```jsx
import { browserHistory } from 'react-router'

// ...
handleSubmit(event) {
  event.preventDefault()
  const userName = event.target.elements[0].value
  const repo = event.target.elements[1].value
  const path = `/repos/${userName}/${repo}`
  browserHistory.push(path)
},
```

第二种方法是使用context对象。

```jsx
export default React.createClass({

  // ask for `router` from context
  contextTypes: {
  router: React.PropTypes.object
  },

  handleSubmit(event) {
  // ...
  this.context.router.push(path)
  },
})
```

### 路由的钩子

每个路由都有 `Enter` 和 `Leave` 钩子，用户进入或离开该路由时触发。

```jsx
<Route path="about" component={About} />
  <Route path="inbox" component={Inbox}>
  <Redirect from="messages/:id" to="/messages/:id" />
</Route>
```

上面的代码中，如果用户离开 `/messages/:id`，进入 `/about` 时，会依次触发以下的钩子。

```
/messages/:id 的 onLeave
/inbox 的 onLeave
/about 的 onEnter
```

下面是一个例子，使用 `onEnter` 钩子替代 <Redirect> 组件。

```jsx
<Route path="inbox" component={Inbox}>
  <Route
    path="messages/:id"
    onEnter={
      ({params}, replace) => replace(`/messages/${params.id}`)
    }
  />
</Route>
```