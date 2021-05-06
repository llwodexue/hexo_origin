---
title: JS 原型继承
tags:
  - JavaScript
  - 原型链继承
  - 借用构造函数
  - 寄生组合继承
  - 中间类继承
categories: JavaScript
author: LiLyn
copyright: ture
abbrlink: 3792d7db
---

## 继承

继承机制使得不同的实例可以共享构造函数的原型对象的属性和方法

<!--more-->

### 原型链继承

> 基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。我们知道每个构造函数都有一个原型对象（`prototype`），原型对象都包含一个指向构造函数的指针（`constructor`），而实例都包含一个指向原型对象的内部指针（`__proto__`）
>
> 那么，如果让一个构造函数的原型对象等于另一个类型的实例，此时的原型对象将包含一个指向另一个原型的指针，相应地，另一个原型中也包含着一个指向另一个构造函数的指针

- 想要让 B 所有的实例继承 A 类私有和公有方法：

  私有属性：使用构造函数时添加给实例的属性

  公有属性：构造函数原型上的属性

```js
function A() {
    this.a = "a";
}
A.prototype.getA = function () {
    console.log(this.a);
};

function B() {
    this.b = "b";
}
// 让 B 类的原型指向 A 类的实例
B.prototype = new A();
var b = new B();
```

- 让 B 类的原型（prototype）指向 A 类的实例，此时 B 类的原型是 A 类的实例（继承 A 类私有属性），同时 B 类的原型的 `__proto__` 指向 A 类的原型（继承 A 的公有属性）

![原型继承](https://gitee.com/lilyn/pic/raw/master/js-img/%E5%8E%9F%E5%9E%8B%E7%BB%A7%E6%89%BF.png)

**原型链继承的问题：**通过原型链继承后，B 的原型继承了 A 的实例属性变成了 B 的原型属性，若继承 A 的实例属性里面有引用数据类型，更改 B 的实例属性 colors，后续 B 的实例的实例属性 colors 都会被修改

```js
function A() {
    this.colors = ["red", "green"];
}
function B() {}
// new A()创建实例，实例拥有 colors 属性
// B 的原型改为该实例，则 B 的原型中添加 colors 属性
B.prototype = new A();
var arr1 = new B();
arr1.colors.push("yellow");
console.log(arr1.colors); // [ 'red', 'green', 'yellow' ]
var arr2 = new B();
console.log(arr2.colors); // [ 'red', 'green', 'yellow' ]
```

**原型链继承的另一个问题：**在创建子类型的实例时，不能向超类型的构造函数中传递参数。实践中很少会单独使用原型链继承



### 中间类继承

某些实例不属于某些类，但是想要用这个类原型上的方法，可以手动的去更改实例的 `__proto__` ，让它指向这个类的原型，这样这个实例就可以使用这个类的原型上的方法

```js
function fn() {
    arguments.__proto__ = Array.prototype;
    // arguments 是一个对象，对象上没有 sort 方法会报错
    return arguments.sort(function (a, b) {
        return a - b;
    });
}
console.log(fn(5, 3, 6, 4)); // Array { '0': 3, '1': 4, '2': 5, '3': 6 }
```



### 借用构造函数（call 继承）

> 基本思想：在子类型构造函数的内部调用超类型构造函数，通过 apply() 和 call() 方法可以在新创建的对象上执行构造函数

B 类和 A 类，想要让 B 类的实例拥有 A 类的私有属性（但不拥有公有属性），我们可以让 A 当成普通函数执行，把里面的 this 指向改成 B 类的实例（往 B 的实例中添加属性）

```js
function A() {
    this.a = "a";
    this.x = 100;
}
function B() {
    // 实例 b this
    // 去执行 A()，但A里this和B里this不是一个，需要用call绑定下this
    A.call(this)
    this.b = "b";
    this.y = 200;
}
var b = new B();
```

**借用构造函数问题：**因为方法都是在构造函数中定义，因此就没有函数复用。而且，超类型的原型中定义的方法，对子类型来说是不可见的（只能添加私有属性）。实践中很少会单独使用借用构造函数



### 寄生组合继承

> 基本思想：借用原型链实现对原型属性和方法的继承，通过借用构造函数来实现实例属性的继承。这样既通过原型上定义方法实现了函数的复用，又保证每个实例都有自己的属性

需求：B 类要继承 A 类的私有属性和公有属性

```js
function A() {
    this.a = "a";
    this.x = 100;
}
A.prototype.getA = function () {
    console.log("A");
};
function B() {
    this.b = "b";
    this.y = 200;
}
```

- 我们很容易就能想到最简单的方法：A、B类的原型指向同一个堆内存地址。但修改时会相互影响，耦合性高，不推荐这么做

```js
B.prototype = A.prototype;
B.prototype.getB = function () {
    console.log("B");
};
```

1. 我们可以先用借用构造函数获取 A 类的私有属性

```js
function B() {
    A.call(this);
}
```

2. 再用原型链继承获取 A 类公有属性

```js
// 可以考虑用普通对象做中间件
var obj = {};
obj.__proto__ = A.prototype;
B.prototype = obj;
```

```js
// 可以考虑用函数做中间件
function F(){}
F.prototype = A.prototype;
B.prototype = new F();
```

2. 上述方法可以简写一下，需要使用如下方法：

```js
// 创建一个空对象，并且让空对象的 __proto__ 指向参数
B.prototype = Object.create(A.prototype);
```

- Object.create() 方法，它可以传入一个对象作为该方法返回的对象的原型对象，原理如下：

```js
Object.create = function (obj) {
    // 版本1
    var o = {};
    o.__proto__ = obj;
    return obj;
    
    // 版本2
    function F() {}
    F.prototype = obj;
    return new F();
};
```

3. 另一种方法

```js
function B() {
    // 继承私有属性
    A.call(this);
}
// 继承公有属性
B.prototype = new A();
B.prototype.constructor = B;
```

- 整理一下上述思路，合并一下代码

```js
function A() {
    this.a = "a";
    this.x = 100;
}
A.prototype.getA = function () {
    console.log("A");
};
function B() {
    // 1.获取私有属性
    A.call(this);
    this.b = "b";
    this.y = 200;
}
// 2.可以考虑用普通对象做中间件
var obj = {};
obj.__proto__ = A.prototype;
B.prototype = obj;
// 2.可以考虑用函数做中间件
function F(){}
F.prototype = A.prototype;
B.prototype = new F();
// 2.使用 Object.create() 方法
B.prototype = Object.create(A.prototype);

// 3.使用原型链继承
B.prototype = new A();
B.prototype.constructor = B;
var b = new B();
```

组合继承融合了原型链和借用构造函数的优点，而且 instanceof 也能用于识别基于组合继承创建的对象，所以实践中最常用组合继承