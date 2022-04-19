---
title: Axios 二次封装
tags:
  - Vue
  - Axios
  - 封装
categories: Vue
author: LiLyn
copyright: ture
abbrlink: 84a23fec
---

## Axios 二次封装

> 目的：把当前项目中，所有请求的公共部分进行统一处理

- `axios.defaults` 设置公共的配置项
- `axios.interceptors` 基于拦截器做统一处理

<!--more-->

配置请求接口的统一前缀

- 开发 `development`
- 测试 `test`
- 灰度 `grayscale`
- 生成 `production`

```js
import axios from 'axios'
import Qs from 'qs'
import router from '@/router'
import { Message } from 'element-ui'
import Cookies from 'js-cookie'

axios.defaults.baseURL =
  process.env.NODE_ENV === 'development' ? '' : 'http://127.0.0.1:9999'
axios.defaults.withCredentials = true // 设置跨域请求中是否携带资源凭证
axios.defaults.timeout = 1000
axios.defaults.headers.post['Content-Type'] =
  'application/x-www-form-urlencoded;charset=UTF-8' // 配置公共的自定义请求头信息

// POST系列请求对于请求主体信息的统一格式化
axios.defaults.transformRequest = function (data, headers) {
  if (data === null || typeof data !== 'object') return data
  let contentType = headers['Content-Type'] || headers.post['Content-Type']
  if (contentType.includes('urlencoded')) return Qs.stringify(data)
  if (contentType.includes('json')) return JSON.stringify(data)
  return data
}
// 设置响应状态码的校验处理{状态码哪些是算请求成功，哪些算失败}
axios.defaults.validateStatus = function (status) {
  return status >= 200 && status < 400
}

// 请求拦截器，在向服务器发送请求之前，拦截到现有的配置，再去做一些统一修改
axios.interceptors.request.use(
  config => {
    // 是否需要设置 token
    const token = Cookies.get('token')
    if (token) {
      config.headers['Authorization'] = token
    }
    return config
  },
  error => {
    console.log(error) // for debug
    return Promise.reject(error)
  }
)

// 响应拦截器，当前请求有结果之后，做一些成功或者失败的统一提示处理等...
axios.interceptors.response.use(
  response => {
    // 服务器正常返回结果 & validateStatus状态码校验成功
    return response.data
  },
  error => {
    // 取消请求也会在这里拦截到
    if (error.message === '取消请求') return Promise.resolve('取消请求')
    /*
    失败情况：
    1.服务器返回了结果但是状态码没有经过validateStatus校验
    2.服务器压根没有返回任何的结果
    3.请求中断或者超时
    */
    let response = error.response
    if (response) {
      // @1
      switch (response.status) {
        case 401:
          router.push({ path: '/login' })
          break
        case 500:
          router.push({ path: '/login' })
          break
        // ...
      }
      Message({
        message: '访问失败，请联系系统管理员',
        type: 'warning',
        duration: 3 * 1000,
      })
    } else {
      if (error && error.code === 'ECONNABORTED') {
        Message({
          message: '服务器拒绝了您的请求',
          type: 'warning',
          duration: 3 * 1000,
        })
      }
      if (!navigator.onLine) {
        Message({
          message: '设备已离线',
          type: 'warning',
          duration: 3 * 1000,
        })
      }
    }
    return Promise.reject(error)
  }
)

export default axios
```