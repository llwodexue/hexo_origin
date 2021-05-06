---
title: Vue 自定义指令应用场景
tags:
  - Vue
  - 自定义指令
  - 权限校验
  - 复制
  - 防抖节流
categories: Vue
author: LiLyn
copyright: ture
abbrlink: 13e591df
---

## Vue 自定义指令应用场景

> 这段是从官网 copy 过来的，相信应该都一看就明白的

- bind: 只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置
- inserted: 被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)
- update: 所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前。指令的值可能发生了改变，也可能没有
- componentUpdated：指令所在组件的 VNode 及其子 VNode 全部更新后调用
- unbind: 只调用一次，指令与元素解绑时调用

<!--more-->

### 1.权限校验 v-permission

权限校验一般分为页面级别和按钮级别，其两种思路基本一致

详细可以参考：[手摸手，带你用 vue 撸后台 系列二(登录权限篇)](https://juejin.cn/post/6844903478880370701)，如下简单说一下

**页面级别：**

1. 最快想到的可能就是使用 beforeEach，首先把所有路由都注册，用户登录后拿到后端返回的权限信息，如果没有此权限统一个提示信息
   不过大部分需求都不希望你这么做，没有权限就不显示
2. 首先把所有路由都注册，不过上来给所有路由做一个标识（前后端约定一下），根据这个标识显示/隐藏
3. 路由完全由后端负责，按照后端返回的信息生成最终用户可访问的路由表，最后通过 `router.addRoutes` 动态挂载

**按钮级别：**

1. 逻辑比较简单，使用 v-if 根据权限显示/隐藏
2. 逻辑稍微复杂一点，使用自定义指令显示/隐藏

**思路：**

1. 自定义一个权限数组（这里我放到 vuex 中）
2. 判断用户的权限是否在这个数组内，如果在就显示，不在就移除

```js
import store from '@/store'
function checkPermission(el, binding) {
  const { value } = binding
  const roles = store.getters && store.getters.roles

  if (value && Array.isArray(value)) {
    if (value.length > 0) {
      const hasPermission = roles.some(role => value.includes(role))
    }
    if (!hasPermission) {
      el.parentNode && el.parentNode.removeChild(el)
    }
  } else {
    throw new Error(`please use such as v-permission="['admin', 'editor']"`)
  }
}

Vue.directive('permission', {
  inserted(el, binding) {
    checkPermission(el, binding)
  },
  update(el, binding) {
    checkPermission(el, binding)
  },
})
```

### 2.复制指令

**思路：**

1. 动态创建 `textarea` 标签，并设置 `readOnly` 属性，并将其移出可视区域
2. 将要复制的值赋给 `textarea` 标签的 `value` 属性，并插入到 body
3. 选中值 `textarea` 并复制
4. 将 body 中插入的 `textarea` 移除
5. 在第一次调用时绑定事件，在解绑时移除事件

```js
import { Message } from 'element-ui'

Vue.directive('copy', {
  bind(el, { value }) {
    el.$value = value
    el.$handle = function () {
      let textValue = el.$value
      if (!text) {
        Message({
          message: '复制内容不能为空',
          type: 'warning',
        })
        return
      }
      // 步骤1
      const textarea = document.createElement('textarea')
      textarea.readOnly = 'readonly'
      textarea.style.cssText = 'opacity:0;position:fixed;left:-9999px'
      // 步骤2
      textarea.value = textValue
      document.body.appendChild(textarea)
      // 步骤3
      textarea.select()
      const copyText = document.execCommand('copy')
      if (copyText) {
        Message({
          message: '复制成功',
          type: 'success',
        })
      }
      // 步骤4
      document.body.removeChild(textarea)
    }
    el.addEventListener('click', el.$handle)
  },
  // 当传入进来的值更新的时候触发
  componentUpdated(el, { value }) {
    el.$value = value
  },
  unbind(el) {
    el.removeEventListener('click', el.$handle)
  },
})
```

### 3.防抖节流

钩子函数参数

- `arg`：传给指令的参数，可选。例如 `v-my-directive:foo` 中，参数为 `"foo"`
- `modifiers`：一个包含修饰符的对象。例如：`v-my-directive.foo.bar` 中，修饰符对象为 `{ foo: true, bar: true }`

**思路：**

- 通过 `addEventListener`监听事件
- 调用方式：`<button v-debounce:300.immediate="fn">click</button>`

```js
function debounce(func, wait, immediate) {
    var timer = null,
        result;
    return function proxy() {
        var self = this,
            params = [].slice.call(arguments),
            callNow = !timer && immediate;
        if (timer) clearTimeout(timer);
        timer = setTimeout(function () {
            clearTimeout(timer);
            timer = null;
            if (!immediate) result = func.apply(self, params);
        }, wait);
        if (callNow) result = func.apply(self, params);
        return result;
    }
}

function throttle(func, wait) {
    var timer = null,
        previous = 0,
        result;
    return function proxy() {
        var self = this,
            params = [].slice.call(arguments);
        var now = +new Date,
            remaining = wait - (now - previous);
        if (remaining <= 0) {
            if (timer) {
                clearTimeout(timer);
                timer = null;
            }
            previous = now;
            result = func.apply(self, params);
            return result;
        }
        if (!timer) {
            timer = setTimeout(function () {
                clearTimeout(timer);
                timer = null;
                previous = +new Date;
                result = func.apply(self, params);
            }, remaining);
        }
        return result;
    };
}

const config = {
  bind(el, binding) {
    let { value } = binding.value,
      immediate = false,
      wait = 500,
      type = 'click',
      params = [],
      func,
      handle = binding.name === 'debounce' ? debounce : throttle
    if (value === null) return
    if (typeof val !== 'object' && typeof value !== 'function') return
    if (binding.arg) wait = +binding.arg
    if (binding.modifiers && binding.modifiers.immediate) immediate = true
    if (typeof value === 'function') func = value
    if (typeof value === 'object') {
      func = value.func || function() {}
      type = value.type || 'click'
      params = value.params || []
    }
    el.$type = type
    el.$handle = handle(
      function proxy(...arg) {
        return func.call(this, ...params.concat(arg))
      },
      wait,
      immediate
    )
    el.addEventListener(el.$type, el.$handle)
  },
  unbind(el) {
    el.removeEventListener(el.$type, el.$handle)
  },
}

Vue.directive('debounce', config)
Vue.directive('throttle', config)
```

