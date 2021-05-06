---
title: JS 实现奇偶行变色
tags:
  - JavaScript
  - 问题
  - 前端
  - 闭包
  - this
categories: JavaScript
author: LiLyn
copyright: ture
abbrlink: 97acee74
---

# JS实现奇偶行变色，鼠标滑上改变颜色，离开恢复颜色

> css 奇偶行变色是通过 `li:nth(odd)` 和 `li:nth(even)` 实现的，鼠标滑上改变颜色，离开恢复颜色是通过 `:hover` 实现的

<!--more-->

- 首先搭一下基本结构

![隔行变色-基本样式](https://gitee.com/lilyn/pic/raw/master/js-img/%E9%9A%94%E8%A1%8C%E5%8F%98%E8%89%B2-%E5%9F%BA%E6%9C%AC%E6%A0%B7%E5%BC%8F.jpg)

```html
<ul id="main">
	<li>走在前方不迷路，迷路就去找麋鹿</li>
	<li>走在前方不迷路，迷路就去找麋鹿</li>
	<li>走在前方不迷路，迷路就去找麋鹿</li>
	<li>走在前方不迷路，迷路就去找麋鹿</li>
</ul>
<script>
    var main = document.getElementById("main");
    var lst = main.getElementsByTagName("li");
    for (var i = 0; i < lst.length; i++) {
        var el = lst[i];
        if (i % 2 == 0) {
            el.style.background = "lightblue";
        } else {
            el.style.background = "lightgreen";
        }
    }
</script>
```

## 错误示范

- 在 for 循环里添加如下代码（添加在尾部）

  这里希望 `oldColor` 保存每一个 li 鼠标滑上之前的颜色

```js
var oldColor = el.style.background;
el.onmouseover = function () {
    // 鼠标滑上变色
    this.style.background = "green";
};
el.onmouseout = function () {
    // 鼠标离开之后恢复原色
    this.style.background = oldColor;
};
```

- 测试一下效果，发现出现如下问题：鼠标滑过，所有 li 都变成最后一行 li 的颜色

![隔行变色-出现问题](https://gitee.com/lilyn/pic/raw/master/js-img/%E9%9A%94%E8%A1%8C%E5%8F%98%E8%89%B2-%E5%87%BA%E7%8E%B0%E9%97%AE%E9%A2%98.gif)

- 剖析一下问题所在

  **首先 for 循环会先执行一遍，并给每一个 li 添加鼠标滑上和离开事件，当鼠标滑上或离开时，此时 for 循环已经结束，此时调用的是每一个 li 已经添加上的事件**。循环结束后 oldColor 已经变成最后一个 li 的颜色，所以触发离开事件更改的颜色永远是最后一个 li 的颜色，因此会出现如上问题

## 问题示范

- 与上面的问题，区别只在于奇偶行在赋值颜色时，是通过类添加的

```html
<style>
	.color1 {
        background: lightblue;
    }
    .color2 {
        background: lightgreen;
    }
</style>
<script>
    var main = document.getElementById("main");
    var lst = main.getElementsByTagName("li");
    for (var i = 0; i < lst.length; i++) {
        if (i % 2 == 0) {
            lst[i].style.background = "lightblue";
        } else {
            lst[i].style.background = "lightgreen";
        }
    }
</script>
```

- 在 for 循环里添加如下代码（添加在尾部），发现这个效果是正常的

```js
var oldColor = lst[i].style.background;
lst[i].onmouseover = function () {
    this.style.background = "green";
};
lst[i].onmouseout = function () {
    this.style.background = oldColor;
};
```

![隔行变色-正确显示](https://gitee.com/lilyn/pic/raw/master/js-img/%E9%9A%94%E8%A1%8C%E5%8F%98%E8%89%B2-%E6%AD%A3%E7%A1%AE%E6%98%BE%E7%A4%BA.gif)

- 但是有一些问题

  `var oldColor = el.style.background` 这个只能获取行内样式

  但是如上代码是**通过添加类的方法去设置 li 的颜色，使用 style 属性没办法获取，所以 oldColor 实际存储的是一个空字符串**。因此，鼠标滑上实际上是给 li 添加行内式代码，鼠标离开后，把行内代码变为 "" 去掉，就相当于没添加效果，此时采用的是内嵌样式（也是就通过类名添加的样式）

## 正确方法

### 方法一：给 this 添加新属性

给每个 li 上添加一个 bgColor 属性，鼠标离开时，只需访问这个 bgColor 查到值，恢复原有颜色即可

```js
// 鼠标滑上li的时候，改变那个li的背景颜色
lst[i].onmouseover = function () {
    // 把原有的颜色绑定上去
    this.bgColor = this.style.backgroundColor;
    this.style.background = "green";
};
// 鼠标离开的时候，恢复原有的颜色
lst[i].onmouseout = function () {
    this.style.background = this.bgColor;
};
```

### 方法二：闭包

手动添加一层作用域（用闭包形式），里面把鼠标事件赋值给 li 标签，被占用形成不销毁的作用域，x 是私有变量（形参），之后把私有变量 oldColor 存到闭包作用域中，当鼠标事件发生在 li 标签时，就会找到闭包作用域中的私有变量 oldColor

![闭包-隔行变色lst[1]](https://gitee.com/lilyn/pic/raw/master/js-img/%E9%97%AD%E5%8C%85-%E9%9A%94%E8%A1%8C%E5%8F%98%E8%89%B2lst%5B1%5D.jpg)

```js
(function (x) {
    var oldColor = lst[x].style.background;
    lst[x].onmouseover = function () {
        this.style.background = "green";
    };
    lst[x].onmouseout = function () {
        this.style.background = oldColor;
    };
})(i);
```

### 方法三：let

- 其实跟方法二是一样的，原理在剖析处

在 for 循环里添加如下代码（添加在尾部），这里只需把 oldColor 前面的 var 改成 let 即可

```js
let oldColor = lst[i].style.background;
lst[i].onmouseover = function () {
    this.style.background = "green";
};
lst[i].onmouseout = function () {
    this.style.background = oldColor;
};
```

- 接下来**剖析**一下这个为什么会成功

  在 for 循环中，**变量 oldColor 是用 var 声明的，在全局范围都有效，所以全局只有一个变量 oldColor。每一次循环，变量 oldColor 的值都发生改变**。也就是说，所有 li 的鼠标离开事件绑定的颜色都是最后一个 li 的颜色。我们可以小试验验证一下

  方法：在 for 循环外面添加 `oldColor = "red"` ，发现鼠标离开后所有 li 都变成红色，证明每一次循环 oldColor 被改变后，前面几次循环， li 离开事件绑定的颜色也都发生了改变

  ![隔行变色-小试验](https://gitee.com/lilyn/pic/raw/master/js-img/%E9%9A%94%E8%A1%8C%E5%8F%98%E8%89%B2-%E5%B0%8F%E8%AF%95%E9%AA%8C.gif)

  
  
  而 **let 声明的 oldColor 只在本轮循环中有效，所以每一次循环的 oldColor 其实都是一个新的变量**，因此所有 li 的鼠标离开事件绑定的颜色都是当前 li 的颜色，所以效果会成功
  
  可以去 [babel](babeljs.cn/repl) 网站，看一下 let 都做了什么
  
  ![闭包-隔行变色let](https://gitee.com/lilyn/pic/raw/master/js-img/%E9%97%AD%E5%8C%85-%E9%9A%94%E8%A1%8C%E5%8F%98%E8%89%B2let.png)

<hr>

## 完整代码

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>Document</title>
        <style>
            * {
                margin: 0;
                padding: 0;
            }
            body {
                background: darkturquoise;
            }
            ul {
                list-style: none;
            }
            #main {
                width: 500px;
                margin: auto;
            }
            #main li {
                height: 50px;
                line-height: 50px;
                text-align: center;
            }
        </style>
    </head>
    <body>
        <ul id="main">
            <li>走在前方不迷路，迷路就去找麋鹿</li>
            <li>走在前方不迷路，迷路就去找麋鹿</li>
            <li>走在前方不迷路，迷路就去找麋鹿</li>
            <li>走在前方不迷路，迷路就去找麋鹿</li>
        </ul>
    </body>
</html>
<script>
    var main = document.getElementById("main");
    var lst = main.getElementsByTagName("li");
    for (var i = 0; i < lst.length; i++) {
        var el = lst[i];
        if (i % 2 == 0) {
            el.style.background = "lightblue";
        } else {
            el.style.background = "lightgreen";
        }
        var oldColor = el.style.background;
        el.onmouseover = function () {
            this.bgColor = this.style.backgroundColor;
            this.style.background = "green";
        };
        el.onmouseout = function () {
            this.style.background = this.bgColor;
        };
    }
</script>
```

![隔行变色-正确显示](https://gitee.com/lilyn/pic/raw/master/js-img/%E9%9A%94%E8%A1%8C%E5%8F%98%E8%89%B2-%E6%AD%A3%E7%A1%AE%E6%98%BE%E7%A4%BA.gif)