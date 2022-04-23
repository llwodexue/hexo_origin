---
title: VScode 前端常用配置/设置/问题
tags:
  - 主题
  - 字体
  - 图标
  - 插件
  - 配置
categories: 基础配置
description: 介绍 VScode 在前端领域中的常用配置、常用设置以及可能会出现的问题
author: LiLyn
abbrlink: 735a614e
---

## 主题

### Snazzy Operator

![](https://img-blog.csdnimg.cn/img_convert/44b0beb30e83fe54f1f1948ecb00d613.png)

### Oceanic Next

![](https://img-blog.csdnimg.cn/img_convert/92d5103fc2c57d04b198741d635eb776.png)

## 字体

### [JetBrains Mono](https://www.jetbrains.com/lp/mono/)

- 字体效果

![](https://img-blog.csdnimg.cn/img_convert/b10162b4e1145d08d9685ac6ebfdeaa4.gif)

```js
//控制字体体系
"editor.fontFamily": "'JetBrains Mono', Consolas, 'Courier New', monospace",
```

### [SourceCodePro](https://github.com/adobe-fonts/source-code-pro)

## 图标

### Material Icon Theme

![](https://img-blog.csdnimg.cn/img_convert/59234ecb0e2943a88377ea26165825c7.png)

## 扩展插件

### Auto Close Tag ——代码闭合

![](https://img-blog.csdnimg.cn/img_convert/075e43fb700f26a1240de4be016c8d37.png)

### Auto Rename Tag ——重命名匹配标签

- 自动重命名配对的 HTML / XML 标签，也可以在 JSX 中使用

![](https://img-blog.csdnimg.cn/img_convert/d6e428b65dd8131b492659d4f3004107.png)

```json
"auto-rename-tag.activationOnLanguage": ["html", "xml", "php", "javascript"]
```

### Better Comments—— 代码注释

![](https://img-blog.csdnimg.cn/img_convert/9ad8792e7e802de910dd0070cd7eacbb.png)

```js
/* MyMethod
  * 绿色的高亮注释
  ! 红色的高亮注释
  ? 蓝色的高亮注释
  todo 橙色的高亮注释
  // 灰色带删除线的注释
*/
```

![](https://img-blog.csdnimg.cn/img_convert/1eb567481b1aea535769d85a4e9f07d4.png)

### Bracket Pair Colorizer 2 ——括号颜色

- 此扩展名允许用颜色标识匹配的括号

![](https://img-blog.csdnimg.cn/img_convert/612dc7c99c0304caae1c183dfdc71d62.png)

### Change-case ——切换命名规则

![](https://img-blog.csdnimg.cn/img_convert/eb96264dcbd7b4e87d0aee3c1e4d59dc.png)

Ctrl + Shift + P 执行命令的输入框，选择 `Change Case Commands`，选择其规范格式即可

![](https://img-blog.csdnimg.cn/img_convert/ea0f9eb2f34906f683d24a7b01cb3b87.png)

### Code Runner—— 代码运行

- 代码一键运行，万能语言运行环境

![](https://img-blog.csdnimg.cn/img_convert/3f96b4d903074054eda29cb333cfef7f.png)

```json
"code-runner.executorMap": {
	"python": "set PYTHONIOENCODING=utf8 && python",
	"java": "cd $dir && javac -encoding utf-8 $fileName && java $fileNameWithoutExt",
},
"code-runner.runInTerminal": true,
```
### Color Highlight —— 高亮显示颜色
![](https://img-blog.csdnimg.cn/5b19c5fc4dcc4779a6ec93c2584994a2.png)

### Chinese (Simplified) Language Pack ——汉化包

![](https://img-blog.csdnimg.cn/img_convert/8523c757447f0bb63150699bbdb9a120.png)

### CSS Peek ——快速编辑查看css

比较常用的快捷键

- Go to：直接跳转到 CSS 文件或在新的编辑器（F12）中打开
- Hover：在符号上悬停显示定义（Ctrl + hover）

![](https://img-blog.csdnimg.cn/img_convert/40252b907528a66d3431bb27604f5d74.png)

### Code Spell Checker ——代码检查

- 检查单词拼写是否有错

![](https://img-blog.csdnimg.cn/img_convert/a2137bc49325c18fd8ae8bbbfe19c623.png)

### Document This ——增加代码注释
选中代码，键入 `Ctrl + Alt + D` 即可生成注释模板
![](https://img-blog.csdnimg.cn/c083b8bc1ac24d068ea8a758175295b4.png)

### DotENV ——env 高亮

env 可能大致分为如下几种情况

- `.env.development` 开发环境
- `.env.production` 生产环境
- `.env.stage` 预发布环境 `.env.grayscale` 灰度测试环境
- `.env.sit` 系统集成测试环境 `.env.test` 测试环境

![](https://img-blog.csdnimg.cn/img_convert/9f4397e73ceb9d2ac0446d7bf2b63151.png)

### ESLint ——代码风格

- 语法规则和代码风格的检查工具

![](https://img-blog.csdnimg.cn/img_convert/f0d36b33675af3ab96846851b2ca8cba.png)

[ESLint](https://docs.shanyuhai.top/tools/vscode/format-with-eslint.html)

### filesize——文件大小

![](https://img-blog.csdnimg.cn/img_convert/c13c3954830cf70f8cbd26707f385e63.png)

在 Vscode 左下角显示文件大小

![](https://img-blog.csdnimg.cn/img_convert/9ad77a5030fb2942a20764c71ba3e90b.png)

### Image preview——图片预览

- 悬停时显示图像预览或装订线左侧可以预览大小图片

![](https://img-blog.csdnimg.cn/img_convert/4b2a642aa6c9e54f5d6a64cecde17eaa.png)
### Indenticator —— 强调缩进深度
![](https://img-blog.csdnimg.cn/11f8580c2ea74100a9252b9c66fb6cbe.png)


### Indent Rainbow——文本缩进颜色

![](https://img-blog.csdnimg.cn/img_convert/421edebf8a64c9c16e96b93416afe03f.png)

缩进效果如下：

![](https://img-blog.csdnimg.cn/img_convert/770b16dd075a82ccacf03961569fe96b.png)
### koroFileHeader —— 函数注释
详细配置可参考：[vscode添加新建文件头部注释和函数注释](https://blog.csdn.net/D_claus/article/details/85243454)
![](https://img-blog.csdnimg.cn/400f341f7947476680cd3388ddffec4d.png)

### Live Server——热更新

![](https://img-blog.csdnimg.cn/img_convert/4646c18eaa5bdf9d1381b18997b5313d.png)

### open in browser——打开默认浏览器

![](https://img-blog.csdnimg.cn/img_convert/a2a03e1007b443545624f583db933b79.png)

### Prettier——代码格式化

![](https://img-blog.csdnimg.cn/img_convert/128f90e8aa946c031c3f6e752e0d208f.png)

常用配置表如下：

```json
{
    // 多少行时开始折行
    "prettier.printWidth": 140,
    // flase 不要分号
    "prettier.semi": false,
    // true 使用单引号
    "prettier.singleQuote": true,
    // 箭头函数参数只有一个时是否要有小括号。avoid：省略括号
    "prettier.arrowParens": "avoid",
    // 自动格式化
    "editor.formatOnSave": true,
}
```

### Path Intellisense——自动显示文件名/路径

![](https://img-blog.csdnimg.cn/img_convert/416f5adc4c14224ad9bc77855a1c27df.png)

### ES7 React/Redux/GraphQL/React-Native snippets——高亮jsx

![](https://img-blog.csdnimg.cn/img_convert/9bbac98df915d7f543a333ce869da317.png)

### Settings Sync——Vscode配置同步

- 使用 GitHub Gist 在多台机器上同步设置，代码片段，主题，文件图标，启动，键绑定，工作区和扩展

![](https://img-blog.csdnimg.cn/img_convert/a8fc4b6ac8688f3e10e2b1b1c2287f07.png)

- 上传快捷键 : Shift + Alt + U

- 下载快捷键 : Shift + Alt + D

### Vetur——Vue工具

- 支持多种功能，比如语法高亮、错误检测、Emmet 和 Snippet 等等

![](https://img-blog.csdnimg.cn/img_convert/79d6d89a327cb3f407004a26dc155dc6.png)

### Vue CSS Peek——Vue 查看 CSS 定义

- 允许在 Vue 中跳转到 CSS 定义，补足 CSS Peek 无法定义的部分

![](https://img-blog.csdnimg.cn/img_convert/224c68fa7e8b2711d96db0d7031bfb09.png)

### Vue-helper——Vue

- Element、iView 代码提示和属性解读
- 允许查看方法、组件的定义

![](https://img-blog.csdnimg.cn/img_convert/5e3d38f8d10495270f313177a4f6207f.png)

**注意：** 下载的不是下面那个，下面那个是语法提示、简化的插件

![](https://img-blog.csdnimg.cn/img_convert/b3b8d8deabe36111a8ce1d2b7dbfd674.png)

### Vue Peek——Vue 查看组件定义

- 允许在 Vue 中跳转相对/绝对文件路径
- 允许查看组件的定义

![](https://img-blog.csdnimg.cn/img_convert/5d2ebfc340f21579bfcb217a2cb8046a.png)

## Emmet

目前所有的前端编辑器都支持 Emmet

[Emmet 作弊表](https://docs.emmet.io/cheat-sheet/)

## settings.json

在 VS Code 中，按 **Ctrl + P**，输入 **settings.json** 并打开该文件

```json
{
  /* 配置 terminal 为 Git */
  "terminal.integrated.profiles.windows": {
    "Git-Bash": {
      "path": "D:\\Develop\\Git\\bin\\bash.exe",
      "args": []
    }
  },
  "terminal.integrated.defaultProfile.windows": "Git-Bash",
  "code-runner.runInTerminal": true,
  /* 配置编辑器字体、贯标 */
  "editor.fontSize": 16,
  "editor.lineHeight": 20,
  "editor.letterSpacing": 0.5,
  "editor.fontWeight": "400",
  "editor.fontFamily": "'JetBrains Mono', Consolas, 'Courier New', monospace",
  "editor.fontLigatures": true, // 连体字
  "editor.tabSize": 2,
  "editor.cursorStyle": "line",
  "editor.cursorWidth": 5,
  "editor.cursorBlinking": "solid", // 光标动画样式
  "editor.suggestSelection": "first",
  "editor.wordWrap": "on", // 视区自动折行
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.detectIndentation": false, // 打开文件时不自动检查tabSize
  // "editor.formatOnSave": true, // 自动格式化
  "terminal.integrated.fontSize": 15,
  "files.autoSave": "onFocusChange", // 文件焦点变化自动保存
  /* 配置编辑器样式 */
  "window.zoomLevel": 0, // 窗口缩放级别
  "explorer.compactFolders": false, // 紧凑显示名称
  "workbench.iconTheme": "material-icon-theme",
  "workbench.colorTheme": "Snazzy Operator",
  "workbench.sideBar.location": "right",
  "indentRainbow.colors": [
    "rgba(255,255,64,0.07)",
    "rgba(127,255,127,0.07)",
    "rgba(255,127,255,0.07)",
    "rgba(79,236,236,0.07)"
  ],
  /* 配置格式化 */
  "prettier.singleQuote": true, // 使用单引号
  "prettier.semi": false, // 使用分号
  "prettier.arrowParens": "avoid", // (x) => {} 只有一个参数省略括号
  "prettier.tabWidth": 2,
  "prettier.printWidth": 100,
  /* 配置编辑器确认方式 */
  "open-in-browser.default": "chrome",
  "liveServer.settings.CustomBrowser": "chrome",
  "git.confirmSync": false, // 同步 Git 是否确认
  "explorer.confirmDelete": false, // 删除文件是否确认
  "security.workspace.trust.untrustedFiles": "open", // 允许打开不信任文件
  "javascript.updateImportsOnFileMove.enabled": "always", // 移动文件更新导入路径
  /* 文件不会显示在工作空间中 */
  "files.exclude": {
    "**/.git": true,
    "**/.svn": true,
    "**/.DS_Store": true,
    "**/.vscode": true,
    "node_modules": true,
    "target": true,
    "logs": true
  },
  /* 搜索功能时，文件夹/文件排除在外 */
  "search.exclude": {
    "**/node_modules": true,
    "**/target": true,
    "**/logs": true,
    "**/dist": true
  },
  "emmet.syntaxProfiles": {
    "vue-html": "html",
    "vue": "html"
  },
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact",
    "vue",
    "html",
    "jsx"
  ],
  "typescript.locale": "en"
}

```



## jsconfig.json

绝对路径、相对路径跳转需要在根目录增加 `jsconfig.json` 文件

```js
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    },
    "target": "ES6",
    "module": "commonjs",
    "allowSyntheticDefaultImports": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## 快捷键

[vscode 快捷键](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-windows.pdf)

- **Ctrl + P** ：转到文件，您可以在 Visual Studio Code 中移动到打开的文件/文件夹的任何文件。
- **Ctrl + `** ：在 VS Code 中打开 terminal
- **Alt + Down**：下移一行
- **Alt + Up**：上移一行
- **Ctrl + D**：将选定的字符移动到下一个匹配字符串上
- **Ctrl + Space**：触发建议
- **Shift + Alt + Down**：向下复制行
- **Shift + Alt + Up**：向上复制行
- **Ctrl + Shift + T**：重新打开最新关闭的窗口

## VSCODE 问题

1. vscode 保存文件时自动删除行尾空格：搜索 `files.trimTrailingWhitespace` ，然后将选项勾选即可
2. Code-runner 在终端运行：搜索 `code-runner.runInTerminal` ，然后将选项勾选即可
3. VSCode 中调用 cv2，代码一直显示红色波浪线（pylint 只支持自己的标准库）：搜索 `Pylint Args` 点击 add item 添加 `--generate-members` 即可
4. 代码补全失效：搜索 `auto Complete` 添加第三方库的路径
5. 如果打开终端的时候弹出了系统的 cmd 窗口。解决方法：打开系统 cmd，然后左上角右键属性，取消使用旧版控制台
   ![弹出cmd窗口](https://img-blog.csdnimg.cn/20200811234121722.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4Njg5Mzk1,size_16,color_FFFFFF,t_70#pic_center)

## 扩展工具其他篇——Jypyter

[工具篇-vscode 效率提升插件](https://zhuanlan.zhihu.com/p/73452541)

[你真的会用 Jupyter 吗？这里有 7 个进阶功能助你效率翻倍](http://news.eeworld.com.cn/mp/QbitAI/a58625.jspx)

如果出错：以管理员方式运行

```python
# 更新pip到最新版本
pip install --upgrade pip
# 如果没有pip
python -m ensurepip
easy_install pip
# 安装Jupyter
pip install jupyter
# 安装nbextensions
python -m pip install jupyter_contrib_nbextensions
jupyter contrib nbextension install --user --skip-running-check
或
conda install -c conda-forge jupyter_nbextensions_configurator
# 安装完成后，勾选 “Table of Contents” 以及 “Hinterland”
# 启动
jupyter notebook
```