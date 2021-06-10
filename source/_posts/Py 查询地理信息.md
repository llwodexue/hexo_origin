---
title: Python 查询并爬取地理信息
tags:
  - Python
  - 爬取地图
  - 批处理
categories: Python
author: LiLyn
copyright: ture
abbrlink: 676d36dc
---

看了 B站up主 应心小栈 的视频有感 [应心小栈](https://space.bilibili.com/95113417/)

<!--more-->

## 一、高德地图
### 地理编码
![gaode](https://img-blog.csdnimg.cn/20191018183654671.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4Njg5Mzk1,size_16,color_FFFFFF,t_70)

操作步骤：

1. 申请 **Web服务API 密钥（key）**
[申请密钥链接](https://lbs.amap.com/dev/index)
2. 拼接 HTTP 请求 URL
这里选用的是【地理编码】
**修改 address 和 key**
- address=<搜索的地址>
- key=<用户的key>
`https://restapi.amap.com/v3/geocode/geo?address=<搜索的地址>&output=XML&key=<用户的key>`
3. 接收HTTP请求返回的数据（JSON 或 XML 格式），解析数据。
```python
# _*_coding:utf-8

import re
import requests
from bs4 import BeautifulSoup
base_url = 'https://restapi.amap.com/v3/geocode/geo?address='
#填写key
fun_url = '&output=XML&key=您的key'
address = '北京大学'
url = base_url + address + fun_url
response = requests.get(url)
soup = BeautifulSoup(response.text,'lxml')
location_source = str(soup.find('location'))
location_down = re.split('<|>',location_source)[2]
location_out = re.split(',',location_down)
longitude = location_out[0]
latitude = location_out[1]
print('经度为：{} 纬度为：{}'.format(longitude,latitude))

>>> 经度为：116.308264 纬度为：39.995304
```

### 搜索POI
![使用POI](https://img-blog.csdnimg.cn/20200526152357308.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4Njg5Mzk1,size_16,color_FFFFFF,t_70)
![接口_高德](https://img-blog.csdnimg.cn/20200526152415980.png)
详细参数参考链接：[https://lbs.amap.com/api/webservice/guide/api/search](https://lbs.amap.com/api/webservice/guide/api/search)

```python
import requests
import json
import pandas as pd
import time
headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36'}
url = 'https://restapi.amap.com/v3/place/text?'
page = 1
while True:
  data = {'key': '您的key',
          'keywords': '海底捞',
          'city': '南京',
          'output': 'json',
          'page': page}
  html = requests.get(url,params=data,headers=headers)
  j = html.json()
  for i in range(len(j['pois'])):
    data1 = [(j['pois'][i]['name'],
            j['pois'][i]['address'],
            j['pois'][i]['pname'],
            j['pois'][i]['cityname'],
            j['pois'][i]['adname'],
            j['pois'][i]['tel'],
            j['pois'][i]['location'],
            j['pois'][i]['biz_ext']['rating'])]
    data2 = pd.DataFrame(data1)
    data2.to_csv('map_gaode.csv',header=False,index=False,mode='a+')
print(str(page)+'has done')
page += 1
time.sleep(1)

```
爬取效果如下：

![高德爬取](https://img-blog.csdnimg.cn/2020061615172698.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4Njg5Mzk1,size_16,color_FFFFFF,t_70)

## 二、百度地图
### 地理编码
![baidu](https://img-blog.csdnimg.cn/20191018183153897.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4Njg5Mzk1,size_16,color_FFFFFF,t_70)

操作步骤：

1. 申请 **Web 服务API 密钥（ak）**
[申请密钥链接](https://lbsyun.baidu.com/apiconsole/key/create)
**Referer 白名单：**  *
Referer 白名单：后面填写 * 否则会出现 **message：APP Referer校验失败**
2. 拼接 HTTP 请求 URL
这里选用的是【地理编码】
**修改 address 和 key**
- address=<搜索的地址>
- key=<用户的key>
`http://api.map.baidu.com/geocoding/v3/?address=<搜索的地址>&output=xml&ak=<用户的key>`
3. 接收 HTTP 请求返回的数据（JSON 或 XML 格式），解析数据。

```python
# _*_coding:utf-8

import re
import requests
from bs4 import BeautifulSoup
base_url = 'http://api.map.baidu.com/geocoding/v3/?address='
#填写ak
fun_url = '&output=xml&ak=您的ak'
address = '北京大学'
url = base_url + address + fun_url
response = requests.get(url)
soup = BeautifulSoup(response.text,'lxml')
location_lng = str(soup.find('lng'))
location_down_lng = re.split('<|>',location_lng)
longitude = location_down_lng[2]
location_lat = str(soup.find('lat'))
location_down_lat = re.split('<|>',location_lat)
latitude = location_down_lat[2]
print('经度为：{} 纬度为：{}'.format(longitude,latitude))

>>> 经度为：116.304152442 纬度为：39.9670892266
```

### 搜索POI
百度可能出现的问题（请求人数过多报错）：`{'status': 401, 'message': '当前并发量已经超过约定并发配额，限制访问'}` 加一个`try:except:`
![接口_百度](https://img-blog.csdnimg.cn/20200526165701771.png)
详细参数参考链接：[http://lbsyun.baidu.com/index.php?title=webapi/guide/webservice-placeapi#service-page-anchor-1-3](http://lbsyun.baidu.com/index.php?title=webapi/guide/webservice-placeapi#service-page-anchor-1-3)

```python
import requests
import json
import pandas as pd
import time

headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36'}
url = 'http://api.map.baidu.com/place/v2/search?'
page_num = 1
while True:
  data = {'ak': '您的ak',
          'query': '海底捞',
          'region': '南京',
          'output': 'json',
          'page_num': page_num}
  html = requests.get(url,params=data,headers=headers)
  j = html.json()
  try:
    for i in range(len(j['results'])):
      data1 = [(j['results'][i]['name'],
              j['results'][i]['address'],
              j['results'][i]['province'],
              j['results'][i]['city'],
              j['results'][i]['area'],
              j['results'][i]['telephone'],
              str(j['results'][i]['location']['lat'])+','+str(j['results'][i]['location']['lng']))]
      data2 = pd.DataFrame(data1)
      data2.to_csv('map_baidu.csv',header=False,index=False,mode='a+')
  except KeyError:
    time.sleep(1)

print(str(page_num)+'has done')
page_num += 1
time.sleep(1)
```
爬取效果如下：

![百度爬取](https://img-blog.csdnimg.cn/20200616151556844.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4Njg5Mzk1,size_16,color_FFFFFF,t_70)

## 三、腾讯地图
### 地理编码
![tengxun](https://img-blog.csdnimg.cn/20191018185450427.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4Njg5Mzk1,size_16,color_FFFFFF,t_70)

操作步骤：

1. 申请 **Web 服务 API 密钥（key）**
[申请密钥链接](https://lbs.qq.com/dev/console/key/add)
2. 拼接 HTTP 请求 URL
这里选用的是【地理编码】
**修改 address 和 key**
- address=<搜索的地址>
- key=<用户的key>
`https://apis.map.qq.com/ws/geocoder/v1/?address=<搜索的地址>&key=<用户的key>`
3. 接收 HTTP 请求返回的数据（JSON 或 XML 格式），解析数据。

```python
# _*_coding:utf-8

import re
import requests
from bs4 import BeautifulSoup
base_url = 'https://apis.map.qq.com/ws/geocoder/v1/?address='
fun_url = '&output=xml&key=您的key'
address = '北京大学'
url = base_url + address + fun_url
response = requests.get(url)
location_text = response.text
longitude = re.findall("lng\": (.*?),",location_text)[0]
latitude = re.findall("lat\": (.*?)\n",location_text)[0]
print('经度为：{} 纬度为：{}'.format(str(longitude),str(latitude)))

>>> 经度为：116.310249 纬度为：39.99287
```
### 搜索POI
详细参数参考链接：[https://lbs.qq.com/service/webService/webServiceGuide/webServiceSearch](https://lbs.qq.com/service/webService/webServiceGuide/webServiceSearch)

这部分就不进行爬取了...