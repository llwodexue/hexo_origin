---
title: JS this指向问题
tags:
  - JavaScript
  - 默认绑定
  - 隐式绑定
  - new绑定
  - 箭头函数
categories: JavaScript
author: LiLyn
copyright: ture
abbrlink: ea5004b1
---

在绝大多数情况下，函数的调用方式决定了 `this` 的值。`this` 不能在执行期间被赋值，并且在每次函数被调用时 `this` 的值也可能会不同，所以总结 `this` 值的规律是有必要的

<!--more-->

## 全局作用域 this 指向

无论是否在严格模式下，在**全局作用域中 `this` 都指向全局对象**

```js
console.log(this); // Window 

"use strict"
console.log(this); // Window 
```

## 私有作用域 this 指向

在函数内部，`this` 的值取决于函数调用的方式

通俗的来说，就是函数执行的时候，看函数名前面有没有 `.`：

- 如果函数调用时在某个对象上触发的（典型：`xxx.fun()`），会触发**隐式绑定**，**this 就是该对象**，即 `xxx`
- 如果函数调用时前面没有 `.` ，会触发**默认绑定**
  - 浏览器环境：Window
  - 严格模式：undefined

```js
function fn() {
    console.log(this);
}
var obj = {
    fn: fn,
};

obj.fn(); // obj
fn(); // Window

"use strict"
obj.fn(); // obj
fn(); // undefined
```

## 构造函数 this 指向（new绑定）

通过构造函数创建的实例，**构造函数中的 this 指的就是当前类的实例**

- new 运算符做了这样一件事：将新对象和函数调用的 this 绑定起来

```js
function fn() {
    this.name = "lion";
    this.age = 13;
    console.log(this); // fn
}
var f = new fn();

"use strict";
function fn() {
    this.name = "lion";
    this.age = 13;
    console.log(this); // fn
}
var f = new fn();
```

## 显示绑定 this 指向

**通过 `call` 、 `apply` 、 `bind` 的方式，显示的指定 this 所指向的值**

`call` 方法第一个参数是要绑定 this 的值，后面传入的是一个参数列表

- 浏览器环境：不传参或传 null、undefined，this 都是 Window
- 严格模式：不传参或传 undefined，this 都是 undefined，传 null，this 是 null

```js
function Fn(){
    console.log(this.name);
}
var person = {
    name: 'bird',
    Fn: Fn
}
var name = 'dog';
var Hi = function(fn) {
    // 触发默认绑定，this 为 winodw
    fn(); // dog
    // 如果不希望绑定丢失
    fn.call(this); // bird
}
Hi.call(person, person.Fn);
```

**call 硬绑定情况**

强制把 foo 的 this 绑定到了 obj，无论之后如何调用 bar 之前的操作都不会被覆盖，它总会在 obj 上调用 foo

```js
function foo() {
    console.log(this.a);
}
var a = 1;
var obj = {
    a: 2,
};
var bar = function () {
    foo.call(obj);
};
bar(); // 2
setTimeout(bar, 100); // 2
bar.call(window); // 2
```

## 自执行/回调函数 this 指向

其实和 私有作用域 this 指向 里的默认绑定是一个道理，**与函数定义地方无关，取决于函数在哪调用**

- 浏览器环境：Window
- 严格模式：undefined

```js
~(function () {
    console.log(this);
})(); // Window

"use strict";
~(function () {
    console.log(this);
})(); // undefined
```

回调函数：一个函数作为参数传递给另一函数

- 浏览器环境：Window
- 严格模式：Window

这里需要注意：**如果是 JS 内置方法，如 `setTimeout` ，回调函数内部 this 都会指向 Window**，非内置方法严格模式下为 undefined

```js
setTimeout(function () {
    console.log(this);
}, 100); // Window

"use strict";
setTimeout(function () {
    console.log(this);
}, 100); // Window
```

## 元素绑定事件 this 指向

给元素绑定事件，当事件触发，函数执行的时候，里面的 this 就是当前绑定的元素

- 浏览器环境：当前绑定事件的元素
- 严格模式：当前绑定事件的元素

```js
el.onclick = function () {
    console.log(this);
}; // el

"use strict";
el.onclick = function () {
    console.log(this);
}; // el
```

## 箭头函数 this

**箭头函数中没有 this**，也没有 this 绑定机制，还没有 arguments

```js
function fn1() {
    this.a = 55;
    return {
        a: 10,
        b: function () {
            return this;
        },
    };
}
var f1 = new fn1();
console.log(f1.b()); // {a: 10, b: ƒ}
```

**如果在箭头函数中用 this，this 仅仅相当于一个变量，会向上级作用域去找**

```js
function fn2() {
    this.a = 55;
    return {
        a: 10,
        b: () => {
            return this;
        },
    };
}

var f2 = new fn2();
console.log(f2.b()); // fn2 { a: 55 }
```

