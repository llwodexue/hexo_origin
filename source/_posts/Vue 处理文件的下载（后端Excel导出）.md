---
title: Vue 处理文件的下载（后端Excel导出）
tags:
  - Vue
  - Excel
  - 跨域处理
categories: Vue
author: LiLyn
copyright: ture
abbrlink: 14d3abe4
---

大概有两种方法（通常对应的是需要不需要携带 token），原理都是通过 `a` 标签下载

1. 通过 Ajax 请求，拿到 `response` ，转换为 blob 格式（主要是为了处理 type），为其生成下载链接，下载即可
2. 直接拼接 URL，拼出来对应请求链接，直接访问即可（不需要二次 token 认证）

<!--more-->

## 后端文件流

- 首先点击导出 Excel ，这里调用接口成功

![](https://gitee.com/lilyn/pic/raw/master/company-img/%E5%AF%BC%E5%87%BAExcel%E6%8E%A5%E5%8F%A3%E6%88%90%E5%8A%9F.jpg)

- 接下来看一下后台返回的数据是什么样，是文件流格式（`OutputStream`）

![](https://gitee.com/lilyn/pic/raw/master/company-img/%E5%90%8E%E7%AB%AF%E8%BF%94%E5%9B%9E%E7%9A%84%E6%95%B0%E6%8D%AE%E6%B5%81.jpg)

在处理之前，说几个要注意的点！！！

1. **注意：后端在这里一般会设置如下几个请求头**

   后端还可能开启 jwt token 验证，如果开启请移步第 2 点请求拦截设置 headers
   
   **注意： 由于跨域浏览器处于安全考虑不让自定义响应头通过 JS 获取** （详见： [JS 无法获取响应 header 的 Content-Disposition 字段](https://blog.csdn.net/PGguoqi/article/details/106824957) ），也就是说 `Content-Disposition` 前端在 Network 里是能看到的，但是无法通过 JS 获取到，这里后端需要将其暴露出去
   
   跨域情况默认只暴露：`Cache-Control`、`Content-Language`、`Content-Type`、`Expires`、`Last-Modified`、`Pragma` 六个属性

```java
// 设置返回类型为excel
response.setContentType("application/vnd.ms-excel; charset=UTF-8");  
// 设置返回文件名为filename.xls 
response.setHeader("Content-Disposition", "filename.xls"); 
// 请求或响应消息不能走缓存
response.setHeader("Cache-Control", "no-cache");
// 将Content-Disposition暴露出去，这样就可以用过JS获取到了
response.setHeader("Access-Control-Expose-Headers", "Content-Disposition");
```

2. **注意：前端在 Axios 请求和响应拦截的时候，需要对其进行处理**

   请求拦截一般我们都是会设置 headers，这里只是简单处理一下，实际会根据不同情况设置 headers

   响应拦截一般我们都是把 `response.data` 进行返回，但是这里我们需要把整个 `response` 返回（因为文件名在 headers 里面）

```js
import axios from 'axios'
import { getToken } from '@/utils/auth'
import { AUTHOR_KEY } from '@/global'

const service = axios.create({
  baseURL: process.env.NODE_ENV === 'development' ? '' : 'http://127.0.0.1:9999'
  withCredentials: true,
  timeout: 5000
})

// 请求拦截器
service.interceptors.request.use(
  config => {
    config.headers[AUTHOR_KEY] = getToken()
    return config
  },
  error => console.log(error)
)

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

由于有浏览器问题可能会出现 `content-disposition` 匹配不到，最好做一下判断看 `content-disposition` 和 `Content-Disposition` 哪个能取到

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
  let fileName = ''
  if (res.headers['content-disposition']) fileName = res.headers['content-disposition']
  if (res.headers['Content-Disposition']) fileName = res.headers['Content-Disposition']
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
export function resolveBlob(res, mimeType) {
  const aLink = document.createElement('a')
  const blob = new Blob([res.data], { type: mimeType })
  const pat = new RegExp('filename=([^;]+\\.[^\\.;]+)')
  let contentDisposition
  if (res.headers['content-disposition']) contentDisposition = res.headers['content-disposition']
  if (res.headers['Content-Disposition']) contentDisposition = res.headers['Content-Disposition']
  const result = pat.exec(decodeURI(contentDisposition))
  let fileName = result && result[1]
  const URL = window.URL || window.webkitURL
  aLink.href = URL.createObjectURL(blob)
  // 如果Content-Disposition没有暴露，给文件一个默认名字
  if (fileName == null) fileName = '日报表'
  aLink.setAttribute('download', fileName)
  document.body.appendChild(aLink)
  aLink.click()
  // 释放URL对象
  window.URL.revokeObjectURL(aLink.href)
  document.body.removeChild(aLink)
}
```

## 拼接 URL 下载

如果可以直接通过 URL 下载文件，则可以不需要发送 Ajax 请求（前提是没有 token、headers 验证），直接下载

- 可以使用 `a` 标签进行下载

```js
import qs from 'qs'

export function downloadExcel(params) {
  const url = window.location.origin + '/dayReportToExcel/toExcel?' + qs.stringify(params)
  const aLink = document.createElement('a')
  aLink.setAttribute('download', '')
  aLink.setAttribute('target', '_blank')
  aLink.href = url
  aLink.click()
}
```

- 可以使用 `window.open(url, '_blank')`

```js
import qs from 'qs'

export function downloadExcel(params) {
  const url = window.location.origin + '/dayReportToExcel/toExcel?' + qs.stringify(params)
  window.open(url, '_blank')
}
```

