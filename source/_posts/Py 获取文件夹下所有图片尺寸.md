---
title: 获取文件夹下所有图片尺寸
tags:
  - Python
  - 获取图片尺寸
  - 批处理
categories: Python
author: LiLyn
copyright: ture
abbrlink: 5a7a946b
---

有时候我们爬取了一堆图片，需要知道这些图片尺寸大小或名字以便日后处理。一个一个弄很是费劲，我们可以用 python 批量获取一下并生成 csv

- 如果图片后缀比较多（png jpg jpeg bmp），可以在 suffix里进行添加

<!--more-->

用法：输入文件夹地址即可生成 csv，不过需要预先安装一些环境，已经安好跳过即可

- PIL库python3.7版本前： `pip install PIL`  python3.7版本之后改名为 pillow `pip install pillow`
- openpyxl库 `pip install openpyxl`

### 代码如下：

```python
# -*- coding:utf-8 -*-
import os
from PIL import Image
import pandas as pd

file_list = []
width_list = []
height_list = []
root_path = input('请输入 图片 所在地址:')
suffix = ['.jpg', '.png']

for dirpath, dirnames, files in os.walk(root_path):
    for file in files:
        file_path = os.path.join(dirpath, file)
        for suf in suffix:
            if file.endswith(suf):
                img = Image.open(file_path)
                file_list.append(file)
                width_list.append(img.size[0])
                height_list.append(img.size[1])

content_dict = {
    'dir_name':file_list,
    'width':width_list,
    'height':height_list
}

df = pd.DataFrame(content_dict)
csv_path = os.path.join(root_path,'image_size.csv')
df.to_csv(csv_path, encoding='utf_8_sig')
```

### 生成 csv 效果图：

![获取文件夹所有图片尺寸](https://gitee.com/lilyn/pic/raw/master/python-img/获取文件夹所有图片尺寸.jpg)

### 生成Excel

如果要生成 Excel，可以使用 openpyxl 读一下生成的 csv，之后用`,` 分割转存为Excel，封装后的代码如下：

- 不过所有数字类型数据都会转换成字符串类型，不希望这个可以写了 is_number 方法，在 append 之前修改数据类型

```python
from openpyxl import Workbook
def xls(path):
    #生成Excel文件
    wb = Workbook()
    ws = wb.active
    first_row = []
    datas = []
    with open(path,encoding='utf-8') as f:
        is_first_row = True
        for line in f:
            line = line[:-1]
            #存储第一行
            if is_first_row:
                is_first_row = False
                first_row = line.split(',')
                continue
            #存储第二行至max_row
            datas.append(line.split(','))
    ws.append(first_row)
    for data in datas:
        ws.append(data)
    wb.save(xls_path)
```

