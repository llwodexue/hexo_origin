---
title: Vue 大量数据展示 卡顿解决方案（滚动展示）
tags:
  - Vue
  - 滚动展示
  - 大数据
categories: Vue
author: LiLyn
copyright: ture
---
## 需求分析（长列表展示）

页面某处需要渲染 **1w+** 条数据，并需要**滚动展示**，这时如果直接把这些数据渲染到页面上，会导致系统内存大量被占用，导致页面卡顿或崩溃

我们都知道，每次 DOM 修改，浏览器都会重新计算元素布局，再重新渲染（回流 / 重绘）。如果数据量很大，页面计算时间就会加成，造成页面卡顿

<!--more-->

## 解决方案

- 根源：DOM 元素太多
- 思路：限制元素数量 / 虚拟DOM

![](https://gitee.com/lilyn/pic/raw/master/company-img/bigData.gif)

后台数据格式：

```js
[
    {
        checked: 0，
        data: "xxx"，
        label: "xxx"，
    }
]
```

Vue 中就有一个现成的轮子可以解决这个问题：[vue-virtual-scroller](https://github.com/Akryum/vue-virtual-scroller#recyclescroller)

1. 安装 `vue-virtual-scroller` 插件

```bash
npm install --save vue-virtual-scroller
```

2. 在 `main.js` 引入

```js
// 注意：别忘了引用这个css
import 'vue-virtual-scroller/dist/vue-virtual-scroller.css'
import Vue from "vue";
import { RecycleScroller } from 'vue-virtual-scroller'

Vue.component('RecycleScroller', RecycleScroller)
```

3. 在组件里使用
   - `items`：呈现数据
   - `item-size`：呈现数据的大小 / 高度
   - `key-filed`：如果 `items` 是对象，需要用这个做唯一标识

```html
<RecycleScroller
  style="height: 200px; overflow: auto"
  class="scroller"
  :items="listItem"
  :item-size="20"
  key-field="data"
>
  <template v-slot="{ item }">
    <el-checkbox :key="item.data" :label="item.label" />
  </template>
</RecycleScroller>

<script>
export default {
	data() {
        return {
            listItem: []
        }
    }
}
</script>
```

## 总结

使用  `vue-virtual-scroller` 插件展示大量数据，是可以比较流畅的渲染 / 滚动的