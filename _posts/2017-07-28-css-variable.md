---
layout: post
title:  CSS 变量
date:   2017-07-28 19:58:00 +0800
categories: CSS
tag: CSS
---

* content
{:toc}

之前css中使用变量要使用Sass/Less等预编译的工具，现在原生的css也开始支持css变量了。

[查看兼容性](https://caniuse.com/#search=CSS%20Variables)

这里先尝试试用下，记录下用法和特性。

[MDN 使用CSS变量](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Using_CSS_variables)

### 基本用法

声明一个变量：

```
element {
  --main-bg-color: brown;
}
```
 
使用变量：

```
element {
  background-color: var(--main-bg-color);
}
```

无论是变量的定义和使用只能在声明块`{}`里面

var()函数还可以用在变量的声明。

```
:root {
  --primary-color: red;
  --logo-text: var(--primary-color);
}
```

`var()`函数还可以使用第二个参数，表示变量的默认值。如果该变量不存在，就会使用这个默认值。

第二个参数不处理内部的逗号或空格，都视作参数的一部分。

如果变量不和方法，例如下面这种

```
body {
  --color: 20px;
  background-color: #369;
  background-color: var(--color, #cd0000);
}
```

背景色会是什么？ ==> `transparent`.

CSS默认值的使用仅限于变量未定义的情况，并不包括变量不合法。

### 变量值类型

如果变量值是一个字符串，可以与其他字符串拼接。

```
:root{
	--bar: 'hello';
	--foo: var(--bar)' world';
}
```

变量值是数值，不能与数值单位直接连用

```
.foo {
  --gap: 20;
  margin-top: var(--gap)px; // 无效 
}
```

上面代码中，数值与单位直接写在一起，这是无效的。必须使用`calc()`函数，将它们连接。

```
.foo {
  --gap: 20;
  margin-top: calc(var(--gap) * 1px);
}
```

### 作用域

类似这样

```
<style>
	:root { --color: purple; }
	div { --color: green; }
	#alert { --color: red; }
	* { color: var(--color); }
</style>

<p>我的紫色继承于根元素</p>
<div>我的绿色来自直接设置</div>
<div id='alert'>
  ID选择器权重更高，因此阿拉是红色！
  <p>我也是红色，占了继承的光</p>
</div>
```

1、变量也是跟着CSS选择器走的，变量的作用域就是它所在的选择器的有效范围。

例如#alert定义的变量，只有id为alert的元素才能享有。如果你想变量全局使用，则你可以设置在:root选择器上；

2、当存在多个同样名称的变量时候，变量的覆盖规则由CSS选择器的权重决定的.

但并无!important这种用法，因为没有必要，!important设计初衷是干掉JS的style设置，但对于变量的定义则没有这样的需求。


### JavaScript 操作

JavaScript 也可以检测浏览器是否支持 CSS 变量。

```
const isSupported =
  window.CSS &&
  window.CSS.supports &&
  window.CSS.supports('--a', 0);

if (isSupported) {
  /* supported */
} else {
  /* not supported */
}
```

JavaScript 操作 CSS 变量的写法如下。

```
// 设置变量
document.body.style.setProperty('--primary', '#7F583F');

// 读取变量
document.body.style.getPropertyValue('--primary').trim();
// '#7F583F'

// 删除变量
document.body.style.removeProperty('--primary');
```

============================= 我是分割线 =============================

参考链接：

[阮一峰 - CSS 变量教程](http://www.ruanyifeng.com/blog/2017/05/css-variables.html)

[了解CSS/CSS3原生变量var](http://www.zhangxinxu.com/wordpress/2016/11/css-css3-variables-var/)