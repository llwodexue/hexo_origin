---
title: JS 深浅拷贝
tags:
  - JavaScript
  - 深拷贝
  - 浅拷贝
  - ES6方法
  - 数组
categories: JavaScript
description: >-
  需要先了解一下JS基本数据类型和存储方式，之后进一步学习JS深浅拷贝
author: LiLyn
copyright: ture
abbrlink: 6362a5
---

## JS基本数据类型

- 基础数据类型按值进行访问的，可以操作保存在变量中的实际值
- 引用数据类型，不允许直接访问值，不能直接操作对象的内存空间，在操作对象时，实际操作的是引用

![JS数据类型](https://gitee.com/lilyn/pic/raw/master/js-img/JS%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.jpg)

<br>

## 存储方式

再看一下存储方式，结合深浅拷贝的定义就会理解一些了

- 基础类型存在栈中
- 引用类型同时保存在栈内存和堆内存

![存储方式](https://gitee.com/lilyn/pic/raw/master/js-img/JS%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E5%AD%98%E5%82%A8%E6%96%B9%E5%BC%8F.jpg)

<br>

## 深拷贝和浅拷贝

### 浅拷贝

1. 直接进行赋值

   赋值引用 a 和 b 都指向同一个对象

   ```js
   var a = [1, 2, 3];
   var b = a;
   ```

2. 如果拷贝的是普通对象

   `Object.assign(target, source)`

   ES6 新增的对象方法，它可以实现第一层的“深拷贝”，但无法实现多层深拷贝

   ```js
   var a = { name: "lion", age: 6 }
   var c = {};
   Object.assign(c, a);
   ```

3. 如果拷贝的是数组

   - `Array.prototype.concat()`
   - ES6 扩展运算符
   - `Array.prototype.slice()`

   ```js
   var a = [1, 2, 3];
   var b = a.concat();
   var [...c] = a;
   var d = a.slice();
   ```
   
4. 没有递归的 for 循环

   ```js
   let obj = {
     name: 'bird',
     age: 12,
     arr: [10, 20, 30]
   }
   
   let obj2 = {}
   for (let key in obj) {
     if (!obj.hasOwnProperty(key)) break
     obj2[key] = obj[key]
   }
   ```

### 深拷贝

1. `JSON.parse() 和 JSON.stringify`

   只能转换 JSON 支持的数据类型，可以参考 [JSON 数据类型](https://www.json.org/json-en.html)

   - `bigint` 不能直接转换为字符串
   - 对于 `symbol/undefined/function` 转换为字符串的时候就丢失了
   - 正则对象、Error 错误对象会变为空对象，Date日期对象会变为字符串
   - 无法处理 "套娃" 操作

   ```js
   let obj = {
     // num: 10n, // Uncaught TypeError: Do not know how to serialize a BigInt
     sy: Symbol('AA'),
     un: undefined,
     fn: function() {},
     reg: /\d+/,
     err: new Error('error'),
     time: new Date(),
   }
   // obj.obj = obj // Uncaught TypeError: Converting circular structure to JSON
   
   let obj2 = JSON.parse(JSON.stringify(obj))
   console.log(obj2)
   // { err: {}, reg: {}, time: "2020-07-10T01:43:11.344Z" }
   ```

   只能处理数字、字符串、布尔、null、object（普通对象/数组对象）...

2. 递归赋值

   Lodash/underscore 类库有 clone/deepClone

   如下方法写是不行的，比如正则、错误对象、日期对象会变为普通对象

   ```js
   let obj = {
     name: 'bird',
     age: 12,
     arr: [10, 20, 30],
     child: {
       chinese: 90,
       math: 100,
       english: 80,
     },
     num: 10n,
     sy: Symbol('AA'),
     un: undefined,
     ul: null,
     fn: function() {},
     reg: /\d+/,
     err: new Error('error'),
     time: new Date(),
   }
   
   function deepClone(obj) {
     let objClone = Array.isArray(obj) ? [] : {}
     if (obj && typeof obj === 'object') {
       for (let key in obj) {
         if (!obj.hasOwnProperty(key)) return
         if (obj[key] && typeof obj[key] === 'object') {
           objClone[key] = this.deepClone(obj[key])
         } else {
           objClone[key] = obj[key]
         }
       }
     }
     return objClone
   }
   
   let obj2 = deepClone(obj)
   console.log(obj2)
   ```

   

