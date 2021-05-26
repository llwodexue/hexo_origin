---
title: JS 不同数据类型的相互转换规则
tags:
  - JavaScript
  - 数据类型转换
  - 题解
categories: JavaScript
author: LiLyn
copyright: ture
abbrlink: 8a7b4226
---

**对象和对象进行比较的时候：比较的是空间地址**，如果空间地址相同就是 true，不同就是 false

- `{}=={}(false)`

不同的数据类型再进行比较的时候，**除了以下的，剩下的都是先转换为数字在比较**：

- 对象和字符串进行比较的时候，把对象转换为字符串，再进行比较

- null 和 undefined 永远不等于任何一种数据类型，但是 `null==undefined(true)` `null===undefined(false)`

- **NaN 永远不等于任何一种数据类型，包括它自己**

  可以使用 `Object.is(NaN, NaN)->true` 检测

<!--more-->

```js
![] == true       // false ![]=>false转为布尔取反
![] == []         // true  ![]=>false=>Number(false)=>0 Number([].toString())=>0
[] = false        // true
![] == false      // true
NaN == NaN        // false
null == undefined // true
```



## 对象数据转换规则

> 原始值：number string boolean null undefined symbol bigint

**对象数据转换规则：**

1. 首先检测对象 `Symbol.toPrimitive` 这个属性，获取其原始值
2. 如果没有这个属性，继续调用它的 `valueOf`，也是获取原始值
3. 如果值不是原始值，则继续调用 `toString` 转换字符串
4. 如果需要转换数字，再把字符串基于 Number 转换为数字

```js
let obj = {
    name: "lion",
};
console.log(obj - 10);
obj[Symbol.toPrimitive]; // undefined
obj.valueOf(); // {name: "lion"} 不是原始值
obj.toString(); //"[object Object]"
Number("[object Object]"); // NaN
NaN - 10; // NaN
```

- 了解数据转换规则后，来看一道题

```js
var a = ?
if (a == 1 && a == 2 && a == 3) {
    console.log("OK");
}
```

- 第一类方法：隐式进行数据类型转换

  修改 `Symbol.toPrimitive`、 `valueOf`、  `toString`任意一个即可 

```js
// 通过 ++i 改值
var a = {
    i: 0,
    [Symbol.toPrimitive]: function () {
        return ++this.i;
    },
};
if (a == 1 && a == 2 && a == 3) {
    console.log("OK");
}

// 通过调用函数，删除值
var a = [1, 2, 3];
a.valueOf = a.shift;
if (a == 1 && a == 2 && a == 3) {
    console.log("OK");
}
```

- 第二类方法：ES6 数据劫持

  用 `defineProperty` 定义

  - 第一个参数是对象
  - 第二个参数是属性名
  - 第三个参数是特征值

```js
var i = 0;
Object.defineProperty(global, "a", {
    get() {
        return ++i;
    },
});
if (a == 1 && a == 2 && a == 3) {
    console.log("OK");
}
```



## 其他类型转换为数字

### Number

一般用于隐式转换【数学运算、isNaN、==比较...】

- 字符串：
  - 如果是空字符穿，转换结果 0
  - 如果说字符串中包含非有效数字，转换结果就是 NaN
- 布尔类型
  - true：转换为1
  - false：转换为0
- **null：转换为0**
- **undefined：NaN**
- Symbol：报错
- BigInt ：正常转换
- 对象：**遵循对象数据转换规则**

```js
Number("123px")     // NaN
Number(undefined)   // NaN
Number(false)       // 0
Number(true)        // 1
Number(null)        // 0
Number("")          // 0

Number([])          // 0  Number([]) ==> [].toString() ==> "" ==> Number("") ==> 0
```

### parseInt

`parseInt([val], [radix])`

- [val] **必须是一个字符串，如果不是，也要默认转换为字符串**
- [radix] 不设置（写零）：按十进制处理，如果字符串以`0x`开头，默认是16进制

**从左往右查找 [val] 中，找到所有符合 [radix] 进制的内容，直到遇到不符合的停止查找**（不论后面是否还有符合的）

```js
parseInt('12px')    // 12
parseInt('12px', 1) // NaN

Number(null) //0
parseInt(null))     // NaN ->parseInt('null', 10)
Number('') //0
parseInt('') //NaN
```

- radix 2-36 之间

```js
let arr = [27.2, 0, "0013", "14px", 123];
arr = arr.map(parseInt);
console.log(arr); // [ 27, NaN, 1, 1, 27 ]
```



## 其他类型转换为字符串

- `toString()`
  **null、undefined 没有 `toString()` 这个方法**，用了会报错，可以使用 `String()`

  需要排除 `Object.prototype.toString` 检测数据类型
  
- 字符串/模板字符串拼接（`+` 除了数学运算还有字符串拼接）

### + 字符串拼接

**情况1：**`+` 左右两边有一边出现字符串或部分对象，则按照字符串拼接处理

- 特殊：`{} + 10 -> 10` {} 看做代码块（ES6块级上下文）

**注意：不是所有对象都是字符串拼接**

- 先去调取对象的 `Symbol.toPrimitive` 属性值，如果没有这个属性
- 再去调取对象的 `valueOf` 获取原始值，如果不是原始值
- 再去调用对象的 `toString` 转换为字符串（如果是想转换为数字，则还会调用 Number 处理）

```js
10 + "10" // "1010"
10 + [10] // 1010; [10].toString() = '10'
10 + new Number(10) // 20; new Number(10).valueOf() = 10
{} + 10   // 10
10 + {}   // "10[object Object]"; {}.toString() = [object Object]
({} + 10) // "[object Object]10"

let obj = {
  x: 10,
  // obj[Symbol.toPrimitive valueOf toString]
  [Symbol.toPrimitive](hint) {
    // console.log(hint) // default string number
    return this.x
  },
}
console.log(10 + obj) // 20
```

**情况2：**只有一边有 `+` 

- `+` 只有一边或`++x`或`x++` ，都是数学运算

  `10+(++x) `先把x累加1，然后和10运算

  `10+(x++)` 先把x的值和10运算，然后x累加1

- 注意：`x++ `只会进行数学运算，`x+=1` 和 `x=x+1` 不仅会进行数学运算还可能进行字符串拼接

```js
+"10" // 10
var x = "10"
10+(x++) // 20

var x = "10"
10+(++x) // 21

var x = "10"
x+=1    // "101"
```

- 了解数据转换规则和字符串拼接后，来看一道题

```js
let result = 100 + true + 21.2 + null + undefined + "Tencent" + [] + null + 9 + false;
console.log(result); // NaNTencentnull9false
```



## 其他类型转换为布尔

- ![值]  转换为布尔并取反

- !![值]  转换为布尔

**除了以下几种结果都是false，剩余的都是true**

- NaN
- 0
- ""
- null
- undefined
