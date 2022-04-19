---
title: Vue 原理（虚拟DOM、diff算法、模板编译）
tags:
  - Vue
  - 虚拟DOM
  - diff算法
  - 模板编译
  - 前端路由原理
categories: Vue
author: LiLyn
copyright: ture
---

## Vue 原理

> 其他扩展：[vue面试中常见的面试题](https://blog.csdn.net/Govern66/article/details/104452545)

**原理的意义**

- 知其然知其所以然——各行业通用的道理
- 了解原理，才能应用的更好
- 大厂造轮子（业务定制、技术 KPI）

<!--more-->

**如何考察**

- 考察重点，而不是考察细节。掌握好 2/8 原则
- 和使用相关联的原理，例如：vdom、模板渲染
- 整体流程是否全面？热门技术是否有深度？

**重要的原理**

- 组件化
- 响应式
- vdom 和 diff
- 模板编译
- 渲染过程
- 前端路由

**原理题**

- 为何 v-for 中要用 key
- 描述组件渲染和更新的过程
- 双向数据绑定 v-model 的实现原理

## 如何理解 MVVM

- “很久以前” 的组件化
  - asp、jsp、php 已经有组件化了
  - nodejs 中也有类似的组件化
- 数据驱动视图（MVVM、setState）
  - 传统组件，只是静态渲染，更新还要依赖于操作 DOM
  - 数据驱动视图——Vue MVVM
  - 数据驱动视图——React setState

![](https://gitee.com/lilyn/pic/raw/master/jslearn-img/MVVM.png)

## Vue 响应式（数据拦截）

> 推荐文章：[简单手写实现Vue2.x](https://juejin.cn/post/6963079075938172936)

- 组件 data 数据一旦变化，立刻触发视图的更新
- 实现数据驱动视图的第一步

### 数据响应式原理

> 源码级别可以参考： [Vue.js 技术揭秘——深入响应式原理](https://ustbhuangyi.github.io/vue-analysis/v2/reactive/reactive-object.html#object-defineproperty)

在 JavaScript 的对象 Object 中有一个属性叫访问器属性，其中有 `[[Get]]` 和 `[[Set]]` 特性，它们分别是获取函数和设置函数

**核心 API**：`Object.defineProperty`

- 存在一些问题，Vue 3.0 启动 `Proxy`

  `Proxy` 可以原生支持监听数组变化
  
  但是 `Proxy` 兼容性不好，且无法 `polyfill`

**问题**

- 深度监听，需要递归到底，一次性计算量大
- 无法监听新增属性/删除属性（`Vue.set`、`Vue.delete`）
- 不能监听数组变化（重新定义原型，重写 `push`、`pop` 等方法）

> `Object.defineProperty` 实现响应式
>
> - `observe` 的功能：给非 VNode 的对象类型数据添加一个 `Observer` ，用来监听数据的变化
> - `Observer` 的作用：给对象的属性添加 getter 和 setter，用来依赖收集和派发更新。对于数组会调用 `observeArray` 方法，对于对象会对对象的 key 调用 `defineReactive` 方法
> - `defineReactive` 的功能：定义一个响应式对象，给对象动态添加 getter 和 setter。对子对象递归调用 `observe` 方法，这样就保证了无论 `obj` 的结构多复杂，它的所有子属性也能变成响应式的对象，这样我们访问或修改 `obj` 中一个嵌套较深的属性，也能触发 getter 和 setter
> - setter 的时候，会通知所有的订阅者
> - `arrayMethods` 首先继承了 `Array`，然后对数组中的所有能改变数组自身的方法进行重写，重写后的方法会先执行它们本身原有的逻辑，然后把新添加的值

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

## 虚拟 DOM

- DOM 操作非常耗费性能
- 以前用 jQuery，可以自行控制 DOM 操作的时机，手动调整
- Vue 和 React 是数据驱动视图，如何有效控制 DOM 操作？

### vdom

- 有了一定复杂度，想减少计算次数比较难
- 能不能把计算，更多的转移为 JS 计算？因为 JS 执行速度很快
- vdom——用 JS 模拟 DOM 结构，计算出最小的变更，操作 DOM

**用 JS 模拟 DOM 结构**

- DOM 结构

```html
<div id="div1" class="container">
  <p>vdom</p>
  <ul style="font-size: 20px;">
    <li>a</li>
  </ul>
</div>
```

- JS 模拟

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

- Vue 中的真实 DOM

```html
<div id="app">
  <p class="text">HelloWorld</p>
</div>
```

- Vue 中的虚拟 DOM

```js
{
  "tag": "div",                     // 标签名
  "key": undefined,                 // key值
  "elm": div#app,                   // 真实DOM节点
  "text": undefined,                // 文本信息
  "data": {attrs: {id:"app"}},      // 节点属性
  "children": [{    	            // 子节点属性
    "tag": "p",
    "key": undefined,
    "elm": p.text,
    "text": undefined,
    "data": {attrs: {class: "text"}},
    "children": [{
      "tag": undefined,
      "key": undefined,
      "elm": text,
      "text": "helloWorld",
      "data": undefined,
      "children": []
    }]
  }]
}
```

### snabbdom 使用

- 通过 **[snabbdom](https://github.com/snabbdom/snabbdom)** 学习 vdom

- 核心概念：h、vnode、patch、diff、key 等

- vdom 价值：数据驱动视图，控制 DOM 操作

  `patch(elem, vnode)` 和 `patch(vnode, newVnode)`

```html
<!DOCTYPE html>
<html>
  <body>
    <div id="container"></div>
    <button id="btn-change">change</button>
  </body>
</html>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/snabbdom.js"></script>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/snabbdom-class.js"></script>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/snabbdom-props.js"></script>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/snabbdom-style.js"></script>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/snabbdom-eventlisteners.js"></script>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/h.js"></script>
<script>
  const snabbdom = window.snabbdom

  // 定义 patch
  const patch = snabbdom.init([
    snabbdom_class,
    snabbdom_props,
    snabbdom_style,
    snabbdom_eventlisteners,
  ])

  // 定义 h
  const h = snabbdom.h

  const container = document.getElementById('container')

  // 生成 vnode
  const vnode = h('ul#list', {}, [h('li.item', {}, 'Item 1'), h('li.item', {}, 'Item 2')])
  patch(container, vnode)

  document.getElementById('btn-change').addEventListener('click', () => {
    // 生成 newVnode
    const newVnode = h('ul#list', {}, [
      h('li.item', {}, 'Item 1'),
      h('li.item', {}, 'Item B'),
      h('li.item', {}, 'Item 3'),
    ])
    patch(vnode, newVnode)
  })
</script>
```

- jQuery 版本

```html
<!DOCTYPE html>
<html>
  <body>
    <div id="container"></div>
    <button id="btn-change">change</button>
  </body>
</html>
<script type="text/javascript" src="https://cdn.bootcss.com/jquery/3.2.0/jquery.js"></script>
<script type="text/javascript">
  const data = [
    {
      name: '张三',
      age: '20',
      address: '北京',
    },
    {
      name: '李四',
      age: '21',
      address: '上海',
    },
    {
      name: '王五',
      age: '22',
      address: '广州',
    },
  ]

  // 渲染函数
  function render(data) {
    const $container = $('#container')

    // 清空容器，重要！！！
    $container.html('')

    // 拼接 table
    const $table = $('<table>')

    $table.append($('<tr><td>name</td><td>age</td><td>address</td>/tr>'))
    data.forEach(item => {
      $table.append(
        $(
          '<tr><td>' + item.name + '</td><td>' + item.age + '</td><td>' + item.address + '</td>/tr>'
        )
      )
    })

    // 渲染到页面
    $container.append($table)
  }

  $('#btn-change').click(() => {
    data[1].age = 30
    data[2].address = '深圳'
    // re-render  再次渲染
    render(data)
  })

  // 页面加载完立刻执行（初次渲染）
  render(data)
</script>
```

- vdom 版本

```html
<!DOCTYPE html>
<html>
  <body>
    <div id="container"></div>
    <button id="btn-change">change</button>
  </body>
</html>

<script src="https://cdn.bootcss.com/snabbdom/0.7.3/snabbdom.js"></script>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/snabbdom-class.js"></script>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/snabbdom-props.js"></script>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/snabbdom-style.js"></script>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/snabbdom-eventlisteners.js"></script>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/h.js"></script>
<script type="text/javascript">
  const snabbdom = window.snabbdom
  // 定义关键函数 patch
  const patch = snabbdom.init([
    snabbdom_class,
    snabbdom_props,
    snabbdom_style,
    snabbdom_eventlisteners,
  ])

  // 定义关键函数 h
  const h = snabbdom.h

  // 原始数据
  const data = [
    {
      name: '张三',
      age: '20',
      address: '北京',
    },
    {
      name: '李四',
      age: '21',
      address: '上海',
    },
    {
      name: '王五',
      age: '22',
      address: '广州',
    },
  ]
  // 把表头也放在 data 中
  data.unshift({
    name: '姓名',
    age: '年龄',
    address: '地址',
  })

  const container = document.getElementById('container')

  // 渲染函数
  let vnode
  function render(data) {
    const newVnode = h(
      'table',
      {},
      data.map(item => {
        const tds = []
        for (let i in item) {
          if (item.hasOwnProperty(i)) {
            tds.push(h('td', {}, item[i] + ''))
          }
        }
        return h('tr', {}, tds)
      })
    )

    if (vnode) {
      // re-render
      patch(vnode, newVnode)
    } else {
      // 初次渲染
      patch(container, newVnode)
    }

    // 存储当前的 vnode 结果
    vnode = newVnode
  }

  // 初次渲染
  render(data)

  const btnChange = document.getElementById('btn-change')
  btnChange.addEventListener('click', () => {
    data[1].age = 30
    data[2].address = '深圳'
    // re-render
    render(data)
  })
</script>
```

## diff 算法

> 推荐文章：[图文并茂地来详细讲讲Vue Diff算法](https://juejin.cn/post/6971622260490797069#refetch)

- diff 算法是 vdom 中最核心、最关键的部分
- diff 算法能在日常使用 Vue、React 中体现出来（如：key）

**diff 算法概述：**

- diff 即对比。是一个广泛的概念，如 linux diff 命令、git diff 等
- 两个 js 对象也可以做 diff，**[jiff](https://github.com/cujojs/jiff)**
- 两棵树做 diff，如这里的 vdom diff

![](https://gitee.com/lilyn/pic/raw/master/jslearn-img/diff算法概述.png)

**树 diff 的时间复杂度 O(n^3)**

- 对于旧树上的点 E 来说，它要和新树上的所有点比较，复杂度为 O(n)

- 点 E 在新树上没有找到，点 E 会被删除，然后遍历新树上的所有点找到对应点（X）去填空，复杂度增加到 O(n^2)

- 这样的操作会在旧树的每个点进行，最终复杂度为 O(n^3)

  1000 个节点，要计算 1 亿次，算法不可用

**优化时间复杂度到 O(n)**

- 只比较同一层级，不跨级比较
- tag 不相同，则直接删掉重建，不再深度比较
- tag 和 key，两者都相同，则认为是相同节点，不再深度比较

![](https://gitee.com/lilyn/pic/raw/master/jslearn-img/diff算法时间复杂度.png)

![](https://gitee.com/lilyn/pic/raw/master/jslearn-img/diff算法时间复杂度2.png)

### snabbdom 源码

> [snabbdom](https://github.com/snabbdom/snabbdom)

- `src\h.ts`

  最后返回 vnode

```js
export function h(sel: string): VNode;
export function h(sel: string, data: VNodeData | null): VNode;
export function h(sel: string, children: VNodeChildren): VNode;
export function h(
  sel: string,
  data: VNodeData | null,
  children: VNodeChildren
): VNode;
export function h(sel: any, b?: any, c?: any): VNode {
  // ...
  return vnode(sel, data, children, text, undefined);
}
```

- `src\vnode.ts`

  最后返回一个 JS 对象

```js
export function vnode(
  sel: string | undefined,
  data: any | undefined,
  children: Array<VNode | string> | undefined,
  text: string | undefined,
  elm: Element | DocumentFragment | Text | undefined
): VNode {
  const key = data === undefined ? undefined : data.key;
  return { sel, data, children, text, elm, key };
}
```

- `src\init.ts`

  触发页面更新就会调用 `patch` 方法，之后我们看一下 `patch` 的源码

```js
return function patch(
  oldVnode: VNode | Element | DocumentFragment,
  vnode: VNode
): VNode {
  let i: number, elm: Node, parent: Node;
  const insertedVnodeQueue: VNodeQueue = [];
  // 执行 pre hooks
  for (i = 0; i < cbs.pre.length; ++i) cbs.pre[i]();

  // 第一个参数不是 vnode
  if (isElement(api, oldVnode)) {
    // 创建一个空的 vnode，关联到这个 DOM 元素
    oldVnode = emptyNodeAt(oldVnode);
  } else if (isDocumentFragment(api, oldVnode)) {
    oldVnode = emptyDocumentFragmentAt(oldVnode);
  }

  // 相同的 vnode（key 和 sel 都相同）
  if (sameVnode(oldVnode, vnode)) {
    // vnode 对比
    patchVnode(oldVnode, vnode, insertedVnodeQueue);
  // 不同的 vnode，直接删掉重建
  } else {
    elm = oldVnode.elm!;
    parent = api.parentNode(elm) as Node;

    // 重建
    createElm(vnode, insertedVnodeQueue);

    if (parent !== null) {
      api.insertBefore(parent, vnode.elm!, api.nextSibling(elm));
      removeVnodes(parent, [oldVnode], 0, 0);
    }
  }

  for (i = 0; i < insertedVnodeQueue.length; ++i) {
    insertedVnodeQueue[i].data!.hook!.insert!(insertedVnodeQueue[i]);
  }
  for (i = 0; i < cbs.post.length; ++i) cbs.post[i]();
  return vnode;
};
```

如果 `patch` 第一个参数不是 vnode，创建一个空的 vnode

```js
function emptyNodeAt(elm: Element) {
  const id = elm.id ? "#" + elm.id : "";
  const classes = elm.getAttribute("class");
  const c = classes ? "." + classes.split(" ").join(".") : "";
  return vnode(
    api.tagName(elm).toLowerCase() + id + c,
    {},
    [],
    undefined,
    elm
  );
}
```

比较 vnode 是否相同（比较 key 和 selector）

```js
function sameVnode(vnode1: VNode, vnode2: VNode): boolean {
  // key 和 sel 或 data 都相等就是相等
  const isSameKey = vnode1.key === vnode2.key;
  const isSameIs = vnode1.data?.is === vnode2.data?.is;
  const isSameSel = vnode1.sel === vnode2.sel;

  return isSameSel && isSameKey && isSameIs;
}
```

### patchVnode

vnode 相同的话执行 `patchVnode`

- 两者都有 children -> 进行 children 对比，之后更新 children（updateChildren）
- 新 children 有，旧 children 无 -> 清空旧 text，添加 children（addVnodes）
- 新 children 无，旧 children 有 -> 移除旧 children（removeVnodes）
- 新 children 无，旧 children 无，旧 text 有  -> 清空旧 text
- 新旧 text 不一样 -> 移除旧 children(removeVnodes)，设置 text

```js
function patchVnode(
  oldVnode: VNode,
  vnode: VNode,
  insertedVnodeQueue: VNodeQueue
) {
  // 执行 prepatch hook，生命周期钩子
  const hook = vnode.data?.hook;
  hook?.prepatch?.(oldVnode, vnode);
  // 设置 vnode.elm
  const elm = (vnode.elm = oldVnode.elm)!;
  // 旧的 children
  const oldCh = oldVnode.children as VNode[];
  // 新的 children
  const ch = vnode.children as VNode[];
  if (oldVnode === vnode) return;
  // hook 相关
  if (vnode.data !== undefined) {
    for (let i = 0; i < cbs.update.length; ++i)
      cbs.update[i](oldVnode, vnode);
    vnode.data.hook?.update?.(oldVnode, vnode);
  }
  // 新的 vnode.text===undefined (vnode.children 一般有值)
  if (isUndef(vnode.text)) {
    // 新旧都有 children
    if (isDef(oldCh) && isDef(ch)) {
      if (oldCh !== ch) updateChildren(elm, oldCh, ch, insertedVnodeQueue);
    // 新 children 有，旧 children 无（旧 text 有）
    } else if (isDef(ch)) {
      // 清空 text
      if (isDef(oldVnode.text)) api.setTextContent(elm, "");
      // 添加 children
      addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue);
    // 旧 children 有，新 children 无
    } else if (isDef(oldCh)) {
      // 移除 children
      removeVnodes(elm, oldCh, 0, oldCh.length - 1);
    // 旧 text 有
    } else if (isDef(oldVnode.text)) {
      api.setTextContent(elm, "");
    }
  // vnode.text!==undefined（vnode.children 无值）
  } else if (oldVnode.text !== vnode.text) {
    // 移除旧 children
    if (isDef(oldCh)) {
      removeVnodes(elm, oldCh, 0, oldCh.length - 1);
    }
    api.setTextContent(elm, vnode.text!);
  }
  hook?.postpatch?.(oldVnode, vnode);
}
```

### addVnodes removeVnodes

- 新增 vnode

```js
function addVnodes(
  parentElm: Node,
  before: Node | null,
  vnodes: VNode[],
  startIdx: number,
  endIdx: number,
  insertedVnodeQueue: VNodeQueue
) {
  for (; startIdx <= endIdx; ++startIdx) {
    const ch = vnodes[startIdx];
    if (ch != null) {
      api.insertBefore(parentElm, createElm(ch, insertedVnodeQueue), before);
    }
  }
}
```

- 删除 vnode

```js
function removeVnodes(
  parentElm: Node,
  vnodes: VNode[],
  startIdx: number,
  endIdx: number
): void {
  for (; startIdx <= endIdx; ++startIdx) {
    let listeners: number;
    let rm: () => void;
    const ch = vnodes[startIdx];
    if (ch != null) {
      if (isDef(ch.sel)) {
        invokeDestroyHook(ch); // hook操作
        // 移除DOM元素
        listeners = cbs.remove.length + 1;
        rm = createRmCb(ch.elm!, listeners);
        for (let i = 0; i < cbs.remove.length; ++i) cbs.remove[i](ch, rm);
        const removeHook = ch?.data?.hook?.remove;
        if (isDef(removeHook)) {
          removeHook(ch, rm);
        } else {
          rm();
        }
      } else {
        // Text node
        api.removeChild(parentElm, ch.elm!);
      }
    }
  }
}
```

### updateChildren

新旧都有 children，会对 children 进行 updateChildren

```js
function updateChildren(
  parentElm: Node,
  oldCh: VNode[],
  newCh: VNode[],
  insertedVnodeQueue: VNodeQueue
) {
  while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
    if (oldStartVnode == null) { /* ... */ }
    // 开始和开始对比
    } else if (sameVnode(oldStartVnode, newStartVnode)) {
    // 结束和结束对比
    } else if (sameVnode(oldEndVnode, newEndVnode)) {
    // 开始和结束对比
    } else if (sameVnode(oldStartVnode, newEndVnode)) {
    // 结束和开始对比
    } else if (sameVnode(oldEndVnode, newStartVnode)) {
    // 以上4个都未命中
    } else {
      // 拿新节点的 key，能否对应上 oldCh 中的某个节点的 key
      idxInOld = oldKeyToIdx[newStartVnode.key as string];
      // 没对应上
      if (isUndef(idxInOld)) {
        // New element
        api.insertBefore(
          parentElm,
          createElm(newStartVnode, insertedVnodeQueue),
          oldStartVnode.elm!
        );
      } else {
        // 对应上 key 的节点
        elmToMove = oldCh[idxInOld];
        // sel 是否相同（sameVnode 的条件）
        if (elmToMove.sel !== newStartVnode.sel) {
          // New element
          api.insertBefore(
            parentElm,
            createElm(newStartVnode, insertedVnodeQueue),
            oldStartVnode.elm!
          );
        // sel 相等，key 相等
        } else {
          patchVnode(elmToMove, newStartVnode, insertedVnodeQueue);
          oldCh[idxInOld] = undefined as any;
          api.insertBefore(parentElm, elmToMove.elm!, oldStartVnode.elm!);
        }
      }
      newStartVnode = newCh[++newStartIdx];
    }
  }
}
```

![](https://gitee.com/lilyn/pic/raw/master/jslearn-img/updateChildren2.png)

不使用 key 就会全部删掉然后插入；使用 key ，key 相同会直接移动过来，不用做销毁然后重新渲染的过程

![](https://gitee.com/lilyn/pic/raw/master/jslearn-img/updateChildren3.png)

### 总结

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

在 `sameVnode` 中，比较两个节点是否相同时，第一个判断条件就是 `vnode.key` ，并且在后面是用暴力解法时，第一选择也是通过 `key` 去匹配

```html
<!--  old  -->
<div>
  <p>A</p>
  <p>B</p>
  <p>C</p>
</div>

<!--  new  -->
<div>
  <p>B</p>
  <p>C</p>
  <p>A</p>
</div>
```

如果没有设置 `key` 值，通过 `diff` 需要操作 DOM 次数会很多，因为 `key` 为 `undefined`，即使每个标签都是相同节点，也会一一进行替换，需要操作 3 次 DOM

如果分别给对应添加了 `key` 值，通过 `diff` 只需操作 1 次 DOM

> 图片来源：[图文并茂地来详细讲讲Vue Diff算法](https://juejin.cn/post/6971622260490797069#heading-2)

![diff.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d84848aa035b4472b4d3f8e4b0d708c5~tplv-k3u1fbpfcp-watermark.awebp)

## 模板编译

- with 语法
- 模板到 render 函数，再到 vnode，再到渲染和更新
- vue 组件可以用 render 代替 template

**with 语法**

- 使用 `with`，能改变 `{}` 内自由变量的查找规则
- 如果找不到匹配的 `obj` 属性，就会报错
- with 要慎用，它打破了作用域规则，易读性变差

```js
const obj = { a: 100 }

console.log(obj.a)
console.log(obj.b) // undefined

with (obj) {
  console.log(a)
  console.log(b) // 报错
}
```

### vue 模板被编译成什么

- html 是标签语言，只有 JS 才能实现判断、循环，因此模板一定是转换为某种 JS 代码，即编译模板
- 使用 webpack `vue-loader` ，会在开发环境下编译模板

```js
const compiler = require('vue-template-compiler')

// *插值
const template = `<p>{{message}}</p>`

/* with (this) {
  return createElement('p', [createTextVNode(toString(message))])
} */

// 编译
const res = compiler.compile(template)
console.log(res.render)
```

- 三元运算符

```js
// *三元运算符
const template = `<p>{{flag ? message : 'no message found'}}</p>`

with (this) {
  return createElement('p', [createTextVNode(toString(flag ? message : 'no message found'))])
}
```

- 有 `class、id` 属性和动态属性

```js
// *属性和动态属性
const template = `
  <div id="div1" class="container">
    <img :src="imgUrl"/>
  </div>`

with (this) {
  return createElement('div', { staticClass: 'container', attrs: { id: 'div1' } }, [
    createElement('img', { attrs: { src: imgUrl } }),
  ])
}
```

- `v-if` 条件

```js
// *条件
const template = `
  <div>
    <p v-if="flag === 'a'">A</p>
    <p v-else>B</p>
  </div>`

with (this) {
  return createElement('div', [
    flag === 'a'
      ? createElement('p', [createTextVNode('A')])
      : createElement('p', [createTextVNode('B')]),
  ])
}
```

- `v-show` 

```js
// *v-show
const template = `<p v-show="flag === 'a'">A</p>`

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

- `v-for` 循环

```js
// *循环
const template = `
  <ul>
    <li v-for="item in list" :key="item.id">{{item.title}}</li>
  </ul>`

with (this) {
  return createElement(
    'ul',
    renderList(list, function(item) {
      return createElement('li', { key: item.id }, [createTextVNode(toString(item.title))])
    }),
    0
  )
}
```

- `event` 事件

```js
// *事件
const template = `<button @click="clickHandler">submit</button>`

with (this) {
  return createElement('button', { on: { click: clickHandler } }, [createTextVNode('submit')])
}
```

- `v-model`

```js
// *v-model
const template = `<input type="text" v-model="name">`

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

### vue 组件使用 render 函数

- 有些复杂情况中，不能用 template，可以考虑用 render

```js
Vue.component('heading', {
  render: function(createElement) {
    return createElement('h' + this.level, [
      createElement(
        'a',
        {
          attrs: {
            name: 'headerId',
            href: '#' + 'headerId',
          },
        },
        'this is a tag'
      ),
    ])
  },
})
```

## Vue 组件渲染和更新

**初次渲染过程：**

- 解析模板为 `render` 函数（或在开发环境已完成，vue-loader）
- 触发响应式，监听 `data` 属性 `getter` 和 `setter`
- 执行 `render` 函数，生成 `vnode`，`patch(elem, vnode)`

```html
<template>
  <p>{{ message }}</p>
</template>

<script>
export default {
  data() {
    return {
      message: 'hello', // 会触发get
      city: '北京',     // 不会触发get，因为模板没有用到，即和视图没关系
    }
  },
}
</script>
```

**更新过程：**

- 修改 `data`，触发 `setter`（此前在 `getter` 中已被监听）
- 重新执行 `render` 函数，生成 `newVnode`
- `patch(vnode, newVnode)`

![](https://gitee.com/lilyn/pic/raw/master/jslearn-img/vue渲染更新流程图.png)

- 生成 `render` 函数，其生成一个 `vnode`，它会 `touch` 触发 `getter` 进行收集依赖
- 在模板中哪个被引用了就会将其用 `Watcher` 观察起来，发生了 `setter` 也会将其 `Watcher` 起来
- 如果之前已经被 `Watcher` 观察起来，发生更新进行重新渲染

**异步渲染**

> 源码级别可以参考：[Vue.js 技术揭秘——nextTick](https://ustbhuangyi.github.io/vue-analysis/v2/reactive/next-tick.html)

- `$nextTick`
- 汇总 `data` 的修改，一次性更新视图
- 减少 DOM 操作次数，提高性能

## 前端路由原理

### hash 路由

**hash 的特点：**

- `hash` 变化会触发网页跳转，即浏览器的前进、后退
- `hash` 变化不会刷新页面，`SPA` 必需的特点
- `hash` 永远不会提交到 `server` 端（前端自生自灭）

```js
http://127.0.0.1:5500/src/1.html?a=100&b=20#/aaa/bbb/
location.protocol // 'http:'
location.hostname // '127.0.0.1'
location.host // '127.0.0.1:5500'
location.port // '5500'
location.pathname // '/src/1.html'
location.search // '?a=100&b=20'
location.hash // '#/aaa/bbb/'
```

**hash 变化：**

1. JS 修改 `url`
2. 手动修改 `url` 的 `hash`
3. 浏览器前进、后退

```html
<!DOCTYPE html>
<html>
  <body>
    <p>hash test</p>
    <button id="btn1">修改 hash</button>
  </body>
</html>
<script>
  window.onhashchange = event => {
    console.log('old url', event.oldURL)
    console.log('new url', event.newURL)

    console.log('hash:', location.hash)
  }

  // 页面初次加载，获取 hash
  document.addEventListener('DOMContentLoaded', () => {
    console.log('hash:', location.hash)
  })

  // JS 修改 url
  document.getElementById('btn1').addEventListener('click', () => {
    location.href = '#/user'
  })
</script>
```

### history 路由

- 用 `url` 规范的路由，但跳转时不刷新页面
- 需要 `server` 端配合，可参考：[后端配置例子](https://router.vuejs.org/zh/guide/essentials/history-mode.html#%E5%90%8E%E7%AB%AF%E9%85%8D%E7%BD%AE%E4%BE%8B%E5%AD%90)
- `pushState` 不会触发 `hashchange` 事件，`popstate` 事件只会在浏览器某些行为下触发，比如点击后退、前进按钮

```html
<!DOCTYPE html>
<html lang="en">
  <body>
    <p>history API test</p>
    <button id="btn1">修改 url</button>
  </body>
</html>
<script>
  // 页面初次加载，获取 path
  document.addEventListener('DOMContentLoaded', () => {
    console.log('load', location.pathname)
  })

  // 打开一个新的路由 【注意】用 pushState 方式，浏览器不会刷新页面
  document.getElementById('btn1').addEventListener('click', () => {
    const state = { name: 'page1' }
    console.log('切换路由到', 'page1')
    history.pushState(state, '', 'page1')
  })

  // 监听浏览器前进、后退
  window.onpopstate = event => {
    console.log('onpopstate', event.state, location.pathname)
  }
</script>
```

**两者选择**

- `to B` 的系统推荐用 `hash`，简单易用，对 `url` 规范不敏感
- `to C` 的系统，可以考虑选择 `H5 history` ，但需要服务端支持
- 能选择简单的，就别用复杂的，要考虑成本和收益
