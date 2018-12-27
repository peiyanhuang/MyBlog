---
layout: post
title:  Angular6 基础--模板
date:   2018-09-05 19:58:00 +0800
categories: JS
tag: Angular
---

* content
{:toc}

### 1. 模板表达式和模板语句

#### 1.1 模板表达式

表达式(`{ { } }`)可以把计算后的字符串插入到 HTML 元素标签内的文本或对标签的属性进行赋值。

在属性绑定中也会看到模板表达式，它出现在 = 右侧的引号中，像这样：`[property]="expression"`。

```html
<h3>
  { { title } }
  <img src="{ { imageUrl } }" style="height:30px">
  <img [src]="heroImageUrl">
  <p>The sum of 1 + 1 is {{1 + 1}}</p>
  <p>The sum of 1 + 1 is {{ getValue() }}</p>
</h3>
```

可以调用宿主组件的方法，就像上面用的 `getVal()`。

JavaScript 中那些具有或可能引发副作用的表达式是被禁止的，包括：

- 赋值 (`=, +=, -=, ...`)
- `new` 运算符
- 使用 `;` 或 `,` 的链式表达式
- 自增和自减运算符：`++` 和 `--`

和 JavaScript 语法的其它显著不同包括：

- 不支持位运算 `|` 和 `&`
- 具有新的模板表达式运算符，比如 `|`(管道操作符)、`?.`(安全导航操作符) 和 `!`(非空断言操作符)。

模板表达式应遵循下列原则：

1. 没有可见的副作用
2. 执行迅速
3. 非常简单
4. 幂等性

#### 1.2 模板语句

模板语句用来响应由绑定目标（如 HTML 元素、组件或指令）触发的事件。

```html
<button (click)="deleteHero()">Delete hero</button>
```

和模板表达式一样，模板语句使用的语言也像 JavaScript。 模板语句解析器和模板表达式解析器有所不同，特别之处在于它支持基本赋值 (=) 和表达式链 (; 和 ,)。然而，某些 JavaScript 语法仍然是不允许的：

- `new` 运算符
- 自增和自减运算符：`++` 和 `--`
- 操作并赋值，例如 `+=` 和 `-=`
- 位操作符 `|` 和 `&`
- 模板表达式运算符

### 2. 属性的绑定

HTML attribute 与 DOM property 的对比

要想理解 Angular 绑定如何工作，重点是搞清 HTML attribute 和 DOM property 之间的区别。

`attribute` 是由 HTML 定义的。`property` 是由 DOM (Document Object Model) 定义的。

- 少量 HTML attribute 和 property 之间有着 1:1 的映射，如 id。
- 有些 HTML attribute 没有对应的 property，如 colspan。
- 有些 DOM property 没有对应的 attribute，如 textContent。
- 大量 HTML attribute 看起来映射到了 property…… 但却不像你想的那样！

最后一类尤其让人困惑…… 除非你能理解这个普遍原则：

`attribute` 初始化 `DOM property`，然后它们的任务就完成了。`property` 的值可以改变；`attribute` 的值不能改变。

例如，当浏览器渲染 `<input type="text" value="Bob">` 时，它将创建相应 DOM 节点， 它的 `value` 这个 `property` 被初始化为 “Bob”。

当用户在输入框中输入 “Sally” 时，DOM 元素的 `value` 这个 `property` 变成了 “Sally”。 但是该 HTML 的 `value` 这个 `attribute` 保持不变。如果你读取 `input` 元素的 `attribute`，就会发现确实没变： `input.getAttribute('value') // 返回 "Bob"`。

HTML 的 `value` 这个 `attribute` 指定了初始值；DOM 的 `value` 这个 `property` 是当前值。

`disabled` 这个 `attribute` 是另一种特例。按钮的 `disabled` 这个 `property` 是 `false`，因为默认情况下按钮是可用的。 当你添加 `disabled` 这个 `attribute` 时，只要它出现了按钮的 `disabled` 这个 `property` 就初始化为 `true`，于是按钮就被禁用了。

添加或删除 `disabled` 这个 `attribute` 会禁用或启用这个按钮。但 `attribute` 的值无关紧要，这就是你为什么没法通过 `<button disabled="false">` 仍被禁用 `</button>` 这种写法来启用按钮。

设置按钮的 `disabled` 这个 `property`（如，通过 Angular 绑定）可以禁用或启用这个按钮。 这就是 `property` 的价值。
就算名字相同，HTML attribute 和 DOM property 也不是同一样东西。

模板绑定是通过 `property` 和事件来工作的，而不是 `attribute`。在 Angular 的中，`attribute` 唯一的作用是用来初始化元素和指令的状态。 当进行数据绑定时，只是在与元素和指令的 `property` 和事件打交道，而 `attribute` 就完全靠边站了。

`property` 属性的绑定使用 `[...]` 来标识，也可以用 `bind-` 前缀的可选形式，并称之为规范形式。例如：

```html
<img [src]="imageUrl">
<img bind-src="imageUrl">
```

`attribute` 绑定的语法与属性绑定类似。 但方括号中的部分不是元素的属性名，而是由 `attr.*` 组成。例如：

```html
<table border=1>
  <tr>
    <td [attr.colspan]="1 + 1">One-Two</td>
  </tr>
  <tr>
    <td>Five</td>
    <td>Six</td>
  </tr>
</table>
```

#### 2.1 CSS类绑定

CSS 类绑定绑定的语法与属性绑定类似。但方括号中的部分不是元素的属性名，而是由class前缀，一个点 (.)和 CSS 类的名字组成，其中后两部分是可选的。形如：`[class.class-name]`。

```html
<!-- 当 badCurly 有值时 class 这个 attribute 设置的内容会被完全覆盖 -->
<div class="bad curly special" [class]="badCurly">Bad curly</div>
<!-- 当模板表达式的求值结果是真值时，Angular 会添加这个类，反之则移除它。 -->
<div [class.special]="isSpecial">The class binding is special</div>
```

用 `NgClass` 指令来同时管理多个类名更方便。

#### 2.2 样式绑定

样式绑定的语法与属性绑定类似。 但方括号中的部分不是元素的属性名，而由 style 前缀，一个点 (.) 和 CSS 样式的属性名组成。 形如：`[style.style-property]`。

```html
<button [style.color]="isSpecial ? 'red': 'green'">Red</button>
<button [style.background-color]="canSave ? 'cyan': 'grey'" >Save</button>
<!-- 根据条件用 “em” 和 “%” 来设置字体大小的单位 -->
<button [style.font-size.em]="isSpecial ? 3 : 1" >Big</button>
<button [style.font-size.%]="!isSpecial ? 150 : 50" >Small</button>
```

用 `NgStyle` 指令 来同时设置多个内联样式更方便。

### 3. 事件绑定

绑定事件可以使用 `()`，也可以使用带 on- 前缀的备选形式。

```html
<button (click)="onSave()">Save</button>
<button on-click="onSave()">On Save</button>
<!-- $event -->
<input [value]="currentHero.name" (input)="currentHero.name=$event.target.value" >
```

绑定会通过名叫 `$event` 的事件对象传递关于此事件的信息（包括数据值）。

#### 3.1 使用 EventEmitter 实现自定义事件

```html
<!-- 父组件绑定了 HeroDetailComponent 的 deleteRequest 事件 -->
<app-hero-detail (deleteRequest)="deleteHero($event)" [hero]="currentHero"></app-hero-detail>
```

```html
<!-- HeroDetailComponent html -->
<div>
  <img src="{{heroImageUrl}}">
  <span [style.text-decoration]="lineThrough">
    {{prefix}} {{hero?.name}}
  </span>
  <button (click)="delete()">Delete</button>
</div>
<!-- HeroDetailComponent ts -->
deleteRequest = new EventEmitter<Hero>();

delete() {
  this.deleteRequest.emit(this.hero);
}
```

组件定义了 `deleteRequest`，它是 `EventEmitter` 实例。当用户点击删除时，组件会调用 `delete()` 方法，让 `EventEmitter` 发出一个 `Hero` 对象。触发父组件绑定的 `deleteRequest` 事件，Angular 调用父组件的 `deleteHero` 方法， 在 `$event` 变量中传入要删除的英雄。

### 4. 双向数据绑定

Angular 提供一种特殊的[双向数据绑定](https://angular.cn/guide/template-syntax#two-way-binding--span-classsyntaxspan-)语法：`[(x)]`，此语法结合了属性绑定的方括号 `[x]` 和事件绑定的圆括号 `(x)`。双向绑定语法实际上是属性绑定和事件绑定的语法糖。

Angular 也提供了 `NgModel` 允许在表单元素上使用双向数据绑定。

### 5. 模板引用变量 (#var)

模板引用变量通常用来引用模板中的某个 DOM 元素，它还可以引用 Angular 组件或指令或 Web Component。

使用井号 `#` 来声明引用变量。 `#phone` 的意思就是声明一个名叫 `phone` 的变量来引用 `<input>` 元素。你可以在模板中的任何地方引用模板引用变量。

也可以用 `ref-` 前缀代替 `#`。

```jsx
<input #phone placeholder="phone number">

<button (click)="callPhone(phone.value)">Call</button>
```

如果应用导入过 `FormsModule`，那么对于 `<form>` 的模板引用变量会有不同。

```html
<form (ngSubmit)="onSubmit(heroForm)" #heroForm="ngForm">
  <div class="form-group">
    <label for="name">Name
      <input class="form-control" name="name" required [(ngModel)]="hero.name">
    </label>
  </div>
  <button type="submit" [disabled]="!heroForm.form.valid">Submit</button>
</form>
<div [hidden]="!heroForm.form.valid">
  {{submitMessage}}
</div>
```

这里的 `heroForm` 实际上是一个 Angular `NgForm` 指令的引用，因此具备了跟踪表单中的每个控件的值和有效性的能力。

原生的 `<form>` 元素没有 `form` 属性，但 `NgForm` 指令有。这就解释了为何当 `heroForm.form.valid` 是无效时你可以禁用提交按钮，并能把整个表单控件树传给父组件的 `onSubmit` 方法。

**模板引用变量**的作用范围是整个模板。

### 6. 输入和输出属性

输入属性是一个带有 `@Input` 装饰器的可设置属性。当它通过**属性绑定**的形式被绑定时，值会“流入”这个属性。

输出属性是一个带有 `@Output` 装饰器的可观察对象型的属性。 这个属性几乎总是返回 Angular 的 `EventEmitter`。 当它通过事件绑定的形式被绑定时，值会“流出”这个属性。