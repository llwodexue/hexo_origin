---
title: Element 可选日期范围限制或检验（开始日期结束日期）
tags:
  - Vue
  - Element
  - 问题
categories: Vue
author: LiLyn
copyright: ture
abbrlink: 94396d70
---
## 需求：

2. 选择起始日期后截止日期不能小于起始日期
3. 选择截止日期后起始日期不能大于截止日期

<!--more-->

有两种实现方法：

1. 选择起始日期，起始日期之前的日期就不让你选择

   效果如下：

   ![](https://gitee.com/lilyn/pic/raw/master/company-img/%E8%B5%B7%E5%A7%8B%E6%97%A5%E6%9C%9F%E6%88%AA%E6%AD%A2%E6%97%A5%E6%9C%9F.gif)

2. 选择起始日期，表单校验截止日期不能小于起始日期

   效果如下：

   ![](https://gitee.com/lilyn/pic/raw/master/js-img/%E8%A1%A8%E5%8D%95%E9%AA%8C%E8%AF%81_%E8%B5%B7%E5%A7%8B%E6%97%A5%E6%9C%9F%E5%92%8C%E6%88%AA%E6%AD%A2%E6%97%A5%E6%9C%9F.gif)

## 第一种情况：不让选

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

## 第二种情况：表单校验

- 使用 DatePicker 的 validateField 来对部分表单字段进行校验，如下需动态校验 startDate 和 stopDate

**注意：**

1. 这里需要结合 watch 来使用，不然会出现`截止日期必须大于起始日期` 不会随着只改变起始日期的情况下消失
2. 这里可以把表单校验方法抽出去，以后后续修改

```js
<template>
  <el-form ref="form" :model="form" label-width="100px" :rules="rules">
    <el-form-item label="起始日期" prop="startDate">
      <el-date-picker
        v-model="form.startDate"
        value-format="yyyy-MM-dd"
        clearable
        type="date"
        placeholder="请选择起始日期"
      />
    </el-form-item>
    <el-form-item label="截止日期" prop="stopDate">
      <el-date-picker
        v-model="form.stopDate"
        value-format="yyyy-MM-dd"
        clearable
        type="date"
        placeholder="请选择截止日期"
      />
    </el-form-item>
  </el-form>
</template>

<script>
export default {
  data() {
    const checkEndTime = (rule, value, callback) => {
      this.ValidatorEndTime(rule, value, callback, this.form.startDate)
    }
    return {
      form: {
        startDate: '',
        stopDate: ''
      },
      rules: {
        startDate: [{ required: true, message: '请选择起始日期', trigger: 'change' }],
        stopDate: [{ required: true, validator: checkEndTime, trigger: 'change' }]
      }
    }
  },
  watch: {
    'form.startDate'() {
      this.validateReset(['startDate', 'stopDate'])
    },
    'form.stopDate'() {
      this.validateReset(['startDate', 'stopDate'])
    }
  },
  methods: {
    validateReset(arr) {
      this.$refs['form'].validateField(arr)
    },
    ValidatorEndTime(rule, value, callback, start) {
      if (!value) {
        callback(new Error('请选择截止日期'))
      } else {
        if (!start) {
          callback(new Error('请选择起始日期！'))
        } else if (Date.parse(start) > Date.parse(value)) {
          callback(new Error('截止日期必须大于起始日期！'))
        } else {
          callback()
        }
      }
    }
  }
}
</script>
```

