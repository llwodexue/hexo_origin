---
title: HTTP网络层性能优化
tags:
  - HTTP网络层
  - 优化
  - HTTP通信
categories: 网络
author: LiLyn
copyright: ture
abbrlink: 2623edef
---
## 客户端和服务端之间的信息通信

- ajax / fetch 数据交互
- 跨域处理方案：ajax、fetch、jsonp、postMessage
- 资源获取【html|、css、js、image、音视频】
- webscoket

<!--more-->

请求：客户端把信息传递给服务器或者向服务器发送请求

响应：服务器接受客户端信息并且返回给客户端相关的内容

HTTP 报文：客户端和服务器之间的传输的所有内容

- 起始行：基本信息【包含 HTTP 的版本等】

  请求起始行：GET【请求方式】 xxx【请求地址】 HTTP/1.1【HTTP 版本号】

  响应起始行：HTTP/1.1【HTTP 版本】 200【HTTP 响应状态码】 OK【状态码描述】

- 首部（头）：请求头【客户端->服务器】、响应头【服务器->客户端】

- 主体：请求主体【客户端->服务器】、响应主体【服务器->客户端】

客户端和服务器之间的数据传输，依托于网络【通信模式 TCP/IP... 传输协议 HTTP/HTTPS/FTP...】

## 从输入 URL 地址到看到页面，中间的经历

1. URL 解析
2. 检查缓存【强缓存、协商缓存（针对资源文件请求）；本地缓存（针对数据请求）】
3. DNS 服务器解析【域名解析：根据域名解析出服务器外网 IP】
4. TCP 三次握手【建立客户端和服务器之间的网络连接通道】
5. 基于 HTTP/HTTPS 等协议，实现客户端和服务端之间的信息通信
6. TCP 四次挥手【把建立好的网络通道释放掉】
7. 客户端渲染【呈现出页面和效果】

### URL 解析

![URL解析](https://gitee.com/lilyn/pic/raw/master/js-img/URL%E8%A7%A3%E6%9E%90.jpg)

URI：统一资源标识符

- URL：统一资源定位符
- URN：统一资源名称

**传输协议：** 用什么样的协议负责客户端和服务端的信息传输

- HTTP：超文本传输协议

  除了传输文本还可以传输其余的信息，例如：文件流、二进制或者 Buffer 格式或者 BASE64 格式的数据

- HTTPS：HTTP + SSL（TSL） 更安全的 HTTP，传输的内容经过加密

- FTP：文件的上传下载

**域名：** 对服务器外网 IP 的一个重命名

`www.baidu.com`

- 顶级域名 `baidu.com`
- 一级域名 `www.baidu.com`
- 二级域名 `video.baidu.com`
- 三级域名 `image.video.baidu.com`

域名和服务器购买完后，需要在 DNS 服务器生成一条解析记录，用于以后的 DNS 解析

**端口号：** 区分同一台服务器上不同的服务的

- 取值范围：0~65535 之间

- 默认端口号：浏览器会根据输入的协议，给与默认端口号

  HTTP -> 80

  HTTPS -> 443

  FTP -> 21

**请求资源的路径名称：** 基于路径找到客户端需要的资源文件

看到的 URL 地址可能是重写后的【看到的地址在文件目录不存在】

- ajax 数据请求 `/api/list`

- url 重写

  动态网址，页面中的内容是无法被搜索引擎收录的（不利于 SEO 优化）

  静态化地址 `https://item.jd.com/....`，通过 URL 重写为 `https://item.jd.com/detail.jsp?id=...`

  `https://item.jd.com/info/100000` 路径参数【导航】

  `https://www.baidu.com/info?id=100000`

**问号传参：**

- 把信息参数传递给服务器，GET 系列请求一般都是这样传递参数

  `xxx=xxx&xxx=xxx` -> x-www-form-urlencoded 格式

- 如果是页面跳转，把信息传递给另一个页面

**HASH 值：**

- 锚点定位
- HASH 路由

URL 编译问题：

- encodeURI decodeURI：编译空格和中文，一般编译整个 URL 中的信息（前后端都支持的 API）
- encodeURIComponent decodeURIComponent：编译空格和中文以及一些特殊符号，所以一般只用来编译传递的信息的值，而不是整个 URL（前后端都支持的 API）
- escape unescape（用于客户端页面信息传递或一些信息的编译【cookie 中的中文内容编译】）
- 也可以基于自己设定的加密机制规则处理（对称加密）
- 对于某些数据，需要采用不可解密的（非对称加密），例如：md5

### 缓存检测

**缓存处理是基于 HTTP 网络层进行优化的一个非常重要的手段** 【针对资源文件请求】

强缓存还是协商缓存都是服务器设置的，客户端浏览器自己会根据返回的一些信息，进行相关处理，无需前端单独设置东西

**缓存位置：**

- Memory Cache : 内存缓存（页面没有关闭，只是刷新）
- Disk Cache：硬盘缓存（页面关闭后重新打开）

打开网页：查找硬盘缓存中是否有匹配，如有则使用，如没有则发送网络请求

普通刷新（F5）：因 TAB 没关闭，因此内存缓存是可用的，会被优先使用，其次才是硬盘缓存

强制刷新（Ctrl + F5）：浏览器不使用缓存，因此发送的请求头均带有 `Cache-Control: no-cache`，服务器直接返回 200 和最新内容

**强缓存：**  `Expires / Cache-Control`

> Expires：缓存过期时间，用来指定资源到期的事件（HTTP/1.0）
>
> Cache-Control：`cache-control: public, max-age=2592000` 第一次拿到资源后的 2592000 秒内（30 天），再次发生请求，读取缓存中的信息（HTTP/1.1）

- 如果获取的是强缓存信息，HTTP 状态码是 **200**
- 如果是从服务器成功重新获取，HTTP 状态码也是 **200**

问题：本地缓存了文件，但是服务对应的资源文件更新了，如何保证获取的是最新的内容？

1. 所有请求的资源文件（css / js / 图片）后面都带一个时间戳
2. 每一次资源的更新，基于 webpack 生成不同的资源名称（HASH 值）

所以 HTML 永远不会做强缓存，资源文件一般会使用 强缓存+协商缓存

**协商缓存：** `Last-Modified / ETag`

> Last-Modified：记录服务器资源文件最后一次更新的时间（HTTP/1.0）请求头： `If-Modified-Since`
>
> ETag：只要服务器资源文件改变，会生成一个不同的标识（HTTP/1.1）请求头： `If-None-Match`

当强缓存失效（不存在）【html 可以做协商缓存】，会校验协商缓存，每一次都会向服务器校验资源是否更新

- 如果没有更新，返回 **304** 通知客户端读取缓存信息，从本地缓存中获取内容进行渲染
- 如果有更新，返回 **200** 及最新资源信息，直接渲染，并把最新的 Last-Modified / ETag 和最新的资源信息缓存到本地

**数据缓存：**

没有缓存数据，从服务器拉取最新数据；有缓存数据，直接读取缓存数据【减少和服务器之间的交互频率，降低服务器压力，也可以提高页面的渲染速度】

- 页面不刷新，某些内容频繁操作，但是数据不是需要实时更新，可以做缓存【不经常更新的数据】
- 页面只要不关闭，直接读取缓存，如果页面关闭，重新打开我们也可以读取缓存中的数据【数据更新频率更低，可以设置过期时间】

客户端存储数据的方案：

1. （全局）变量存储【vuex / redux】：页面刷新或关闭后打开，之前存储的数据都没有（内存释放）
2. cookie
3. webStorage：LocalStorage SessionStorage
4. IndexedDB 浏览器数据库存储
5. Cache
6. Manifest 离线存储

**LocalStorage V.S. SessionStorage**

- LocalStorage 持久化本地存储（没有过期时间），页面关闭存储内容也是存在的，除非用户手动清除 `removeItem clear`
- SessionStorage 会话存储，页面关闭后，存储的信息会消息【页面刷新不消失】

**Cookie V.S. LocalStorage**

- Cookie 只允许一个源下最多存储 4KB 内容，所以不能存储太多的数据

  本地存储的数据是由同源访问限制的，只允许读取本源下存储的内容

- LocalStorage 可以在同源下存储 5MB 内容

  <br>

- Cookie 需要设置过期时间，超过时间就失效，并且有路径限制

- LocalStorage 持久化存储，没有过期时间，除非手动清除

  <br>

- Cookie 不稳定

  基于安全卫士或浏览器自带的清除操作，会把 Cookie 删除掉，开启无痕浏览，不能存储 Cookie

- LocalStorage 不受这些操作影响

  <br>

- Cookie 兼容低版本浏览器

- LocalStorage HTML5 新增的 API【不兼容 IE8 以下浏览器】

  <br>

- Cookie 不算严格本地存储

  客户端向服务器发送请求，会默认把本地的 Cookie 基于请求头发送给服务器，并且服务器返回的响应头中有 Set-Cookie 字段，浏览器会默认把这些信息种在客户端本地中

- LocalStorage 严格本地存储，默认情况下和服务器没有任何关系

想要基于 ajax 获取数据，必须要保证当前页面的运行是在 http/https 协议下，file 文件协议不行

```js
function getData() {
  return new Promise(resolve => {
    let xhr = new XMLHttpRequest()
    xhr.open('get', './data.json')
    xhr.onreadystatechange = () => {
      if (xhr.status === 200 && xhr.readyState === 4) {
        resolve(JSON.parse(xhr.responseText))
      }
    }
    xhr.send()
  })
}
;(async function () {
  let cache_data = localStorage.getItem('cache-data')
  if (cache_data) {
    cache_data = JSON.parse(cache_data)
    if (+new Date() - cache_data.time <= 10000) {
      return
    }
  }
  let result = await getData()
  localStorage.setItem(
    'cache-data',
    JSON.stringify({
      time: +new Date(),
      data: result,
    })
  )
})()
```

### DNS 解析

- 递归查询

![](https://gitee.com/lilyn/pic/raw/master/company-img/DNS%E8%A7%A3%E6%9E%90-%E9%80%92%E5%BD%92%E6%9F%A5%E8%AF%A2.jpg)

- 迭代查询

![](https://gitee.com/lilyn/pic/raw/master/company-img/DNS%E8%A7%A3%E6%9E%90-%E8%BF%AD%E4%BB%A3%E6%9F%A5%E8%AF%A2.jpg)

多服务器部署

- 弊端：增加了 DNS 解析次数
- 优势：资源合理利用、抗压能力增强、提高 HTTP 并发性【同源并发 HTTP 5~7 个】

每一次 DNS 解析时间预计在 20~120 毫秒

- 减少 DNS 请求次数
- DNS 预获取（DNS Prefetch）

```js
<meta http-equiv="x-dns-prefetch-control" content="on">
<link rel="dns-prefetch" href="//static.360buyimg.com"/>
```

### TCP 三次握手

- seq 序号，用来标识从 TCP 源端向目的端发送的字节流，发起方发送数据时对此进行标记
- ack 确认序号，只有 ACK 标识为 1 时，确认序号字段才有效，ack=seq+1
- 标识位
  - ACK：确认序号有效（acknowledge）
  - RST：重置连接 （reset）
  - SYN：发起一个新连接（synchronous）
  - FIN：释放一个新连接（finish）
  - seq：序号（sequence）

![三次握手](https://img-blog.csdnimg.cn/20200430211404285.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4Njg5Mzk1,size_16,color_FFFFFF,t_70#pic_center)

TCP 三次握手通俗理解，C→S：在不在，S→C：我在你在不在，C→S：我在

**三次握手为什么不用两次或四次？**

- TCP 作为一种可靠传输控制协议，其核心思想： **既要保证数据可靠传输，又要提高传输的效率**
- 两次握手只能保证客户端给服务器端的信息收到了，不能保证服务端给客户端的信息收到了（不够稳定）
- 四次握手就多余了，因为服务端已经知道客户端收到信息了，再给客户端发信息就没有意义了

![](https://gitee.com/lilyn/pic/raw/master/company-img/OSI%20TCP.png)

UDP(User Datagram Protocol) 连接没有三次握手机制

1. 相对于 TCP 来讲快
2. 不稳定可靠

### 数据传输

HTTP 报文

- 请求报文
- 响应报文

响应状态码

- 200 OK：请求已成功
- 202 Accepted：服务器已接受请求，但尚未处理（异步）
- 204 No Content：服务器成功处理了请求，但不需要返回任何实体内容
- 301 Moved Permanently：永久重定向
- 302 Move Temporarily：临时重定向
- 304 Not Modified：文档内容没有改变，走协商缓存
- 400 Bad Request : 请求参数有误
- 401 Unauthorized：请求需要权限验证
- 404 Not Found：请求失败，服务器没有这个资源
- 405 Method Not Allowed：请求方法不能由于请求相应资源
- 500 Internal Server Error：服务器未知错误
- 502 Bad Gateway：网关有误
- 503 Service Unavailable：服务器维护或过载，无法处理请求

### TCP 四次挥手

服务受到信息和标识后

1. 准备客户端需要的东西【需要时间】
2. 把信息返回给客户端

但是为了保证消息的及时反馈，此时需要立即告诉客户端：我收到你的东西了，我现在开始准备等我一会

![四次握手](https://img-blog.csdnimg.cn/20200430211431423.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4Njg5Mzk1,size_16,color_FFFFFF,t_70#pic_center)

TCP 四次挥手通俗理解，C→S：我要走了，S→C：等下，我看看还有没有数据要传输，C→S：好了，没事了，挂了吧（已经挂了），C→S：挂了

**为什么连接的时候是三次握手，关闭的时候却是四次挥手？**

- 服务端收到客户端的 SYN 连接请求，可以直接发送 SYN+ACK 报文
- 但关闭连接时，当服务器收到 FIN 报文时，很可能并不会立即关闭连接，所以只能先回复以一个 ACK 报文，告诉客户端：你发的 FIN 报文我收到了，只有等到服务端所有的报文都发送完了，才能发送 FIN 报文，因此不能一起发送，故需要四次握手

**为了减少 TCP 握手和挥手的时间，一般都使用 `Connection: keep-alive`**

数据请求：

- 长轮询：设置定时器，每隔多久发送一次请求，拿到最新数据
- 长连接：如果数据没有更新则连接不中断（服务器挂起），监听数据改变

### 页面渲染

## 性能优化汇总

1. 利用缓存

   - 对于静态资源文件实现强缓存和协商缓存（扩展：文件有更新，如何保证及时刷新？）
   - 对于不经常更新的接口数据采用本地存储做数据缓存（扩展：cookie / localStorage / vuex|redux 区别？）

2. DNS 优化

   - 分服务器部署，增加 HTTP 并发性（导致 DNS 解析变慢）
   - DNS Prefetch

3. TCP 的三次握手和四次挥手

   - Connection: keep-alive

4. 数据传输

   - 减少数据传输的大小

     内容或者数据压缩（webpack 等）

     服务器端一定要开启 GZIP 压缩（一般能压缩 60%左右）

     大批量数据分批次请求（例如：下拉刷新或者分页，保证首次加载请求数据少）

   - 减少 HTTP 请求的次数

     资源文件合并处理

     字体图标

     雪碧图 CSS-Sprit

     图片的 BASE64

5. CDN 服务器“地域分布式“

6. 采用 HTTP2.0

网络优化是前端性能优化的中的重点内容，因为大部分的消耗都发生在网络层，尤其是第一次页面加载，如何减少等待时间很重要“减少白屏的效果和时间”

- loading 人性化体验
- 骨架屏：客户端骨屏 + 服务器骨架屏
- 图片延迟加载

## HTTP1.0 VS HTTP1.1 VS HTTP2.0

### HTTP1.0 VS HTTP1.1 区别：

- **缓存处理：** HTTP1.0 中主要使用 Last-Modified、Expires 来做缓存判断标准，HTTP1.1 则引入了更多的缓存控制策略：ETag、Cache-Control...
- **带宽优化及网络连接的使用：** HTTP1.1 支持断点续传，即返回码是 206（Partial Content）
- **长连接：** HTTP1.1 中默认开启 `Connection: keep-alive`，一定程度上弥补了 HTTP1.0 每次请求都要创建连接的缺点
- **错误通知的管理：** 在 HTTP1.1 中新增了 24 个错误状态响应码，如 409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除…
- **Host 头处理：** 在 HTTP1.0 中认为每台服务器都绑定一个唯一的 IP 地址，因此，请求消息中的 URL 并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个 IP 地址。HTTP1.1 的请求消息和响应消息都应支持 Host 头域，且请求消息中如果没有 Host 头域会报告一个错误（400 Bad Request）

### HTTP2.0 VS HTTP1.x 的新特性：

- **新的二进制格式（Binary Format）：** HTTP1.x 的解析是基于文本，基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认 0 和 1 的组合，基于这种考虑 HTTP2.0 的协议解析决定采用二进制格式，实现方便且健壮

- **header 压缩：** HTTP1.x 的 header 带有大量信息，而且每次都要重复发送，HTTP2.0 使用 encoder 来减少需要传输的 header 大小，通讯双方各自 cache 一份 header fields 表，既避免了重复 header 的传输，又减小了需要传输的大小

- **服务端推送（server push）：** 例如我的网页有一个 sytle.css 的请求，在客户端收到 sytle.css 数据的同时，服务端会将 sytle.js 的文件推送给客户端，当客户端再次尝试获取 sytle.js 时就可以直接从缓存中获取到，不用再发请求了

  ```html
  Link: </styles.css>; rel=preload; as=style, </example.png>; rel=preload; as=image
  ```
  
- **多路复用（MultiPlexing）**

  HTTP/1.0 每次请求响应，建立一个 TCP 连接，用完关闭

  HTTP/1.1 「长连接」 若干个请求排队串行化单线程处理，后面的请求等待前面请求的返回才能获得执行机会，一旦有某请求超时等，后续请求只能被阻塞，毫无办法，也就是人们常说的线头阻塞；

  HTTP/2.0 「多路复用」多个请求可同时在一个连接上并行执行，某个请求任务耗时严重，不会影响到其它连接的正常执行；
