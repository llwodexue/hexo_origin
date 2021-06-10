---
title: Element 限制可选日期范围（开始日期结束日期）
tags:
  - Vue
  - Element
  - 问题
categories: Vue
author: LiLyn
copyright: ture
abbrlink: 53e5f3e7
---
## 需求：

1. 起始日期和截止日期是两个单独组件
2. 选择起始日期后截止日期不能小于起始日期
3. 选择截止日期后起始日期不能大于截止日期

<!--more-->

效果如下：

![](https://gitee.com/lilyn/pic/raw/master/company-img/%E8%B5%B7%E5%A7%8B%E6%97%A5%E6%9C%9F%E6%88%AA%E6%AD%A2%E6%97%A5%E6%9C%9F.gif)

## 代码：

- 首先不能使用 Element 中的 DatePicker 选择日期范围
- 给 DatePicker 动态配置 pick-options 中的 disableDate 即可对可选日期进行限制

**注意：**

1. 如果加` value-format` 需要给日期格式中间加 `-`
2. 需要给 DatePicker  加 `type="date"`
3. 截止日期如果不能超过当前日期需要减一天（`24 * 3600 * 1000 = 8.64e7 `）

```html
<template>
  <el-form ref="form" :model="form" label-width="100px">
    <el-form-item label="起始日期" prop="startDate">
      <el-date-picker
        v-model="form.startDate"
        value-format="yyyy-MM-dd"
        clearable
        type="date"
        placeholder="请选择起始日期"
        :picker-options="dateTimeStartFunc"
      />
    </el-form-item>
    <el-form-item label="截止日期" prop="stopDate">
      <el-date-picker
        v-model="form.stopDate"
        value-format="yyyy-MM-dd"
        clearable
        type="date"
        placeholder="请选择截止日期"
        :picker-options="dateTimeEndFunc"
      />
    </el-form-item>
  </el-form>
</template>

<script>
export default {
  data() {
    return {
      form: {
        startDate: '',
        stopDate: ''
      },
      dateTimeStartFunc: {
        disabledDate: time => {
          if (this.form.stopDate) {
            return time.getTime() > new Date(this.form.stopDate).getTime()
          }
        }
      },
      dateTimeEndFunc: {
        disabledDate: time => {
          if (this.form.startDate) {
            return time.getTime() < new Date(this.form.startDate).getTime() - 8.64e7
          }
        }
      }
    }
  }
}
</script>
```

