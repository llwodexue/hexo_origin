---
title: 实现JS一些重要的API
tags:
  - 方法实现
  - new
  - instanceof
categories: JavaScript
description: >-
  介绍new、Object.create、instanceof、forEach、call、bind、promise
author: LiLyn
copyright: ture
abbrlink: e8c036a5
---
## 实现 new

new 原理：

1. 创建Ctor的一个实例对象

   `实例.__proto__ = Ctor.prototype`

2. 把构造函数当做普通函数执行（让方法中的 this -> 实例对象）

3. 确认方法的返回值（如果没有返回值或者返回的是原始值，我们让其默认返回实例对象即可）

需要考虑的点：

- Symbol、BigInt 不能被 new

  我们知道 instanceof 是不能检测基本数据类型的 `Symbol() instanceof Symbol` 是 false。不过可以利用 call 方法来强迫装箱：当然这个也是 Function 和 Object 的实例

  注意：使用 `Object.prototype.toString.call(f)` 也是 symbol，因为会先去找 `Symbol.toStringTag` 这个属性（返回的是 `'[object Symbol]'`），所以需要结合 typeof 来区分（进行了装箱操作把基本数据类型转换为对应的引用数据类型）

  ```js
  let f = function () { return this }.call(Symbol(''))
  f instanceof Symbol // true
  
  Object.prototype.toString.call(f) // '[object Symbol]'
  typeof f // 'object'
  ```

  生成器函数不需要基于 new 执行，就可以创造函数实例。其实相当于内部做了 `f.__proto__ = fn.prototype` 的处理

  ```js
  function* fn() {}
  let f = fn()
  f instanceof fn // true
  ```

- 箭头函数不能被 new

  箭头函数没有 this，this 继承的是外层代码块的 this（也不能用 call 方法改变 this）
  
  箭头函数不能使用 arguments，不过可以使用 rest 运算符
  
  箭头函数不能使用 yield，箭头函数不能用作 Generator 函数

```js
/*
Ctor -> constructor缩写（构造函数）
params -> 后期给Ctor传递的所有实参信息
*/
function _new(Ctor, ...params) {
  let obj,
    result,
    proto = Ctor.prototype
  // 校验规则
  if (Ctor === Symbol || Ctor === BigInt || typeof Ctor !== 'function' || !proto)
    throw new TypeError(`${Ctor} is not a constructor`)
  obj = Object.create(Ctor.prototype)
  result = Ctor.call(obj, ...params)
  if (/^(object|function)$/.test(typeof result)) return result
  return obj
}

function Dog(name) {
  this.name = name
}
Dog.prototype.bark = function () {
  console.log('wangwang')
}
let bear = _new(Dog, '比尔')
bear.bark() // wangwang
console.log(bear instanceof Dog) // true
```

## 实现 Object.create

`Object.create([obj])`：创建一个空对象，并让 `空对象.__proto__` 指向 `[obj]`

需要考虑的点：

- `[obj]` 可以是一个对象或者是 null，但不能是其他的值
- `Object.create(null)` 创建一个不具备 `__proto__` 属性的对象（不是任何类的实例）

```js
Object.create = function create(prototype) {
  if (prototype !== null || typeof prototype !== 'object') throw new TypeError('Object prototype may only be')
  function Proxy() {}
  Proxy.prototype = prototype
  return new Proxy()
}
// 如果传null是无法删除 obj.__proto__ 的
```

## 实现 instanceof

原理：查找 `[构造函数][Symbol.hasInstance](实例)` 

- 如果存在这个属性方法，则方法执行返回的值就是最后检测的结果
- 如果不存在这个属性方法，则会查找当前实例的原型链（一直找到 Object.prototype 为止），如果查找中，找到某个原型等于构造函数的原型，则返回true，反之false

**注意：** 基于 ES6 设置静态私有属性 `static [Symbol.hasInstance](value) { return true }` 是有效的

获取原型链需要考虑的点：

- `Object.getPrototypeOf()` 不能获取 `null/undefined` 的原型

instanceof 需要考虑的点：

- 检测的所属类型（instanceof 右侧的所属类型）不能是箭头函数及非函数（基本数据类型的值、`{}` ...）

```js
function instance_of(obj, Ctor) {
  // 数据格式准确性校验
  if (Ctor == null) throw new TypeError("Right-hand side of 'instanceof' is not callable")
  if (!Ctor.prototype) throw new TypeError("Function has non-object prototype 'undefined' in instanceof check")
  if (typeof Ctor !== 'function') throw new TypeError("Right-hand side of 'instanceof' is not callable")

  // 原始值类型直接忽略
  if (obj === null) return false
  if (!/^(object|function)$/.test(typeof obj)) return false

  // 先检测是否有Symbol.hasInstance这个属性
  if (typeof Ctor[Symbol.hasInstance] === 'function') return Ctor[Symbol.hasInstance](obj)

  // 最后才会按照原型链进行处理
  let prototype = Object.getPrototypeOf(obj)
  while (prototype) {
    if (prototype === Ctor.prototype) return true
    prototype = Object.getPrototypeOf(prototype)
  }
  return false
}

console.log(instance_of([12, 23], Array)) // true
// console.log(instance_of(null, null))
// console.log(instance_of(null, () => {}))
// console.log(instance_of(null, {}))
```

## 实现 forEach 方法（支持对象遍历）

需要考虑的点：

- Symbol 无法用 `for...in...` 遍历的，可以使用 `Object.getOwnPropertySymbols()` 获取，之后结合 for 循环或 `Object.keys()` 使用即可

```js
function each(obj, callback) {
  let keys = Object.keys(obj),
    key = null,
    value = null,
    i = 0,
    len = 0
  if (typeof Symbol !== 'undefined') {
    // 支持Symbol
    keys = keys.concat(Object.getOwnPropertySymbols(obj))
  }
  len = keys.length
  if (typeof callback !== 'function') callback = function () {}
  for (; i < len; i++) {
    key = keys[i]
    value = obj[key]
    callback(value, key)
  }
}
```

其它常用数组方法的封装可以看我这篇文章：[JS 常用数组方法封装（包含splice）](https://editor.csdn.net/md/?articleId=111084399)

## 实现 call bind

需要考虑的点：

- 临时设置的属性，不能和原始对象冲突，所以属性采用唯一值处理

  使用 `Symbol` 或 生成随机字符串 `Math.random() * new Date()`

- 如果 context 不是对象（基本数据类型的值），需要将其处理成对象。如果 context 是 null 则需要单独处理

```js
function call(context, ...params) {
  /*
  this -> 要执行的函数
  context -> 要改变函数中this的指向
  params -> 未来要传递给函数func的实参信息
  */
  context == null ? (context = window) : null
  if (!/^(object|function)$/.test(typeof context)) context = Object(context)
  let self = this,
    key = Symbol('KEY'),
    result
  context[key] = self
  result = context[key](...params)
  delete context[key]
  return result
}
Function.prototype._call = call

let obj = { name: 'Bird' }
function func(x, y) {
  this.total = x + y
  return this
}
let res = func._call(obj, 1, 2) // {name:'Bird', total:3}
```

bind 不是立即把函数执行，只是预先把 this 和后期需要传递的参数存储起来（柯理化函数）

```js
function bind(context, ...params) {
  /*
  this -> func
  context -> obj
  params -> [100,200]
  */
  context == null ? (context = window) : null
  if (!/^(object|function)$/.test(typeof context)) context = Object(context)
  let self = this,
    key = Symbol('KEY'),
    result
  return function proxy(...args) {
    // args -> 事件触发传递的信息，例如：ev
    params = params.concat(args)
    context[key] = self
    result = context[key](...params)
    delete context[key]
    return result
  }
}
Function.prototype._bind = bind

let obj = { name: 'Jhon' }
document.body.onclick = func._bind(obj, 1, 2)
```

## 实现 Promise

先看一下 Promise 具有的方法：

![](https://gitee.com/lilyn/pic/raw/master/js-img/20210910105320.png)

Promise 原理：

1. 首先会给当前实例增加状态（默认是 pending 等待态）和两个事件池 成功的事件池和失败的事件池（用来存放 then 中还不知道实例状态的事件）

2. 其次最重要的就是 Promise 内部传递的 executor 函数，此函数会立即执行并会传递两个回调函数 resolve 和 reject，当执行任何一个回调函数时，都会做两件事情：**更改状态、发布对应事件池的状态方法** 

   注意：更改完状态就不能再更改了（是 pending 才会改状态）；执行回调函数时如果基于 then 存储过方法，依次通知对应状态的事件池方法执行，且执行是异步的微任务

3. new Promise 会创造实例，实例可以调用其所属类原型的方法，之后重构了一下 Promise 构造函数原型上的方法

4. then 方法返回一个全新的 Promise 实例，有两种情况：

   - 如果已经知道对应实例的状态，则创建一个异步的微任务，后期执行对应的 onFulfilled/onRejected
   - 如果此时还不知道实例状态，就先把 onFulfilled/onRejected 存储起来，后期更改其状态之后，再通知方法执行即可，也是异步微任务

   注意：返回新的 Promise 实例不能和执行结果是同一个 `let p2 = p1.then(() => p2)` 

   如何验证返回的是否是一个新的 Promise 实例：类型是 object 或 function 且它得具有 then 方法（是一个函数）。如果返回的不是 Promise 实例，且执行没有报错，则返回一个成功的 Promise 实例

5. 如果 then 中的某个回调函数（resolve、reject）不设置，其默认具备穿透性，顺延到下一个 then 中的同状态的方法上

   如果没有传 onFulfilled 或 onRejected，就给对应的默认传递 resolve 或 reject 即可

6. catch 相当于调用 then，第一个参数不写，第二个参数传递错误原因

7. all 传入的必须是一个数组，如果数组中某一项不是 Promise 实例，需要把它转换为成功的 Promise 实例。之后把每一项依次执行，只要有一项失败返回就是失败的，必须全部成功才是成功

   注意：返回结果的顺序跟传入的顺序一致（不能使用 push，因为不能确定谁先到，需要使用索引）

需要注意的点：

- Promise 是一个构造函数，需要 new 执行
- Promise 参数必须是一个函数
- Promise then 里的方法实际上是异步的微任务，可以基于 queueMicrotask 去创造异步的微任务（兼容性差）；这里采用基于定时器创造一个异步的宏任务，来模拟微任务 

ES6 方法实现 Promise，可以参考我另外一篇文章 [手撕 Promise（内附then链实现）](https://blog.csdn.net/qq_38689395/article/details/114014869)

```js
;(function () {
  function Promise(executor) {
    var self = this,
      change
    // 必须new执行
    if (!(self instanceof Promise)) throw new TypeError('undefined is not a promise')
    // executor必须是一个函数
    if (typeof executor !== 'function') throw new TypeError('Promise resolver ' + executor + ' is not a function')

    // 实例具备的属性，状态和结果
    self.state = 'pending'
    self.result = undefined
    self.onFulfilledCallbacks = []
    self.onRejectedCallbacks = []

    // 修改实例的状态
    change = function change(state, result) {
      if (self.state !== 'pending') return
      self.state = state
      self.result = result
      // 创建异步微任务，通知集合中存储的方法执行（前提：集合中有东西才执行）
      var callbacks = state === 'fulfilled' ? self.onFulfilledCallbacks : self.onRejectedCallbacks,
        i = 0,
        len = callbacks.length,
        callback
      if (callbacks.length > 0) {
        setTimeout(function () {
          for (; i < len; i++) {
            callback = callbacks[i]
            if (typeof callback === 'function') callback(result)
          }
        }, 0)
      }
    }

    // new Promise的时候会立即执行executor函数
    try {
      executor(
        function resolve(result) {
          change('fulfilled', result)
        },
        function reject(reason) {
          change('rejected', reason)
        }
      )
    } catch (error) {
      // 执行executor报错，实例也是失败的状态
      change('rejected', error)
    }
  }

  // 验证是否为Promise
  function isPromise(x) {
    if (x == null) return false
    if (/^(object|function)$/i.test(typeof x)) {
      if (typeof x.then === 'function') {
        return true
      }
    }
    return false
  }

  // 处理onFulfilled/onRejected方法执行返回结果的处理
  function handle(promiseNew, x, resolve, reject) {
    if (x === promiseNew) throw new TypeError('TypeError: Chaining cycle detected for promise #<Promise>')
    if (isPromise(x)) {
      try {
        x.then(resolve, reject)
      } catch (err) {
        reject(err)
      }
      return
    }
    // 返回不是promise实例，而且执行没有报错，则promiseNew一定是成功的，x是它的结果
    resolve(x)
  }

  Promise.prototype = {
    constructor: Promise,
    self: true,
    then: function (onFulfilled, onRejected) {
      var self = this,
        promiseNew,
        x
      if (typeof onFulfilled !== 'function') {
        onFulfilled = function onFulfilled(result) {
          return result
        }
      }
      if (typeof onRejected !== 'function') {
        onRejected = function onRejected(reason) {
          throw reason
        }
      }
      promiseNew = new Promise(function (resolve, reject) {
        // resolve reject可以设置返回的@TP（新promise实例是成功还是失败以及结果等）
        // 但是执行哪个方法，由监听onFulfilled/onRejected方法报错以及返回值来决定
        switch (self.state) {
          case 'fulfilled':
            setTimeout(function () {
              try {
                x = onFulfilled(self.result)
                handle(promiseNew, x, resolve, reject)
              } catch (err) {
                reject(err)
              }
            }, 0)
            break
          case 'rejected':
            setTimeout(function () {
              try {
                x = onRejected(self.result)
                handle(promiseNew, x, resolve, reject)
              } catch (err) {
                reject(err)
              }
            }, 0)
            break
          default:
            self.onFulfilledCallbacks.push(function (result) {
              try {
                x = onFulfilled(result)
                handle(promiseNew, x, resolve, reject)
              } catch (err) {
                reject(err)
              }
            })
            self.onRejectedCallbacks.push(function (reason) {
              try {
                x = onRejected(reason)
                handle(promiseNew, x, resolve, reject)
              } catch (err) {
                reject(err)
              }
            })
        }
      })
      return promiseNew
    },
    catch: function (onRejected) {
      return this.then(null, onRejected)
    },
  }
  if (typeof Symbol !== 'undefined') Promise.prototype[Symbol.toStringTag] = 'Promise'

  // 普通对象：私有静态方法
  Promise.all = function all(promises) {
    var promiseNew,
      results = [],
      n = 0
    if (!Array.isArray(promises)) throw new TypeError(promises + ' is not iterable')
    // 控制集合中的每一项都是Promise实例
    promises = promises.map(function (promise) {
      if (!isPromise(promise)) return Promise.resolve(promise)
      return promise
    })
    promiseNew = new Promise(function (resolve, reject) {
      promises.forEach(function (promise, index) {
        promise
          .then(function (result) {
            // result存储的是当前迭代这一项的成功结果
            n++
            results[index] = result
            // 都处理成功
            if (n >= promises.length) resolve(results)
          })
          .catch(function (reason) {
            // 只要有一项失败，整体就是失败
            reject(reason)
          })
      })
    })
    return promiseNew
  }
  Promise.resolve = function resolve(result) {
    return new Promise(function (resolve) {
      resolve(result)
    })
  }
  Promise.reject = function reject(reason) {
    return new Promise(function (_, reject) {
      reject(reason)
    })
  }

  /* 暴露API */
  if (typeof window !== 'undefined') window.Promise = Promise
  // if (typeof module === 'object' && typeof module.exports === 'object') module.exports = Promise
})()

let p1 = new Promise(function (resolve, reject) {
  Math.random() < 0.6 ? resolve('OK') : reject('NO')
})
// try catch 一般情况下是无法捕捉到异步任务执行报错信息
let p2 = p1.then(function (result) {
  console.log('成功', result)
  return 200
})

let p3 = p2.then(
  function (result) {
    console.log('成功', result)
  },
  function (reason) {
    console.log('失败', reason)
  }
)

let p4 = Promise.resolve(10)
let p5 = new Promise(resolve => {
  setTimeout(() => {
    resolve(20)
  }, 1000)
})
let p6 = 30
Promise.all([p4, p5, p6])
  .then(results => {
    console.log('成功', results)
  })
  .catch(reason => {
    console.log('失败', reason)
  })
```

## 接下来参考

[关于JS中一些重要的api实现, 巩固你的原生JS功底](https://juejin.cn/post/6844903924520992782)

