---
title: Element 日期选择器清除问题（Axios get请求问题）
tags:
  - Vue
  - Axios
  - Element
  - 源码分析
  - 问题
categories: Vue
author: LiLyn
copyright: ture
abbrlink: 379f43e
---

## Element 日期选择器问题

条件查询中需要根据日期进行筛选，为了用户便利性展示清除按钮

![](https://gitee.com/lilyn/pic/raw/master/company-img/%E6%98%BE%E7%A4%BA%E6%B8%85%E9%99%A4%E6%8C%89%E9%92%AE.jpg)

- 当点击清除按钮后进行查询（**get 请求**），报 500 了，看一下 Network，并**没有进行 url 拼接**（startDate 没有拼接到 url）
- 检查 api 文件的 params 参数，发现 params 中的 startDate 值为 null

<!--more-->

![](https://gitee.com/lilyn/pic/raw/master/company-img/headersget%E8%AF%B7%E6%B1%82500.jpg)

到这里就出现了两个问题：

1. **startDate 为什么赋值为 null 了**（input 清除之后值变为空字符）
2. 我使用的是 Axios 为什么，**值为 null 没有进行 url 拼接**（空字符串的就可以进行拼接）

接下来先看先一下业务代码长什么样，之后就是 Axios 和 Element 源码分析

### vue 代码如下：

```html
<template>
  <div>
    <el-form ref="queryForm" label-width="110px" :model="listQuery">
      <el-form-item label="开车日期：">
        <el-date-picker
          v-model="listQuery.startDate"
          value-format="yyyy-MM-dd"
          type="date"
          clearable
          placeholder="选择开车日期"
          style="width: 100%"
        />
      </el-form-item>
    </el-form>
    <el-button type="primary" icon="el-icon-search" @click="getList"
      >查 询</el-button
    >
  </div>
</template>

<script>
  import { getStationBus } from '@/api/station-bus'
  export default {
    data() {
      return {
        listQuery: {
          pageNumber: 1,
          pageSize: 20,
          startDate: '',
        },
        list: [],
      }
    },
    created() {
      this.getList()
    },
    methods: {
      getList() {
        getStationBus(this.listQuery).then(res => {
          this.list = res.data
        })
      },
    },
  }
</script>
```

### api 代码如下：

```js
import request from '@/utils/request'

export const getStationBus = params => {
  return request({
    method: 'get',
    url: '/stationBus/list',
    params,
  })
}
```

## Axios 源码分析

[https://github.com/axios/axios/blob/master/dist/axios.js](https://github.com/axios/axios/blob/master/dist/axios.js) 622 行左右（搜素 buildURL）

- 当请是 get 系列请求时，如果 params 对象里面有值为 null/undefined，不会进行 url 拼接，直接 return 出去了

![](https://gitee.com/lilyn/pic/raw/master/company-img/get%E7%B3%BB%E5%88%97%E8%AF%B7%E6%B1%82.png)

## Element 源码分析[TimePicker组件]

[https://github.com/ElemeFE/element/blob/dev/packages/date-picker/src/picker.vue](https://github.com/ElemeFE/element/blob/dev/packages/date-picker/src/picker.vue) ，搜 handleClickIcon

**点击清除按钮后，把值赋值为 null**。这里需要注意！！！与 Input 组件不同

![](https://gitee.com/lilyn/pic/raw/master/company-img/%E7%82%B9%E5%87%BB%E6%B8%85%E9%99%A4%E6%8C%89%E9%92%AE%E5%90%8E%E8%B5%8B%E5%80%BC%E4%B8%BAnull%E6%97%A5%E6%9C%9F%E7%BB%84%E4%BB%B6.jpg)

## Element 源码分析[Input组件]

[https://github.com/ElemeFE/element/blob/dev/packages/input/src/input.vue](https://github.com/ElemeFE/element/blob/dev/packages/input/src/input.vue)，搜 clear

**点击清除按钮后，把值赋值为 空字符串**

![](https://gitee.com/lilyn/pic/raw/master/company-img/%E7%82%B9%E5%87%BB%E6%B8%85%E9%99%A4%E6%8C%89%E9%92%AE%E5%90%8E%E8%B5%8B%E5%80%BC%E4%B8%BAnull%E8%BE%93%E5%85%A5%E6%A1%86%E7%BB%84%E4%BB%B6.jpg)