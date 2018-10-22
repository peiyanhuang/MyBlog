---
layout: post
title:  typeScript 基础类型和接口
date:   2018-02-10 19:58:00 +0800
categories: TypeScript
tag: TypeScript
---

* content
{:toc}

### 1. 基础类型

类型注解是一种为 `函数` 或 `变量` 添加约束的方式，即规定变量的类型。

```typescript
function greeter(person: string) {
    return "Hello, " + person;
}

var user = [0, 1, 2];

document.body.innerHTML = greeter(user);
```

#### 1.1 基础类型

```js
// 1.布尔值: 最基本的数据类型就是简单的true/false值。
let isDone: boolean = false; 

// 2.数字: 和JavaScript一样，TypeScript里的所有数字都是浮点数。支持二进制、八进制、十进制和十六进制字面量。
let num: number = 0x12f;

// 3.字符串: 和JavaScript一样，可以使用双引号（"）或单引号（'）表示字符串, 可以使用模版字符串。
let name: string = `Bob is ${age} years old`;

// 4.数组: 有两种方式可以定义数组。 
// 第一种，可以在元素类型后面接上 []，表示由此类型元素组成的一个数组：
let list: string[] = ['a', 'b', 'c'];
// 第二种方式是使用数组泛型，Array<元素类型>：
let list: Array<number> = [1, 2, 3];

// 5.元组 Tuple: 
// 元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。
let x: [string, number, boolean];

// 6.枚举
// enum类型是对JavaScript标准数据类型的一个补充。默认情况下，从0开始为元素编号。也可以手动的指定成员的数值。例如，改成从 1开始编号：
enum Color {Red = 1, Green, Blue}
let c: Color = Color.Green;
let colorName: string = Color[2];  //Green

// 7.Any
// 有时候，我们不希望类型检查器对某些值进行检查而是直接让它们通过编译阶段的检查。 那么我们可以使用 any类型来标记这些变量：
let notSure: any = 4;

// 8.Void
// 某种程度上来说，void类型像是与any类型相反，它表示没有任何类型。 当一个函数没有返回值时，你通常会见到其返回值类型是 void：
function warnUser(): void {
    alert("This is my warning message");
}
// 声明一个void类型的变量没有什么大用，因为你只能为它赋予undefined和null：
let unusable: void = undefined;

// 9.Null 和 Undefined
// 默认情况下null和undefined是所有类型的子类型。就是说你可以把 null 和 undefined 赋值给 number 类型的变量。然而，当你指定了--strictNullChecks标记，null和undefined只能赋值给void和它们各自。

// 10.Never
// never类型表示的是那些永不存在的值的类型。 例如， never类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型。
// never类型是任何类型的子类型，也可以赋值给任何类型；然而，没有类型是never的子类型或可以赋值给never类型（除了never本身之外）。 即使 any也不可以赋值给never。
```

#### 1.2 类型断言

通过类型断言这种方式可以告诉编译器，“相信我，我知道自己在干什么”。 类型断言好比其它语言里的类型转换，但是不进行特殊的数据检查和解构。 它没有运行时的影响，只是在编译阶段起作用。 TypeScript会假设你，程序员，已经进行了必须的检查。

类型断言有两种形式。 其一是“尖括号”语法：

```js
let someValue: any = "this is a string";

let strLength: number = (<string>someValue).length;
```

另一个为as语法：

```js
let someValue: any = "this is a string";

let strLength: number = (someValue as string).length;
```

两种形式是等价的。 至于使用哪个大多数情况下是凭个人喜好；然而，当你在TypeScript里使用 `JSX` 时，只有 `as` 语法断言是被允许的。

### 2. 接口

使用接口来描述一个拥有 `firstName` 和 `lastName` 字段的对象。 在 TypeScript 中，只要两个类型内部的结构兼容那么那么可以认为他们是兼容的。 这就允许我们在实现接口时候只要保证包含了接口要求的结构就可以，而不必明确地使用 `implements` 语句。

```js
interface Person {
    firstName: string;
    lastName: string;
}

function greeter(person: Person) {
    return "Hello, " + person.firstName + " " + person.lastName;
}

var user = { firstName: "Jane", lastName: "User" };

document.body.innerHTML = greeter(user);
```

#### 2.1 可选属性

接口里的属性不一定全都是必需的。 有些是只在某些条件下存在，或者根本不存在。即给函数传入的参数对象中只有部分属性赋值了。

```typescript
interface SquareConfig {
    color?: string;
    width?: number;
}
```

带有可选属性的接口与普通的接口定义差不多，只是在可选属性名字定义的后面加一个 `?` 符号。

```typescript
function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ color: "red", width: 100 });
```

#### 2.2 只读属性

一些对象属性只能在对象刚刚创建的时候修改其值。 你可以在属性名前用`readonly`来指定只读属性:

```typescript
interface Point {
    readonly x: number;
    readonly y: number;
}

let p1: Point = { x: 10, y: 20 };
p1.x = 5; // error!
```

TypeScript具有 `ReadonlyArray<T>` 类型，它与 `Array<T>` 相似，只是把所有可变方法去掉了，因此可以确保数组创建后再也不能被修改。

```typescript
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro.length = 100; // error!
a = ro; // error!
```

上面代码的最后一行，可以看到就算把整个 `ReadonlyArray` 赋值到一个普通数组也是不可以的。 但是你可以用类型断言重写：

```typescript
a = ro as number[];
```

#### 2.3 额外的属性检查

```typescript
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ rgb: "red", width: 100 });
```

如上，我们可能会认为其是正确的：因为 width 属性是兼容的，不存在 color 属性，而且额外的 rgb 属性是无意义的。

然而，TypeScript会认为这段代码可能存在bug -- 对象字面量会被特殊对待而且会经过 *额外属性检查*，当将它们赋值给变量或作为参数传递的时候。 如果一个对象字面量存在任何“目标类型”不包含的属性时，你会得到一个错误。

绕开这些检查非常简单。 最简便的方法是使用类型断言：

```typescript
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
```

然而，最佳的方式是能够添加一个字符串索引签名：

```typescript
interface SquareConfig {
    color?: string;
    width?: number;
    [propName: string]: any;
}
```

还有最后一种跳过这些检查的方式它就是将这个对象赋值给另一个变量： 因为 squareOptions 不会经过额外属性检查，所以编译器不会报错。

```typescript
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions);
```

#### 2.4 函数类型

为了使用接口表示函数类型，我们需要给接口定义一个调用签名。 它就像是一个只有参数列表和返回值类型的函数定义。参数列表里的每个参数都需要名字和类型。

```typescript
interface SearchFunc {
    (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(src: string, sub: string): boolean {
    let result = src.search(sub);
    return result > -1;
}
```

函数的参数名不需要与接口里定义的名字相匹配。函数的参数会逐个进行检查，要求对应位置上的参数类型是兼容的。也可以这样调用：

```typescript
let mySearch: SearchFunc;
mySearch = function(src, sub) {
    let result = src.search(sub);
    return result > -1;
}
```

#### 2.5 可索引的类型

与使用接口描述函数类型差不多，我们也可以描述那些能够“通过索引得到”的类型，比如a[10]或ageMap["daniel"]。可索引类型具有一个 索引签名，它描述了对象索引的类型，还有相应的索引返回值类型。 

```typescript
interface StringArray {
  [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```

#### 2.6 类类型

使用 `implements` 关键字，可以在接口中描述一个方法，在类里实现它:

```typescript
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

#### 2.7 混合类型

```typescript
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = <Counter>function (start: number) { };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```

#### 2.8 继承接口

和类一样，接口也可以相互继承。

```typescript
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

当接口继承了一个类类型时，它会继承类的成员但不包括其实现:

```typescript
class Control {
    private state: any;
}

interface SelectableControl extends Control {
    select(): void;
}

class Button extends Control implements SelectableControl {
    select() { }
}

// 错误：“Image”类型缺少“state”属性。
class Image implements SelectableControl {
    select() { }
}
```

在Control类内部，是允许通过SelectableControl的实例来访问私有成员state的。 实际上， SelectableControl就像Control一样，并拥有一个select方法。 Button和TextBox类是SelectableControl的子类（因为它们都继承自Control并有select方法），但Image和Location类并不是这样的。