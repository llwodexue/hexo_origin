---
title: 前端 JS 加密常用方法（RSA MD5 SHA）
tags:
  - Vue
  - jsencrypt
  - md5
  - sha
categories: Vue
description: >-
  介绍jsencrypt、md5、sha等加密算法在Vue中如何使用
author: LiLyn
copyright: ture
abbrlink: 9f5130f2
---

## jsencrypt 进行 RSA 加密

```bash
npm i jsencrypt
```

[生成 RSA 密钥对](http://web.chacuo.net/netrsakeypair)

![](https://gitee.com/lilyn/pic/raw/master/company-img/rsa%E5%85%AC%E9%92%A5%E7%A7%81%E9%92%A5.jpg)

- 将生产的公钥和私钥复制过来

```js
import JSEncrypt from 'jsencrypt/bin/jsencrypt'

const publicKey = `非对称加密公钥`
const privateKey = `非对称加密私钥`
const txt = '123'

/* 加密 */
function encrypt(pass) {
  const encryptor = new JSEncrypt() // 创建加密对象实例
  encryptor.setPublicKey(publicKey) // 设置公钥
  return encryptor.encrypt(pass) // 对内容进行加密
}

/* 解密 */
function decrypt(pass) {
  const decryptor = new JSEncrypt() // 创建解密对象实例
  decryptor.setPrivateKey(privateKey) // 设置私钥
  return decryptor.decrypt(pass) // 拿私钥解密内容
}

console.log(decrypt(encrypt(txt))) // '123'
```

## blueimp-md5 进行 MD5 加密

```bash
npm i blueimp-md5
```

- 使用 MD5 加密一般会结合处理字符串的方法

```js
import md5 from 'blueimp-md5'

const txt = '123'

function passTrans(pass) {
  return md5(pass).substring(4).split('').reverse().join('').substring(4)
}

console.log(passTrans(txt)) // 32d25170b469b57095ca269b
```

## sha 进行 SHA 加密

SHA 家族的五个算法，分别是 SHA-1、SHA-224、SHA-256、SHA-384，和 SHA-512，由美国国家安全局（NSA）所规划，并由美国国家规范与技能研究院（NIST）发布，这里只对 SHA-1 和 SHA-256 进行演示

- SHA-1

```bash
npm install js-sha1
```

```js
import sha1 from 'js-sha1'

const txt = '123'

function passTrans(pass) {
  return sha1(pass)
}

console.log(passTrans(txt)) // 40bd001563085fc35165329ea1ff5c5ecbdbbeef
```

- SHA-256

```bash
npm install js-sha256
```

```js
import { sha256 } from 'js-sha256'

const txt = '123'

function passTrans(pass) {
  return sha256(pass)
}

console.log(passTrans(txt)) // a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3
```