---
title: JS 垃圾回收与性能优化
tags:
  - JavaScript
  - 垃圾回收
  - 性能优化
categories: JavaScript
author: LiLyn
copyright: ture
abbrlink: 3344fd09
---

JavaScript 通过自动内存管理实现内存分配和闲置资源回收，基本思路：确定哪个变量不会再使用，然后释放它占用的内存。这个过程是周期性的，即垃圾回收程序每隔一定时间就会自动运行。但是某块内是否还有用，属于“不可判断的”问题，意味着靠算法是解决不了的

<!--more-->

## 垃圾回收方法

### 标记清理

JavaScript 最常用的垃圾回收策略是标记清理（mark-and-sweep）。当变量进入环境时就将其标记为`进入环境`，当变量离开环境时将其标记为`离开环境`。随后每隔一段时间就会检测一下当前作用域中的内存，是否被占用，如果没有被占用，在空闲的时候就将其释放掉

### 引用计数

思路是对每个值都记录它被引用的次数。当一个空间地址被占用一次，引用数 +1，如果不占用，就引用数 -1。当一个值引用数为 0时，说明已经没有被占用了，在空闲的时候就将其释放掉

```js
function problem() {
    let o1 = new Object();
    let o2 = new Object();
    o1.A = o2;
    o2.B = o1;
}
```

o1 和 o2 相互引用，意味着它们引用数都是 2，当函数结束后, o1 和 o2 还会存在，如果函数调用多次，会导致大量内存永远不被释放

### 手动进行内存释放

在IE8及以前的IE版本中，并非所有对象都是原生 JavaScript 对象，BOM 和 DOM 中的对象是 C++ 实现的组件对象模型（COM），而 COM 对象使用引用计数实现垃圾回收

```js
let el = document.getElementById("div");
let o = new Object();
o.el = el;
el.obj = o;
```

由于存在循环引用，因此 DOM 元素内存永远不会被回收，需要将变量设置为 null （解除引用），切断变量与其引用值之间的关系

```js
o.el = null;
el.obj = null;
```

IE9 把 BOM 和 DOM 对象都改成了 JavaScript 对象，也避免了存在两套垃圾回收算法而导致的问题

## 提升性能

### const 和 let 提升性能

let 和 const 都以块（而非函数）为作用域，使用这两个关键字能更早让垃圾回收程序介入

### 隐藏类和删除操作

避免“先创建再补充” 式的动态属性赋值。在构造函数中一次声明所有属性。这样，两个实例基本就一样了，因此可以共享一个隐藏类，从而带来潜在的性能提升

- 动态删除属性与动态添加属性导致后果一样，最好的方法时把不想要的属性设置为 null，这样可以保持隐藏类不变和继续共享，同时也能达到删除引用值提供垃圾回收的效果

```js
function Person(name) {
    this.age = 12;
    this.name = name;
}
let a1 = new Person();
let a2 = new Person("lion");

// 尽量不要这么做
// 此时 Person 实例就会有两个不同的隐藏类，根据操作频率和隐藏类的大小，可能对性能产生影响
a2.name = "lion"
// 尽量不要这么做
delete a1.age;
```

### 避免内存泄露

**情况1：**

意外声明全局变量是最常见的内存泄露，解释器会把变量 name 当做 window 的属性来创建。在 window 对象上创建的属性，只要 window 本身不被清理就不会消失

```js
function say() {
    name = "lion";
}
```

在全局作用域声明 arr（fn() 里没传形参arr；在函数里没有声明 arr），arr 在全局作用域中赋值

```js
let arr = [];
function fn() {
    arr = [1, 2, 3];
}
fn();
```

**情况2：**

定时器也导致内存泄露，只要定时器一直运行，回调函数中引用的 name 就会一直占用内存

```js
let name = "lion";
setInterval(() => {
    console.log(name);
}, 100);
```

**情况3：**

调用 out() 会导致分配给 name 的内存被泄露，执行的时候创建一个内部闭包，只要返回的函数存在就不能清理 name，因为闭包一直在引用着它

```js
let out = function () {
    let name = "lion";
    return function () {
        return name;
    };
};
```

## 总结

解除变量的引用不仅可以减少消除循环引用，而且对垃圾回收也有帮助。为促进内存回收，全局对象、全局对象的属性和循环引用都应该在不需要时解除引用