---
title: Vue 处理文件的下载和上传（FormData/Blob）
tags:
  - Vue
  - FormData
  - Blob
categories: Vue
author: LiLyn
copyright: ture
abbrlink: 4cfe3e8b
---

## FormData 上传

将本地数据上传或导入数据库，有时候需要使用 FormData 对象。FormData 接口提供了一种表示表单数据的键值对 `key/value` 的构造方式，组成一个 queryString 提交到后台

<!--more-->

### 创建一个 FormData 对象实例

```js
// js中
let formDataJs = new FormData()
// vue中
let formDataVue = new window.FormData()
```

### FormData 常用方法

formData 里存储的数据格式是 `key/value`形式

```js
let formData = new FormData()
// 获取值
formData.get('key')
// 添加数据
formData.append('key', value)
// 设置、修改数据
formData.set('key', value)
// 删除数据
formData.delete('key')
// 判断数据是否存在
formData.has('key')
```

### Vue 上传数据

```html
<template>
  <div>
    <input
      ref="uploadFile"
      type="file"
      multiple
      accept=".png"
      style="display: none"
      @change="upload($event)"
    />
    <el-button size="small" type="primary" @click="$refs.uploadFile.click()"
      >浏览</el-button
    >
  </div>
</template>

<script>
  export default {
    methods: {
      upload(e) {
        const files = e.target.files
        const suffix = '.png'
        let flag = true
        files.forEach(item => {
          const isSuffix = item.name.endsWith(suffix)
          if (!isSuffix) {
            this.$message.error(
              `${item.name} 文件类型不符，请上传后缀名为 ${suffix} 的文件`
            )
            return (flag = false)
          }
        })
        if (flag) {
          if (files.length === 0) return
          const formData = new window.FormData()
          files.forEach(item => {
            formData.append('file', item)
          })
          const config = {
            headers: {
              'Content-Type': 'multipart/form-data',
              type: 'file',
            },
          }
          this.$axios
            .post('····请求接口····', formData, config)
            .then(() => {
              this.$message.success('导入成功!')
            })
            .catch(() => {
              this.$message.error('导入失败!')
            })
        }
      },
    },
  }
</script>
```

## Blob 下载

Blob 通常用于存储大文件，典型的 Blob 内容是一张图片或一个音频

1. 默认情况下 axios 不会处理二进制数据，即请求可以正常被浏览器接收，但 axios 不会去处理。需要在请求的时候设置 `responseType: 'blob'` 才可以
2. 拿到文件流之后，需要生成一个 URL 才可以下载，可以通过`URL.createObjectURL()`方法生成一个链接
3. a 标签添加文件名
   正常情况下，通过 `window.location = url` 就可以下载文件。浏览器判断这个链接是一个资源而不是页面的时候，就会下载文件
   但是通过文件流生成的 url 对应的资源是没有文件名的，需要添加文件名。这时候可以用到 a 标签
   ，a 标签我们通常会用到 href 属性，但是其实还有一个 download 属性，这个属性就指定了下载的文件名

```js
const mimeMap = {
  xlsx: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
  zip: 'application/zip',
}

export function downLoadZip(url) {
  axios
    .get({
      url: url,
      responseType: 'blob',
    })
    .then(res => {
      resolveBlob(res, mimeMap.zip)
    })
}

export function resolveBlob(res, mimeType) {
  // 创建a标签，并处理二级制数据
  const aLink = document.createElement('a')
  const blob = new Blob([res.data], { type: mimeType })
  // 生成下载链接
  const URL = window.URL || window.webkitURL
  aLink.href = URL.createObjectURL(blob)
  // 匹配出文件名
  const disposition = res.headers['content-disposition']
  const fileName = /filename="([^;]+)"/.exec(a)[1]
  // 设置下载文件名称并下载
  aLink.setAttribute('download', fileName)
  document.body.appendChild(aLink)
  aLink.click()
  // 释放URL对象
  window.URL.revokeObjectURL(aLink.href)
  document.body.removeChild(aLink)
}
```