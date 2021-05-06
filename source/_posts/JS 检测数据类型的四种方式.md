---
title: JS 检测数据类型的四种方式
tags:
  - JavaScript
  - typeof
  - instanceof
  - constructor
  - toString.call
categories: JavaScript
author: LiLyn
copyright: ture
abbrlink: 5d2789f
---

JavaScript 有八种数据类型：

基本数据类型：Boolean、Number、String、null、undefined、Symbol(ES6新增)、BigInt（ES2020引入）

引用数据类型：Object

> Symbol：表示独一无二的值
>
> BigInt：用来解决 JavaScript 中数字只能到 53 个二进制位，大于这个范围的整数，无法精确表示

<!--more-->

### typeof

> 底层原理：typeof 是按照"值"在计算机存储的"二进制"值来检测的，凡是以000开始的都认为是对象

返回结果是一个字符串（全小写字母），可返回的类型有：

- "number"
- "string"
- "boolean"
- "undefined"
- "object"
- "function"
- "symbol"
- "bigint"

**注意：**`typeof null` 返回的是 "object"，`typeof 变量（不存在）` 返回的是 "undefined"

```js
typeof null;  // "object"
typeof say;   // "undefined"（上下文没有定义）
```

ECMAScript 提供的内置类型在计算机底层是按照二进制存储的

- 1：数字 010：浮点数
- 100：字符串
- -2^30：undefined
- 000000：null
- 000：对象

JS 最初为了性能考虑使用低位存储变量信息，000 开头代表对象，然而 null 表示全 0，所以将它错误的判断为 object

### instanceof

> 底层原理：首先查找 `Symbol.hasInstance`，如果存在，基于这个检测。如果没有，则基于原型链`__proto__`查找，只要出现这个类的原型，结果就是true

![dir(Function.prototype)](https://gitee.com/lilyn/pic/raw/master/js-img/dir(Function.prototype).jpg)

```js
class Fn {
    // 基于ES6 class方式构建Symbol.hasInstance才会生效
    static[Symbol.hasInstance]() {
        console.log("OK");
        return false;
    }
}
let f = new Fn;
console.log(f instanceof Fn); // OK false
```

**注意：**

- instanceof 不能正确判断基本数据类型
- 原型链可以重构，导致结果可能不准确

```js
var arr = [1, 2, 3];
arr instanceof Array;   // true
arr instanceof Object;  // true

null instanceof Object;  // false
123 instanceof Number;   // false
"" instanceof String;    // false
true instanceof Boolean; // false
```

#### 封装 instanceOf

```js
function instance_of(target, ctor) {
    let tType = typeof target,
        cType = typeof ctor;
    // 保证ctor是一个构造函数
    if (cType !== "function" || !ctor.prototype) throw new TypeError("ctor is not a constructor!");
    // 不处理原始值，排除null undefined 字面量
    if (target == null) return false;
    if (!/^(object|function)$/i.test(tType)) return false;
    // 优先检测 Symbol.hasInstance
    if (typeof ctor[Symbol.hasInstance] === "function") {
        return ctor[Symbol.hasInstance](target);
    }
    // 没有这个属性，再按照 ctor.prototype 是否出现在 example 的原型链上检测
    let prototype = Object.getPrototypeOf(target);
    while (prototype) {
        if (prototype == ctor.prototype) return true;
        prototype = Object.getPrototypeOf(prototype);
    }
    return false;
}

console.log(instance_of([], Array));  // true
console.log(instance_of([], RegExp)); // false
console.log(instance_of([], Object)); // true
```



### constructor

constructor 可以得知某个实例对象，到底是哪一个构造函数产生的

**注意：**constructor 可以手动更改（如果手动更改原型指向，检测就不准确了）；如果修改了原型对象，一般也会同时修改 constructor。null 和 undefined 是无效的对象，所以不存在 constructor 

```js
true.constructor === Boolean; // true
var a = {};
a.constructor; // Object()
a.constructor = 3;
a.constructor; // 3
```

### Object.prototype.toString.call()

> 底层原理：除了null/undefined，大部分数据类型所属类的原型上，都有toString方法；但是除了 Object.prototype.toString 用来检测数据类型，其余的都是转换为字符串的
>
> - 返回值："[object ?]"
>   - 先查找 [val] 的 `Symbol.toStringTag` （先找私有的，私有没有则向所属类原型上找），属性值就是"?"的值
>   - 没有，则内部是返回当前实例所属构造函数的名字 `“[object Number/String/Null/Undefined/Object/Array/Function/GeneratorFunction...]”`

```js
class Fn {}
Fn.prototype[Symbol.toStringTag] = 'Fn';
let f = new Fn;
console.log(Object.prototype.toString.call(f)); // [object Fn]
```



对于 Object，直接调用 toString 就可以返回 `"[object Object]"`。而对于其他对象，则需要通过 call/apply 来调用才能正确返回类型信息

```js
Object.prototype.toString.call(null); // "[object Null]"
```

我们可以封装一个 isType 方法对类型进行判断

```js
let isType = (type, obj) =>
    Object.prototype.toString.call(obj) === `[object ${type}]`;
console.log(isType("Number", 12)); // true
```



思考：每次都写 `Object.prototype.toString` 是否可以简写？

- 如果调用 toString 方法，obj 首先会在自己私有方法里找，如果没有则顺着原型链往上找，所以 prototype 可以省略

```js
var obj = {};
obj.toString.call(null); // "[object Null]"
```

不过上面还是不够简洁，可否把 obj 直接省去？

- 浏览器全局环境：window（是一个对象）
- node全局环境：global（是一个对象）

obj 省去的话，则默认是 window 调用 toString，我们来看一下 window 的原型链，最终指向 Object

![window原型](https://gitee.com/lilyn/pic/raw/master/js-img/window%E5%8E%9F%E5%9E%8B.jpg)

```js
toString.call(null); // 浏览器："[object Null]"
toString.call(null); // node：[object Null]
```

进一步验证一下，window 原型链上的 toString 是否和 Object.prototype 上的 toString 一致

```js
window.toString === Object.prototype.toString; // 浏览器：true
global.toString === Object.prototype.toString; // node：true
```

不过不推荐这么写，毕竟查找原型链还得花上一阵子时间
