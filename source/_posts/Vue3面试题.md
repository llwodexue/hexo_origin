---
title: Vue3 面试题
tags:
  - Vue
  - 生命周期
  - Composition API
  - 响应式原理
  - Vite
categories: Vue
author: LiLyn
copyright: ture
abbrlink: 5381f975
---

## Vue3

**新功能**

- `createApp`

- `emits` 属性

- 多事件处理

- `Fragment`

  不再限于模板中的单个根节点

- 移除 `.sync` 改为 `v-model` 参数

- 异步组件的引用方式

- 移除 `filter`

- `Teleport`

  以前称为 `<Portal>`，译作传送门（之前都是放在 APP 里，用这个可随意放置）

- `Suspense`

  可以嵌套层级中等待嵌套的异步依赖项

- **Composition API**

  - `reactive`

  - `ref`、`toRef`、`toRefs`

  - `readonly`

  - `computed`

  - `watch`、`watchEffect`

  - 钩子函数生命周期

<!--more-->

**原理**

- `Proxy` 实现响应式

- 编译优化
  - `PatchFlag` 静态标记
  
  - `hoistStatic` 静态提升
  
  - `cacheHandler` 缓存事件
  
  - `SSR` 优化
  
  - `Tree-shaking` 优化
  
    可以将无用模块"剪辑"，仅打包需要

**Vite**

- `ES6 module`

**面试题**

- Vue3 比 Vue2 有什么优势
- 描述 Vue3 生命周期
- 如何看待 `Composition API` 和 `Options API`
- 如何理解 `ref`、`toRef` 和 `toRefs`
- Vue3 升级了哪些重要的功能
- `Composition API` 如何实现代码逻辑复用
- Vue 如何实现响应式
- `watch` 和 `watchEffect` 的区别是什么
- `setup` 中如何获取组件实例
- Vue3 为何比 Vue2 快
- Vite 是什么
- `Composition API` 和 `React Hooks` 的对比

## Vue3 比 Vue2 有什么优势

- 性能更好
- 体积更小
- 更好的 TS 支持
- 更好的代码组织
- 更好的逻辑抽离
- 更多新功能

## Vue3 生命周期

| 选项式 API                             | 组合式 API                        |
| -------------------------------------- | --------------------------------- |
| `beforeCreate`                         | 不需要（直接写到 `setup` 函数中） |
| `created`                              | 不需要（直接写到 `setup` 函数中） |
| `beforeMount`                          | `onBeforeMount`                   |
| `mounted`                              | `onMounted`                       |
| `beforeUpdate`                         | `onBeforeUpdate`                  |
| `updated`                              | `onUpdated`                       |
| `beforeDestroy` Vue 3：`beforeUnmount` | `onBeforeUnmount`                 |
| `destroyed` Vue 3： `unmounted`        | `onUnmounted`                     |
| `errorCaptured`                        | `onErrorCaptured`                 |
| `activated`                            | `onActivated`                     |
| `deactivated`                          | `onDeactivated`                   |

## composition VS options

`composition API` 优点：

- 更好的代码组织
- 更好的逻辑复用
- 更好的类型推导

如何选择：

- 不建议共用，会引起混乱
- 小型项目，业务逻辑简单，用 `Options API`
- 中大型项目，逻辑复杂，用 `Composition API`

**选项式 API**（`Options API`）

- 所有方法都写在 `methods` 中，如果 `data` 中数据越来越多，找数据会非常困难

```html
<template>
  <h1 @click="changeCount">{{ count }}</h1>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      count: 0,
    }
  },
  methods: {
    changeCount() {
      this.count++
    },
  },
}
</script>
```

**组合式 API**（`Composition API`）

- 逻辑会清晰，可以让功能的代码集中抽取到一个函数中进行逻辑复用

```html
<template>
  <h1 @click="changeNum">{{ num }}</h1>
</template>

<script>
import { ref } from 'vue'
function useNum() {
  const num = ref(0)
  function changeNum() {
    num.value++
  }
  return { changeNum, num }
}
export default {
  name: 'App',
  setup() {
    const { changeNum, num } = useNum()
    return {
      changeNum,
      num,
    }
  },
}
</script>
```

## 如何理解 ref toRef 和 toRefs

### ref

- 生成值类型的响应式数据
- 可用于模板和 `reactive`
- 通过 `.value` 修改值

```html
<template>
  <p>值类型响应式：{{ ageRef }} {{ state.name }}</p>
  <p ref="elemRef">templateRef</p>
</template>

<script>
import { ref, reactive, onMounted } from 'vue'

export default {
  name: 'Ref',
  setup() {
    const ageRef = ref(20)
    const nameRef = ref('cat')
    const elemRef = ref('elemRef')
    const state = reactive({
      name: nameRef,
    })
    setTimeout(() => {
      ageRef.value = 30 // .value 修改值
      nameRef.value = 'dog'
    }, 1500)

    onMounted(() => {
      console.log(elemRef.value)
    })
    return {
      ageRef,
      state,
      elemRef,
    }
  },
}
</script>
```

### toRef

- 针对一个响应式对象（`reactive` 封装）的 `prop`
- 创建一个 `ref`，具有响应式
- 两者保持引用关系

```html
<template>
  <p>{{ ageRef }} {{ state.age }}</p>
</template>

<script>
import { toRef, reactive } from 'vue'
export default {
  name: 'ToRef',
  setup() {
    const state = reactive({
      age: 20,
      name: 'cat',
    })
    const ageRef = toRef(state, 'age')
    setTimeout(() => {
      state.age = 25
    }, 1000)
    setTimeout(() => {
      ageRef.value = 30
    }, 3000)
    return { state, ageRef }
  },
}
</script>
```

`toRef` 如果用于普通对象（非响应式对象），产出的结果不具备响应式

```js
const state = {
  age: 20,
  name: 'cat',
}
```

### toRefs

- 将响应式对象（`reactive` 封装）转换为普通对象
- 对象的每个 `prop` 都是对应的 `ref`
- 两者保持引用关系

注意：直接解构 `state` ，页面能显示到但结果不是响应式的

```html
<template>
  <p>{{ age }} {{ name }}</p>
</template>

<script>
import { toRefs, reactive } from 'vue'

export default {
  name: 'ToRef',
  setup() {
    const state = reactive({
      age: 20,
      name: 'cat',
    })
    // 将响应式对象，变为普通对象
    const stateAsRefs = toRefs(state)
    setTimeout(() => {
      state.age = 25
    }, 1000)
    // const { age: ageRef, name: nameRef } = stateAsRefs
    return { ...stateAsRefs }
  },
}
</script>
```

### ref toRef 和 toRefs的最佳使用方式

1. 用 `reactive` 做对象的响应式，用 `ref` 做值类型响应式

   `setup` 中返回 `toRefs(state)` 或者 `toRef(state, 'xxx')`

   `ref` 的变量命名都用 `xxxRef`

2. 合成函数返回响应式对象，使用 `toRefs`

   ```js
   function useFeatureX() {
     const state = reactive({
       x: 1,
       y: 2
     })
     return toRefs(state)
   }
   
   export default {
     name: 'WhyRef',
     setup() {
       const { x, y } = useFeatureX()
       return {
         x,
         y
       }
     }
   }
   ```

### 为何需要 ref

- 返回值类型，会丢失响应式

  在 `setup`、`computed`、合成函数，都有可能返回值类型

  Vue 如不定义 `ref`，用户将自造 `ref`，反而混乱

```html
<template>
  <p>{{ age }}</p>
  <p>{{ age1 }}</p>
</template>

<script>
import { reactive, computed } from 'vue'

export default {
  name: 'WhyRef',
  setup() {
    const state = reactive({
      age: 20,
      name: 'dog',
    })
    // computed返回的是一个类似于ref的对象，也有.value
    const age1 = computed(() => {
      return state.age + 1
    })
    setTimeout(() => {
      state.age = 25
    }, 1000)
    return {
      ...state, // 这样不是响应式的
      age1,
    }
  },
}
</script>
```

### 为何需要 .value

- `ref` 是一个对象（不丢失响应式），`value` 存储值
- 通过 `.value` 属性的 `get` 和 `set` 实现响应式
- 用于模板、`reactive` 时，不需要 `.value`，其他情况都需要

简单理解 `computed` 运算逻辑

```js
// 错误
function computed(getter) {
  let value
  watchEffect(() => { // 可以改为setTimeout进行模拟测试
    value = getter()
  })
  return value
}

// 正确
function computed(getter) {
  const ref = {
    value: null,
  }
  watchEffect(() => {
    ref.value = getter()
  })
  return ref
}
```

### 为何需要 toRef 和 toRefs

- 初衷：不丢失响应式的情况下，把对象数据 **分解/扩展**
- 前提：针对的是响应式对象（`reactive` 封装）非普通对象
- 注意：**不创造** 响应式，而是 **延续** 响应式

## Vue3 升级了哪些重要功能

> [Vue3 官网](https://v3.cn.vuejs.org/)

- `createApp`
- `emits` 属性
- 生命周期
- 多事件
- `Fragment`
- 移除 `.sync`
- 异步组件的写法
- 移除 `filter`
- `Teleport`
- `Suspense`
- `Composition API`

### createApp

```js
// vue2.x
const app = new Vue({ /* ... */ })
Vue.use(/* ... */)
Vue.mixin(/* ... */)
Vue.component(/* ... */)
Vue.directive(/* ... */)

// vue3
const app = Vue.createApp({ /* ... */ })
app.use(/* ... */)
app.mixin(/* ... */)
app.component(/* ... */)
app.directive(/* ... */)
```

### emits 属性

- 父组件

```html
<template>
  <son :msg="msg" @onSayHello="sayHello" />
</template>

<script>
import Son from './views/Son.vue'
export default {
  components: { Son },
  data() {
    return {
      msg: 'hello vue3',
    }
  },
  methods: {
    sayHello(info) {
      console.log('hello', info)
    },
  },
}
</script>
```

- 子组件

```html
<template>
  <p>{{ msg }}</p>
</template>

<script>
export default {
  props: {
    msg: String,
  },
  emits: ['onSayHello'],
  setup(props, { emit }) {
    emit('onSayHello', 'vue3')
  },
}
</script>
```

### 多事件处理

```html
<button @click="one($event), two($event)">Submit</button>
```

### Fragment

```html
<!-- vue2.x组件模板 -->
<template>
  <div>
    <p>{{ msg }}</p>
    <p>{{ content }}</p>
  </div>
</template>

<!-- vue3组件模板 -->
<template>
  <p>{{ msg }}</p>
  <p>{{ content }}</p>
</template>
```

### 移除 .sync

```html
<!-- vue2.x -->
<MyComponent :title.sync="title" />

<!-- vue3.x -->
<MyComponent v-model:title="title" />
```

### 异步组件

```js
// vue2.x
new Vue({
  components: {
    'my-component': () => import('./async-com.vue'),
  },
})

// vue3.x
import { createApp, defineAsyncComponent } from 'vue'
createApp({
  components: {
    AsyncComponent: defineAsyncComponent(() => import('./async-com.vue')),
  },
})
```

### 移除 filter

```html
<!-- vue2.x -->
<div>{{ message | capitalize }}</div>

<!-- vue3.x -->
<div :id="rawId | formatId">/div>
```

### Teleport

```html
<button @click="modalOpen = true">Open full screen modal!</button>
<teleport to="body">
  <div v-if="modalOpen" class="modal">
    <div>
      telePort 弹窗（父元素是 body）
      <button @click="modalOpen = false">Close</button>
    </div>
  </div>
</teleport>
```

### Suspense

```html
<Suspense>
  <template>
    <!-- 是一个异步组件 -->
    <Test1 />
  </template>
  <!-- #fallback 就是一个具名插槽。即Suspense组件内部，有两个slot，其中一个具名为fallback -->
  <template #fallback> Loading.. </template>
</Suspense>
```

### Composition API

- `reactive`
- `ref` 相关
- `readonly`
- `watch` 和 `watchEffect`
- `setup`
- 生命周期钩子函数

## Composition API 实现逻辑复用

- 抽离逻辑代码到一个函数
- 函数命名约定为 `useXxx` 格式（React Hooks 也是）
- 在 `setup` 中引用 `useXxx` 函数

```html
<template>
  <p>mouse position {{ x }} {{ y }}</p>
</template>

<script>
import useMousePosition from './useMousePosition'

export default {
  name: 'MousePosition',
  setup() {
    const { x, y } = useMousePosition()
    return {
      x,
      y,
    }
  },
}
</script>
```

- `useMousePosition.js`

```js
import { ref, onMounted, onUnmounted } from 'vue'

function useMousePosition() {
  const x = ref(0)
  const y = ref(0)
  function update(e) {
    x.value = e.pageX
    y.value = e.pageY
  }
  onMounted(() => {
    window.addEventListener('mousemove', update)
  })
  onUnmounted(() => {
    window.removeEventListener('mousemove', update)
  })
  return { x, y }
}

export default useMousePosition
```

如果不使用 `ref` 使用 `reactive`，需要将整个 `reactive` 暴露出去。在父组件接收的时候不能直接解构，否则会失去响应式

## Vue3 如何实现响应式

`Object.defineProperty` 的缺点：

- 深度监听需要一次性递归
- 无法监听新增属性/删除属性（`Vue.set`、`Vue.delete`）
- 无法原生监听数组，需要特殊处理

```js
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
```

`Proxy` 实现响应式优点

```js
// 测试数据
const data = {
  name: 'bird',
  age: 20,
  info: {
    city: 'beijing',
  },
}

function reactive(target = {}) {
  if (typeof target !== 'object' || target == null) {
    // 不是对象或数组，则返回
    return target
  }
  // 代理配置
  const proxyConf = {
    get(target, key, receiver) {
      // 只处本身（非原型）属性
      const ownKeys = Reflect.ownKeys(target)
      if (ownKeys.includes(key)) {
        console.log('get', key) // 监听
      }
      const result = Reflect.get(target, key, receiver)
      // 惰性深度监听。什么时候用什么时候监听
      return reactive(result) // 返回结果
    },
    set(target, key, val, receiver) {
      // 重复的数据，不处理
      if (val === target[key]) {
        return true
      }
      const ownKeys = Reflect.ownKeys(target)
      if (ownKeys.includes(key)) {
        console.log('已有的key', key)
      } else {
        console.log('新增的key', key)
      }
      const result = Reflect.set(target, key, val, receiver)
      console.log('set', key, val)
      return result // 是否设置成功
    },
    deleteProperty(target, key) {
      const result = Reflect.deleteProperty(target, key)
      console.log('delete property', key)
      return result // 是否删除成功
    },
  }
  // 生成代理对象
  const observed = new Proxy(target, proxyConf)
  return observed
}
const proxyData = reactive(data)
```

`Reflect` 的作用：

- 和 `Proxy` 能力一一对应

-  规范化、标准化、函数式

  `'a' in obj` -> `Reflect.has(obj, 'a')`

  `delete obj.b` -> `Reflect.deleteProperty(obj, 'b')`

- 替代 `Object` 上的工具函数

  `Object.getOwnPropertyNames(obj)` -> `Reflect.ownKeys(obj)`

**总结**

- `Proxy` 能规避 `Object.defineProperty` 的问题
- `Proxy` 无法兼容所有浏览器，无法 `polyfill`

## v-model 参数用法

Vue2 的 `.sync` 修饰符

```html
<text-document v-bind:title.sync="doc.title" />

<!-- 语法糖 -->
<text-document
  v-bind:title="doc.title"
  v-on:update:title="doc.title = $event"
/>
```

Vue3 的 `v-model` 

```html
<ChildComponent :title.sync="pageTitle" />

<!-- 替换为 -->
<ChildComponent v-model:title="pageTitle" />
```

`v-model` 相当于传递了 `modelValue` prop 并接受抛出的 `update:modelValue` 事件

```html
<template>
  <p>{{ name }} {{ age }}</p>
  <user-info v-model:name="name" v-model:age="age" />
</template>

<script>
import { reactive, toRefs } from 'vue'
import UserInfo from './UserInfo.vue'
export default {
  components: { UserInfo },
  setup() {
    const state = reactive({
      name: 'bird',
      age: '20',
    })
    return toRefs(state)
  },
}
</script>
```

- `UserInfo.vue`

```html
<template>
  <input :value="name" @input="$emit('update:name', $event.target.value)" />
  <input :value="age" @input="$emit('update:age', $event.target.value)" />
</template>

<script>
export default {
  props: {
    name: String,
    age: String,
  },
}
</script>
```

## watch 和 watchEffect 的区别

- 两者都可监听 `data` 属性变化

- `watch` 需要明确监听哪个属性

  默认是惰性执行，监听源可以是一个具有返回值的 `getter` 函数，也可以直接是一个 `ref`

- `watchEffect` 会根据其中的属性，自动监听其变化

```html
<template>
  <p>{{ numberRef }}</p>
  <p>{{ name }} {{ age }}</p>
</template>

<script>
import { reactive, toRefs, ref, watch, watchEffect } from 'vue'
export default {
  setup() {
    const numberRef = ref(10)
    const state = reactive({
      name: 'bird',
      age: 20,
    })
    watchEffect(() => {
      // 初始化时，一定会执行一次（收集需要监听的数据）
      console.log('watchEffect', state.age)
    })
    watchEffect(() => {
      console.log('watchEffect', numberRef)
    })
    // watch监听ref属性
    watch(numberRef, (newNum, oldNum) => {
      console.log('ref watch', newNum, oldNum)
    })
    // watch监听state属性
    watch(
      // 1.确定监听哪个属性
      () => state.age,
      // 2.回调函数
      (newState, oldState) => {
        console.log('state watch', newState, oldState)
      },
      // 3.配置项
      {
        immediate: true, // 初始化之前就监听
        // deep: true // 深度监听
      }
    )
    setTimeout(() => {
      numberRef.value = 100
    }, 1000)
    setTimeout(() => {
      state.age = 25
    }, 1500)
    return { numberRef, ...toRefs(state) }
  },
}
</script>
```

## setup 中如何获取组件实例

- 在 `setup` 和其他 `Composition API` 中没有 `this`
- 可通过 `getCurrentInstance` 获取当前实例
- 若用 `Options API` 可照常使用 `this`

```html
<template>
  <p>getInstance</p>
</template>

<script>
import { getCurrentInstance, onMounted } from 'vue'
export default {
  data() {
    return {
      x: 1,
      y: 2,
    }
  },
  setup() {
    console.log('setup this', this) // undefined
    onMounted(() => {
      console.log('onMounted this', this) // undefined
      console.log('x', instance.data.x) // 1
    })
    const instance = getCurrentInstance()
    console.log('instance', instance) // 组件实例
  },
  mounted() {
    console.log('mounted this', this) // Proxy 实例
    console.log('y', this.y) // 2
  },
}
</script>
```

## Vue3 为何比 Vue2 块

- `proxy` 响应式
- `PatchFlag`
- `hoistStatic`
- `cacheHandler`
- `SSR` 优化
- `tree-shaking`

### PatchFlag（标记）

- 编译模板时，动态节点做标记
- 标记，分为不同的类型，如：`TEXT`、`PROPS`
- `diff` 算法时，可以区分静态节点，以及不同类型的动态节点

> [Vu3 在线编译](https://vue-next-template-explorer.netlify.app/)
>
> [Vue2 在线编译](https://vue-template-explorer.netlify.app/)

Vue2 和 Vu3 diff 算法比较

- Vue2 没有区分静态节点和动态节点

![](https://gitee.com/lilyn/pic/raw/master/jslearn-img/vue2vue3diff.png)

- Vue3 新增静态标记 `patchFlag` 与上次虚拟节点比较时，只比较有  `patchFlag`  的节点

```html
<div>
  <div>1</div>
  <div>2</div>
  <div>{{ name }}</div>
</div>

<script>
export function render() {
  return (
    _openBlock(),
    _createBlock('div', null, [
      _createVNode('div', null, '1'),
      _createVNode('div', null, '2'),
      _createVNode('div', null, _toDisplayString(_ctx.name), 1 /* TEXT */),
    ])
  )
}
</script>
```

### hoistStatic（静态提升）

- 将静态节点的定义，提升到父作用域，缓存起来
- 多个相邻静态节点，会被合并起来
- 典型的拿空间换时间的优化策略

Vue2 无论元素是否参与更新，每次都会重新创建然后再渲染。Vue3 对于不参与更新的元素，做静态提升，只会被创建一次，在渲染时直接复用即可

```html
<div>
  <div>1</div>
  <div>2</div>
  <div>{{ name }}</div>
</div>

<script>
const _hoisted_1 = /*#__PURE__*/ _createVNode('div', null, '1', -1 /* HOISTED */)
const _hoisted_2 = /*#__PURE__*/ _createVNode('div', null, '2', -1 /* HOISTED */)

export function render() {
  return (
    _openBlock(),
    _createBlock('div', null, [
      _hoisted_1,
      _hoisted_2,
      _createVNode('div', null, _toDisplayString(_ctx.name), 1 /* TEXT */),
    ])
  )
}
</script>
```

### cacheHandler（缓存事件）

- 缓存事件

Vue2 绑定事件每次触发都要重新生成全新 Function 去更新。Vue3 提供事件缓存对象，当开启 `cacheHandler` 会自动生成一个内联函数，同时生成一个静态节点，当事件再次触发时，只需从缓存中调用即可

```html
<div>
  <div @click="todo">something</div>
</div>

<script>
export function render() {
  return (
    _openBlock(),
    _createBlock('div', null, [
      _createVNode(
        'div',
        {
          onClick: _cache[1] || (_cache[1] = (...args) => _ctx.todo(...args)),
        },
        'something'
      ),
    ])
  )
}
</script>

```

### SSR 渲染

- 静态节点直接输出，直接绕过 `vdom`，当做字符串推进 `buffer` 里
- 动态节点，还是需要动态渲染

### tree shaking（按需编译）

- 编译时，根据不同的情况，引入不同的 API

  可以静态分析模块依赖并删除未使用的导出相关的代码

## Vite

- 一个前端打包工具，Vue 作者发起的项目
- 借助 Vue 的影响力，发展较快，和 webpack 竞争
- 优势：开发环境下无需打包，启动快

**Vite 为何启动快**

- 开发环境使用 ES6 Module，无需打包——非常快
- 生成环境使用 rollup，并不会快很多

### CommonJs

> [「万字进阶」深入浅出 Commonjs 和 Es Module](https://juejin.cn/post/6994224541312483336#heading-18)

**CommonJs 实现原理**

- 在编译的过程中，实际 CommonJs 对 JS 的代码进行了首尾包装，被包装之后的样子如下：

```js
;(function (exports, require, module, __filename, __dirname) {
  const sayName = require('./hello.js')
  module.exports = function say() {
    return {
      name: sayName(),
      author: '我不是外星人',
    }
  }
})
```

- 在 CommonJs 规范下模块中，会形成一个包装函数，包装函数本质如下：

  最后将包装函数执行

```js
function wrapper(script) {
  return '(function (exports, require, module, __filename, __dirname) {' + script + '\n})'
}
```

**require 加载原理**

- `module`：在 Node 中每一个 JS 文件都是一个 `module`，`module` 上保存了 `exports` 等信息之外，还有一个 `loaded` 表示该模块是否被加载
- `Module`：以 Node 为例，整个系统运行之后，会用 `Module` 缓存每一个模块加载的信息

```js
// id 为路径标识符
function require(id) {
  /* 查找  Module 上有没有已经加载的 js  对象*/
  const cachedModule = Module._cache[id]

  /* 如果已经加载了那么直接取走缓存的 exports 对象  */
  if (cachedModule) {
    return cachedModule.exports
  }

  /* 创建当前模块的 module  */
  const module = { exports: {}, loaded: false }

  /* 将 module 缓存到  Module 的缓存属性中，路径标识符作为 id */
  Module._cache[id] = module
  /* 加载文件 */
  runInThisContext(wrapper('module.exports = "123"'))(
    module.exports,
    require,
    module,
    __filename,
    __dirname
  )
  /* 加载完成 */
  module.loaded = true
  /* 返回值 */
  return module.exports
}
```

`require` 大致流程如下：

1. `require` 会接收一个参数——文件标识符，然后分析定位文件，接下来会从 `Module` 上查找有没有缓存，如果有缓存，那么直接返回缓存的内容
2. 如果没有缓存，会创建一个 `module` 对象，缓存到 `Module` 上，然后执行文件，加载完文件后，将 `loaded` 设置为 `true`
3. `exports` 和 `module.exports` 持有相同的引用，所以对 `exports` 赋值会导致 `exports` 操作的不再是 `module.exports` 的引用

### ES Module

ES6 之后，JS 才有了真正意义上的模块化规范。ES Module 的优势：

- 借助 ES Module 的静态导入导出的优势，实现了 `tree shaking`
- ES Module 还可以 `import()` 懒加载方式实现代码分割

**基本使用**

- 使用 `export default` 导出

```html
<script type="module">
  import add from './add.js'
  console.log(add(10, 20)) // 30
</script>

<script>
// add.js
export default function add(a, b) {
  return a + b
}
</script>
```

- 使用 `export` 导出，需要解构

```html
<script type="module">
  import { add } from './math.js'
  console.log(add(10, 20)) // 30
</script>

<script>
// math.js
export function add(a, b) {
  return a + b
}
</script>
```

**外链使用**

```html
<script type="module">
  import { createStore } from 'https://unpkg.com/redux@latest/es/redux.mjs'
  console.log(createStore) // Function
</script>
```

**动态引入**

```html
<button id="btn1">load1</button>

<script type="module">
  document.getElementById('btn1').addEventListener('click', async () => {
    const res = await import('./math.js')
    console.log(res.add(10, 20)) // 30
  })
</script>

<script>
// math.js
export function add(a, b) {
  return a + b
}
</script>
```

## Composition API 和 React Hooks 对比

- Composition API 只会被调用一次，React Hooks 函数会被多次调用
- Composition API 无需 `useMemo`、`useCallback`，因为 `setup` 只调用一次
- Composition API 无需顾虑调用顺序，React Hooks 需要保证 `hooks` 的顺序一致
- Composition API `reactive + ref` 比 React Hooks  要难理解

