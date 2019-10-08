---
layout: post
title:  typeScript 函数、泛型和枚举
date:   2018-02-12 19:58:00 +0800
categories: TypeScript
tag: TypeScript
---

* content
{:toc}

## 函数

和 JavaScript 一样，TypeScript 函数可以创建有名字的函数和匿名函数。

### 1.1 函数类型

我们可以给每个参数添加类型之后再为函数本身添加返回值类型。TypeScript 能够根据返回语句自动推断出返回值类型，因此我们通常省略它。

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

第二部分是返回值类型。对于返回值，我们在函数和返回值类型之前使用 `=>` 符号，使之清晰明了。如之前提到的，返回值类型是函数类型的必要部分，如果函数没有返回任何值，你也必须指定返回值类型为 `void` 而不能留空。

* 可选参数

在参数名旁使用 `?` 表示可选参数。可选参数必须跟在必须参数后面。

```ts
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

let result1 = buildName("Bob");  // works correctly now
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");  // ah, just right
```

* 默认参数

在 TypeScript 里，我们也可以为参数提供一个 默认值 当用户没有传递这个参数或传递的值是 `undefined` 时。它们叫做有默认初始化值的参数。在所有必须参数后面的带默认初始化的参数都是可选的。

与普通可选参数不同的是，带默认值的参数不需要放在必须参数的后面。如果带默认值的参数出现在必须参数前面，用户必须明确的传入 `undefined` 值来获得默认值。

* 剩余参数

必要参数，默认参数和可选参数有个共同点：它们表示某一个参数。有时，你想同时操作多个参数，或者你并不知道会有多少参数传递进来。在 JavaScript 里，你可以使用 `arguments` 来访问所有传入的参数。

在 TypeScript 里，你可以把所有参数收集到一个变量里：

```ts
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

### 1.2 重载

JavaScript 本身是个动态语言。JavaScript 里函数根据传入不同的参数而返回不同类型的数据。这时，可以为同一个函数提供多个函数类型定义来进行函数重载。编译器会根据这个列表去处理函数的调用。

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

它查找重载列表，尝试使用第一个重载定义。如果匹配的话就使用这个。因此，在定义重载的时候，一定要把最精确的定义放在最前面。

注意，`function pickCard(x): any` 并不是重载列表的一部分，因此这里只有两个重载：一个是接收对象另一个接收数字。

## 泛型

我们需要一种方法使返回值的类型与传入参数的类型是相同的。 这里，我们使用了 *类型变量*，它是一种特殊的变量，只用于表示类型而不是值。

```typescript
function identity<T>(arg: T): T {
    return arg;
}
```

我们给 identity 添加了类型变量 `T`。 `T` 帮助我们捕获用户传入的类型（比如：number），之后我们就可以使用这个类型。 之后我们再次使用了 `T` 当做返回值类型。现在我们可以知道参数类型与返回值类型是相同的了。

我们定义了泛型函数后，可以用两种方法使用。 第一种是，传入所有的参数，包含类型参数：

    let output = identity<string>("myString");

这里我们明确的指定了 `T` 是 `string` 类型，并做为一个参数传给函数，使用了 `<>` 括起来而不是 `()`。

第二种方法更普遍。利用了类型推论 -- 即编译器会根据传入的参数自动地帮助我们确定 `T` 的类型：

    let output = identity("myString");

### 2.1 泛型接口

```typescript
interface GenericIdentityFn {
    <T>(arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn = identity;

<!-- 把泛型参数当作整个接口的一个参数 -->
interface GenericIdentityFn<T> {
    (arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

### 2.2 泛型类

```typescript
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

### 2.3 泛型约束

想要限制函数去处理任意带有 `.length` 属性的所有类型。只要传入的类型有这个属性，我们就允许，就是说至少包含这一属性。 为此，我们需要列出对于 `T` 的约束要求。

定义一个接口来描述约束条件。 创建一个包含 `.length` 属性的接口，使用这个接口和 `extends` 关键字来实现约束：

```typescript
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);
    return arg;
}

loggingIdentity({length: 10, value: 3});
```

## 枚举

枚举通过 `enum` 关键字来定义。使用枚举我们可以定义一些有名字的数字常量。

一个枚举类型可以包含零个或多个枚举成员。 枚举成员具有一个数字值，它可以是 *常数* 或是 *计算得出的值* 当满足如下条件时，枚举成员被当作是常数：

1.不具有初始化函数并且之前的枚举成员是常数。 在这种情况下，当前枚举成员的值为上一个枚举成员的值加1。 但第一个枚举元素是个例外。 如果它没有初始化方法，那么它的初始值为 0。

2.枚举成员使用常数枚举表达式初始化。 常数枚举表达式是TypeScript表达式的子集，它可以在编译阶段求值。 当一个表达式满足下面条件之一时，它就是一个常数枚举表达式：

    * 数字字面量
    * 引用之前定义的常数枚举成员（可以是在不同的枚举类型中定义的） 如果这个成员是在同一个枚举类型中定义的，可以使用非限定名来引用。
    * 带括号的常数枚举表达式
    * `+, -, ~` 一元运算符应用于常数枚举表达式
    * `+, -, *, /, %, <<, >>, >>>, &, |, ^` 二元运算符，常数枚举表达式做为其一个操作对象。若常数枚举表达式求值后为 `NaN` 或 `Infinity`，则会在编译阶段报错。

所有其它情况的枚举成员被当作是需要计算得出的值。如下：

```typescript
enum FileAccess {
    // constant members
    None,
    Read    = 1 << 1,
    Write   = 1 << 2,
    ReadWrite  = Read | Write,
    // computed member
    G = "123".length
}

<!-- 数字枚举 -->
enum Direction {
    Up = 1,
    Down,
    Left,
    Right
}
```

如上，我们定义了一个数字枚举 `Direction`，`Up` 使用初始化为 1 (默认从 0 开始)。 其余的成员会从 1 开始自动增长。 换句话说，`Direction.Up` 的值为 1，`Down` 为 2，`Left` 为 3，`Right` 为 4。

```ts

enum Response {
    No = 0,
    Yes = 1,
}

function respond(recipient: string, message: Response): void {
    // ...
}

respond("Princess Caroline", Response.Yes)
```

使用枚举很简单，如上：通过枚举的属性来访问枚举成员，和枚举的名字来访问枚举类型。

### 联合枚举与枚举成员的类型

存在一种特殊的非计算的常量枚举成员的子集：字面量枚举成员。 字面量枚举成员是指不带有初始值的常量枚举成员，或者是值被初始化为

* 任何字符串字面量（例如： "foo"， "bar"， "baz"）
* 任何数字字面量（例如： 1, 100）
* 应用了一元 -符号的数字字面量（例如： -1, -100）

```ts
enum ShapeKind {
    Circle,
    Square,
}

interface Circle {
    kind: ShapeKind.Circle;
    radius: number;
}

interface Square {
    kind: ShapeKind.Square;
    sideLength: number;
}

let c: Circle = {
    kind: ShapeKind.Square, //    ~~~~~~~~~~~~~~~~ Error!
    radius: 100,
}
```

如上：枚举成员成为了类型。

### 反向映射

除了创建一个以属性名做为对象成员的对象之外，数字枚举成员还具有了**反向映射**，从枚举值到枚举名字。例如：

```ts
enum Enum {
    A
}
let a = Enum.A;
let nameOfA = Enum[a]; // "A"
```

要注意的是 不会为字符串枚举成员生成反向映射。

### const 枚举

为了避免在额外生成的代码上的开销和额外的非直接的对枚举成员的访问，我们可以使用 `const` 枚举。常量枚举通过在枚举上使用 `const` 修饰符来定义:

```ts
const enum Enum {
    A = 1,
    B = A * 2
}
```

常量枚举只能使用常量枚举表达式，并且不同于常规的枚举，它们在编译阶段会被删除。常量枚举成员在使用的地方会被内联进来。之所以可以这么做是因为，常量枚举不允许包含计算成员。

### 外部枚举

外部枚举用来描述已经存在的枚举类型的形式。

```typescript
declare enum Enum {
    A = 1,
    B,
    C = 2
}
```

外部枚举和非外部枚举之间有一个重要的区别，在正常的枚举里，没有初始化方法的成员被当成常数成员。对于非常数的外部枚举而言，没有初始化方法时被当做需要经过计算的。