---
title: Vue 中使用 Upload 组件上传 Excel
tags:
  - Vue
  - Element
  - 封装
categories: Vue
author: LiLyn
copyright: ture
abbrlink: 5c61fd6a
---
vue 中使用 Element 的 upload 组件上传 Excel，大致可以分两种情况

1. 使用 `action` 上传到服务器
2. 使用 `axios` 上传到服务器

<!--more-->

**注意：上传文件可能由于前后端格式不统一导致上传失败**

- `application/x-www-form-urlencoded` 一般情况下使用这个比较多
- `multipart/form-data`
- `application/json`

## 使用 `action`

使用 `action` 时，首先会使用OPTIONS方法发起一个**预检请求**，从而获知服务器是否允许该跨域请求。服务器确认允许之后，才发起实际的 HTTP 请求。后端返回 204，以防处理 POST 请求时访问错误

![](https://gitee.com/lilyn/pic/raw/master/company-img/upload_options.jpg)

**注意：使用 `action` ，需要后端做跨域处理。比如 Nginx [反向代理](https://juejin.cn/post/6844903782556368910)、CORS 等**

效果如下：

![](https://gitee.com/lilyn/pic/raw/master/js-img/%E4%B8%8A%E4%BC%A0xlsx.gif)

备注：

1. 如果希望使用 ajax 发送请求可以配置 `http-request`
2. Window 电脑可以选择 `所有文件（*.*）` ，之后可以上传任意文件，最好在上传之前做个 `before-upload` 判断类型处理

代码如下：

```html
<template>
  <div>
    <el-upload
      ref="upload"
      :accept="fileType.join(',')"
      :limit="1"
      :headers="upload.headers"
      :action="upload.url"
      :disabled="upload.isUploading"
      :before-upload="beforeUpload"
      :on-progress="handleFileProgress"
      :on-success="handleFileSuccess"
      :auto-upload="false"
      drag
    >
      <i class="el-icon-upload" />
      <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
      <div slot="tip" class="el-upload__tip" style="color: red">
        提示：仅允许导入“xls”或“xlsx”格式文件！
      </div>
    </el-upload>
    <el-button type="success" size="mini" @click="submitUpload">上传到服务器</el-button>
  </div>
</template>

<script>
import { getToken } from '@/utils/auth'
export default {
  name: 'Upload',
  data() {
    return {
      fileType: ['.xlsx', '.xls'],
      upload: {
        // 设置上传的请求头部
        headers: { Authorization: getToken() },
        // 上传地址
        url: 'https://jsonplaceholder.typicode.com/posts/',
        // 是否更新已经存在的用户数据
        isUploading: false
      }
    }
  },
  methods: {
    // 文件上传中处理
    handleFileProgress() {
      this.upload.isUploading = true
    },
    // 文件上传成功处理
    handleFileSuccess() {
      this.upload.isUploading = false
      this.$refs.upload.clearFiles()
    },
    // 提交上传文件
    submitUpload() {
      this.$refs.upload.submit()
    },
    // 上传文件之前的钩子
    beforeUpload(file) {
      const isXlsx = file.type === 'application/vnd.ms-excel'
      if (!isXlsx) {
        this.$message.error('上传文件只能是 xlsx 或 xls 格式')
      }
      return isXlsx
    }
  }
}
</script>
```

## 不使用 `action`

上面需要后端配合使用，沟通起来还是比较麻烦的，还是推荐不使用 `action`，自己处理 ajax 请求可以更自由些

效果如下（数据是拿 mock 随机生成存入 Excel 的）：

![](https://gitee.com/lilyn/pic/raw/master/company-img/%E5%AF%BC%E5%85%A5Excel.gif)

[FileReader - MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader)

- 想把文件以断点续传的形式传给服务器，一般使用 `readAsArrayBuffer()` 读取文件
- 想把文件中的数据展示到页面上，一般使用 `readAsBinaryString()` 读取文件

[Element Upload](https://element.eleme.cn/#/zh-CN/component/upload)

- `:on-change` 文件状态改变时的钩子，添加文件、上传成功和上传失败时都会被调用

  第一个参数是 file，里面有文件内容、状态等信息

  ![](https://gitee.com/lilyn/pic/raw/master/company-img/upload_file_raw.jpg)

- 把文件内容通过 FileReader 转换成二进制文件后

  再通过 `xlsx.read` 读取，即可拿到 Excel 数据内容

  ![](https://gitee.com/lilyn/pic/raw/master/company-img/upload_xlsx.jpg)

- 最后通过 `xlsx.utils.sheet_to_json` 即可转换成我们需要的数据格式

[Element Loading](https://element.eleme.cn/#/zh-CN/component/loading)

- `Loading.service(options)` 以服务的方式调用的 Loading 需要异步关闭

  需结合 `this.$nextTick()` 使用

代码如下：

```html
<template>
  <div class="uploadBox">
    <!-- 上传文件按钮 -->
    <div class="buttonBox">
      <el-upload
        action
        accept=".xlsx,.xls"
        :show-file-list="false"
        :on-change="handleChange"
        :auto-upload="false"
      >
        <el-button slot="trigger" type="primary">选取Excel文件</el-button>
        <el-button type="success" :disabled="disabled" @click="submit">提交到服务器</el-button>
      </el-upload>
    </div>

    <!-- 解析出来的数据 -->
    <div v-show="show" class="tableBox">
      <h3>
        <i class="el-icon-info">请您检查无误后，再点击“提交到服务器”按钮</i>
      </h3>
      <el-table :data="tempData" border style="width: 100%" :height="height">
        <el-table-column prop="name" label="姓名" min-width="50%" />
        <el-table-column prop="phone" label="电话" min-width="50%" />
      </el-table>
    </div>
  </div>
</template>

<script>
import xlsx from 'xlsx'
import { Loading } from 'element-ui'
import uploadExcel from '@/api'
export default {
  name: 'Upload',
  data() {
    return {
      height: document.documentElement.clientHeight - 130,
      tempData: [],
      show: false,
      disabled: false,
      character: {
        name: {
          text: '姓名',
          type: 'string'
        },
        phone: {
          text: '电话',
          type: 'string'
        }
      }
    }
  },
  methods: {
    // 采集excel数据
    async handleChange(file) {
      const originData = file.raw
      if (!originData) return
      this.show = false
      const loadingInstance = Loading.service({
        text: '努力加载中!!!',
        background: 'rgba(0, 0, 0, 0.8)'
      })
      const binaryData = await this.readFile(originData)
      const workbook = xlsx.read(binaryData, { type: 'binary' })
      const worksheet = workbook.Sheets[workbook.SheetNames[0]]
      const data = xlsx.utils.sheet_to_json(worksheet)
      this.tempData = this.handleData(data)
      await this.delay(300)
      this.show = true
      loadingInstance.close()
    },
    // 把读取出来的数据转换为服务器需要的格式
    handleData(data) {
      const arr = []
      const char = this.character
      data.forEach(item => {
        const obj = {}
        for (const key in char) {
          if (Object.hasOwnProperty.call(char, key)) {
            const el = char[key]
            let val = item[el.text] || ''
            const type = el.type
            type === 'string' ? (val = String(val)) : null
            type === 'number' ? (val = Number(val)) : null
            obj[key] = val
          }
        }
        arr.push(obj)
      })
      return arr
    },
    // 提交数据给服务器
    async submit() {
      if (this.tempData.length <= 0) {
        this.$message({
          message: '请先选择Excel文件',
          type: 'warning',
          showClose: true
        })
        return
      }
      this.disabled = true
      const loadingInstance = Loading.service({
        text: '努力加载中!!!',
        background: 'rgba(0, 0, 0, 0.8)'
      })
      await this.delay(300)
      // 发送API请求
      uploadExcel(this.tempData).then(() => {
        this.$message({
          message: 'Excel文件已上传完毕',
          type: 'success',
          showClose: true
        })
        this.show = false
        this.disabled = false
        loadingInstance.close()
      })
    },
    readFile(file) {
      return new Promise(resolve => {
        const reader = new FileReader()
        reader.readAsBinaryString(file)
        reader.onload = e => {
          resolve(e.target.result)
        }
      })
    },
    delay(interval = 0) {
      return new Promise(resolve => {
        const timer = setTimeout(_ => {
          clearTimeout(timer)
          resolve()
        }, interval)
      })
    }
  }
}
</script>

<style scoped>
.buttonBox {
  padding: 15px;
  display: flex;
}
.el-button {
  margin-right: 20px !important;
}
.tableBox {
  padding: 0 15px;
}
h3 {
  font-size: 18px;
  color: #f56c6c;
  padding-bottom: 15px;
}
</style>
```

**导出数据**

如下方法使用 xlsx 导出，也可以使用 xlsx + file-saver 导出

- `@selection-change` ，当选择项发生变化时会触发该事件

  参数为 selection，拿到后处理一下

- 之后通过 `xlsx.utils.json_to_sheet` 将其变成 sheet

  再新建一个表格 `xlsx.utils.book_new`

  往表格插入数据 `xlsx.utils.book_append_sheet`

  最后通过 `xlsx.writeFile` 即可下载

```js
const arr = this.selectionList.map(item => {
  return {
    编号: item.id,
    姓名: item.name,
    电话: item.phone
  }
})
const sheet = xlsx.utils.json_to_sheet(arr)
const book = xlsx.utils.book_new()
xlsx.utils.book_append_sheet(book, sheet, '表格名')
xlsx.writeFile(book, `user${new Date().getTime()}.xls`)
```

