---
title: Vue 封装无限层级树形菜单组件（后台传的是扁平数组）
tags:
  - Vue
  - 树形菜单
  - Element
  - 封装
categories: Vue
author: LiLyn
copyright: ture
abbrlink: 9fef765
---

项目原因，需要把一个扁平/线性数组转换成树形数组（符合 el-tree 数据要求）

```js
const resData = [
  { id: '1', label: '动物', parentId: '', icon: '' },
  { id: '2', label: '狗', parentId: '1', icon: 'icon-chongwutubiao13' },
  { id: '3', label: '哈士奇', parentId: '2', icon: 'icon-hashiqi' },
  { id: '4', label: '柯基', parentId: '2', icon: 'icon-keji-' },
  { id: '6', label: '猫', parentId: '1', icon: 'icon-chongwutubiao04' },
  { id: '7', label: '植物', parentId: '', icon: '' },
  { id: '8', label: '微生物', parentId: '', icon: '' },
]
```

<!--more-->

## JS 代码：扁平数组转换成树形数组

- 直接上代码，不需要递归

```js
function transformData(data) {
  // 深拷贝一份（以防连续调用出错）
  const cData = JSON.parse(JSON.stringify(data))
  const map = {}
  const tData = []
  // 注意：这里item的引用地址指向cData
  cData.forEach(item => (map[item.id] = item))
  cData.forEach(child => {
    const mapItem = map[child.parentId]
    if (mapItem) {
      // 注意：这里mapItem引用地址指向也是指向cData
      if (!mapItem.children) mapItem.children = []
      mapItem.children.push(child)
    } else {
      // 顶级节点
      tData.push(child)
    }
  })
  return tData
}
```

## 项目应用：el-tree 制作一个树形多级嵌套菜单栏

实现效果：

![](https://img-blog.csdnimg.cn/img_convert/fd270ef62d9c02db91a340ae0b0a173f.png)

- Vue 代码

```html
<template>
  <el-scrollbar wrap-class="scrollbar-wrapper" style="height: 100%">
    <el-tree
      ref="tree"
      v-loading="treeDataLoading"
      node-key="id"
      :data="treeData"
      :show-checkbox="showCheckBox"
      :default-expand-all="defaultExpandAll"
      style="height: 100%; padding-bottom: 20px"
    >
      <span slot-scope="{ node, data }" class="custom-tree-node">
        {{ node.label }}
        <template v-if="data.icon">
          <el-tooltip effect="dark" :content="data.label" placement="top-start">
            <i :class="['iconfont', data.icon]"></i>
          </el-tooltip>
        </template>
      </span>
    </el-tree>
  </el-scrollbar>
</template>

<script>
export default {
  props: {
    // 节点是否可被选择（显示前面的复选框）
    showCheckBox: {
      type: Boolean,
      default: false,
    },
    // 是否默认展开所有节点
    defaultExpandAll: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      treeData: [],
      treeDataLoading: false,
      resData: [
        { id: '1', label: '动物', parentId: '', icon: '' },
        { id: '2', label: '狗', parentId: '1', icon: 'icon-chongwutubiao13' },
        { id: '3', label: '哈士奇', parentId: '2', icon: 'icon-hashiqi' },
        { id: '4', label: '柯基', parentId: '2', icon: 'icon-keji-' },
        { id: '6', label: '猫', parentId: '1', icon: 'icon-chongwutubiao04' },
        { id: '7', label: '植物', parentId: '', icon: '' },
        { id: '8', label: '微生物', parentId: '', icon: '' },
      ],
    }
  },
  methods: {
    init() {
      this.treeDataLoading = true
      // 发送数据请求（这里不发送请求以resData模拟）
      new Promise(res => {
        const tree = this.resData
        this.treeData = this.transformData(tree)
        res()
      })
        .catch(() => {})
        .finally(() => {
          this.treeDataLoading = false
        })
    },
    transformData(data, params = {}) {
      const cData = JSON.parse(JSON.stringify(data))
      const map = {}
      const tData = []
      const attr = {
        id: 'id',
        parentId: 'parentId',
      }
      const arg = Object.assign({}, attr, params)
      cData.forEach(item => (map[item[arg.id]] = item))
      cData.forEach(child => {
        const mapItem = map[child[arg.parentId]]
        if (mapItem) {
          if (!mapItem.children) mapItem.children = []
          mapItem.children.push(child)
        } else {
          tData.push(child)
        }
      })
      return tData
    },
    // 全选
    checkAll() {
      this.$refs.tree.setCheckedNodes(this.treeData)
    },
    // 取消全选
    cancelAll() {
      this.$refs.tree.setCheckedKeys([])
    },
  },
}
</script>

<style lang="less" scoped>
.scrollbar-wrapper {
  overflow-x: hidden !important;
}
.custom-tree-node {
  font-size: 16px;
}
</style>
```

## 父组件调用

```html
<template>
  <div>
    <tree-select ref="tree" defaultExpandAll />
  </div>
</template>
<script>
import TreeSelect from './components/TreeSelect'
export default {
  components: {
    TreeSelect,
  },
  mounted() {
    this.$refs['tree'].init()
  },
}
</script>
```

## 还可以参考这篇文章，他使用的双 filter

[js 实现无限层级树形数据结构（创新算法）](https://blog.csdn.net/Mr_JavaScript/article/details/82817177)