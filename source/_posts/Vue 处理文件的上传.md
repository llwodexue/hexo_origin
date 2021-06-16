---
title: Vue 处理文件的上传
tags:
  - Vue
  - FormData
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
