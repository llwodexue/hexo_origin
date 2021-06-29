---
title: Element tree树组件 鼠标双击事件
tags:
  - Vue
  - Element
  - 双击
categories: Vue
author: LiLyn
copyright: ture
abbrlink: e082dc34
---
## 需求

双击 tree上的子节点，把节点文字显示在输入框中（这里简化一下，双击 tree 显示文字即可，省略 $emit 那一步）

- 注解：**props 可以接收 defaultExpandAll**（是否默认展开所有节点）

<!--more-->

效果如下：

![](https://gitee.com/lilyn/pic/raw/master/company-img/%E5%8F%8C%E5%87%BB%E6%95%88%E6%9E%9C.gif)

因为 Element 的 tree 组件不支持双击事件，这时可以曲线救国试一下

## 基本代码

### 树组件代码

```html
<template>
  <el-tree
    id="tree"
    v-loading="treeDataLoading"
    node-key="id"
    :data="treeData"
    :default-expand-all="defaultExpandAll"
    :expand-on-click-node="false"
    style="height: 100%;"
  />
</template>

<script>
export default {
  name: 'TreeDbl',
  props: {
    defaultExpandAll: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      treeData: [],
      treeDataLoading: false,
      resData: [
        {
          id: '1',
          label: '动物',
          parentId: ''
        },
        {
          id: '2',
          label: '狗',
          parentId: '1'
        },
        {
          id: '3',
          label: '哈士奇',
          parentId: '2'
        },
        {
          id: '4',
          label: '柯基',
          parentId: '2'
        },
        {
          id: '6',
          label: '猫',
          parentId: '1'
        },
        {
          id: '7',
          label: '植物',
          parentId: ''
        }
      ]
    }
  },
  created() {
    this.getTree()
  },
  methods: {
    dblClickItem(e) {
      if (e.target.nodeName === 'SPAN') {
        console.log(e.target.innerText)
      }
    },
    /*
    以下处理数据逻辑可以直接跳过
    */
    getTree() {
      this.treeDataLoading = true
      // 发送数据请求（这里不发送请求以resData模拟）
      new Promise(resovle => {
        const tree = this.resData
        this.treeData = this.transfromData(tree)
        resovle()
      })
        .catch(() => {})
        .finally(() => {
          this.treeDataLoading = false
        })
    },
    transfromData(data, params = {}) {
      // 深拷贝一份（以防连续调用出错）
      const cData = JSON.parse(JSON.stringify(data))
      const map = {}
      const tData = []
      const attr = {
        id: 'id',
        parentId: 'parentId'
      }
      const arg = Object.assign({}, attr, params)
      // 注意：这里item的引用地址指向cData
      cData.forEach(item => (map[item[arg.id]] = item))
      cData.forEach(child => {
        const mapItem = map[child[arg.parentId]]
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
  }
}
</script>
```

### 父组件代码

```html
<tree-dbl defaultExpandAll />
```

### 效果如下：

![](https://gitee.com/lilyn/pic/raw/master/company-img/TreeDbl1.jpg)

## 一个不太好的方案：直接添加事件绑定

可以直接给 tree添加事件绑定，利用事件委托显示每一个元素的文字信息，可以在 getTree 方法中的 finally 中添加如下代码

```js
document.getElementById('tree').addEventListener('dblclick', this.dblClickItem)
```

- 也可以借助 `@node-expand`（节点被展开时触发的事件） 和 `@node-collapse` （节点被关闭时触发的事件）添加/解除事件绑定，不过有些麻烦
- 也可以使用 `@node-click` （节点被点击时的回调），记录点击次数，**由于 `@node-click` 可以拿到点击节点的 data，可以对其进行是否是根节点的判断**

## 推荐方案：使用自定义节点内容

可以在节点区添加按钮或图标等内容

```html
<template>
  <el-tree
    id="tree"
    ref="tree"
    v-loading="treeDataLoading"
    node-key="id"
    :data="treeData"
    :default-expand-all="defaultExpandAll"
    :expand-on-click-node="false"
    style="height: 100%;"
  >
    <span class="custom-tree-node" slot-scope="{ node }">
      <span @dblclick="dblClickItem">{{ node.label }}</span>
    </span>
  </el-tree>
</template>
>

<style scoped>
.custom-tree-node {
  font-size: 14px;
}
</style>
```

