---
title: JS 实现 sizeOf 函数，计算 object 占用了多少个 bytes
tags:
  - 字节数
  - JavaScript
categories: JavaScript
author: LiLyn
copyright: ture
---
## 实现 sizeOf 函数，传入一个 object，计算这个 Object 占用了多少个 bytes

可以参考：[https://github.com/miktam/sizeof](https://github.com/miktam/sizeof)

- Number：一个数字 8 字节（64 位存储）
- String：一个字符 2 字节
- Boolean：4 个字节

<!--more-->

```js
const same = {}
const testData = {
  a: 111,
  b: 'ccc',
  222: false,
  c: same,
  d: same,
}

const seen = new WeakSet()
function sizeOfObj(obj) {
  if (obj === null) return 0

  let bytes = 0
  // 对象里的key也是占用内存空间的
  const props = Object.keys(obj)
  for (let i = 0; i < props.length; i++) {
    const key = props[i]
    // 无论value是否重复，都需要计算key
    bytes += calculator(key)
    if (typeof obj[key] === 'object' && obj[key] !== null) {
      // 这里需要注意value使用相同内存空间（只需计算一次内存）
      if (seen.has(obj[key])) continue
      seen.add(obj[key])
    }
    bytes += calculator(obj[key])
  }
  return bytes
}

function calculator(obj) {
  const objType = typeof obj
  switch (objType) {
    case 'string':
      return obj.length * 2
    case 'boolean':
      return 4
    case 'number':
      return 8
    case 'object':
      if (Array.isArray(obj)) {
        // 数组处理 [1,2] [{x:1},{y:2}]
        return obj.map(calculator).reduce((res, cur) => res + cur, 0)
      } else {
        // 对象处理
        return sizeOfObj(obj)
      }
    default:
      return 0
  }
}
console.log(calculator(testData)) // 32
```

