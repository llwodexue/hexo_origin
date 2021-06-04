---
title: Vue 把表格导出 Excel 文件及打印
tags:
  - Vue
  - 打印
  - Element
  - 导出Excel
categories: Vue
author: LiLyn
copyright: ture
abbrlink: 9fef765
---
## 安装文件

1. 安装 `xlsx` 库

   ```bash
   npm install xlsx
   ```

2. 安装 `file-saver` 库

   ```bash
   npm install file-saver
   ```

3. 如果需要打印表格，安装 `vue-print-nb` 库

   ```bash
   npm install vue-print-nb
   ```

<!--more-->

![](https://gitee.com/lilyn/pic/raw/master/company-img/%E5%AF%BC%E5%87%BA%E6%89%93%E5%8D%B0.gif)

## 导出表格

主要使用的是 `js-xlsx`，导出 Excel 文件，主要是如何生成一个 sheet，之后将这个 sheet 转成最终 Excel 文件的 blob 对象，然后利用 `URL.createObject` 下载

1. 配置生成 Excel 的配置项

   `bookType` 要生成的文件类型

   `bookSST` 是否生成 `Shared String Table`，如果开启生成速度会下降，但在低版本 IOS 设备上有更好的兼容性

   `type` 可以使用 `base64` 、`binary` （BinaryString）、`string` （utf8 字符串）、`buffer`（node buffer） 、`array`（Unit8Array 8位无符号数组） 、`file` （文件路径，仅 node 支持）

2. 字符串转 ArrayBuffer 进行文件下载，这里使用 `file-saver`

   `FileSaver.saveAs(Blob/File/Url, filename)`

```js
import FileSaver from 'file-saver'
import XLSX from 'xlsx'

function htmlToExcel(dom, title = '默认标题') {
  const wb = XLSX.utils.table_to_book(document.querySelector(dom))
  const wbout = XLSX.write(wb, { bookType: 'xlsx', bookSST: true, type: 'array' })
  const blob = new Blob([wbout], { type: 'application/octet-stream' })
  FileSaver.saveAs(blob, `${title}.xlsx`)
}
```

- 组件里使用。直接调用 `htmlToExcel` 方法，传入选择器和标题名称即可

```html
<el-table id="printTest" :data="tableData" style="width: 500px">
  <el-table-column prop="date" label="日期"></el-table-column>
  <el-table-column prop="name" label="姓名"></el-table-column>
  <el-table-column prop="age" label="年龄"></el-table-column>
</el-table>
<el-button type="success" @click="htmlToExcel('#printTest', '表格')">导出Excel</el-button>
```

## 打印表格

`main.js`

```js
import Print from 'vue-print-nb'

Vue.use(Print)
```

组件里使。用 `v-print` 指定需要打印的容器，如下示例 `"'#printTest'"`：

- 注意：选择器需要用单引号包裹

```html
<template>
  <div>
    <el-table id="printTest" :data="tableData" style="width: 500px">
      <el-table-column prop="date" label="日期"></el-table-column>
      <el-table-column prop="name" label="姓名"></el-table-column>
      <el-table-column prop="age" label="年龄"></el-table-column>
    </el-table>
    <el-button type="primary" v-print="'#printTest'">打印</el-button>
  </div>
</template>
```

