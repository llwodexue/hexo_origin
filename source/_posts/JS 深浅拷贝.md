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

### JS基本数据类型

- 基础数据类型按值进行访问的，可以操作保存在变量中的实际值
- 引用数据类型，不允许直接访问值，不能直接操作对象的内存空间，在操作对象时，实际操作的是引用

![JS数据类型](https://gitee.com/lilyn/pic/raw/master/js-img/JS%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.jpg)

<br>

### 存储方式

再看一下存储方式，结合深浅拷贝的定义就会理解一些了

- 基础类型存在栈中
- 引用类型同时保存在栈内存和堆内存

![存储方式](https://gitee.com/lilyn/pic/raw/master/js-img/JS%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E5%AD%98%E5%82%A8%E6%96%B9%E5%BC%8F.jpg)

<br>

### 深拷贝 和 浅拷贝

**浅拷贝方式**

1. 直接进行赋值

   赋值引用 a 和 b 都指向同一个对象

   ```js
   var a = [1, 2, 3];
   var b = a;
   ```

2. 如果拷贝的是普通对象

   - `Object.assign(target, source)`

   ES6 新增的对象方法，它可以实现第一层的“深拷贝”，但无法实现多层深拷贝

   对于 a 对象所有的属性和方法都进行了深拷贝，但是 a 对象的属性 data 是对象，它拷贝的是地址，也就是浅拷贝

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

<br>

**拷贝对象**

```js
var a = {
    name: "bird",
    data: {num:10}
};
var b = a;
b.name = "lion";
console.log(a.name); // lion

var c = {};
Object.assign(c, a);
c.name = "cat";
console.log(a.name); // lion，发现 a中的name并没有改变
c.data.num = 5;
console.log(c.data.num); // 5，说明 data 对象没有被深拷贝
```

**拷贝数组**

```js
// concat 不修改原数组，会浅拷贝原数组
var a = [1,2,{name:"bird"}];
var b = a.concat();
b[2].name = "cat";
console.log(a[2].name); // cat

// ES6扩展运算符
var [...c] = a;

// slice 不修改原数组，会浅拷贝原数组
var d = a.slice();
```

<br>

**深拷贝方式**

1. 递归赋值
2. `JSON.parse() 和 JSON.stringify`

<br>

**实例**

```js
var a = {
    name: "bird",
    data: {num:10},
    say: ()=>console.log("say")
};
var b = {};

// 递归赋值
function deepClone(obj) {
    let objClone = Array.isArray(obj) ? [] : {};
    if (obj && typeof obj === "object") {
        for (let key in obj) {
            // 判断是不是自有属性，而不是继承属性
            if (obj.hasOwnProperty(key)) {
                // 判断 obj 子元素是否为对象或数组，如果是，递归复制
                if (obj[key] && typeof obj[key] === "object") {
                    objClone[key] = this.deepClone(obj[key]);
                } else {
                    // 如果不是，简单复制
                    objClone[key] = obj[key];
                }
            }
        }
    }
    return objClone;
}
var c = deepClone(a);
c.name = 'cat';
console.log(a.name); // bird

// JSON.parse 和 JSON.stringify()
var d = JSON.parse(JSON.stringify(a));
d.name = "lion";
console.log(a.name); // bird
```

