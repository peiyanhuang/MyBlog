---
layout: post
title:  '__proto__'和'prototype'的区别和关系
date:   2017-01-19 20:38:00 +0800
categories: JS
tag: JS
---

* content
{:toc}

首先，要明确几个点：

1.在JS里，万物皆对象。方法（Function）是对象，方法的原型(Function.prototype)是对象。因此，它们都会具有对象共有的特点。
即：对象具有属性__proto__，可称为隐式原型，一个对象的隐式原型指向构造该对象的构造函数的原型，这也保证了实例能够访问在构造函数原型中定义的属性和方法。

注：普通对象没有prototype,但有__proto__属性。

2.方法(Function)
方法这个特殊的对象，除了和其他对象一样有上述_proto_属性之外，还有自己特有的属性——原型属性（prototype），这个属性是一个指针，指向一个对象，这个对象的用途就是包含所有实例共享的属性和方法（我们把这个对象叫做原型对象）。原型对象也有一个属性，叫做constructor，这个属性包含了一个指针，指回原构造函数。

![images]({{"/images/proto.png" | prepend: site.baseurl}})

好啦，知道了这两个基本点，我们来看看上面这副图。

1.构造函数Foo()

构造函数的原型属性Foo.prototype指向了原型对象，在原型对象里有共有的方法，所有构造函数声明的实例（这里是f1，f2）都可以共享这个方法。

2.原型对象Foo.prototype

Foo.prototype保存着实例共享的方法，有一个指针constructor指回构造函数。

3.实例

f1和f2是Foo这个对象的两个实例，这两个对象也有属性__proto__，指向构造函数的原型对象，这样子就可以像上面1所说的访问原型对象的所有方法啦。

例子代码：

```
var person = function(){
	...
};

var one = new person();

console.log(one.__proto__ === person.prototype) //true
console.log(person.prototype.__proto__ === Object.prototype) //true
console.log(person.prototype.constructor === person) //true
console.log(Object.prototype.__proto__) //null
```

总结：

1.`实例对象的__proto__`属性,指向该对象的构造函数的`原型对象(prototype)`。

2.`原型对象(prototype)`的`__proto__`属性，指向`Object.prototype`

3.`原型对象(prototype)`的`constructor`,指向`构造函数本身`

4.`Object.prototype`的`__proto__`属性指向`null`

```
Object.__proto__ === Function.prototype // true

Function.__proto__ === Function.prototype // true

Function.prototype.__proto__ === Object.prototype //true

Function.prototype.constructor === Function //true

Object.prototype.constructor === Object //true

Object.constructor === Function；//true 
```

看看以下打印什么？

```
var animal = function(){};
var dog = function(){};

animal.price = 2000;//
dog.prototype = animal;
var tidy = new dog();

console.log(dog.price);
```

结果：

```
dog.price
//undefined

dog.prototype.price
//2000
```