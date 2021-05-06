---
title: Element 二次封装——select选择框
tags:
  - Vue
  - select
  - Element
  - v-model
  - $attr $listeners
categories: Vue
author: LiLyn
copyright: ture
abbrlink: 8bcae64c
---

项目中出现多个地方使用相同的 `<el-select>` 标签，且更改的时候，下面的数据会自动发生改变。为减少代码对其进行二次封装是很有必要的，一般可以考虑如下几点：

1. 在封装的组件里发送数据请求，这样能降低耦合性（解耦）
2. 可以使用 `v-bind="$attrs"` 将父组件调用子组件时传入的属性展开（排除被 prop 注册、class、style）
3. 可以使用 `v-on="$listeners"` 将父组件调用子组件时传入的方法展开
4. 可以使用 `v-model` 当子组件的值发生变化，自动修改父组件的传入的值

<!--more-->

- 父组件

```vue
<template>
  <train-select v-model="listQuery.name" />
</template>
```

- 子组件

```vue
<template>
  <el-select
    :value="selected"
    placeholder="请选择信息"
    clearable
    filterable
    class="select-option_table"
    @change="selectChange"
  >
    <template>
      <li>
        <span>姓名</span>
        <span>年龄</span>
      </li>
    </template>
    <el-option
      v-for="(item, i) in infoList"
      :key="i"
      :label="item.label"
      :value="item.name"
      class="select-option_li"
    >
      <span>{{ item.name }}</span>
      <span>{{ item.age }}</span>
    </el-option>
  </el-select>
</template>

<script>
import { API_Info } from '@/api/common'
export default {
  name: 'Select',
  model: {
    prop: 'selected',
    event: 'parent-event',
  },
  props: ['selected'],
  data() {
    return {
      infoList: [],
    }
  },
  created() {
    this.getSelect()
  },
  methods: {
    getSelect() {
      API_Info().then(res => {
        this.infoList = res.data
      })
    },
    selectChange(val) {
      this.$emit('parent-event', val)
      this.$emit(
        'change',
        this.infoList.find(item => item.name === val)
      )
    },
  },
}
</script>
```

### 使用 `$attrs` 和 `$listeners`

- `$attrs` 包含了父作用域中不作为 prop 被识别 (且获取) 的 attribute 绑定 (class 和 style 除外)

```html
<fu-input placeholder="请输入" maxlength="50" />

<!-- fu-input组件内部实现 -->
<input v-bind="$attrs" />
<!-- 上面代码渲染出的效果如下-->
<input placeholder="请输入" maxlength="50" />
```

- `$listeners` 包含了父作用域中的（不含 .native 修饰器的）v-on 事件监听器

```html
<fu-input @focus="focus" @input="xxx" @change="xxx" />

<!-- fu-input组件内部实现 -->
<input v-on="$listeners" />
<!-- 上面代码渲染出的效果如下-->
<input @focus="focus" @input="xxx" @change="xxx" />
```

### 使用 `v-model`

- 自定义组件上，使用 v-model 指令，默认会向子组件传递一个字段名为 value 的 prop 属性，以及绑定一个名为 input 的事件
- model 选项，允许一个自定义组件在使用 v-model 时定制 prop 和 event，可以回避一些冲突

```html
<template>
  <my-div v-model="someValue"></my-div>
  <!-- 等价于 -->
  <my-div :value="someValue" @input="someValue = $event"></my-div>
</template>

<script>
  export default {
    props: {
      value: {
        type: String,
        required: true,
      },
    },
    methods: {
      modifyParentCompsValue() {
        this.$emit('input', '要设置的值')
      },
    },
  }
</script>
```