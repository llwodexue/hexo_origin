---
title: Python 爬取必应（壁纸+搜索词）
tags:
  - Python
  - 爬取必应
  - 批处理
categories: Python
author: LiLyn
copyright: ture
abbrlink: d6422dce
---

## 爬取必应壁纸

经常使用必应应该可以发现，其主页每天都会更新一张图片，这些图片很好看，希望每天能够下载收藏每张图片。具体请看这个网站：必应每日高清壁纸([https://bing.ioliu.cn/](https://bing.ioliu.cn/))

<!--more-->

效果如下：

![效果](https://img-blog.csdnimg.cn/20200628162535832.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4Njg5Mzk1,size_16,color_FFFFFF,t_70)

代码如下：

```python
import requests
import re
import os

headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.131 Safari/537.36'}


def get_page(num):
    page_list = []
    for i in range(1, num+1):
        url = f'https://bing.ioliu.cn/?p={i}'
        page_list.append(url)
    return page_list


def get_html(url):
    r = requests.get(url, headers=headers)
    html = r.text
    return html


def parse_html(html):
    pattern1 = re.compile(r'data-progressive.*?src="(.*?)"')
    pattern2 = re.compile(r'<h3>(.*?)</h3>')
    img_list = re.findall(pattern1, html)
    title_list = re.findall(pattern2, html)
    return img_list, title_list


def download(path, img_list, title_list):
    for i in range(len(img_list)):
        img_url = img_list[i]
        title = title_list[i]
        img_url = img_url.replace('640', '1920').replace('480', '1080')
        pattern3 = re.compile(r'[()-/_]')
        title = re.sub(pattern3, '', title)
        print(f'正在爬取: {img_url}')
        img_floder = 'D:/图片/'+keyword
        if not os.path.exists(img_floder):
            os.makedirs(img_floder)
        with open(f'{img_floder}/{title}.jpg', 'wb') as f:
            img_content = requests.get(img_url).content
            f.write(img_content)
        # 将爬取失败的删除
		if os.path.getsize(img_path) < 50:
            os.remove(img_path)


if __name__ == '__main__':
    num = 20
    keyword = '必应壁纸'
    path = 'D:/图片/'
    page_list = get_page(num)
    for page in page_list:
        html = get_html(page)
        img_list, title_list = parse_html(html)
        download(path, img_list, title_list)
```



## 根据搜索词爬取必应图片

这里需要注意： `requests.get(url, headers=headers).text` 会有很多 html 转义编码的字符，比如：引号变为`&quot`，会影响使用正则

**解决方法：**

1. 正则中加入`&quot`
2. 使用 `etree.HTML` 重新加载一下，再用 `xpath` 定位到此处

**出现问题：**

1. 请求超时

   设置请求超时时间，防止长时间停留在同一个请求

   `socket.setdefaulttimeout(10)`

   ```python
   requests.exceptions.ConnectionError: HTTPConnectionPool(host='www.iutour.cn', port=80): 
           
   Max retries exceeded with url: /uploadfile/bjzb/20141126124539763.jpg 
   (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x000001A46192EC50>: 
                                 
   Failed to establish a new connection: [WinError 10060] 由于连接方在一段时间后没有正确答复或连接的主机没有反应，连接尝试失败。',))
   ```

2. 需要验证证书

   `requests.get(img_url, verify=False)`

   ```python
   requests.exceptions.SSLError: HTTPSConnectionPool(host='bbp.jp', port=443):
   
   Max retries exceeded with url: /wp-content/uploads/2016/05/2-20.jpg 
   (Caused by SSLError(SSLError("bad handshake: Error([('SSL routines', 'tls_process_server_certificate', 'certificate verify failed')],)",),))
   ```


直接使用 `try:catch`

```python
import requests
import re
import os
from lxml import etree

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.131 Safari/537.36'}


def get_page(num):
    img_list = []
    for i in range((num // 35) + 1):
        url = f'https://cn.bing.com/images/async?q={keyword}&first={i*35}&count=35&relp=35&scenario=ImageBasicHover&datsrc=I&layout=RowBased_Landscape&mmasync=1'
        r = requests.get(url, headers=headers)
        html = r.text
        html = etree.HTML(html)
        conda_list = html.xpath('//a[@class="iusc"]/@m')
        for j in conda_list:
            pattern = re.compile(r'"murl":"(.*?)"')
            img_url = re.findall(pattern, j)[0]
            img_list.append(img_url)
    return img_list


def download(path, img_list):
    for i in range(len(img_list)):
        img_url = img_list[i]
        print(f'正在爬取: {img_url}')
        img_floder = 'D:/图片/'+keyword
        if not os.path.exists(img_floder):
            os.makedirs(img_floder)
        try:
            with open(f'{img_floder}/{i}.jpg', 'wb') as f:
                img_content = requests.get(img_url).content
                f.write(img_content)
        except:
            continue

if __name__ == '__main__':
    num = 100
    keyword = '食品街'
    path = 'D:/图片/'
    img_list = get_page(num)
    download(path, img_list)
```

