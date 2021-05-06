---
title: Vue + Element 国际化(i18n)路由页面切换语言
tags:
  - Vue
  - i18n
  - Element
  - 切换语言
categories: Vue
author: LiLyn
copyright: ture
abbrlink: 82482c19
---
## 安装 i18n

`internationalization` 这个单词中，i 和 n 之间有 18 个字母

<!--more-->

**注意：**当前 i18n 最新版本为 9 版本，9 版本没有 VueI18n，`import VueI18n from 'vue-i18n'` 解构会报错 `Cannot read property 'install' of undefined`

- 这里为了让 Element 兼容，安装的是 8 版本的

```bash
npm install vue-i18n@8
```

## 页面中使用国际化

### i18n 文件夹下的 index.js 文件

`require.context`

> 它允许传入一个目录进行搜索，一个标志指示是否应该搜索子目录，还有一个正则表达式来匹配文件

- 第一个参数：`directory` 要搜索的文件夹目录不能是变量，否则在编译阶段无法定位目录
- 第二个参数：`useSubdirectories` 是否搜索子目录
- 第三个参数：`regExp` 匹配文件的正则表达式

```js
require.context('demo', false, /\.js$/)
```

`function` 返回一个具有 resolve、keys、id 三属性的方法

- `resolve` 返回请求被解析后得到的模板
- `keys` 返回一个数组，由所有符合上下文模板处理的请求组成
- `id` 是上下文模板里面所包含的模板，可能在你使用 `module.hot.accept` 的时候被调用到

```js
import Vue from 'vue'
import VueI18n from 'vue-i18n'

Vue.use(VueI18n)

let langFiles = require.context('./config', false, /\.js$/)
let regExp = /\.\/([^\.\/]+)\.([^\.]+)$/
let messages = {}

langFiles.keys().forEach(key => {
  let prop = regExp.exec(key)[1]
  messages[prop] = langFiles(key).default
})

function getLanguage() {
  let locale = sessionStorage.getItem('lang') || 'zh'
  return locale
}

export default new VueI18n({
  locale: getLanguage(),
  messages
})
```

- messages 对象

![](https://gitee.com/lilyn/pic/raw/master/company-img/%E5%9B%BD%E9%99%85%E5%8C%96%E4%BF%A1%E6%81%AF.jpg)

### main.js 引入

```js
import Vue from 'vue'
import App from './App.vue'
import i18n from './i18n'

new Vue({
  render: h => h(App),
  i18n
}).$mount('#app')
```

### 展示页面

效果图如下：

![](https://gitee.com/lilyn/pic/raw/master/company-img/%E5%88%87%E6%8D%A2%E8%AF%AD%E8%A8%80.gif)

```vue
<template>
  <div>
    <el-dropdown
      trigger="click"
      class="international"
      @command="handleSetLanguage"
    >
      <div><i class="el-icon-s-tools" />切换语言</div>
      <el-dropdown-menu slot="dropdown">
        <el-dropdown-item :disabled="language === 'zh'" command="zh">
          中文
        </el-dropdown-item>
        <el-dropdown-item :disabled="language === 'en'" command="en">
          English
        </el-dropdown-item>
      </el-dropdown-menu>
    </el-dropdown>
    <el-table :data="tableData" style="width: 100%">
      <el-table-column prop="date" :label="$t('table.date')" width="180">
      </el-table-column>
      <el-table-column prop="name" :label="$t('table.name')" width="180">
      </el-table-column>
      <el-table-column prop="age" :label="$t('table.age')"> </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      tableData: [
        {
          date: '2016-05-02',
          name: '王小虎',
          age: '23'
        },
        {
          date: '2016-05-04',
          name: '李小鸭',
          age: '14'
        }
      ],
      language: sessionStorage.getItem('lang') || 'zh'
    }
  },
  methods: {
    handleSetLanguage(lang) {
      this.$i18n.locale = lang
      this.language = lang
      sessionStorage.setItem('lang', lang)
      location.reload();
    }
  }
}
</script>
```

### 语言配置文件

```js
// en.js
export default {
  table: {
    date: 'Date',
    name: 'Name',
    age: 'Age'
  }
}

// zh.js
export default {
  table: {
    date: '日期',
    name: '姓名',
    age: '年龄'
  }
}
```

## 路由菜单国际化

- 首先需要处理一下标题名，如果这个名在语言配置文件中就直接返回，如果不在返回原信息中的标题

  `$te`这个方法用以判断需要翻译的`key`在你提供的`语言包(messages)`中是否存在

```js
export function routeTitle(item) {
  const name = `route.${item.path.substring(1)}`
  if (this.$te(name)) {
    return item.meta.title
  }
  return this.$t(name)
}
```

### 展示页面中

把每一项传到处理标题的方法中

```html
<template v-for="(item, index) in routesSystem">
    <el-menu-item :index="index.toString()">{{ routeTitle(item) }}</el-menu-item>
</template>

<script>
import { routeTitle } from '@/utils/get-page-title'
export default {
    methods: {
        routeTitle // 声明一下
    }
}
</script>
```

### 配置语言文件

```js
// zh.js
export default {
  route: {
    application: '总计划',
    plan: '计划',
    equipment: '设备'
  }
}

// en.js
export default {
  route: {
    application: 'Application',
    plan: 'Plan',
    equipment: 'Equipment'
  }
}
```

### 路由表

```js
[
    {
        path: '/application',
        meta: { title: '总计划' },
	},
    {
        path: '/plan',
        meta: { title: '计划' },
	},
    {
        path: '/equipment',
        meta: { title: '设备' },
	}
]
```

## Element 国际化

### main.js 引入

- 参考兼容 `vue-i18n@6.x` 的步骤（需要手动处理）

```js
import Vue from 'vue'
import App from './App.vue'
import i18n from './i18n'

import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'
// 因为是8版本需要这么使用
Vue.use(ElementUI, {
  i18n: (key, value) => i18n.t(key, value),
  size: 'mini'
})

new Vue({
  render: h => h(App),
  i18n
}).$mount('#app')
```

### 语言配置文件

- 这样 Element 的国际化也就完成了

```js
// en.js
import enLocale from 'element-ui/lib/locale/lang/en'
export default {
  ...enLocale
}

// zh.js
import zhLocale from 'element-ui/lib/locale/lang/zh-CN'
export default {
  ...zhLocale
}
```

## 推荐参考

[前端国际化之Vue-i18n源码分析](https://segmentfault.com/a/1190000008752459)

[vue中如何使用i18n实现国际化](https://segmentfault.com/a/1190000016445415)

[Element国际化](https://element.eleme.cn/#/zh-CN/component/i18n)