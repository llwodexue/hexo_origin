---
title: vscode造成电脑卡顿
tags:
  - Vscode
  - 电脑卡顿
  - 问题
categories: 基础配置
author: LiLyn
copyright: ture
abbrlink: fc53734b
---

### 主流解决方案

最近使用 Vscode 总是特别卡顿，网上大部分的解决方案如下（但是没有什么效果）

- search.followSymlinksd: false （控制是否在搜索中跟踪符号链接）
- git.enabled: false （是否启用Git）
- git.autorefresh: false （是否启用自动刷新）

<!--more-->

### 语言设置问题

都设置后还是很卡，很是头疼。去查看一下资源管理器，看一下哪个占用较多的cpu资源，发现是一个 Microsoft.Python.LanguageServer 的进程占用

![cpu-python](https://gitee.com/lilyn/pic/raw/master/Basics/python%E5%8D%A0%E7%94%A8%E8%BF%87%E9%AB%98.png)

勾选 Jedi 不使用 Microsoft

- Python: Language Server: jedi

![修改python语言](https://gitee.com/lilyn/pic/raw/master/Basics/%E4%BF%AE%E6%94%B9python%E8%AF%AD%E8%A8%80.png)

```json
// 最好在 settings.json 中加上这个，要不然可能随时变回 Microsoft
"python.languageServer": "Jedi",
```

同样， cpptools.exe 的进程占用也很高

![cpu-c](https://gitee.com/lilyn/pic/raw/master/Basics/C%E5%8D%A0%E7%94%A8%E8%BF%87%E9%AB%98.png)

- C_Cpp: Intelli Sense Engine: disabled

![修改c语言](https://gitee.com/lilyn/pic/raw/master/Basics/%E4%BF%AE%E6%94%B9c%E8%AF%AD%E8%A8%80.png)



### 插件问题

都设置后，还是会时不时卡顿，再去查看资源管理器，发现有一些插件也会导致 CPU 过高

- Auto Rename Tag （其实按F2重构即可）

你可以在命令面板（Ctrl + Shift + P）输入 `Developer: Startup Performance` 查看各个插件启动时间

可以参考 [那些你应该考虑卸载的 VSCode 扩展](https://juejin.cn/post/6844904115798016008)
