---
title: Vue 处理文件的下载（后端Excel导出）
tags:
  - Vue
  - Blob
categories: Vue
author: LiLyn
copyright: ture
abbrlink: 14d3abe4
---

大概有两种方法（通常对应的是需要不需要携带 token），原理都是通过 `a` 标签下载

1. 通过 Ajax 请求，拿到 `response` ，转换为 blob 格式（主要是为了处理 type），为其生成下载链接，下载即可
2. 拼接 URL，拼出来对应请求链接，直接访问即可

<!--more-->

## 后端文件流

- 首先点击导出 Excel ，这里调用接口成功

![](https://gitee.com/lilyn/pic/raw/master/company-img/%E5%AF%BC%E5%87%BAExcel%E6%8E%A5%E5%8F%A3%E6%88%90%E5%8A%9F.jpg)

- 接下来看一下后台返回的数据是什么样，是文件流格式（`OutputStream`）

![](https://gitee.com/lilyn/pic/raw/master/company-img/%E5%90%8E%E7%AB%AF%E8%BF%94%E5%9B%9E%E7%9A%84%E6%95%B0%E6%8D%AE%E6%B5%81.jpg)

在处理之前，说几个要注意的点

1. **注意：后端在这里一般会设置如下几个请求头**

```java
// 设置返回类型为excel
response.setContentType("application/vnd.ms-excel; charset=UTF-8");  
	
// 设置返回文件名为filename.xls 
response.setHeader("Content-Disposition", "filename.xls");  
response.setHeader("Cache-Control", "no-cache");  
```

2. **注意：前端在 Axios 响应拦截的时候，需要对其进行处理。**一般我们都是把 `response.data` 进行返回，但是这里我们需要把整个 `response` 返回（因为文件名在 headers 里面）

```js
import axios from 'axios'

const service = axios.create({
  baseURL: process.env.NODE_ENV === 'development' ? '' : 'http://127.0.0.1:9999'
  withCredentials: true,
  timeout: 5000
})

// 响应拦截器
service.interceptors.response.use(
  response => {
    if (response.config.responseType === 'blob') {
      return response
    }
    return response.data
  },
  error => console.log(error)
)

export default service
```

接下来要处理这个文件流，大概有两种方法（通常对应的是需要不需要携带 token），原理都是通过 `a` 标签下载

1. 通过 Ajax 请求，拿到 `response` ，转换为 blob 格式（主要是为了处理 type），为其生成下载链接，下载即可
2. 拼接 URL，拼出来对应请求链接，直接访问即可

## 通过 Blob 下载

Blob 通常用于存储大文件，典型的 Blob 内容是一张图片或一个音频

1. 默认情况下 axios 不会处理二进制数据，即请求可以正常被浏览器接收，但 axios 不会去处理。**需要在请求的时候设置 `responseType: 'blob'` 才可以**
2. 拿到文件流之后，需要生成一个 URL 才可以下载，可以**通过`URL.createObjectURL()`方法生成一个链接**
3. a 标签添加文件名
   正常情况下，通过 `window.location = url` 就可以下载文件。浏览器判断这个链接是一个资源而不是页面的时候，就会下载文件。但是通过文件流生成的 url 对应的资源是没有文件名的，需要添加文件名。这时候可以用到 download 属性指定下载的文件名

```js
const mimeMap = {
  xlsx: 'application/vnd.ms-excel',
  zip: 'application/zip',
}
export const toExcel = params => {
  return request({
    method: 'get',
    url: '/dayReportToExcel/toExcel',
    responseType: 'blob',
    params
  }).then(res => resolveBlob(res, mimeMap.xlsx))

export function resolveBlob(res, mimeType) {
  // 创建a标签，并处理二级制数据
  const aLink = document.createElement('a')
  const blob = new Blob([res.data], { type: mimeType })
  // 生成下载链接
  const URL = window.URL || window.webkitURL
  aLink.href = URL.createObjectURL(blob)
  // 设置下载文件名称
  const fileName = res.headers['content-disposition']
  aLink.setAttribute('download', fileName)
  // 下载
  document.body.appendChild(aLink)
  aLink.click()
  // 释放URL对象
  window.URL.revokeObjectURL(aLink.href)
  document.body.removeChild(aLink)
}
```

注意：一般情况下文件名都是需要匹配的，后端传过来的可能是这样的，首选需要 `decodeURI` 解码一下，再用正则把文件名匹配出来（替换设置下载文件名那里即可）

![](https://gitee.com/lilyn/pic/raw/master/company-img/Excel%E6%96%87%E4%BB%B6%E5%90%8D.jpg)

```js
const pat = new RegExp('filename=([^;]+\\.[^\\.;]+);*')
const contentDisposition = decodeURI(res.headers['content-disposition'])
var result = pat.exec(contentDisposition)
const fileName = result[1]
aLink.setAttribute('download', fileName)
```

## 拼接 URL 下载

如果可以直接通过 URL 下载文件，则可以不需要发送 Ajax 请求（前提是没有 token、headers 验证），直接下载

- 可以使用 `window.open(url)`
- 可以使用 `a` 标签进行下载

```js
import qs from 'qs'

export function downloadExcel(params) {
  const url = window.location.origin + '/dayReportToExcel/toExcel?' + qs.stringify(params)
  // window.open(url)
  const aLink = document.createElement('a')
  aLink.setAttribute('download', '')
  aLink.setAttribute('target', '_blank')
  aLink.href = url
  aLink.click()
}
```
