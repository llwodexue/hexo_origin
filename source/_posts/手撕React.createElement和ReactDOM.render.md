---
title: 手撕React.createElement和ReactDOM.render
tags:
  - React
  - createElement
  - render
  - 源码分析
categories: React
author: LiLyn
copyright: ture
abbrlink: 7e319c36
---

### 准备工作

```bash
npm install create-react-app -g
create-react-app demo
cd demo
yarn start
```

<!--more-->

- 这时已经新建好了一个 react 项目，接下来 index.js 写入如下代码

```html
console.log(
  <h1 id="h1" className="title" key="1">
    hello world
    <span style={{ color: 'red' }}>cat</span>
  </h1>
)
```

- 如下对象就是一个 React 对象，也就是虚拟 DOM

![虚拟dom对象示例](https://img-blog.csdnimg.cn/img_convert/723b629d0e10b5bd476871bed912c40b.png)

- 接下来我们打开[babel 官网](https://www.babeljs.cn/repl)，勾选 `react` ，粘贴上去。通过 babel 把 jsx 对象转换成 React 中的 createElement 函数

![babel_react](https://img-blog.csdnimg.cn/img_convert/c9c1436f832951ebbe63cb587d77585f.png)

- 在项目中写入如下代码

```js
import React from 'react'
import ReactDOM from 'react-dom'

ReactDOM.render(
  <h1 id="h1" className="title" key="1">
    hello world
    <span style={{ color: 'red' }}>cat</span>
  </h1>,
  document.getElementById('root')
)
```

- 效果图如下：

![渲染示例](https://img-blog.csdnimg.cn/img_convert/1e643e0830b8155a555ee3261eed6a4a.png)

**过程：**

1. 通过 babel（babel-preset-react-app）转换成 jsx 对象
2. 默认会把 createElement 函数执行，得到 JSX 对象（虚拟 DOM 对象）
   - 第一个参数：type，后期要创建元素的标签名（或是组件）
   - 第二个参数：props，属性对象，包含当前元素标签上设置的各个属性及属性值（不设置，props 值是 null）
   - 第三个以后的参数：children，当前元素的所有子节点（没有写，则不存在这个参数），如果有，有几个就依次传递几个
     - 如果是文本节点，则直接传递文本内容
     - 如果是元素节点，则把元素依次变为 createElement 格式，把执行的返回值，作为参数传递进来
3. ReactDOM.render，把虚拟 DOM 对象转换为真实 DOM 对象

### React 中 createElement 函数

**React.createElement 返回值**

```js
{
  $$typeof: Symbol(react.element),
  key: null,
  ref: null,
  // 存储的是传递给createElement的第一个参数
  type: 'h1',
  // 首先会把传递给createElement的属性对象，一项项的赋值给对象的props
  props: {
    // 并且返回对象的prop还包含children，记录当前元素的子节点（可能是一个值{字符串或JSX返回的对象}，如果有多个子节点，则其是一个数组）
    children: 'hello world',
    className: 'title',
    id: 'title',
  }, // 即使没有传递任何属性，也没有任何的子节点，返回对象的props也是一个{}
}
```

- 这里需要注意，key 和 ref 虽然都在 props 里，但是这两个不在 props 里添加，而与 props 同级

```js
React.createElement = function createElement(type, props, ...children) {
  let len = children.length,
    obj = {
      type,
      props: {},
      key: null,
      ref: null,
    }
  // 处理传递进来的属性
  if (props !== null && typeof props === 'object') {
    each(props, (value, key) => {
      if (/^(key|ref)$/.test(key)) {
        obj[key] = value[key]
        return
      }
      obj.props[key] = value[key]
    })
    // for in 循环性能较差
    /* for (let key in props) {
      if (!props.hasOwnProperty(key)) break
      if (/^(key|ref)$/.test(key)) {
        obj[key] = props[key]
        continue
      }
      obj.props[key] = props[key]
    } */
  }
  // 处理子节点
  if (len > 0) {
    obj.props.children = len === 1 ? children[0] : children
  }
  return obj
}
```

- 由于 `for...in...` 循环性能较差，这里稍微优化一下（封装一个 each 方法）

```js
const each = function each(obj, callback) {
  if (obj == null || typeof obj !== 'object')
    throw new TypeError('obj must be an object')
  let keys = Object.keys(obj),
    key,
    i = 0
  if (typeof Symbol !== 'undefined') {
    keys = keys.concat(Object.getOwnPropertySymbols(obj))
  }

  for (; i < keys.length; i++) {
    key = keys[i]
    if (typeof callback === 'function') {
      callback(obj[key], key)
    }
  }
}
```

### ReactDOM 中 render 函数

```js
ReactDOM.render = function render(obj, container, callback) {
  let { type, props } = obj,
    element
  // 核心思想：动态创建指定类型的元素，插入到指定容器当中
  if (typeof type === 'string') {
    element = document.createElement(type)
    each(props, (value, key) => {
      if (key === 'className') {
        element.setAttribute('class', value)
        return
      }
      if (key === 'style') {
        // 把style对象中的每一项分别给元素设置样式 value:style对象
        each(value, (item, attr) => {
          element.style[attr] = item
        })
        return
      }
      if (key === 'children') {
        // 肯定有子节点 value:children属性值
        if (!Array.isArray(value)) value = [value] // 让它都是数组
        value.forEach((item, index) => {
          // item每个子节点
          // 1.文本子节点：直接创建一个文本节点，插入到element中
          // 2.元素子节点：新的jsx对象（createElement），我们需要把它也创建成为一个元素标签，插入到element中即可
          if (typeof item === 'string') {
            let textNode = document.createTextNode(item)
            element.appendChild(textNode)
            return
          }
          // 递归处理
          render(item, element)
        })
        return
      }
      element.setAttribute(key, value)
    })
    container.appendChild(element)
    if (typeof callback === 'function') {
      callback()
    }
    return
  }
  // 如果type是一个组件，有不同的处理方案
}
```