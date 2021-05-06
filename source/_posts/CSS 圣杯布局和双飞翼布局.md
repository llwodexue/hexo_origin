---
title: 圣杯布局和双飞翼布局
tags:
  - CSS
  - 圣杯布局
  - 双飞翼布局
categories: CSS
author: LiLyn
copyright: ture
abbrlink: '6669e648'
---

圣杯布局和双飞翼布局，虽然两者的实现方法略有差异，不过都遵循了以下要点：

- 两侧宽度固定，中间宽度自适应
- 中间部分在 DOM 结构上优先，以便先行渲染
- 允许三列中的任意一列成为最高列

<!--more-->

## 圣杯布局

- 页面分为左中右3个部分，其中左右两侧固定宽度，而中间部分自适应

html结构

- 这里把 center 部分放在最前面，然后是 left、right

```html
<div class="container">
    <div class="center">中间自适应</div>
    <div class="left">左侧固定宽度</div>
    <div class="right">右侧固定宽度</div>
</div>
```

### 左右浮动 + 中间 100% 宽度

- 将三列都设置 `float: left` 使其在同一排显示
- left 和 right 设置固定宽度
- 由于 center 宽度为 100% 将父元素占满了，因此 left 和 right 只能换行显示

```html
<style type="text/css">
    .container {
    	padding: 0 150px 0 200px;
    }
    .container>div {
        height: 200px;
        float: left;
    }
    .center {
        background: aqua;
        width: 100%;
    }
    .left {
        width: 200px;
        background: lightblue;
    }
    .right {
        width: 150px;
        background: lightgreen;
    }
</style>
```

![圣杯布局-都设为浮动,中间宽设为100%](https://gitee.com/lilyn/pic/raw/master/css-img/圣杯布局-都设为浮动,中间宽设为100%.jpg)

- 设置 left `margin-left: -100%`，使其上移到 center 一行，并与 center 重叠  
- 设置 right `margin-right: -150px`，使其上移到 center 一行，并与 center 重叠

![圣杯布局-设置margin-left](https://gitee.com/lilyn/pic/raw/master/css-img/%E5%9C%A3%E6%9D%AF%E5%B8%83%E5%B1%80-%E8%AE%BE%E7%BD%AEmargin-left.jpg)

- 设置 left `position: relative; left: -200px`
- 设置 left `position: relative; left: 150px`

![圣杯布局-设置定位](https://gitee.com/lilyn/pic/raw/master/css-img/%E5%9C%A3%E6%9D%AF%E5%B8%83%E5%B1%80-%E8%AE%BE%E7%BD%AE%E5%AE%9A%E4%BD%8D.jpg)

- 如果宽度无法容下3个盒子会换行显示，设置 `min-width: 600px`，可以解决

![圣杯布局-尺寸不够时会换行显示](https://gitee.com/lilyn/pic/raw/master/css-img/%E5%9C%A3%E6%9D%AF%E5%B8%83%E5%B1%80-%E5%B0%BA%E5%AF%B8%E4%B8%8D%E5%A4%9F%E6%97%B6%E4%BC%9A%E6%8D%A2%E8%A1%8C%E6%98%BE%E7%A4%BA.jpg)

### flex

- 将 container 设置为弹性布局，container 变为了 flex 容器，子元素 center、left、right 变为了 flex 项目
- 设置 left 的 order 属性来调整位置，默认为0，值越小越靠前
- left 和 right 设置为固定宽度，使用 width 或 flex-basis
- 让 center 自动填充剩余空间，使用 flex-grow（默认为0） 或 flex 即可

```html
<style media="screen">
    .container {
        display: flex;
    }

    .container>div {
        height: 200px;
    }
    .center {
        background: aqua;
        flex-grow:1;
    }
    .left {
        flex-basis: 200px;
        background: lightblue;
        order: -1;
    }
    .right {
        flex-basis: 150px;
        background: lightgreen;
    }
</style>
```

## 双飞翼布局

html结构

- center 是鸟的身体，left 和 right 是鸟的翅膀，先把 center 放好，再将翅膀移动到合适的位置

```html
<div class="container">
    <div class="center">
        <div class="inner">中间自适应</div>
    </div>
    <div class="left">左侧固定宽度</div>
    <div class="right">右侧固定宽度</div>
</div>
```

- 把之前给 container 的 padding 加给 center，并把它转换成 IE 盒模型
- 再给 left 和 right `margin-left` 即可