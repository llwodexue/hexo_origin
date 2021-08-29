---
title: Vue 中使用 Upload 组件上传图片
tags:
  - Vue
  - Element
  - form-data
  - base64
categories: Vue
author: LiLyn
copyright: ture
abbrlink: 5b8836c3
---

vue 中使用 Element 的 upload 组件上传 Excel，大致可以分两种情况

1. 使用 `action` 上传到服务器

   如下不讨论使用 action 上传服务器，如需了解可以参考：[Vue 中使用 Upload 组件上传 Excel](https://blog.csdn.net/qq_38689395/article/details/118419678)

2. 使用 `axios` 上传到服务器

   这里主要阐述如下两种情况

   - 使用 FormData 上传
   - 使用 base64 上传

<!--more-->

## 基础知识（原理）

**input file 属性**

- accept：可选择的文件类型，例如：`image/*`
- multiple：允许用户选择多个文件

监听 input file 的 onchange 事件，打印 `this.files` 是 **选取的文件集合**，每一项就是选择的文件。每一项 files 包含如下属性：

- lastModified
- lastModifiedDate
- name
- size
- type

## 方案1：FormData 上传

将本地数据上传或导入数据库，有时候需要使用 FormData 对象。FormData 接口提供了一种表示表单数据的键值对 `key/value` 的构造方式，组成一个 queryString 提交到后台

**注意：如下直接打印 formData 里面是空的，formData 需要用 get 方法获取值**

代码如下：（这个例子是只要导入就会立即上传）

```html
<template>
  <el-upload
    ref="upload"
    multiple
    action
    :accept="fileType.join(',')"
    :http-request="submitUpload"
    :before-upload="beforeUpload"
    drag
  >
    <i class="el-icon-upload" />
    <div slot="tip" class="el-upload__tip" style="color:red">
      提示：仅允许导入“png”、“jpg”、“jpeg”格式文件，且不超过2M！
    </div>
  </el-upload>
</template>

<script>
import { uploadFile } from '../api/index'
export default {
  data() {
    return {
      fileType: ['image/jpeg', 'image/jpg', 'image/png']
    }
  },
  methods: {
    submitUpload(req) {
      const formData = new FormData()
      const config = {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }
      formData.append('file', req.file)
      formData.append('filename', req.filename)
      uploadFile('url', formData, config)
    },
    beforeUpload(file) {
      const isImg = this.fileType.includes(file.type)
      const isLimit = file.size / 1024 / 1024 < 2
      if (!isImg) {
        this.$message.error('上传图片只能是“png”、“jpg”、“jpeg”格式!')
      }
      if (!isLimit) {
        this.$message.error('上传图片大小不能超过 2MB!')
      }
      return isImg && isLimit
    }
  }
}
</script>
```

## 方案2：把选择的文件 base64 后上传

有一些小图片可能会采取前端 base64 上传。这里我们使用 H5 FileReader 对象，用 `readAsDataURL` 将文件转 base64 

**注意：`readAsDataURL 操作是异步的`**

代码如下：（这个例子是手动点击上传再上传）

```js
<template>
  <div>
    <el-upload
      ref="upload"
      action
      list-type="picture-card"
      :file-list="imgList"
      :accept="imgTool.type.join(',')"
      :on-change="handleChange"
      :on-remove="hadnleRemove"
      :auto-upload="false"
    >
      <i class="el-icon-upload" />
      <div slot="tip" class="el-upload__tip" style="color:red">
        提示：仅允许导入“png”、“jpg”、“jpeg”格式文件，且不超过2M！
      </div>
    </el-upload>
    <el-button type="success" size="mini" @click="submitUpload">上传到服务器</el-button>
  </div>
</template>

<script>
import { uploadFile } from '../api/index'
export default {
  data() {
    return {
      imgTool: {
        type: ['image/jpeg', 'image/jpg', 'image/png'],
        size: 2 * 1024 * 1024
      },
      imgList: []
    }
  },
  methods: {
    async submitUpload() {
      console.log(this.imgList)
      const base64Pro = this.imgList.map(file => this.fileReader(file.raw))
      const base64List = await Promise.all(base64Pro)
      const config = {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      }
      base64List.forEach((item, i) => {
        const data = {
          base64: decodeURIComponent(item),
          name: this.imgList[i].name
        }
        uploadFile('url', data, config)
      })
    },
    handleChange(file, filList) {
      const isImg = this.imgTool.type.includes(file.raw.type)
      const isLimit = file.size < this.imgTool.size
      if (!isImg) {
        this.$message.error('上传图片只能是“png”、“jpg”、“jpeg”格式!')
        this.imgList = filList.filter(v => v.uid !== file.uid)
        return
      }
      if (!isLimit) {
        this.$message.error('上传图片大小不能超过 2MB!')
        this.imgList = filList.filter(v => v.uid !== file.uid)
        return
      }
      this.imgList.push(file)
    },
    hadnleRemove(_, fileList) {
      this.imgList = fileList
    },
    fileReader(file) {
      return new Promise(resolve => {
        const reader = new FileReader()
        reader.readAsDataURL(file)
        reader.onload = e => {
          resolve(e.target.result)
        }
      })
    }
  }
}
</script>
```

上述案例是本地文件转 base64，如需在线图片转 base64，可以参考 [用Vue来实现图片上传多种方式](https://juejin.cn/post/6844903639086006279#heading-6)

