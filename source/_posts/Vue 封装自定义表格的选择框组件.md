---
title: Vue 封装自定义表格的选择框组件
tags:
  - Vue
  - Element
  - 封装
categories: Vue
author: LiLyn
copyright: ture
abbrlink: 643ce53f
---

有这样一个需求：

1. 下拉框组件希望显示的内容是一个表格
2. 回显只显示其姓名
3. 选择框切换时父组件需要能拿到对应名字的一整行数据

<!--more-->

如图：

![](https://gitee.com/lilyn/pic/raw/master/js-img/%E8%87%AA%E5%AE%9A%E4%B9%89%E8%A1%A8%E6%A0%BC%E7%9A%84%E9%80%89%E6%8B%A9%E6%A1%86.gif)

## 问题分析：

需求其实很简单，主要利用 `v-model` 里的 `@input` 控制其回显和 `@change` 找到对应行的数据，这里主要说一下可能出现的问题：

- 如果后端返回的数据没有唯一值（可能出现重复值的情况），需要自行处理一下，给每一项绑定一个唯一值（下标）
- 当选择框组件有 `clearable` 属性时，`@change` 的值需要对空值进行处理一下

其他方面优化：

- 可以使用 `v-bind="$attrs"` 将父组件调用子组件时传入的属性展开（排除被 prop 注册、class、style）

  ```html
  <fu-input placeholder="请输入" maxlength="50" />
  
  <!-- fu-input组件内部实现 -->
  <input v-bind="$attrs" />
  <!-- 上面代码渲染出的效果如下-->
  <input placeholder="请输入" maxlength="50" />
  ```

- 可以使用 `v-on="$listeners"` 将父组件调用子组件时传入的方法展开

  ```html
  <fu-input @focus="focus" @input="xxx" @change="xxx" />
  
  <!-- fu-input组件内部实现 -->
  <input v-on="$listeners" />
  <!-- 上面代码渲染出的效果如下-->
  <input @focus="focus" @input="xxx" @change="xxx" />
  ```

- 组件上的 v-model 默认会利用名为 value 的 prop 和名为 input 的 event。可以使用 `model` 对其 prop 和 event 修改名字

  ```html
  <my-div v-model="someValue"></my-div>
  
  <!-- 等价于 -->
  <my-div :value="someValue" @input="someValue = $event"></my-div>
  ```

## 组件代码：

```html
<template>
  <el-form-item :label="label" :prop="prop">
    <el-select
      :value="selected"
      :placeholder="placeholder"
      popper-class="select-options_table"
      clearable
      filterable
      default-first-option
      style="width: 100%"
      @change="selectChange"
    >
      <li class="option-header">
        <span>日期</span>
        <span>姓名</span>
        <span>年龄</span>
      </li>
      <el-option
        v-for="item in tableData"
        :key="item.index"
        :label="item.name"
        :value="item.index"
        class="select-option_li"
      >
        <span>{{ item.date }}</span>
        <span>{{ item.name }}</span>
        <span>{{ item.age }}</span>
      </el-option>
    </el-select>
  </el-form-item>
</template>

<script>
const tableData = [
  {
    date: '2019-06-06',
    name: '王小虎',
    age: '23'
  },
  {
    date: '2019-06-06',
    name: '李小鸭',
    age: '14'
  },
  {
    date: '2019-06-06',
    name: '王小虎',
    age: '23'
  },
  {
    date: '2019-06-06',
    name: '张小马',
    age: '18'
  }
]
export default {
  name: 'SelectName',
  model: {
    prop: 'selected',
    event: 'select-event'
  },
  props: {
    prop: String,
    label: {
      type: String,
      default: '姓名'
    },
    placeholder: {
      type: String,
      default: '请选择姓名'
    },
    params: {
      type: Object,
      default() {
        return {}
      }
    },
    selected: {
      type: String,
      value: ''
    }
  },
  data() {
    return {
      tableData: tableData
    }
  },
  created() {
    this.tableData.map((item, i) => {
      item.index = i
      return item
    })
  },
  methods: {
    selectChange(val) {
      if (val === '') return this.$emit('select-event', val)
      const nameList = this.tableData.find(item => item.index === val)
      this.$emit('select-event', nameList.name)
      this.$emit('change', nameList)
    }
  }
}
</script>

<style lang="less" scoped>
.select-options_table {
  span {
    width: 100px;
    height: 30px;
    line-height: 30px;
    text-align: center;
    display: inline-block;
  }
  .select-option_li {
    padding: 0;
  }
  .option-header {
    background: #ffddbb;
  }
}
</style>
```

## 父组件代码：

```html
<el-form :model="listQuery" style="width: 300px" label-width="110px">
  <select-name v-model="name" prop="name" @change="getName" />
</el-form>

<script>
import SelectName from './components/SelectName'
export default {
  data() {
    return {
      name: '李小鸭'
    }
  },
  components: {
    SelectName
  },
  methods: {
    getName(val) {
      console.log(val)
    }
  }
}
</script>
```

