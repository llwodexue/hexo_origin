---
title: Vue2 面试题
tags:
  - Vue
  - diff算法
  - 响应式原理
  - nextTick
categories: Vue
author: LiLyn
copyright: ture
---

## v-show 和 v-if 区别

- `v-show` 通过 CSS display 控制显示和隐藏
- `v-if` 通过判断组件真实渲染和销毁，而不是显示和隐藏
- 频繁切换显示状态用 `v-show`，否则用 `v-if`

`v-if` 

- 当 `v-if` 与 `v-for` 一起使用时，`v-for` 具有比 `v-if` 更高的优先级，意味着：`v-if` 将分别重复运行于每个 `v-for` 循环中，会造成性能问题。所以，不推荐 `v-if` 和 `v-for` 同时使用

<!--more-->

```js
const compiler = require('vue-template-compiler')

const res = compiler.compile(`<div v-if="true" v-for="i in 3">{{message}}</div>`)

with (this) {
  return renderList(3, function(i) {
    return true ? createElement('div', [createTextVNode(toString(message))]) : createEmptyVNode()
  })
}
```

`v-show`

```js
const compiler = require('vue-template-compiler')

const res = compiler.compile(`<p v-show="flag === 'a'">A</p>`)

with (this) {
  return createElement(
    'p',
    {
      directives: [
        { name: 'show', rawName: 'v-show', value: flag === 'a', expression: "flag === 'a'" },
      ],
    },
    [createTextVNode('A')]
  )
}

// v-show 操作的是样式 https://github.com/vuejs/vue/blob/dev/src/platforms/web/runtime/directives/show.js
bind (el: any, { value }: VNodeDirective, vnode: VNodeWithData) {
  vnode = locateNode(vnode)
  const transition = vnode.data && vnode.data.transition
  const originalDisplay = el.__vOriginalDisplay =
    el.style.display === 'none' ? '' : el.style.display
  if (value && transition) {
    vnode.data.show = true
    enter(vnode, () => {
      el.style.display = originalDisplay
    })
  } else {
    el.style.display = value ? originalDisplay : 'none'
  }
}
```

## 为何在 v-for 中用 key

- 必须用 `key`，且不能是 `index` 和 `random`
- `diff` 算法中通过 `tag` 和 `key` 来判断，是否是 `sameNode`
- 减少渲染次数，提升渲染性能

## 描述 Vue 组件生命周期（父子组件）

- `beforeCreate` 在初始化事件生命周期之后，数据被观测（observer）之前调用

- `created` 实例已经创建完成之后被调用

  可以进行一些数据、资源请求。在这个阶段无法与 DOM 进行交互，如果非想要，可以通过 `$nextTick` 访问

- `beforeMount` 在 DOM 挂载之前被调用，相关的 `render` 函数首次被调用（如果有 `template` 会转换成 `render` 函数）

  在此时也可以对数据进行更改，不会触发 `updated`

- `mounted` 创建 `vm.$el` 并替换 `el`，并在挂载之后调用该钩子

  可以访问到 DOM 节点，使用 `$refs` 属性对 DOM 进行操作，也可以像后台发送请求，拿到返回数据

- `beforeUpdate` 数据更新时调用，发生在虚拟 DOM 重新渲染和和打补丁之前

  可以在这个钩子中进一步地更改状态，这不会触发附加的重新渲染

- `updated` 由于数据更改导致虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子

  避免在此期间更改状态，可能会导致更新无限循环

- `beforeDestroy` 实例销毁之前调用

- `destroyed ` Vue 实例销毁后调用，调用后，Vue 实例所有东西都会解绑定，所有事件监听会被移除，所有子实例也会被销毁

  可以执行一些优化操作，清除定时器，解除绑定事件

注意：除了 `beforeCreate` 和 `created` 钩子之外，其他钩子均在服务器端渲染期间不调用

```js
Vue.prototype._init = function (options?: Object) {
  // ...
  initLifecycle(vm)
  initEvents(vm) // 初始化事件相关的属性
  initRender(vm) // vm 添加了一些虚拟 dom、slot 等相关的属性和方法
  callHook(vm, 'beforeCreate')
  initInjections(vm)
  initState(vm) // props、methods、data、watch、computed等数据初始化
  initProvide(vm)
  callHook(vm, 'created')
}
```

`mounted` （渲染完成）执行顺序是先子后父

```js
function patch(oldVnode, vnode, hydrating, removeOnly) {
   // 定义收集所有组件的insert hook方法的数组
  const insertedVnodeQueue = []
  if (isUndef(oldVnode)) {
    createElm(vnode, insertedVnodeQueue)
  }
}
function createElm(vnode, insertedVnodeQueue, ...) {
  // createChildren会递归创建子组件（递归createElm）
  createChildren(vnode, children, insertedVnodeQueue)
  if (isDef(data)) {
    // 执行所有的create钩子
    invokeCreateHooks(vnode, insertedVnodeQueue)
  }
}
function invokeCreateHooks (vnode, insertedVnodeQueue) {
  // 把vnode push到insertedVnodeQueue
  if (isDef(i.insert)) insertedVnodeQueue.push(vnode)
}

// 调用insert方法把DOM插入到父节点，因为是递归调用，子元素会优先调用insert
insert(parentElm, vnode.elm, refElm)
function invokeInsertHook(vnode, queue, initial) {
  // 依次调用insert方法
  queue[i].data.hook.insert(queue[i])
}
const componentVNodeHooks = {
  // 依次执行 mounted 方法
  insert(vnode: MountedComponentVNode) {
    callHook(componentInstance, 'mounted')
  },
}
```

`$destroy` （销毁完成）执行顺序是先子后父

```js
Vue.prototype.$destroy = function() {
  callHook(vm, 'beforeDestroy')
   // 递归触发子组件销毁钩子函数
  vm.__patch__(vm._vnode, null)
  callHook(vm, 'destroyed')
}
```

## Vue 组件如何通讯

- 父 -> 子通过 `props`，子 -> 父通过 `$on $emit`
- 在父组件中提供数据子组件进行消费 `provide`、`inject`
- `ref` 获取实例的方式调用组件的属性或方法
- 自定义事件 `event.$on`、`event.$off`、`event.$emit`
- `vuex` 状态管理实现通信

## 描述组件渲染和更新过程

- 生成 `render` 函数，其生成一个 `vnode`，它会 `touch` 触发 `getter` 进行收集依赖
- 在模板中哪个被引用了就会将其用 `Watcher` 观察起来，发生了 `setter` 也会将其 `Watcher` 起来
- 如果之前已经被 `Watcher` 观察起来，发生更新进行重新渲染

![](https://gitee.com/lilyn/pic/raw/master/jslearn-img/vue渲染更新流程图.png)

## 双向数据绑定 v-model 的实现原理

`v-model`本质上是语法糖，`v-model` 在内部为不同的输入元素使用不同的属性并抛出不同的事件

- `text` 和 `textarea` 元素使用 value 属性和 input 事件
- `checkbox` 和 `radio` 使用 checked 属性和 change 事件
- `select` 字段将 value 作为 prop 并将 change 作为事件

```js
const compiler = require('vue-template-compiler')

const res = compiler.compile(`<input v-model="name" type="text" />`)

with (this) {
  return createElement('input', {
    directives: [{ name: 'model', rawName: 'v-model', value: name, expression: 'name' }],
    attrs: { type: 'text' },
    domProps: { value: name },
    on: {
      input: function($event) {
        if ($event.target.composing) return
        name = $event.target.value
      },
    },
  })
}
```

## 对 MVVM 的理解

- `Model`：代表数据模型，也可以在 `Model` 中定义数据修改和操作的业务逻辑。我们可以把 `Model` 称为数据层，因为它仅仅关注数据本身，不关心任何行为
- `View`：用户操作界面。当 `ViewModel` 对 `Model` 进行更新的时候，会通过数据绑定更新到 `View`
- `ViewModel`：业务逻辑层，`View` 需要什么数据，`ViewModel` 要提供这个数据；`View` 有某些操作，`ViewModel` 就要响应这些操作

**总结**： `MVVM`模式简化了界面与业务的依赖，解决了数据频繁更新。`MVVM` 在使用当中，利用双向绑定技术，使得 `Model` 变化时，`ViewModel` 会自动更新，而 `ViewModel` 变化时，`View` 也会自动变化

## computed 和 watch 的区别

`computed`：

- `computed` 具有缓存性，`computed` 的值在 `getter` 执行后是会缓存的，只有它依赖的属性值改变之后，下一次获取 `computed` 的值时才会重新调用对应的 `getter` 来计算
- `computed` 适用于比较消耗性能的计算场景，可以提高性能

`watch`：

- 更多的是观察作用，类似于数据监听的回调函数，用于观察 `props`、`$emit` 或本组件的值，当数据变化时来执行回调进行后续操作
- 无缓存性，页面重新渲染时值不变化也会执行

`computed` 和 `watch` 都支持对象的写法

```js
vm.$watch('obj', {
  deep: true, // 深度遍历
  immediate: true, // 立即触发
  handler: function(val, oldVal) {}, // 执行的函数
})
var vm = new Vue({
  data: { a: 1 },
  computed: {
    aPlus: {
      // this.aPlus 时触发
      get: function() {
        return this.a + 1
      },
      // this.aPlus = 1 时触发
      set: function(v) {
        this.a = v - 1
      },
    },
  },
})
```

## 为何组件 data 必须是一个函数

- 一个组件被复用多次的话，也就是创建多个实例。本质上，这些实例用的都是同一个构造函数，如果 `data` 是对象的话（引用数据类型），会影响到所有实例
- 为了组件不同实例 `data` 不冲突，`data` 必须是一个函数

## 自定义 v-model

`v-model` 可以看成是 `value + input` 方法的语法糖

- 自定义：自己写 `model` 属性，里面放上 `prop` 和 `event`

```html
<template>
  <input type="text" :value="text" @input="$emit('change', $event.target.value)" />
</template>

<script>
export default {
  model: {
    prop: 'text',
    event: 'change',
  },
  props: {
    text: String,
  },
}
</script>
```

## 相同逻辑如何抽离

- `Vue.mixin` ，给组件每个生命周期、函数都混入一些公共逻辑
- `mixin` 混入的钩子函数会先于组件内的钩子函数执行，并且在遇到同名选项时也会有选择性进行合并

## 何时使用异步组件

核心就是把组件变成一个函数，依赖 `import()` 语法，可以实现文件的分割加载

- 加载大组件
- 路由异步加载

## 何时使用 keep-alive

常用的两个属性：`include`、`exclude`，允许组件有条件的进行缓存

两个生命周期：`activated`、`deactivated`，用来得知当前组件是否处于活跃状态

- 缓存组件实例，用于保留组件状态或避免重复渲染
- 多个静态 Tab 页的切换时，来优化性能

## 何时需要使用 beforeDestory

- 解绑自定义事件 `event.$off`
- 清除定时器
- 解绑自定义的 DOM 事件，如：`window`、`scroll` 等

## action 和 mutation 有何区别

- `action` 中可以处理异步，`mutation` 中不可以
- `mutation` 做的是原子操作，`action` 可以整合多个 `mutation`

## vue-router 常用的路由模式

**hash 路由**

- `hash` 变化会触发网页跳转，即浏览器的前进、后退
- `hash` 变化不会刷新页面，`SPA` 必需的特点
- `hash` 永远不会提交到 `server` 端（前端自生自灭）

**history 路由**

- 用 `url` 规范的路由，但跳转时不刷新页面
- 需要 `server` 端配合，可参考：[后端配置例子](https://router.vuejs.org/zh/guide/essentials/history-mode.html#%E5%90%8E%E7%AB%AF%E9%85%8D%E7%BD%AE%E4%BE%8B%E5%AD%90)
- `pushState` 不会触发 `hashchange` 事件，`popstate` 事件只会在浏览器某些行为下触发，比如点击后退、前进按钮

## vnode 描述一个 DOM 结构

- Vue 中的真实 DOM

```html
<div id="div1" class="container">
  <p>vdom</p>
  <ul style="font-size: 20px;">
    <li>a</li>
  </ul>
</div>
```

- Vue 中的虚拟 DOM

```js
{
  tag: 'div',
  props: {
    className: 'container',
    id: 'div1',
  },
  children: [
    {
      tag: 'p',
      children: 'dom',
    },
    {
      tag: 'ul',
      props: { style: 'font-size: 20px' },
      children: [{ tag: 'li', children: 'a' }],
    },
  ],
}
```

## 数据响应式原理

**核心 API**：`Object.defineProperty`

- 存在一些问题，Vue 3.0 启动 `Proxy`

  `Proxy` 可以原生支持监听数组变化

  但是 `Proxy` 兼容性不好，且无法 `polyfill`

**问题**

- 深度监听，需要递归到底，一次性计算量大
- 无法监听新增属性/删除属性（`Vue.set`、`Vue.delete`）
- 不能监听数组变化（重新定义原型，重写 `push`、`pop` 等方法）

**简单实现 Vue 中的 defineReactive**

```js
// 触发更新视图
function updateView() {
  console.log('视图更新')
}

// 重新定义数组原型
const arrayProto = Array.prototype
// 创建新对象，原型指向 arrayProto ，再扩展新的方法不会影响原型
const arrayMethods = Object.create(arrayProto)
;['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'].forEach(method => {
  arrayMethods[method] = function() {
    arrayProto[method].call(this, ...arguments) // 原始操作
    updateView() // 触发视图更新
  }
})

// 重新定义属性，监听起来
function defineReactive(target, key, value) {
  observer(value) // 进行监听

  // 不可枚举的不用监听
  const property = Object.getOwnPropertyDescriptor(target, key)
  if (property && property.configurable === false) return
  Object.defineProperty(target, key, {
    get() {
      return value
    },
    set(newValue) {
      if (newValue !== value) {
        observer(newValue) // 值修改后进行监听

        // value 一直在闭包中，此处设置完之后，再 get 时也是会获取最新的值
        value = newValue
        updateView() // 触发更新视图
      }
    },
  })
}

// 监测数据的变化
function observer(target) {
  // 不是对象或数组（vue 是判断是否是数组、纯粹对象、可扩展对象）
  if (typeof target !== 'object' || target === null) {
    return target
  }
  // 不要这样写，污染全局的 Array 原型
  /* Array.prototype.push = function () {
      updateView()
  } */
  if (Array.isArray(target)) {
    target.__proto__ = arrayMethods
  }
  // 重新定义各个属性（for in 也可以遍历数组）
  for (let key in target) {
    if (!Object.hasOwnProperty.call(target, key)) return
    defineReactive(target, key, target[key])
  }
}

const data = {
  name: 'zhangsan',
  age: 20,
  info: {
    address: '北京', // 需要深度监听
  },
  nums: [10, 20, 30],
}

observer(data)

data.name = 'lisi'
data.age = 21
data.info.address = '上海' // 深度监听
// data.x = '100' // 新增属性，监听不到 —— 所以有 Vue.set
// delete data.name // 删除属性，监听不到 —— 所有已 Vue.delete
data.nums.push(4) // 监听数组
```

## diff 算法

diff 算法过程：

- 同级元素进行比较，再比较子节点
- 先判断一方有子节点，一方没有子节点情况（如果新的 `children` 没有子节点，将旧的子节点移除）
- 之后比较都有子节点的情况（核心 `diff`），递归比较子节点

正常 `diff` 两个树的时间复杂度是 `O(n^3)`，但实际情况我们很少会跨级移动 DOM。所以，只有当新旧 `children` 都为多个子节点时才需要核心的 `diff` 算法进行同层级比较

- Vue2 核心 `diff` 算法采用了双端比较的算法，同时从新旧 `children` 的两端开始比较，借助 `key` 值找到可复用的节点，再进行相关操作。相比 React 的 `diff` 算法，同样情况可以减少移动节点次数，减少不必要的性能损耗
- Vue3 核心 `diff` 算法采用了最长递增子序列

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/847306f303ab4177891b56cccff1ebd3~tplv-k3u1fbpfcp-watermark.image)

双端比较算法

- 使用 **旧列表** 的头一个节点 `oldStartNode` 与 **新列表** 的头一个节点 `newStartNode` 对比
- 使用 **旧列表** 的最后一个节点 `oldEndNode` 与 **新列表** 的最后一个节点 `newEndNode` 对比
- 使用 **旧列表** 的头一个节点 `oldStartNode` 与 **新列表** 的最后一个节点 `newEndNode` 对比
- 使用 **旧列表** 的最后一个节点 `oldEndNode` 与 **新列表** 的头一个节点 `newStartNode` 对比

![](https://gitee.com/lilyn/pic/raw/master/jslearn-img/diff算法概述.png)

**树 diff 的时间复杂度 O(n^3)**

- 对于旧树上的点 E 来说，它要和新树上的所有点比较，复杂度为 `O(n)`

- 点 E 在新树上没有找到，点 E 会被删除，然后遍历新树上的所有点找到对应点（X）去填空，复杂度增加到 `O(n^2)`

- 这样的操作会在旧树的每个点进行，最终复杂度为 `O(n^3)`

  1000 个节点，要计算 1 亿次，算法不可用

**优化时间复杂度到 O(n)**

- 只比较同一层级，不跨级比较
- `tag` 不相同，则直接删掉重建，不再深度比较
- `tag` 和 `key`，两者都相同，则认为是相同节点，不再深度比较

## 简述 diff 算法过程

在 Vue 中，主要是 `patch()`、`patchVnode()` 和 `updateChildren` 这三个方法来实现 `Diff` 的

- 当 Vue 中的响应式数据发生变化时，就会触发 `updateCompoent()`

- `updateComponent()` 会调用 `patch()` 方法，在该方法中进行比较，调用 `sameVnode` 判断是否为相同节点（判断 `key`、`tag` 等静态属性），如果是相同节点的话执行 `patchVnode` 方法，开始比较节点差异，如果不是相同节点的话，则进行替换操作

  `patch()` 接收新旧虚拟 DOM，即 `oldVnode`、`vnode`

  - 首先判断 `vnode` 是否存在，如果不存在，删除旧节点
  - 如果 `vnode` 存在，再判断 `oldVnode`，如果不存在，只需新增整个 `vnode` 即可
  - 如果 `vnode` 和 `oldVnode` 都存在，判断两者是不是相同节点，如果是，调用 `patchVnode()` ，对两个节点进行详细比较
  - 如果两者不是相同节点，只需将 `vnode` 转换为真实 DOM 替换 `oldVnode`

- `patchVnode` 同样接收新旧虚拟 DOM，即 `oldVnode`、`vnode`

  - 首先判断两个虚拟 DOM 是不是全等，即没有任何变动，是的话直接结束函数，否者继续执行
  - 其次更新节点的属性，接着判断 `vnode.text` 是否存在，存在的话只需更新节点文本即可，否则继续执行
  - 判断 `vnode` 和 `oldVnode` 是否有孩子节点
    - 如果两者都有孩子节点，执行 `updateChildren()` ，进行比较更新
    - 如果 `vnode` 有孩子，`oldVnode` 没有，则直接删除所有孩子节点，并将该文本属性设为空
    - 如果 `oldVnode` 有孩子，`vnode` 没有，则直接删除所有孩子节点
    - 如果两者都没有孩子节点，就判断 `oldVnode.text` 是否有内容，有的话情况内容即可

- `updateChildren` 接收三个参数：`parentElm` 父级真实节点、`oldCh` 为 `oldVnode` 的孩子节点、`newCh` 为 `Vnode` 的孩子节点

  `oldCh` 和 `newCh` 都是一个数组。正常我们想到的方法就是对这两个数组一一比较，时间复杂度为 `O(NM)`。Vue 中是通过四个指针实现的

  - 首先是 `oldStartVnode` 和 `newStartVnode` 进行比较（两头比较），如果比较相同的话，就可以执行 `patchVnode`
  - 如果 `oldStartVnode` 和 `newStartVnode` 匹配不上的话，接下来就是 `oldEndVnode` 和 `newEndVnode` 做比较了（两尾比较）
  - 如果两头和两尾比较都不是相同节点的话，就开始交叉比较，首先是 `oldStartVnode` 和 `newEndVnode` 做比较（头尾比较）
  - 如果 `oldStartVnode` 和 `newEndVnode` 匹配不上的话，就 `oldEndVnode` 和 `newStartVnode` 进行比较（尾头比较）

  如果这四种比较方法都匹配不到相同节点，才是用暴力解法，针对 `newStartVnode` 去遍历 `oldCh` 中剩余的节点，一一匹配

> 图片来源：[图文并茂地来详细讲讲Vue Diff算法](https://juejin.cn/post/6971622260490797069#heading-2)

![diff.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d84848aa035b4472b4d3f8e4b0d708c5~tplv-k3u1fbpfcp-watermark.awebp)

## Vue 为何是异步渲染，$nextTick 何用

- 因为如果不采用异步更新，那么每次更新数据都会对当前组件进行重新渲染。异步渲染（合并 data 修改），再更新视图，可以提高渲染性能

```js
class Watcher {
  update() {
    if (this.computed) {
      // ...
    } else if (this.sync) {
      // ...
    } else {
      // 当数据发生变化时会将watcher放到一个队列中批量更新
      queueWatcher(this)
    }
  }
}

const queue: Array<Watcher> = []
let has: { [key: number]: ?true } = {}
let waiting = false
let flushing = false
// 在派发更新时并不会每次修改都触发watcher的回调
export function queueWatcher(watcher: Watcher) {
  const id = watcher.id
  // has对象保证同一个watcher只添加一次
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      queue.push(watcher)
    } else {
      let i = queue.length - 1
      while (i > index && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(i + 1, 0, watcher)
    }
    // 通过waiting保证nextTick的调用逻辑只有一次
    if (!waiting) {
      waiting = true
      // 调用nextTick方法 批量的进行更新
      nextTick(flushSchedulerQueue)
    }
  }
}

function flushSchedulerQueue () {
  // 1.因为父组件的创建过程是先于子的，所以watcher的创建也是先父后子，执行顺序也是先父后子
  // 2.用户自定义watcher要优先于渲染watcher执行
  // 3.如果一个组件在父组件watcher执行期间被销毁，那么它对应的watcher执行都可以被跳过
  queue.sort((a, b) => a.id - b.id) // 由小到大
}
```

- `$nextTick` 在 DOM 更新完之后，触发回调，用于获得更新后的 DOM

- `$nextTick` 方法主要是使用了 **宏任务** 和 **微任务**，定义一个异步方法，多次调用 `nextTick` 会将方法存入队列中，通过这个异步方法清空当前队列

  Vue 2.4 之前都是使用微任务，但是微任务的优先级过高，有些情况下可能会出现比事件冒泡更快的情况，但如果都是用宏任务，有可能会出现渲染的性能问题

  新版，默认使用微任务，但在特殊情况下会使用宏任务，比如：`v-on`

- 对于实现宏任务，会先判断是否能用 `setImmediate`，不能的话降级为 `MessageChannel` ，以上都不行的话就是用 `setTimeout`

```js
if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  macroTimerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else if (
  typeof MessageChannel !== 'undefined' &&
  (isNative(MessageChannel) ||
    // PhantomJS
    MessageChannel.toString() === '[object MessageChannelConstructor]')
) {
  const channel = new MessageChannel()
  const port = channel.port2
  channel.port1.onmessage = flushCallbacks
  macroTimerFunc = () => {
    port.postMessage(1)
  }
} else {
  macroTimerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}
```

## Vue 常见性能优化方式

- 合理使用 `v-show` 和 `v-if`
- 合理使用 `computed`
- `v-for` 时加 `key`（Vue 会进行复用），以及避免和 `v-if` 同时使用
- 自定义事件、DOM 事件及时销毁
- 合理使用异步组件、路由懒加载
- 合理使用 `keep-alive`（SPA 页面）
- `data` 层级不要太深，不要讲所有数据都放在 `data` 中（会增加 `getter` 和 `setter`，收集对应 `watcher`）
- 使用 `vue-loader` 在开发环境做模板编译（预编译）
- webpack 层面的优化
- 前端通用的性能优化，如图片懒加载、防抖、节流
- 使用 SSR

## 释义参数 vue-template-compiler

- `render` 中的参数释义

```js
_c = createElement
function installRenderHelpers(target) {
  target._o = markOnce
  target._n = toNumber
  target._s = toString
  target._l = renderList
  target._t = renderSlot
  target._q = looseEqual
  target._i = looseIndexOf
  target._m = renderStatic
  target._f = resolveFilter
  target._k = checkKeyCodes
  target._b = bindObjectProps
  target._v = createTextVNode
  target._e = createEmptyVNode
  target._u = resolveScopedSlots
  target._g = bindObjectListeners
  target._d = bindDynamicKeys
  target._p = prependModifier
}
```

