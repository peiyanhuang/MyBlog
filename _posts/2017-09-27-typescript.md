---
layout: post
title:  typeScript 
date:   2017-10-23 19:58:00 +0800
categories: JS
tag: TypeScript
---

* content
{:toc}

### 1. 安装使用

npm 安装:

```
> npm install -g typescript
```

编译方式:

```
tsc ***.ts
```

输出结果为一个`***.js`文件，它包含了和输入文件中相同的JavsScript代码。

### 2. 类型注解

类型注解是一种为 函数 或 变量 添加约束的方式。

```
function greeter(person: string) {
    return "Hello, " + person;
}

var user = [0, 1, 2];

document.body.innerHTML = greeter(user);
```

#### 2.1 基础类型

```
1.布尔值: 最基本的数据类型就是简单的true/false值。
let isDone: boolean = false; 

2.数字: 和JavaScript一样，TypeScript里的所有数字都是浮点数。支持二进制、八进制、十进制和十六进制字面量。
let num: number = 0x12f;

3.字符串: 和JavaScript一样，可以使用双引号（"）或单引号（'）表示字符串, 可以使用模版字符串。
let name: string = `Bob is ${age} years old`;

4.数组: 有两种方式可以定义数组。 
第一种，可以在元素类型后面接上 []，表示由此类型元素组成的一个数组：
let list: string[] = ['a', 'b', 'c'];
第二种方式是使用数组泛型，Array<元素类型>：
let list: Array<number> = [1, 2, 3];

5.元组 Tuple: 
元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。
let x: [string, number, boolean];

6.枚举
enum类型是对JavaScript标准数据类型的一个补充。
默认情况下，从0开始为元素编号。也可以手动的指定成员的数值。例如，改成从 1开始编号：
enum Color {Red = 1, Green, Blue}
let c: Color = Color.Green;
let colorName: string = Color[2];  //Green

7.Any
有时候，我们不希望类型检查器对某些值进行检查而是直接让它们通过编译阶段的检查。 那么我们可以使用 any类型来标记这些变量：
let notSure: any = 4;

8.Void
某种程度上来说，void类型像是与any类型相反，它表示没有任何类型。 当一个函数没有返回值时，你通常会见到其返回值类型是 void：
function warnUser(): void {
    alert("This is my warning message");
}
声明一个void类型的变量没有什么大用，因为你只能为它赋予undefined和null：
let unusable: void = undefined;

9.Null 和 Undefined
默认情况下null和undefined是所有类型的子类型。就是说你可以把 null和undefined赋值给number类型的变量。
然而，当你指定了--strictNullChecks标记，null和undefined只能赋值给void和它们各自。

10.Never
never类型表示的是那些永不存在的值的类型。 例如， never类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型。
never类型是任何类型的子类型，也可以赋值给任何类型；然而，没有类型是never的子类型或可以赋值给never类型（除了never本身之外）。 即使 any也不可以赋值给never。
```

#### 2.2 类型断言

通过类型断言这种方式可以告诉编译器，“相信我，我知道自己在干什么”。 类型断言好比其它语言里的类型转换，但是不进行特殊的数据检查和解构。 它没有运行时的影响，只是在编译阶段起作用。 TypeScript会假设你，程序员，已经进行了必须的检查。

类型断言有两种形式。 其一是“尖括号”语法：

```
let someValue: any = "this is a string";

let strLength: number = (<string>someValue).length;
```

另一个为as语法：

```
let someValue: any = "this is a string";

let strLength: number = (someValue as string).length;
```

两种形式是等价的。 至于使用哪个大多数情况下是凭个人喜好；然而，当你在TypeScript里使用`JSX`时，只有 `as`语法断言是被允许的。

### 3. 接口

使用接口来描述一个拥有`firstName`和`lastName`字段的对象。 在TypeScript中，只要两个类型内部的结构兼容那么那么可以认为他们是兼容的。 这就允许我们在实现接口时候只要保证包含了接口要求的结构就可以，而不必明确地使用 implements语句。

```
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

### 4. 类

在构造函数的参数上使用public等同于创建了同名的成员变量.

```
class Student {
    fullName: string;
    constructor(public firstName, public middleInitial, public lastName) {
        this.fullName = firstName + " " + middleInitial + " " + lastName;
    }
}

interface Person {
    firstName: string;
    lastName: string;
}

function greeter(person : Person) {
    return "Hello, " + person.firstName + " " + person.lastName;
}

var user = new Student("Jane", "M.", "User");

document.body.innerHTML = greeter(user);
```