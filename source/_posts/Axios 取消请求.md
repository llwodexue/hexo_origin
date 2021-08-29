---
title: Axios 取消请求
tags:
  - Vue
  - Axios
  - 二次封装
categories: Vue
author: LiLyn
copyright: ture
abbrlink: a688a8da
---

## 应用场景

取消请求偶尔会用到，以下是两个工作中可能用到的场景

1. 如果一个数据请求量比较大（可能会请求错误），还没请求完就切换路由，可能会出现错误的提示框（响应拦截器中配置错误提示）
2. 导出文件或下载文件时，中途取消
3. 一个请求请求量比较大，发送新请求时需要取消上一个请求

<!--more-->

## 取消单个请求（结合生命周期使用）

- 页面销毁时取消请求
- 重复发送请求取消上一次请求

```js
import axios from 'axios'
import { generatePlan } from '@/api'

export default {
  destroyed() {
    this.cancelPost()
  },
  methods: {
    generateList() {
      this.$confirm('生成时间比较长，您是否确定生成？', '警告', {
        confirmButtonText: '确定',
        cancaelButtonText: '取消',
      })
        .then(() => {
          const self = this
          this.cancelPost()
          // post请求
          return generatePlan({
            params: this.listQuery,
            cancelToken: new axios.CancelToken(function exector(c) {
              self.$cancel = c
            }),
          }).then(res => {
            if (res.code === 200) this.list = res.data.result
          })
        })
        .catch(() => {})
    },
  },
  cancelPost() {
    typeof this.$cancel === 'function' ? this.$cancel('取消请求') : null
  },
}
```

## 取消所有请求（结合导航守卫）

思路：在请求拦截器中，给所有请求加一个 token，设置全局变量 source 控制 cancel token，在路由变化时调用 cancel 方法

```js
http.interceptors.request.use(
  config => {
    config.cancelToken = store.source.token
    return config
  },
  error => {
    return Promise.reject(error)
  }
)

router.beforeEach((to, from, next) => {
  const CancelToken = axios.CancelToken
  store.source.cancel && store.source.cancel()
  store.source = CancelToken.source()
  next()
})
```

## axios 取消请求原理

axios 是对 XMLHttpRequest 的封装，使用 XMLHttpRequest 实例的 abort()方法

```js
// https://github.com/axios/axios/blob/master/lib/adapters/xhr.js
if (config.cancelToken) {
  // Handle cancellation
  config.cancelToken.promise.then(function onCanceled(cancel) {
    if (!request) {
      return
    }

    request.abort()
    reject(cancel)
    // Clean up request
    request = null
  })
}
```