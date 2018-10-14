---
layout: post
title:  CSS Modules 用法
date:   2018-02-19 19:58:00 +0800
categories: 前端工具
tag: CSS
---

* content
{:toc}

### 1.CSS Modules 介绍

要弄懂CSS Modules是什么，可以先看官方介绍：[GitHub](https://github.com/css-modules/css-modules)

CSS Modules既不是官方标准，也不是浏览器的特性，而是在构建步骤（例如使用Webpack或Browserify）中对CSS类名选择器限定作用域的一种方式（通过hash实现类似于命名空间的方法）。使用JS编译原生的CSS文件，使其具备模块化的能力。

### 2.CSS 模块化方案比较

1.CSS 命名约定

规范化CSS的模块化解决方案（比如BEM BEM — Block Element Modifier ,OOCSS,AMCSS,SMACSS,SUITCSS)
但存在以下问题：

-JS CSS之间依然没有打通变量和选择器等

-复杂的命名

2.CSS in JS

彻底抛弃 CSS，用 JavaScript 写 CSS 规则，并内联样式。styled-components 就是其中代表。styled-components可以让CSS真正意义地写到JS里面，同时让标签更具有语意化，这跟HTML5新标签思想相同；该框架让样式也具备组件化思想，让前端完全面向组件化编程，就像java的包装类型。但存在以下问题：

-样式代码也会出现大量重复。

-不能利用成熟的 CSS 预处理器（或后处理器）

3.使用 JS 来管理样式模块(CSS Modules)

CSS Module 还是 JS 和 CSS 分离的写法，不会改变大家的书写习惯，CSS Module 只需修改构建代码和使用模块依赖引入 className 的方式即可使用，且支持 less 和 sass 的语法，

使用 CSS Modules 可以让组件 className 控制权转交给 JS，我们不会去关心命名冲突污染等问题，同时可以灵活控制生成的命名，样式代码不用修改即可让使用 CSS 语法的旧项目零成本接入。

CSS Modules 能最大化地结合现有 CSS 生态(预处理器/后处理器等)和 JS 模块化能力，几乎零学习成本。只要你使用 Webpack，可以在任何项目中使用。是目前最好的 CSS 模块化解决方案。

### 3.配置

CSS Modules 配置非常简单，如果使用 webpack，只需要如下配置 css-loader

```javascript
{
 test: /\.css$/,
 use: [
   {
     loader: 'css-loader',
     options: {
       modules: true,
       localIdentName: '[path][name]__[local]--[hash:base64:5]'
     }
   }
 ]
}
```

加上 `modules` 即为启用，`localIdentName` 是设置生成样式的命名规则。

### 4.使用

下面是一个 React 组件 App.js。

```jsx
import React from 'react';
import style from './App.css';

export default () => {
  return (
    <h1 className={style.title}>
      Hello World
    </h1>
  );
};
```

上面代码中，我们将样式文件 App.css 输入到 style 对象，然后引用 style.title 代表一个 class。

```css
.title {
  color: red;
}
```

构建工具会将类名 style.title 编译成一个哈希字符串。

```html
<h1 class="_3zyde4l1yATCOkgn-DBWEL">
  Hello World
</h1>
```

App.css 也会同时被编译。

```css
._3zyde4l1yATCOkgn-DBWEL {
  color: red;
}
```

### 5.全局作用域

CSS Modules 允许使用 `:global(.className)` 的语法，声明一个全局规则。凡是这样声明的 `class`，都不会被编译成哈希字符串。

App.css加入一个全局 class。

```css
.title {
  color: red;
}

:global(.title) {
  color: green;
}
```

App.js 使用普通的 class 的写法，就会引用全局 class。

```jsx
import React from 'react';
import styles from './App.css';

export default () => {
  return (
    <h1 className="title">
      Hello World
    </h1>
  );
};
```

CSS Modules 还提供一种显式的局部作用域语法 `:local(.className)`，等同于 `.className`，所以上面的 App.css 也可以写成下面这样。

```css
:local(.title) {
  color: red;
}

:global(.title) {
  color: green;
}
```

### 6. Class 的组合

在 CSS Modules 中，一个选择器可以继承另一个选择器的规则，这称为"组合"（"composition"）。

在 App.css 中，让 `.title` 继承 `.className`。

```css
.className {
  background-color: blue;
}

.title {
  composes: className;
  color: red;
}
```

App.js 不用修改。

```jsx
import React from 'react';
import style from './App.css';

export default () => {
  return (
    <h1 className={style.title}>
      Hello World
    </h1>
  );
};
```

选择器也可以继承其他 CSS 文件里面的规则。

```css
.title {
    composes: className from "./style.css";
}
```

继承全局的 `global` 规则：

```css
.otherClassName {
  composes: globalClassName from global;
}
```