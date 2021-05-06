---
title: JS实现商城排序
tags:
  - JavaScript
  - flex
  - DocumentFragment
categories: JavaScript
author: LiLyn
copyright: ture
abbrlink: d2addf30
---

## 实现效果

- 根据上架时间/热度/价格进行升序/降序排序
- 上箭头亮代表升序，下箭头亮代表降序

<!--more-->

![商城排序](https://img-blog.csdnimg.cn/20201220203345422.gif)

### 渲染 li 标签

- 获取 ul 以便让每一个 li 渲染到页面

```js
var lists = document.querySelector(".lists");
var lis = null;
```

- 把每一个 li 渲染到页面上

  这里为了操作方便，给每一个 li 绑定自定义属性，之后根据属性值获取其中内容 `li.getAttribute(...)` 即可

  如果不这样做，则需要通过 `li.querySelector(...).innerText` 获取

  **注意：**`querySelectorAll` 获取到的是类数组，因为需要进行排序，使用数组的 sort() 方法，所以需要用 `[].slice.call()` 或 `Array.from` 转换成数组

```js
function Init(data) {
    let str = "";
    for (let i = 0; i < data.length; i++) {
        const el = data[i];
        str += `<li time="${el.time}" hot="${el.hot}" price="${el.price}">
        <img src="${el.img}" alt="">
        <p class="title">${el.title}</p>
        <p class="time">${el.time}</p>
        <p class="info">
            <span class="price">${el.price}</span>
            <span class="hot">${el.hot}</span>
        </p>
    </li>`;
    }
    lists.innerHTML = str;
    lis = [].slice.call(lists.querySelectorAll("li"));
}
```

### 点击 a 标签进行排序

- 获取 a 标签

```js
var links = document.querySelectorAll(".top a");
```

- 给 a 标签添加鼠标点击事件

  为实现每次排序都是上一次的倒序，需要给每一个 a 标签添加一个自定义属性作为标志 `links[i].flag = -1` ，flag 值为1代表升序，每次点击事件只需 flag 取反即可 `this.flag *= -1`

  **注意：** sort() 方法中回调函数 this 指向 window，所以需要用一个变量存储 this `let that = this;`

```js
for (let i = 0; i < links.length; i++) {
    links[i].flag = -1;
    links[i].onclick = function () {
        this.flag *= -1;
        let that = this;
        let sortFlag = this.getAttribute("sortFlag");
        if (sortFlag == "time") {
            lis.sort(function (a, b) {
                return (a.getAttribute(sortFlag).replace(/-/g, "") - b.getAttribute(sortFlag).replace(/-/g, "")) * that.flag;
            });
        } else {
            lis.sort(function (a, b) {
                return (a.getAttribute(sortFlag) - b.getAttribute(sortFlag)) * that.flag;
            });
        }
        for (let i = 0; i < lis.length; i++) {
            lists.appendChild(lis[i])
        }
    };
}

```

### 点击 a 标签点亮排序箭头

- 升/降序高亮对应箭头字体图标

```js
let arrows = this.querySelectorAll("b");
let arrCur = document.querySelectorAll(".top b.current");
for (let i = 0; i < arrCur.length; i++) {
    arrCur[i].classList.remove("current");
}
if (this.flag == 1) {
    arrows[0].classList.add("current");
} else {
    arrows[1].classList.add("current");
}
```

### 优化

- for 每进行一次循环都会引起浏览器的回流，非常耗性能

```js
for (let i = 0; i < lis.length; i++) {
    lists.appendChild(lis[i]);
}
```

- 可以使用 `DocumentFragment` 来创建一个新的空白的文档片段，把元素附加到这个文档片段中，之后通过这个文档片段附加到 DOM 树

  因为**文档片段存在内存中**，并不在 DOM 树中，所以将子元素插入到文档片段时不会引起页面回流。因此，使用文档片段会带来更好的性能

```js
let frg = document.createDocumentFragment();
for (let i = 0; i < lis.length; i++) {
    frg.appendChild(lis[i]);
}
lists.appendChild(frg)
```

- 如果希望点击完当前a标签，再点击其他a标签时都是升序排列的话，需要每次都将其他标签的flag设置为 -1

```js
for (let i = 0; i < links.length; i++) {
    if (links[i] != this) {
        links[i].flag = -1;
    }
}
```

## 完整代码

### index.html（html+css）

- 创建 `index.html` 文件，粘贴如下代码

```html
<!DOCTYPE html>

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>商城排序</title>
    <style>
        * {
            margin: 0;
            padding: 0;
        }
        ul {
            list-style: none;
        }
        a {
            text-decoration: none;
        }
        body, html {
            background: lavender;
        }
        .main {
            width: 1185px;
            margin: 0 auto;
        }
        .main .topBar {
            height: 50px;
            background: #fff;
            display: flex;
            align-items: center;
            padding-left: 20px;
            box-sizing: border-box;
        }
        .main .shortBtn {
            display: flex;
            align-items: center;
            margin: 0 10px;
            color: blue;
        }
        .main .shortBtn .arrows {
            color: #333;
            display: flex;
            flex-direction: column;
            margin-left: 10px;
        }
        .main .shortBtn .iconfont {
            font-size: 14px;
        }
        .shortBtn:hover>span {
            color: red;
        }
        .lists {
            display: flex;
            flex-wrap: wrap;
        }
        .lists li {
            margin-top: 20px;
            width: 225px;
            background: #fff;
            padding: 20px;
            box-sizing: border-box;
            font-size: 14px;
            margin-right: 15px;
            line-height: 30px;
        }
        .lists li:nth-child(5n){
            margin-right: 0;
        }
        .lists li img {
            width: 140px;
            display: block;
            margin: 0 auto;
        }
        .lists li .numInfo {
            display: flex;
            justify-content: space-between;
        }
        .arrowUp {
            border: 5px solid #333;
            margin-bottom: 10px;
            border-color: transparent transparent #333 transparent;
        }
        .arrowUp.current {
            border-color: transparent transparent red transparent;
        }
        .arrowDown {
            border: 5px solid #333;
            border-color: #333 transparent transparent transparent;
        }
        .arrowDown.current {
            border-color: red transparent transparent transparent;
        }
    </style>
</head>

<body>
    <div class="main">
        <div class="topBar">
            <span>排序</span>
            <a href="javascript:;" class="shortBtn" sortFlag="time">
                <span>上架时间</span>
                <div class="arrows">
                    <b class="arrowUp"></b>
                    <b class="arrowDown"></b>
                </div>
            </a>
            <a href="javascript:;" class="shortBtn" sortFlag="hot">
                <span>热度</span>
                <div class="arrows">
                    <b class="arrowUp"></b>
                    <b class="arrowDown"></b>
                </div>
            </a>
            <a href="javascript:;" class="shortBtn" sortFlag="price">
                <span>价格</span>
                <div class="arrows">
                    <b class="arrowUp"></b>
                    <b class="arrowDown"></b>
                </div>
            </a>
        </div>
        <!-- 图片列表 -->
        <ul class="lists"></ul>
    </div>
</body>

</html>
<script src="index.js"></script>
```

### data.json

- 这里需要用 Ajax 请求一下如下 json 数据
- 创建 `data.json` ，粘贴如下数据

```js
[
    {
        "id": 1,
        "title": "HUAWEI全网通版（亮黑色）",
        "price": 499,
        "time": "2017-03-15",
        "hot": 198,
        "img": "img/1.jpg"
    },
    {
        "id": 2,
        "title": "HUAWEI（曜石黑）",
        "price": 129,
        "time": "2017-02-08",
        "hot": 25,
        "img": "img/2.jpg"
    },
    {
        "id": 3,
        "title": "华为畅享7（香槟金）",
        "price": 895,
        "time": "2017-01-25",
        "hot": 568,
        "img": "img/3.jpg"
    },
    {
        "id": 4,
        "title": "HUAWEI全网通版（曜石黑）",
        "price": 1895,
        "time": "2016-12-30",
        "hot": 20000,
        "img": "img/4.jpg"
    },
    {
        "id": 5,
        "title": "HUAWEI全网通版（玫瑰金）",
        "price": 3587,
        "time": "2016-01-30",
        "hot": 1032654,
        "img": "img/5.jpg"
    },
    {
        "id": 6,
        "title": "华为畅享7（香槟金）",
        "price": 992,
        "time": "2018-01-01",
        "hot": 1,
        "img": "img/6.jpg"
    },
    {
        "id": 7,
        "title": "HUAWEI全网通版（樱语粉）",
        "price": 564,
        "time": "2017-02-19",
        "hot": 400,
        "img": "img/7.jpg"
    },
    {
        "id": 8,
        "title": "HUAWEI全网通版（曜石黑）",
        "price": 420,
        "time": "2017-01-25",
        "hot": 240,
        "img": "img/8.jpg"
    },
    {
        "id": 9,
        "title": "HUAWEI P10（钻雕金）",
        "price": 12,
        "time": "2014-01-01",
        "hot": 12345678,
        "img": "img/9.jpg"
    },
    {
        "id": 10,
        "title": "HUAWEI全网通版（曜石黑）",
        "price": 420,
        "time": "2017-01-25",
        "hot": 240,
        "img": "img/8.jpg"
    }
]
```

### index.js

```js
// 获取页面元素
let lists = document.querySelector(".lists");
let links = document.querySelectorAll(".topBar a");
let lis = lists.getElementsByTagName("li");
// Ajax 请求数据
let data = null;

// 1.数据请求 Ajax四部曲
function getData(url) {
    let xhr = new XMLHttpRequest();
    xhr.open("get", url);
    xhr.onreadystatechange = function () {
        if (xhr.readyState == 4 && xhr.status == 200) {
            data = JSON.parse(xhr.responseText);
            BindHtml(data);
        }
    };
    xhr.send();
}
getData("data/data.json");

// 2.渲染li
function BindHtml(data) {
    var str = "";
    for (let i = 0; i < data.length; i++) {
        const el = data[i];
        str += `<li time="${el.time}" price="${el.price}" hot="${el.hot}">
        <img src="${el.img}" alt="">
        <p class="title">${el.title}</p>
        <p class="time">${el.time}</p>
        <p class="numInfo">
            <span class="price">${el.price}</span>
            <span class="hot">${el.hot}</span>
        </p>
    </li>`;
    }
    lists.innerHTML = str;
    lis = [].slice.call(lis);
}
// 3.给link绑定事件
for (let i = 0; i < links.length; i++) {
    links[i].flag = -1;
    links[i].onclick = function () {
        let that = this;
        for (let i = 0; i < links.length; i++) {
            if (links[i] != this) {
                links[i].flag = -1;
            }
        }
        this.flag *= -1;
        let sortFlag = this.getAttribute("sortFlag");
        let arrows = this.querySelectorAll("b");
        let arrCur = document.querySelectorAll("b.current");
        for (let i = 0; i < arrCur.length; i++) {
            arrCur[i].classList.remove("current");
        }
        if (this.flag == 1) {
            arrows[0].classList.add("current");
        } else {
            arrows[1].classList.add("current");
        }
        let frg = document.createDocumentFragment();
        if (sortFlag == "time") {
            lis.sort(function (a, b) {
                return (a.getAttribute(sortFlag).replace(/-/g, "") - b.getAttribute(sortFlag).replace(/-/g, "")) * that.flag;
            });
        } else {
            lis.sort(function (a, b) {
                return (a.getAttribute(sortFlag) - b.getAttribute(sortFlag)) * that.flag;
            });
        }
        for (let i = 0; i < lis.length; i++) {
            frg.appendChild(lis[i]);
        }
        lists.appendChild(frg);
    };
}
```
