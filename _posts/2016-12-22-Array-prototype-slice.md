---
layout: post
title:  Array.prototype.slice.call(arguments)
date:   2016-12-22 20:58:00 +0800
categories: 2016
tag: JS
---

* content
{:toc}

slice() 方法会浅复制（shallow copy）数组的一部分到一个新的数组，并返回这个新数组。

### 描述

slice 不修改原数组，只会返回一个浅复制了原数组中的元素的一个新数组。原数组的元素会按照下述规则拷贝：

- 如果该元素是个对象引用 （不是实际的对象），slice 会拷贝这个对象引用到新的数组里。两个对象引用都引用了同一个对象。如果被引用的对象发生改变，则新的和原来的数组中的这个元素也会发生改变。
- 对于字符串、数字及布尔值来说（不是 String、Number 或者 Boolean 对象），slice 会拷贝这些值到新的数组里。在别的数组里修改这些字符串或数字或是布尔值，将不会影响另一个数组。
如果向两个数组任一中添加了新元素，则另一个不会受到影响。

在下例中, slice从myCar中创建了一个新数组newCar.两个数组都包含了一个myHonda对象的引用. 当myHonda的color属性改变为purple, 则两个数组中的对应元素都会随之改变.

	// 使用slice方法从myCar中创建一个newCar.
	var myHonda = { color: "red", wheels: 4, engine: { cylinders: 4, size: 2.2 } };
	var myCar = [myHonda, 2, "cherry condition", "purchased 1997"];
	var newCar = myCar.slice(0, 2);

	// 输出myCar, newCar,以及各自的myHonda对象引用的color属性.
	print("myCar = " + myCar.toSource());
	print("newCar = " + newCar.toSource());
	print("myCar[0].color = " + myCar[0].color);
	print("newCar[0].color = " + newCar[0].color);

	// 改变myHonda对象的color属性.
	myHonda.color = "purple";
	print("The new color of my Honda is " + myHonda.color);

	//输出myCar, newCar中各自的myHonda对象引用的color属性.
	print("myCar[0].color = " + myCar[0].color);
	print("newCar[0].color = " + newCar[0].color);
	上述代码输出:

	myCar = [{color:"red", wheels:4, engine:{cylinders:4, size:2.2}}, 2, "cherry condition", "purchased 1997"]
	newCar = [{color:"red", wheels:4, engine:{cylinders:4, size:2.2}}, 2]
	myCar[0].color = red 
	newCar[0].color = red
	The new color of my Honda is purple
	myCar[0].color = purple
	newCar[0].color = purple

### Array.prototype.slice.call(arguments)

slice 方法可以用来将一个类数组（Array-like）对象/集合转换成一个数组。

所谓类似数组的对象，本质特征只有一点，即必须有`length`属性。

你只需将该方法绑定到这个对象上。下述代码中 list 函数中的 arguments 就是一个类数组对象。

	function list() {
	  return Array.prototype.slice.call(arguments);
	}

	var list1 = list(1, 2, 3); // [1, 2, 3]

除了使用 `Array.prototype.slice.call(arguments)`，你也可以简单的使用 `[].slice.call(arguments)` 来代替。另外，你可以使用 bind 来简化该过程。

ES6的 `Array.from`方法也可用于将两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括ES6新增的数据结构Set和Map）。

扩展运算符（`...`）也可以将某些数据结构转为数组。例如字符串

	[...'hello']
	// [ "h", "e", "l", "l", "o" ]

rest参数也会视为数组。

                  