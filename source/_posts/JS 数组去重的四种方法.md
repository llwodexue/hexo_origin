---
title: JS 数组去重的四种方法
tags:
  - JavaScript
  - 基础
  - 前端
  - 数组
categories: JavaScript
author: LiLyn
copyright: ture
abbrlink: 908d091d
---

### 总结

1. 双 for 循环去重
2. 借用对象属性不能相同特点去重
3. 新建数组去存储不同值（includes 与 indexOf 的区别）
4. ES6 Set去重

<!--more-->

### 方法一：双 for 循环

- 原理：依次拿出数组中的每一项，和它后面的所有剩余项进行比较，如果有相同的就删除
- NaN 与任何值都不相等，包括 NaN 本身
- `null==undefined(true)` 
- 对象和对象比较的是空间地址：`{}=={}(false)`

```js
var arr = [1, 2, 2, 2, {}, {}, NaN, NaN, null, null, undefined, undefined];
function unique1(arr) {
    for (var i = 0; i < arr.length - 1; i++) {
        var el1 = arr[i];
        for (var j = i + 1; j < arr.length; j++) {
            var el2 = arr[j];
            if (el1 == el2) {
                arr.splice(j, 1);
                j--;
            }
        }
    }
    return arr;
}
console.log(unique1(arr)); // [ 1, 2, {}, {}, NaN, null]
```

### 方法二：借用对象属性不能相同特点去重

- 原理：创建一个空对象，去遍历数组中的每一项，把数组中的每项当成属性名和属性值，给此对象添加，在添加的过程中，如果此对象已经有此项，说明重复，在数组中删除此项
- 访问对象的属性如果属性不存在返回 undefined，`obj[null]` 返回 undefined

```js
var arr = [1, 2, 2, 2, {}, {}, NaN, NaN, null, null, undefined, undefined];
function unique2(arr) {
    var obj = {};
    for (var i = 0; i < arr.length; i++) {
        var el = arr[i];
        // typeof (obj[el]) != "undefined"
        if (obj[el] == el) {
            arr.splice(i, 1);
            i--;
            continue;
        }
        obj[el] = el;
    }
    return arr;
}
console.log(unique2(arr)); // [ 1, 2, {}, {}, NaN, NaN ]
```

### 方法三：新建数组去存储不同值

- 原理：创建一个新数组，去遍历数组中的每一项，如果新数组没有这个（利用 indexOf） push 进新数组
- indexOf 比较参数与数组中的每一项时，会使用全等操作符（===）

```js
var arr = [1, 2, 2, 2, {}, {}, NaN, NaN, null, null, undefined, undefined];
function unique3(arr) {
    var newArr = [];
    for (var i = 0; i < arr.length; i++) {
        var el = arr[i];
        // newArr.indexOf(el) < 0 说明没有
        if (newArr.indexOf(el) == -1) {
            newArr.push(el);
        }
    }
    return newArr;
}
console.log(unique3(arr)); // [ 1, 2, {}, {}, NaN, NaN, null, undefined ]
```

- 原理：与上面那个同理
- includes 可以去重 NaN

```js
var arr = [1, 2, 2, 2, {}, {}, NaN, NaN, null, null, undefined, undefined];
function unique4(arr) {
    var newArr = [];
    for (var i = 0; i < arr.length; i++) {
        var el = arr[i];
        if (!newArr.includes(el)) {
            newArr.push(el);
        }
    }
    return newArr;
}
console.log(unique4(arr)); // [ 1, 2, {}, {}, NaN, null, undefined ]
```

**includes 与 indexOf 的区别**

- 如果数组里只有 NaN，利用 indexOf 是无法判断的，必须使用 includes 方法

```js
var arr = [NaN]
arr.indexOf(NaN)   // -1
arr.includes(NaN)  // true

var arr1 = [undefined]
arr1.indexOf(undefined)  // 0
arr1.includes(undefined) //true

var arr2 = new Array(1)
arr2.indexOf(undefined)  // -1
arr2.includes(undefined) // true
```

### 方法四：ES6 Set去重

- 这种方法无法去掉 "{}" 空对象

```js
var arr = [1, 2, 2, 2, {}, {}, NaN, NaN, null, null, undefined, undefined];
function unique5(arr) {
    return Array.from(new Set(arr));
}
console.log(unique5(arr)); // [ 1, 2, {}, {}, NaN, null, undefined ]
```
