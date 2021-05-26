---
title: JS中出现undefined和null情况
tags:
  - JavaScript
  - undefined
  - null
categories: JavaScript
author: LiLyn
copyright: ture
abbrlink: ea5004b1
---

undefined：未定义

null：空值

<!--more-->

## undefined

- 变量提升阶段，只声明未定义，返回 undefined
- 函数没设置返回值（return），返回 undefined
- 函数有形参但没传实参，返回 undefined
- 获取对象不存在的属性，返回 undefined
- typeof 一个不存在的变量，返回 undefined
- JS 严格模式，调用函数但函数前面没有`.`（排除显示绑定），this 是 undefined

```js
console.log(a); // undefined
var a = 1;

function fn1() {}
console.log(fn1()); // undefined

function fn2(name) {
    console.log(name); // undefined
}
fn2()

var obj = {};
console.log(obj.a); // undefined

console.log(typeof b); // undefined

"use strict"
function fn3() {
    console.log(this); // undefined
}
fn3()
```



## null

- 手动设置变量的值或某一个属性值为 null
- JS 获取 DOM 元素，如果没有获取到指定的元素对象，返回 null
- 正则捕获时，如果没有捕获到，返回 null
- `Object.prototype.__proto__` 的值是 null
- document 和 body 很多属性都是 null，这里就不列举了

```js
let a = null
console.log(a); // null

// 页面中没有#box的元素
document.getElementById("box") // null

"hello".match(/\d/g) // null
/\d/g.exec("hello")  // null

Object.prototype.__proto__ // null

document.body.offsetParent // null
document.parentNode        // null
```



