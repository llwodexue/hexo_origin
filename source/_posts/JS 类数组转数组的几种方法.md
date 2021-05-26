---
title: JS 类数组转数组的四种方法
tags:
  - JavaScript
  - 数组
categories: JavaScript
author: LiLyn
copyright: ture
abbrlink: f0469b23
---

### 总结

1. for 循环
2. [].slice.call() （calll方法：[].map.call()）
3. ES6语法：Array.from()
4. ES6语法：展开运算符

<!--more-->

原生 js 获取 DOM 元素集合是一个类数组对象，所以不能直接利用数组对象（比如：sort、forEach），需要转换为数组后，才可以使用

下述方法使用 arguments 当伪数组

```js
function fn() {
    var res = toArr(arguments);
    console.log(res);
}
fn("A", 3, 5); // 返回 [ 'A', 3, 5 ]
```

### 1.for 循环

```js
function toArr(cArr) {
    var arr = [];
    for (var i = 0; i < cArr.length; i++) {
        arr.push(cArr[i]);
    }
    return arr;
}
```

### 2.[].slice.call() `call() 方法`

```js
function toArr(cArr) {
    return [].slice.call(cArr);
}
```

- `arr.slice(start, end)` ：提取索引 start 复制到索引 end 的部分（不包括 end
- `Array.prototype.slice.call(arguments, 0)`，就是把 `arguments` 当做当前对象，要调用 `arguments` 的 `slice` 方法，`slice(0)`  获取所有项（不包含索引）
- [] 是一个数组对象，会去原型链上找 `slice` 这个方法，所以可以简写成 `[].slice.call()`

`call() ` 方法可以搭配的比较广，这里再举个例子：可以用 map 方法，返回遍历的 item 即可

```js
function toArr(cArr) {
    return [].map.call(cArr, (item) => item);
}
```

### 3.ES6语法：Array.from()

只要有 length 属性的对象，都可以用此方法转换成数组

```js
function toArr(cArr) {
    return Array.from(cArr);
}
```

### 4.ES6语法：展开运算符

```js
function toArr(cArr) {
    return [...cArr];
}
```

