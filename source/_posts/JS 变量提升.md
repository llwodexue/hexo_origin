---
title: JS 变量提升
tags:
  - JavaScript
  - 作用域
  - 变量提升特殊性
  - 前端
categories: JavaScript
author: LiLyn
copyright: ture
abbrlink: ea26882a
---

当浏览器开辟出供 js 执行的栈内存之后，代码并不是立即自上而下执行，而是需要先做一些事情：把当前作用域中带 var 和 function 的关键字进行提前的声明和定义（变量提升）

- var：只声明，未定义（不赋值）
- function：声明和定义（赋值）一起完成

<!--more-->

## 匿名函数具名化

匿名函数具名化【规范函数执行的顺序】

```js
var fn = function sum() {
  console.log('ok')
  sum = 100 // 默认情况下，其值是不能修改的
  console.log(sum) // 函数本身 // 具名话的名字可以在函数内部的上下文中使用【代表函数本身】

  console.log(sum) // undefined // 虽然具名化的名字会在当前函数的私有上下文中被声明，当做私有变量处理，但是它的优先级是比较低的
  var sum
}

sum() // 具名化设置的函数名不能在函数以外使用【因为没有在当前上下文中声明这个变量】
```

匿名函数具名化可以让匿名函数实现递归调用

```js
'use strict'
;(function anonymous(x) {
  if (x < 0) return
  anonymous(x - 1)
  // arguments.callee(x - 1) // 开启严格模式则会报错
})(100)
```



## 作用域中的变量提升

作用域链查找原则：首先会在当前作用域中查找，如果没有的话会沿着作用域链向上查找， 直至全局作用域

1. 在全局作用域获取不到，报错：` ... is not defined`
2. 如果是赋值语句，就相当于给全局作用域添加了这样一个属性名和属性值

### 全局变量

> 在全局作用域（默认会提供一个最大的 window 对象）中声明的变量

- 函数没有形参，且函数体里没有 var 声明

```js
var a = 6;
function fn() {
    console.log(a);
    a = 1;
}
fn();
console.log(a);
```

答案：

```js
// 6
// 1
```

### 私有变量

> 函数执行的时候形成的作用域是私有的，保护里面的变量不受外界干扰

- 形参

  函数有形参，但没有传实参，且打印在赋值之前

  ```js
  var a = 6;
  function fn(a) {
      console.log(a);
      a = 1;
  }
  fn();
  console.log(a);
  ```

  答案：

  ```js
  // undefined
  // 6
  ```

- 在私有作用域中声明的变量

  函数没有形参，函数体里有 var 声明
  
  ```js
  console.log(a, b);
  var a = 12,
      b = 12;
  function fn() {
      // 变量提升 var a；a 是私有的， b是 windows 全局作用域
      console.log(a, b);
      // 下面语句相当于 b=13; var a=b
      var a = b = 13;
      console.log(a, b);
  }
  fn();
console.log(a, b);
  ```
  
  答案：
  
  ```js
  // undefined undefined
  // undefined 12
  // 13 13
  // 12 13
  ```

- 复合（形参 + 私有作用域中声明变量）

  ```js
  /*
      私有作用域 fn(a)
      1.形参赋值 a=12; a是私有的
      2.变量提升 var b; b是私有的
          只有 c 是全局作用域的
      3.代码自上而下执行
  */
  var a = 12,
      b = 13,
      c = 14;
  function fn(a) {
      console.log(a, b, c);
      var b = c = a = 20;
      console.log(a, b, c);
  }
  fn(a);
  console.log(a, b, c);
  ```

  答案：

  ```js
  // 12 undefined 14
  // 20 20 20
  // 12 13 20
  ```

### 块级作用域

- let 声明的变量不进行变量提升不能重复声明

```js
let a = 10,
    b = 10;
let fn = function () {
    // a 变成私有变量，这个私有作用域没有 b，需要向上级查找
    let a = b = 20;
    console.log(a, b);
};
fn();
console.log(a, b);
```

答案：

```js
// 20 20
// 10 20
```

### 引用类型数据问题

```js
/*
    1.变量提升：fn -> 0x111(全局)；arr -> 0x222(全局)
    2.代码自上而下执行，执行 fn(arr)
        1.形参赋值 arr -> 0x222[12, 13]
        2.重新赋值 arr[0]=100 -> 0x222[100, 13]
        3.arr=[100] -> 0x333[100](函数fn中)
        4.arr[0]=0 -> 0x333[0]
*/
var arr = [12, 13];
function fn(arr) {
    console.log(arr);
    arr[0] = 100;
    arr = [100];
    arr[0] = 0;
    console.log(arr);
}
fn(arr);
console.log(arr);
```

答案：

```js
// [12,13]
// [0]
// [100,13]
```



## * 变量提升特殊性

### * 判断条件

> **不论判断条件是否成立，都会进行变量提升**

#### * 条件不成立

var：**只声明不定义**

```js
// 不管条件是否成立，都会进行变量提升，var a
console.log(a);
if (false) {
    // 条件不成立，无法赋值
    var a = 5;
}
console.log(a);
```

答案：

```js
// undefined
// undefined
```

function：

- 在老版本浏览器中：声明+定义

- 在新版本浏览器中：**只声明不定义**

  可以把它理解成函数表达式 `var fn = function(){}` ，只声明 `var fn` 

  `delete fn` 返回 false（`var 变量` 无法使用 delete 删除）

```js
// 在新版本浏览器中，判断条件中的function相当于只声明未定义，所以undefined
console.log(fn);
console.log(fn());
if (false) {
    function fn() {
        console.log("lion");
    }
}
console.log(fn);
```

答案：

```js
// undefined
// TypeError: fn is not a function
```

#### 条件成立

- **判断条件成立，会对执行体中的 fn 进行变量提升**（声明+赋值）

```js
console.log(fn);
if (true) {
    console.log(fn);
    // 全局作用域没有 fn ，给 fn 进行赋值
    function fn() {
        console.log("hello");
    }
}
console.log(fn);
```

答案：

```js
// undefined
// [Function: fn]
// [Function: fn]
```

#### 特殊情况

- **判断条件成立，如果有 function 定义的变量，在这个 function 函数后面更改变量的值，更改的都是私有变量**

  可以把`if(){} 的 ` `{ }` 理解成块级作用域（特例： function(){} 的 `{ }`是私有作用域）

```js
var a = 0;
if (true) {
    console.log(a);  // [Function: a]
    a = 5;
    function a() {}
    a = 7;
    console.log(a);  // 7
}
console.log(a);  // 5
```

### 自执行函数

- **自执行函数在当前作用域下不进行变量提升**

新版浏览器：

> 1. 在全局作用域中，没有变量提升
> 2. 代码自上而下执行，`window.f = function(){}` 和 `window.g = function(){}`
> 3. 进入自执行函数，走到 if 语句中，函数 g 声明提升，此时 g 只声明未定义，相当于 undefined，所以 g() 会报类型错误

```js
f = function () {
    return true;
};
g = function () {
    return false;
};
~(function () {
    if (g() && [] == ![]) {
        f = function () {
            return false;
        };
        function g() {
            return true;
        }
    }
})();
console.log(f());
console.log(g());
```

答案：

```js
// TypeError: g is not a function
```

老版浏览器：

> 1. 在全局作用域中，没有变量提升
>
> 2. 代码自上而下执行，`window.f = function(){}` 和 `window.g = function(){}`
>
> 3. 进入自执行函数，走到 if 语句中，函数 g 声明提升并定义，
>
>    g() 为 true，参考下图优先级顺序， `==` 优先级高于 `&&` 
>
>    `g() && [] ==  ![]` ，![] 转换为 false，再转换为0；[] 转换为 0；`[] == ![]` 返回 true
>
>    `true && true` 返回 true，进入循环
>
> 4. f 进行重新赋值，f 指向 `function(){return false}`，g 已经声明，不会再重复声明， g没有被修改
>
> 5. 答案：
>
>    ```js
>    // false
>    // false
>    ```

![JavaScript运算符优先级](https://gitee.com/lilyn/pic/raw/master/js-img/JavaScript%E8%BF%90%E7%AE%97%E7%AC%A6%E4%BC%98%E5%85%88%E7%BA%A7.jpg)

### 赋值运算

- **只对等号左边进行变量提升**（函数表达式）

```js
console.log(fn);
console.log(fn(1, 2));
var fn = function (n, m) {
    return n + m;
};
console.log(fn(3, 4));
```

答案：

```js
// undefined
// TypeError: fn is not a function
```

### return 语句

- return 下面的代码虽然不能执行，但是可以进行变量提升（f2 进行变量提升）
- return 后面的代码不能进行变量提升（f1 不进行变量提升）

```js
function fn() {
    console.log(f2);
    console.log(f1);
    return function f1() {};
    function f2() {
        console.log("bird");
    }
}
fn();
```

答案：

```js
// [Function: f2]
// ReferenceError: f1 is not defined
```

### 重复变量名

- var 不会进行重复声明，但会重新赋值

```js
var num = 4;
var num = 5;
console.log(num); // 5
```

- function 在变量提升阶段 声明和定义是一同完成的，如果遇到重复声明定义的，会重新进行赋值

```js
/*
1.变量提升：
    function fn = 0x111
                = 0x222
                = 0x333
                = 0x444
2.代码从上到下执行
 */
fn();
function fn() {
    console.log(1);
}
function fn() {
    console.log(2);
}
fn();
function fn() {
    console.log(3);
}
// fn=100 给fn重新赋值
fn = 100;
function fn() {
    console.log(4);
}
// function 声明和定义早已完成， 100()则会报错
fn();
```

答案：

```js
// 4
// 4
// TypeError: fn is not a function
```