---
title: JS 实现图片延迟加载（懒加载）
tags:
  - JavaScript
  - 延迟加载
  - 懒加载
  - utils
categories: JavaScript
author: LiLyn
copyright: ture
abbrlink: 399b5a63
---

## 实现原理

1. 结构中，我们使用一个盒子包裹着图片（图片不显示的时候，可以先占据着这个位置，并且设置默认背景图或背景颜色）
2. 最开始，img 的 src 设置默认背景图，并把图片真实地址放到自定义属性中（比如：data-src ）
3. 当 JS 监听到该图片元素进入可视窗口时，将自定义属性中的地址放到 src 属性中，达到懒加载效果

<!--more-->

![懒加载](https://img-blog.csdnimg.cn/20201220232703325.gif)

**作用：**

1. 防止页面一次性向服务器发送大量请求，导致页面卡顿
2. 全部加载会耗费大量流量

**预备知识：**

1. `clientHeight`：当前盒子可视区域的高度（height+上下padding）
2. `offsetHeight`：获取当前盒子的总高度（height+上下padding+上下border）
3. `offsetTop`：当前盒子距离父级参照物的上偏移量
4. `offsetParent`：当前盒子的父级参照物
5. `scrollTop`：获取和设置当前盒子纵向滚动条卷曲的高度

## 代码实现

为了显示更加明显，当图片的下边框 = 浏览器可视窗口的下边框时，显示图片真实路径

- 图片下边框：图片距离 body 的上偏移量 + 图片自身的总高度
- 浏览器可视窗口下边框：浏览器滚动条卷曲高度 + 当前浏览器可视窗口的高度

### 单张图片懒加载

```html
<!DOCTYPE html>
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>单张图片延时加载</title>
    <style>
        * {
            margin: 0;
            padding: 0;
        }
        img {
            width: 100%;
            height: 100%;
        }
        #box {
            width: 300px;
            height: 200px;
            margin: 800px auto;
        }
    </style>
</head>

<body>
    <div id="box">
        <img src="img/default.jpg" true-img="img/1.jpg" alt="" />
    </div>
</body>
<script src="utils.js"></script>
<script>
    let box = document.getElementById("box");
    let img = document.getElementsByTagName("img")[0];
    function check() {
        // 让图片只加载一次
        if (img.flag) {
            return;
        }
        // 当前图片盒子的总高度
        let boxH = box.offsetHeight;
        // 获取当前图片盒子距离 body 上偏移量
        let boxT = utils.offset(box).top;
        // 获取浏览器可视区域高度
        let winH = utils.win("clientHeight");
        // 获取浏览器滚动条卷曲高度
        let winT = utils.win("scrollTop");
        if (winH + winT >= boxH + boxT) {
            // 创建一个 img 标签
            let newImg = new Image();
            let trueImg = img.getAttribute("true-img");
            newImg.src = trueImg;
            // 动态创建一个 img 标签用来检测当前的路径是否正确
            newImg.onload = function () {
                // 如果路径正确，执行onload函数
                img.src = trueImg;
                img.flag = true;
                newImg = null;
            };
            newImg.onerror = function () {
                // 如果路径错误，执行onerror函数
                img.src = "img/2.jpg";
                img.flag = true;
            };
        }
    }
    window.onscroll = check;
</script>
</html>
```

### 多张图片懒加载

这里使用 `getElementsByClassName` ，这个具有映射关系，下面简单介绍一下这个映射关系：

> `getElementsByClassName` 返回对象是动态的 HTMLCollection
>
> - 动态 添加/删除 元素 HTMLCollection 的长度会产生变化
>
> - - 把最后一个 li，从 ul 中删除，lis的长度 -1
>
> - - 删除的属性，lis 中访问不到
>
> ```html
> <ul id="lists">
>     <li class="bg">1</li>
>     <li>2</li>
>     <li>3</li>
>     <li id="end">4</li>
> </ul>
> <script>
>     var lis = document.getElementsByTagName("li");
>     lis[0].classList.remove("bg");
>     lists.removeChild(end);
>     console.log(lis.length); // 3
>     console.log(lis[0].classList); // DOMTokenList [value: ""]
> </script>
> ```
>
> `uerySelectorAll` 返回对象是静态 NodeList
>
> - 动态 添加/删除 元素 NodeList 的长度不会产生变化
>
> - - 把最后一个 li，从 ul 中删除，lis的长度并没有 -1
>
> - - 删除的属性，lis 中访问不到
>
> ```html
> <ul id="lists">
>     <li class="bg">1</li>
>     <li>2</li>
>     <li>3</li>
>     <li id="end">4</li>
> </ul>
> <script>
>     var lis = document.querySelectorAll("li");
>     lis[0].classList.remove("bg");
>     lists.removeChild(end);
>     console.log(lis.length); // 4
>     console.log(lis[0].classList); // DOMTokenList [value: ""]
> </script>
> ```

```html
<!DOCTYPE html>

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        * {
            margin: 0;
            padding: 0;
        }
        .box {
            margin: 500px auto;
        }
        .box img {
            display: block;
            margin: auto;
            margin-bottom: 10px;
            height: 200px;
        }
    </style>
</head>

<body>
    <div class="box">
        <img src="img/default.jpg" class="bg" true-img="img/1.jpg" alt="">
        <img src="img/default.jpg" class="bg" true-img="img/2.jpg" alt="">
        <img src="img/default.jpg" class="bg" true-img="img/3.jpg" alt="">
        <img src="img/default.jpg" class="bg" true-img="img/4.jpg" alt="">
        <img src="img/default.jpg" class="bg" true-img="img/5.jpg" alt="">
    </div>
</body>

</html>
<script src="utils.js"></script>
<script>
    // 动态的 HTMLCollection 具有映射关系
    let imgs = document.getElementsByClassName("bg");
    // 判断每一张图片是否要加载
    function delay() {
        // 当函数执行的时候，循环每一张图片，然后判断每一张图片是否需要加载
        for (var i = 0; i < imgs.length; i++) {
            delayImg(imgs[i]);
        }
    }

    function delayImg(img) {
        let imgH = img.offsetHeight;
        let winH = utils.win("clientHeight");
        let imgT = utils.offset(img).top;
        let winT = utils.win("scrollTop");
        if (winT + winH > imgH + imgT) {
            let trueSrc = img.getAttribute("true-img");
            let newImg = new Image();
            newImg.src = trueSrc;
            newImg.onload = function () {
                img.src = trueSrc;
                img.className = "";
                newImg = null;
            };
        }
    }
    // 刚进界面把已经符合的图片显示出来
    delay();
    window.onscroll = delay;
</script>
<script src="pratice2.js"></script>
```

### utils

```js
let utils = (function () {
    // 获取盒子距离body的偏移量
    function offset(el) {
        let left = el.offsetLeft;
        let top = el.offsetTop;
        let parent = el.offsetParent;
        while (parent !== document.body) {
            left += parent.offsetLeft + parent.clientLeft;
            top += parent.offsetTop + parent.clientTop;
            parent = parent.offsetParent;
        }
        return {
            left,
            top,
        };
    }

    // 设置或者获取浏览器的某些属性
    function win(attr, val) {
        if (val == undefined) {
            return document.documentElement[attr] || document.body[attr];
        }
        document.documentElement[attr] = val;
        document.body[attr] = val;
    }
    return {
        offset,
        win,
    };
})();
```

