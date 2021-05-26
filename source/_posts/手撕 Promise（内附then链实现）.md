---
title: 手撕 Promise（内附then链实现）
tags:
  - JavaScript
  - Promise
  - then链
  - 源码分析
categories: JavaScript
author: LiLyn
copyright: ture
abbrlink: c7bac255
---

## Promise

### 了解  Promise

Promise 是一个异步操作返回的对象，用来传递异步操作的消息

**可以解决的问题**

1. 解决回调地狱问题，不会导致难以维护
2. 合并多个异步请求，节约时间

<!--more-->

**Promise 的三种状态**

1. Pending 等待态
2. Fulfilled 成功态
3. Rejected 失败态

### 使用 Promise

**Promise.resolve：将现有对象转为 Promise 对象，这个对象处于 resolve 状态**

作用：

- 改状态
- 发布事件池子里的方法

```js
Promise.resolve([1, 2, 3]).then((data) => {
    console.log(data);
}); // [1, 2, 3]
```



**Promise.reject：将现有对象转为 Promise 对象，这个对象处于 reject 状态**

作用：

- 改状态
- 发布事件池子里的方法

```js
Promise.reject([1, 2, 3]).then(null, (err) => {
    console.log(err);
}); // [1, 2, 3]
```



**Promise.then：用来指定 Promise 对象的状态改变时要执行的操作**

- then 是同步的，then 里的回调函数是异步的

**注意几点：**

- 如果当前 then 中回调函数的执行结果是一个 Promise 实例，那当前这个实例的状态会接管下一个 then 中回调函数的执行
- 如果当前的 then 中的回调函数执行结果不是一个 Promise 实例，那下一个then中会默认执行成功的回调函数
- 当 Promise 的回调函数执行的时候出现了错误，那当前的实例会变成失败态

```js
let p = new Promise(function (resolve, reject) {
    resolve("success");
    //reject('fail');
});

p.then((data) => {
        console.log("data：", data);
    },(err) => {
        console.log("err：", err);
    }
);
```



**Promise.all：接收一个数组，数组内是 Promise 实例。必须全部成功才表示成功**



**Promise.race：接收一个数组，数组内是 Promise 实例。最早返回的对象成功，就变成成功态；如果失败了，就变成失败态**



## 封装 Promise

Promise 是一个 ES6 的内置类

- 当 new Promise 的时候，会给当前实例增加状态（默认是 pending 等待态）和两个事件池（成功和失败的事件池）
- 还会给 Promise 的内部传递一个 executor 函数，此函数会立即执行，而且此函数会传递两个形参函数 resolve 和 reject，形参对应的实参函数在 Promise 内部，当执行任何一个函数的时候，都会做两件事
  1. 去改变当前 Promise 实例的状态（**改状态**）
  2. **发布对应池子里的事件**
- Promise 的类的原型上还有一个 then 方法，用来给当前的 Promise 实例的事件池子订阅方法，而且还会返回一个新的 Promise 实例

### 不考虑 then 链式调用

```js
class MyPromise {
    constructor(executor) {
        this.state = "pending";
        // Promise实例可以多次then，为了保证then中方法按顺序执行，这里把then中成功的回调和失败的回调存放到数组内
        this.fulfilledEvent = [];
        this.rejectedEvent = [];

        let resolve = (result) => {
            if (this.state !== "pending") return;
            this.state = "resolved";
            // resolve其实是一个微任务，这里宏任务替换一下
            setTimeout(() => {
                // 发布方法
                this.fulfilledEvent.forEach((item) => {
                    if (typeof item === "function") {
                        item(result);
                    }
                });
            }, 0);
        };
        let reject = (reason) => {
            if (this.state !== "pending") return;
            this.state = "rejected";
            setTimeout(() => {
                this.rejectedEvent.forEach((item) => {
                    if (typeof item === "function") {
                        item(reason);
                    }
                });
            }, 0);
        };
        try {
            executor(resolve, reject);
        } catch (e) {
            reject(e);
        }
    }
    then(onFulfilled, onRejected) {
        if (this.state === "resolved") {
            // 订阅方法
            this.fulfilledEvent.push((result) => {
                onFulfilled(result);
            });
        }
        if (this.state === "rejected") {
            this.rejectedEvent.push((reason) => {
                onRejected(reason);
            });
        }
    }
}
```

- 可以简单测试一下

```js
let p = new MyPromise((resolve, reject) => {
    resolve(100);
});

p.then((res) => {
        console.log("res: ", res);
    },(err) => {
        console.log("err: ", err);
    }
);
// data: 100
```

### then 链式调用

- 当我们连续调用 then 的时候（p1），如果当前 then 中的回调函数执行不返回一个 Promise 实例，那当前的 then（p1） 默认是成功态，然后去发布它的成功事件池子里的方法

- 如果当前 then 中的回调函数执行返回一个 Promise（p2） 实例，那这个 Promise（p2） 实例就会接管（p1）实例的状态，然后去影响（p1）事件池子里的方法发布

  原理：把（p1）的 resolve 和 reject 放到 （p2） 的对应的事件池子里，然后（p2）去发布自己池子里的的方法时，（p1）的 resolve 和 reject 就会执行，从而达到间接的让 （p2）去发布（p1）池子里的方法

```js
class MyPromise {
    constructor(executor) {
        this.state = "pending";
        this.fulfilledEvent = [];
        this.rejectedEvent = [];

        let resolve = (result) => {
            if (this.state !== "pending") return;
            this.state = "resolved";
            setTimeout(() => {
                this.fulfilledEvent.forEach((item) => {
                    if (typeof item === "function") {
                        item(result);
                    }
                });
            }, 0);
        };
        let reject = (reason) => {
            if (this.state !== "pending") return;
            this.state = "rejected";
            setTimeout(() => {
                this.rejectedEvent.forEach((item) => {
                    if (typeof item === "function") {
                        item(reason);
                    }
                });
            }, 0);
        };
        try {
            executor(resolve, reject);
        } catch (e) {
            reject(e);
        }
    }
    then(onFulfilled = () => {}, onRejected = () => {}) {
        return new MyPromise((resolve, reject) => {
            this.fulfilledEvent.push((result) => {
                let fn = onFulfilled(result);
                fn instanceof MyPromise
                    ? fn.then(resolve, reject)
                    : resolve(result);
            });
            this.rejectedEvent.push((reason) => {
                let fn = onRejected(reason);
                fn instanceof MyPromise
                    ? fn.then(resolve, reject)
                    : resolve(reason);
            });
        });
    }
}
```

- 可以简单测试一下

```js
let p1 = new MyPromise((resolve, reject) => {
    resolve(100);
});

let p2 = p1.then((res) => {
        console.log("p1 res: ", res);
        return new MyPromise((resolve, reject) => {
            resolve(200);
        });
    },(err) => {
        console.log("p1 err: ", err);
    }
);

p2.then(res) => {
        console.log("p2 res: ", res);
    },(err) => {
        console.log("p2 err: ", err);
    }
);
// p1 res:  100
// p2 res:  200
```