---
layout: post
title:  typeScript 类
date:   2017-10-26 19:58:00 +0800
categories: TypeScript
tag: TypeScript
---

* content
{:toc}

### 1. 类

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