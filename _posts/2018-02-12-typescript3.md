---
layout: post
title:  typeScript 函数、泛型
date:   2018-02-12 19:58:00 +0800
categories: TypeScript
tag: TypeScript
---

* content
{:toc}

## 函数

### 1.函数类型

我们可以给每个参数添加类型之后再为函数本身添加返回值类型。 TypeScript能够根据返回语句自动推断出返回值类型，因此我们通常省略它。

```typescript
function add(x: number, y: number): number {
    return x + y;
}

let myAdd = function(x: number, y: number): number { return x + y; };
```

完整函数类型:

```typescript
let myAdd: (baseValue: number, increment: number) => number =
    function(x: number, y: number): number {
        return x + y;
    };
```

只要参数类型是匹配的，那么就认为它是有效的函数类型，而不在乎参数名是否正确。

第二部分是返回值类型。 对于返回值，我们在函数和返回值类型之前使用 `=>` 符号，使之清晰明了。 如之前提到的，返回值类型是函数类型的必要部分，如果函数没有返回任何值，你也必须指定返回值类型为 `void` 而不能留空。

在参数名旁使用 `?` 表示可选参数。可选参数必须跟在必须参数后面。

在TypeScript里，我们也可以为参数提供一个 默认值 当用户没有传递这个参数或传递的值是 undefined 时。 它们叫做有默认初始化值的参数。

### 2. 重载

JavaScript本身是个动态语言。 JavaScript里函数根据传入不同的参数而返回不同类型的数据。这时，可以为同一个函数提供多个函数类型定义来进行函数重载。 编译器会根据这个列表去处理函数的调用。

```typescript
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];

let pickedCard2 = pickCard(15);
```

它查找重载列表，尝试使用第一个重载定义。 如果匹配的话就使用这个。因此，在定义重载的时候，一定要把最精确的定义放在最前面。

注意，`function pickCard(x): any` 并不是重载列表的一部分，因此这里只有两个重载：一个是接收对象另一个接收数字。

## 泛型

### 1.

我们需要一种方法使返回值的类型与传入参数的类型是相同的。 这里，我们使用了 *类型变量*，它是一种特殊的变量，只用于表示类型而不是值。

```typescript
function identity<T>(arg: T): T {
    return arg;
}
```

我们给identity添加了类型变量 `T`。 T帮助我们捕获用户传入的类型（比如：number），之后我们就可以使用这个类型。 之后我们再次使用了 `T` 当做返回值类型。现在我们可以知道参数类型与返回值类型是相同的了。

我们定义了泛型函数后，可以用两种方法使用。 第一种是，传入所有的参数，包含类型参数：

    let output = identity<string>("myString");

这里我们明确的指定了 `T` 是 `string` 类型，并做为一个参数传给函数，使用了 `<>` 括起来而不是 `()`。

第二种方法更普遍。利用了类型推论 -- 即编译器会根据传入的参数自动地帮助我们确定T的类型：

    let output = identity("myString");

