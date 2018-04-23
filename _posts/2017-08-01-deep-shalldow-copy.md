---
layout: post
title:  javascript中的深拷贝和浅拷贝
date:   2018-04-23 13:08:00 +0800
categories: JS
tag: JS
---

* content
{:toc}

### 1.浅拷贝与深拷贝的区别

JS 中的浅拷贝与深拷贝，只是针对复杂数据类型（Object，Array）的复制问题。浅拷贝与深拷贝都可以实现在已有对象上再生出一份的作用。但是对象的实例是存储在堆内存中然后通过一个引用值去操作对象，由此拷贝的时候就存在两种情况了：--拷贝引用和拷贝实例，这也是浅拷贝和深拷贝的区别。

- 浅拷贝：浅拷贝是拷贝引用，拷贝后的引用都是指向同一个对象的实例(指向同一个内存地址)，彼此之间的操作会互相影响

- 深拷贝：重新分配内存，并且把源对象所有属性都进行新建拷贝，以保证深拷贝的对象的引用图不包含任何原有对象或对象图上的任何对象，拷贝后的对象与原来的对象是完全隔离，互不影响

### 2.浅拷贝

这是最简单的浅拷贝，直接引用，例:

```
let obj = { x: 1, y: 2 };
let objCopy = obj;

console.log(obj === objCopy); // 输出true。
objCopy.x = 3;
console.log(obj.x); // 输出 3
```

下面是一个简单的浅复制实现：

```
function shallowCopy(src) {
    var dst = {};
    for (var prop in src) {
        if (src.hasOwnProperty(prop)) {
            dst[prop] = src[prop];
        }
    }
    return dst;
}

var obj = { a:1, arr: [2,3] };
var shallowObj = shallowCopy(obj);
```

上面的例子中，浅复制函数`shallowCopy`只会将对象的各个属性进行依次复制，并不会进行递归复制，而 JavaScript 存储对象都是存地址的，所以浅复制会导致 obj.arr 和 shallowObj.arr 指向同一块内存地址.

还有一种也是浅拷贝：源对象拷贝实例，其属性对象拷贝引用

```
var a = [{c:1}, {d:2}];
var b = a.slice();
console.log(a === b); // 输出false，说明外层数组拷贝的是实例
a[0].c = 3;
console.log(b[0].c); // 输出 3，说明其元素拷贝的是引用
```

对于像 `Array.prototype.slice()`, `Array.prototype.concat()`等方法，可以看出：

外层源对象是拷贝实例，如果其属性元素为复杂数据类型时，内层元素拷贝引用。
对源对象直接操作，不影响另外一个对象，但是对其属性操作时候，会改变另外一个对象的属性的值。

### 3.深拷贝

深拷贝后，两个对象，包括其内部的元素互不干扰。常见方法有 `JSON.parse()`, `JSON.stringify()`, jQury 的 `$.extend(true,{},obj)`, lodash的 `_.cloneDeep` 和 `_.clone(value, true)`。

[jQury 的 $.extend(true,{},obj) 方法](https://github.com/jquery/jquery/blob/1472290917f17af05e98007136096784f9051fab/src/core.js#L121)

```
function deepCopy(value, targets) {
    let target = targets || {};
    if (typeof value !== "object" && !(value instanceof Function)) {
        target = {};
    }
    if (value !== null) {
        for (var i in value) {
            if (value[i] === target[i]) {
                continue;
            }
            if (typeof value[i] === 'object') {
                if (value[i] instanceof Array) {    //is array
                    target[i] = [];
                } else {    // is object
                    target[i] = {};
                }
                target[i] = deepCopy(value[i], target[i]);
            } else {
                target[i] = value[i]
            }
        }
    }
    return target;
}
```

另一个简单的方法是使用 `JSON.parse()、JSON.stringify()`:

```
let _clone = function(obj){
    let str, 
    	  newobj = obj.constructor === Array ? [] : {};
    if(typeof obj !== 'object'){
        return;
    } else if(window.JSON){
        str = JSON.stringify(obj); 
        newobj = JSON.parse(str); 
    } else {
        for(let i in obj){
            newobj[i] = typeof obj[i] === 'object' ? cloneObj(obj[i]) : obj[i]; 
        }
    }
    return newobj;
};
```

简单的 `JSON.parse(JSON.stringify(obj))`, 但只能支持JSON格式的。这是一种可以实现深拷贝的方法.
但这种方法的缺陷是会破坏原型链,并且无法拷贝属性值为function的属性。

### 3.Object.assign()

ES6为我们提供了一种十分好用的方法：`Object.assign(target, ...source)`。

`assign`方法接受多个参数，第一个参数`target`为拷贝目标，剩余参数`...source`是拷贝源。此方法可以将`...source`中的属性复制到`target`中，同名属性会进行覆盖，并且在复制过程中实现了'伪'深拷贝。

```javascript
let foo = {
    a: 1,
    b: 2,
    c: {
        d: 1,
    }
}
let bar = {};
Object.assign(bar, foo);
foo.a++;
foo.a === 2 //true
bar.a === 1 //true
```

乍一看，好像已经实现了深拷贝的效果，对 foo.a 进行的操作并没有体现在 bar.a 中,但是再往后看

```
foo.c.d++;
foo.c.d === 2 //true
bar.c.d === 1 //false
bar.c.d === 2 //true
Object.assign()
```

这是一种可以对非嵌套对象进行深拷贝的方法,如果对象中出现嵌套情况,那么其对被嵌套对象的行为就成了普通的浅拷贝.
如果真的想进行深拷贝,最简单粗暴地方式就是 JSON 操作。