---
title: JS 设计模式浅析
tags:
  - JavaScript
  - 设计模式
categories: JavaScript
author: LiLyn
copyright: ture
abbrlink: f356c3f6
---

> 设计模式：在面向对象软件设计过程中针对特定问题的简洁而优雅的解决方案

一般来说，设计模式（Pattern Design）就是**创建不同类型的对象的套路**

<!--more-->

常见设计模式：

- 单例设计模式：一个类仅有一个一个实例
- 工厂模式：批量生产出具有相似属性和方法的对象
- 构造函数模式：可以理解为是工厂模式的另一种写法
- 发布订阅模式：定义对象间的一对多的关系

## 单例模式

> 单例模式：保证一个类仅有一个实例，并提供一个访问它的全局访问点

简单来说：单例模式就是一个对象

### 极简写法

- 成员暴露出来，初始化占用资源

```js
var utils = {
    num: 6,
    method: function () {
        return this.num;
    },
};
var obj = utils;
console.log(obj.method());
```

### 闭包方式

- 解决成员暴露和初始化占用资源问题

```js
var utils = (function () {
    var num = 6;
    function fn() {
        return this.num;
    }
    return {
        num: num,
        method: fn,
    };
})();
var obj = utils;
console.log(obj.method());
```

应用：弹窗，无论点击多长次，弹窗只应该创建一次

## 工厂模式

> 工厂模式：像工厂一样批量生产相似属性和方法的对象。减少页面冗余代码，实现了“高耦合低内聚”

简单来说，工厂模式就是函数

```js
function Person(name, age) {
    return {
        name: name,
        age: age,
        say: function () {
            console.log(this.name);
        },
    };
}
var p = Person("lion", 13);
p.say();
```

## 构造函数模式

> 构造函数模式：利用构造函数的方式创建对象。函数执行是，前面一旦加了 new，就变成构造函数

```js
function Person(name, age) {
    this.name = name;
    this.age = age;
}
Person.prototype.sayName = function () {
    console.log(this.name);
};
var f = new Person("cat", 17);
f.sayName();
```

## 发布订阅

DOM 0级事件和DOM 2级事件区别：

```js
box.onclick = function(){};
box.addEventListener("click", function(){});
```

- DOM 0级就是给元素的某个属性绑定方法（有效绑定的方法只有一个）
- DOM 2级是基于事件池机制完成，每增加一个绑定的方法，都会往事件池中存放一个...当事件池触发会依次执行事件池的事情（发布订阅其实就是模拟事件池机制）

DOM 2级事件池机制：

1. 基于addEventListener/attachEvent（IE6~8）向事件池中追加方法 ：新版浏览器会根据元素和事件类型对新增的方法做重复校验，但是IE6~8不可以
2. 当事件行为触发，会把事件池中的方法按照增加的顺序依次执行，但是IE6~8中执行的顺序是不固定的

```js
let _subscribe = (function () {
    // Sub：发布订阅类
    class Sub {
        constructor() {
            // 创建一个事件池，用来存储后期需要的方法
            this.$pond = [];
        }
        // 向事件池中追加方法（重复校验）
        add(func) {
            let flag = this.$pond.some((item) => {
                return item === func;
            });
            !flag ? this.$pond.push(func) : null;
        }
        // 从事件池中移除方法
        remove(func) {
            this.$pond = this.$pond.filter((item) => {
                return item !== func;
            });
        }
        // 通知事件池中的方法，按照顺序依次执行
        fire(...args) {
            this.$pond.forEach((item) => {
                // 在三个参数以上，call性能略优于apply
                item.call(this, ...args);
            });
        }
    }
    return function subscribe() {
        return new Sub();
    };
})();
```

- 如果要使用for循环，需要考虑数组塌陷问题

```js
// 从事件池中移除方法
remove(func) {
    let $pond = this.$pond;
    for (let i = 0; i < $pond.length; i++) {
        let item = $pond[i];
        if (item === func) {
            $pond[i] = null;
            break;
        }
    }
}
// 通知事件池中的方法，按照顺序依次执行
fire(...args) {
    let $pond = this.$pond;
    for (let i = 0; i < $pond.length; i++) {
        let item = $pond[i];
        if (typeof item !== "function") {
            $pond.splice(i, 1);
            i--;
            continue;
        }
        item.call(this, ...args);
    }
}
```

