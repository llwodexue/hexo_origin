---
title: 免安装版 MySQL（附带 SQLyog 安装包）
tags:
  - SQL
  - SQLyog
  - 安装步骤
categories: Python
author: LiLyn
copyright: ture
abbrlink: 5a7a946b
---
## MySQL 免安装包

链接：https://pan.baidu.com/s/1T0m9hKmVAfEBRpOTbfTTHg 
提取码：1234 

- 里面有 32 位安装包和 64 位安装包
## SQLyog 免安装包
链接：https://pan.baidu.com/s/1v6jNPZID2vlIfdUsRbR5rw 
提取码：1234 

- 里面有 32 位安装包和 64 位安装包，直接点击exe文件安装，最后输入破解序列号即可永久使用

<!--more-->
## MySQL 配置

1. 以**管理员身份**打开命令行，否则后续命令行部分命令需要权限，出现错误

![](https://img-blog.csdnimg.cn/img_convert/ac5ca3564e090872c1f6603b2d77c741.png)

2. 切换目录。切换到 MySQL 的 bin 目录下，我把 MySQL 放到 E 盘里了： `cd E:\mysql-5.7.15-winx64\bin && e:`

![](https://img-blog.csdnimg.cn/img_convert/6b0e8acb85a135e7a851332ebd4d7575.png)

3. 安装 MySQL的服务：`mysqld --install`

![](https://img-blog.csdnimg.cn/img_convert/0b71b2ba5aa1519011bc3813aa3fda03.png)

4. 初始化 MySQL，初始化会产生一个随机密码，**需要记住这个密码**，后续会用到：`mysqld --initialize --console`

![](https://img-blog.csdnimg.cn/img_convert/9679d8276c66f91db7c42fa185b37f75.png)

5. 开启 MySQL 服务：`net start mysql`

![](https://img-blog.csdnimg.cn/img_convert/bd059673886450637521774116d49b11.png)

6. 登录验证。注意：这里的密码是初始化（步骤4）产生的随机密码；一定要先开启服务，再登录验证，不然会提示拒绝访问

![](https://img-blog.csdnimg.cn/img_convert/5b145a5f42335bf2b07604070ee93b79.png)

7. 修改密码。由于初始化的随机密码太复杂，不便于我们登录（记得结尾加 `;` ）

   ```bash
   SET PASSWORD = PASSWORD('123456');
   ```

   ![](https://img-blog.csdnimg.cn/img_convert/777541fe2e82111e0cf0a3ee999d6051.png)

## 环境变量设置

点击 "我的电脑" --> "属性" --> ''高级系统设置'' --> ''环境变量''

![](https://img-blog.csdnimg.cn/img_convert/ba4c3eabda8a6f4f04de19904ae56d28.png)

配置完成之后，每当我们想要用命令行使用 mysql 时，只需要登录即可使用

![](https://img-blog.csdnimg.cn/img_convert/ff609eee38ff3da7be13555747104899.png)

在 mysql 目录下新建一个 `my.init` 文件，在里面粘贴如下代码

```bash
[mysqld]
character-set-server=utf8mb4
bind-address=0.0.0.0
port=3306
default-storage-engine=INNODB
[mysql]
default-character-set=utf8mb4
[client]
default-character-set=utf8mb4
```

这样，免安装版的 MySQL 就安装并配置完成了