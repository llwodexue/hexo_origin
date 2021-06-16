---
title: Python 批量获取文件夹文件名称和数量
tags:
  - Python
  - 获取文件
  - 批处理
  - 导出Excel
categories: Python
author: LiLyn
copyright: ture
abbrlink: 5a7a946b
---
## 使用方法

- 替换 `image path` 为你想获取的地址

  file 获取文件名称/数量

  floder 获取文件夹名称/数量

- 可以服务器或 CMD 中使用（不能在 Git Bash 中使用）

<!--more-->

```bash
python getFile.py {image path} {file/floder}
```

![](https://gitee.com/lilyn/pic/raw/master/company-img/%E6%89%B9%E9%87%8F%E8%8E%B7%E5%8F%96%E6%96%87%E4%BB%B61.jpg)

## 文件名 getFile.py

效果如下：

![](https://gitee.com/lilyn/pic/raw/master/company-img/%E6%89%B9%E9%87%8F%E8%8E%B7%E5%8F%96%E6%96%87%E4%BB%B62.jpg)

代码如下：

```python
# -*- coding:utf-8 -*-
import os
import pandas as pd
import sys

if len(sys.argv) < 3:
    print("Usage python getFile.py {image path} {file/floder}")
    sys.exit()
root_path = sys.argv[1].replace('\\', '/')
type_f = sys.argv[2]
if type_f != 'file' and type_f != 'floder':
    print("Usage {file/floder}")
    sys.exit()

File_list = []
Primary_list = []
Secondary_list = []
Count = []
Floder = []
name = root_path.split('/')[-1]
for dirpath, dirnames, files in os.walk(root_path):
    if not files:
        continue
    if dirpath == root_path:
        continue
    dir_Primary, dir_Secondary = dirpath.split('/')[-2:]
    for file in files:
        File_list.append(file)
        Primary_list.append(dir_Primary)
        Secondary_list.append(dir_Secondary)
    Floder.append(dir_Secondary)
    Count.append(str(len(files)))

if type_f == 'file':
    print(f'文件数量 {len(File_list)}')
    content_dict = {
        'PrimaryFolder': Primary_list,
        'SecondaryFolder': Secondary_list,
        'Filename': File_list,
    }
    df = pd.DataFrame(content_dict)
    df.to_csv(os.path.join(root_path, f'{name}_file.csv'), encoding='utf_8_sig')
elif type_f == 'floder':
    print(f'文件夹数量 {len(Floder)}')
    content_dict = {
        'Floder': Floder,
        'Count': Count,
    }
    df = pd.DataFrame(content_dict)
    df.to_csv(os.path.join(root_path, f'{name}_floder.csv'), encoding='utf_8_sig')
```