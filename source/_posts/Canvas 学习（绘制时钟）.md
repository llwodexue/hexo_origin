---
title: Canvas 学习（时钟 绘图板）
tags:
  - Canvas
  - JavaScript
categories: JavaScript
description: Canvas画圆、时钟、绘图板
author: LiLyn
copyright: ture
abbrlink: 453bf97e
---
## canvas画圆

- 为什么需要用canvas，如果用`onmousemove`，每一次移动都会进行DOM重绘，非常消耗性能，这时我们需要看一下 canvas

```html
<div id="canvas"></div>
<script>
    canvas.onmousemove = (e) => {
        let div = document.createElement("div");
        div.style.position = "absolute";
        div.style.left = e.clientX + "px";
        div.style.top = e.clientY + "px";
        div.style.width = "6px";
        div.style.height = "6px";
        div.style.marginLeft = "-3px";
        div.style.marginRight = "-3px";
        div.style.borderRadius = "50%";
        div.style.backgroundColor = "black";
        canvas.appendChild(div);
    }
</script>
```

- 我们现在已经给 canvas 设置一屏宽高，但还是出现滚动条，原因是 canvas 是 inline 元素，设置宽高不起作用，所以需要把它转换成块级元素（**不要这样做，这样做会拉伸元素**）

```html
<canvas id="canvas"></canvas>
<style>
    #canvas {
        height: 100vh;
        width: 100vw;
        border: 1px solid red;
        /* display: block; */
    }
</style>
```

- `<canvas>` 标签只有两个属性**——** `width`和`height`，可以使用 JS 设置canvas 宽高，这里不能直接给 canvas 设置 `height: 100vh;` `width: 100vw;` 

```js
let canvas = document.getElementById("canvas");
canvas.width = document.documentElement.clientWidth;
canvas.height = document.documentElement.clientHeight;
```

- canvas画圆

```js
let painting = false;
let ctx = canvas.getContext("2d");
ctx.fillStyle = "black";
ctx.strokeStyle = "none";

canvas.onmousedown = (e) => {
    painting = true;
}
canvas.onmousemove = (e) => {
    if (painting === true) {
        ctx.beginPath();
        ctx.arc(e.clientX, e.clientY, 10, 0, 2 * Math.PI);
        ctx.stroke();
        ctx.fill()
    }
}
canvas.onmouseup = () => {
    painting = false;
}
```

## Canvas 

### 颜色、样式和阴影

| 属性                                                         | 描述                                     |
| ------------------------------------------------------------ | ---------------------------------------- |
| [fillStyle](https://www.w3school.com.cn/tags/canvas_fillstyle.asp) | 设置或返回用于填充绘画的颜色、渐变或模式 |
| [strokeStyle](https://www.w3school.com.cn/tags/canvas_strokestyle.asp) | 设置或返回用于笔触的颜色、渐变或模式     |
| [shadowColor](https://www.w3school.com.cn/tags/canvas_shadowcolor.asp) | 设置或返回用于阴影的颜色                 |
| [shadowBlur](https://www.w3school.com.cn/tags/canvas_shadowblur.asp) | 设置或返回用于阴影的模糊级别             |
| [shadowOffsetX](https://www.w3school.com.cn/tags/canvas_shadowoffsetx.asp) | 设置或返回阴影距形状的水平距离           |
| [shadowOffsetY](https://www.w3school.com.cn/tags/canvas_shadowoffsety.asp) | 设置或返回阴影距形状的垂直距离           |

### 线条样式

| 属性                                                         | 描述                                     |
| ------------------------------------------------------------ | ---------------------------------------- |
| [lineCap](https://www.w3school.com.cn/tags/canvas_linecap.asp) | 设置或返回线条的结束端点样式             |
| [lineJoin](https://www.w3school.com.cn/tags/canvas_linejoin.asp) | 设置或返回两条线相交时，所创建的拐角类型 |
| [lineWidth](https://www.w3school.com.cn/tags/canvas_linewidth.asp) | 设置或返回当前的线条宽度                 |
| [miterLimit](https://www.w3school.com.cn/tags/canvas_miterlimit.asp) | 设置或返回最大斜接长度                   |

### 矩形

| 方法                                                         | 描述                         |
| ------------------------------------------------------------ | ---------------------------- |
| [rect()](https://www.w3school.com.cn/tags/canvas_rect.asp)   | 创建矩形                     |
| [fillRect()](https://www.w3school.com.cn/tags/canvas_fillrect.asp) | 绘制“被填充”的矩形           |
| [strokeRect()](https://www.w3school.com.cn/tags/canvas_strokerect.asp) | 绘制矩形（无填充）           |
| [clearRect()](https://www.w3school.com.cn/tags/canvas_clearrect.asp) | 在给定的矩形内清除指定的像素 |

### 路径

| 方法                                                         | 描述                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------- |
| [fill()](https://www.w3school.com.cn/tags/canvas_fill.asp)   | 填充当前绘图（路径）                                    |
| [stroke()](https://www.w3school.com.cn/tags/canvas_stroke.asp) | 绘制已定义的路径                                        |
| [beginPath()](https://www.w3school.com.cn/tags/canvas_beginpath.asp) | 起始一条路径，或重置当前路径                            |
| [moveTo()](https://www.w3school.com.cn/tags/canvas_moveto.asp) | 把路径移动到画布中的指定点，不创建线条                  |
| [closePath()](https://www.w3school.com.cn/tags/canvas_closepath.asp) | 创建从当前点回到起始点的路径                            |
| [lineTo()](https://www.w3school.com.cn/tags/canvas_lineto.asp) | 添加一个新点，然后在画布中创建从该点到最后指定点的线条  |
| [clip()](https://www.w3school.com.cn/tags/canvas_clip.asp)   | 从原始画布剪切任意形状和尺寸的区域                      |
| [quadraticCurveTo()](https://www.w3school.com.cn/tags/canvas_quadraticcurveto.asp) | 创建二次贝塞尔曲线                                      |
| [bezierCurveTo()](https://www.w3school.com.cn/tags/canvas_beziercurveto.asp) | 创建三次方贝塞尔曲线                                    |
| [arc()](https://www.w3school.com.cn/tags/canvas_arc.asp)     | 创建弧/曲线（用于创建圆形或部分圆）                     |
| [arcTo()](https://www.w3school.com.cn/tags/canvas_arcto.asp) | 创建两切线之间的弧/曲线                                 |
| [isPointInPath()](https://www.w3school.com.cn/tags/canvas_ispointinpath.asp) | 如果指定的点位于当前路径中，则返回 true，否则返回 false |

### 转换

| 方法                                                         | 描述                                           |
| ------------------------------------------------------------ | ---------------------------------------------- |
| [scale()](https://www.w3school.com.cn/tags/canvas_scale.asp) | 缩放当前绘图至更大或更小                       |
| [rotate()](https://www.w3school.com.cn/tags/canvas_rotate.asp) | 旋转当前绘图                                   |
| [translate()](https://www.w3school.com.cn/tags/canvas_translate.asp) | 重新映射画布上的 (0,0) 位置                    |
| [transform()](https://www.w3school.com.cn/tags/canvas_transform.asp) | 替换绘图的当前转换矩阵                         |
| [setTransform()](https://www.w3school.com.cn/tags/canvas_settransform.asp) | 将当前转换重置为单位矩阵。然后运行 transform() |

### 文本

| 属性                                                         | 描述                                     |
| ------------------------------------------------------------ | ---------------------------------------- |
| [font](https://www.w3school.com.cn/tags/canvas_font.asp)     | 设置或返回文本内容的当前字体属性         |
| [textAlign](https://www.w3school.com.cn/tags/canvas_textalign.asp) | 设置或返回文本内容的当前对齐方式         |
| [textBaseline](https://www.w3school.com.cn/tags/canvas_textbaseline.asp) | 设置或返回在绘制文本时使用的当前文本基线 |

### 图像绘制

| 方法                                                         | 描述                         |
| ------------------------------------------------------------ | ---------------------------- |
| [drawImage()](https://www.w3school.com.cn/tags/canvas_drawimage.asp) | 向画布上绘制图像、画布或视频 |

### 像素操作

| 属性   | 描述                                                |
| ------ | --------------------------------------------------- |
| width  | 返回 ImageData 对象的宽度                           |
| height | 返回 ImageData 对象的高度                           |
| data   | 返回一个对象，其包含指定的 ImageData 对象的图像数据 |

## Canvas练习

### 绘制圆形和文本

- 绘制线段

```js
ctx.beginPath();
// 设置绘制的起始点
ctx.moveTo(50, 50);
// 设置经过某个位置
ctx.lineTo(50, 300);
ctx.lineTo(300, 100);
// 设置绘制的结束点
ctx.closePath();

// 起始路径的线段边缘设置为圆角
ctx.lineCap = "round";
// 转折处的线段设置为圆角
ctx.lineJoin = "round"
// 设置颜色样式
ctx.strokeStyle = "aqua";
ctx.lineWidth = 20;
ctx.stroke()
```

- 绘制圆

```js
// 默认为false，顺时针，true逆时针
// ctx,arc(x, y, radius, startAngle, endAngle, anticlockwise)
ctx.arc(300, 300, 100, 0, 2 * Math.PI, true);
ctx.fillStyle = "bisque";
ctx.fill();
ctx.stroke();
```

- 弹幕字体

```js
ctx.font = "50px sans-serif"
var x = 1200;
setInterval(function () {
    ctx.clearRect(0, 0, 1200, 1200);
    x -= 10;
    if (x <= 0) {
        x = 1200;
    }
    ctx.fillText("hello", x, 100);
    ctx.strokeText("world", x, 200);
}, 40)
```

### 绘制图形

- 绘制图像（同一图片）

```js
var img = new Image();
img.src = "./imgs/2.jpg";
img.onload = function () {
    ctx.drawImage(img, 50, 100, 480, 270);
}
var img2 = new Image();
img2.src = "./imgs/3.jpg";
img2.onload = function () {
    ctx.drawImage(img2, 400, 400, 480, 270, 100, 400, 480, 270);
}
```

- 绘制图像（视频抽帧）

```js
var video = document.querySelector("video");
var interval = null;
video.onplay = function () {
    interval = setInterval(function () {
        ctx.drawImage(video, 0, 800, 800, 400)
    }, 32)
}
video.onpause = function () {
    clearInterval(interval);
}
```

## 绘制时钟

ES6版本参考这个（附带效果）：[https://github.com/llwodexue/clock](https://github.com/llwodexue/clock)

```js
let canvas1 = document.querySelector("#canvas1");
let ctx = canvas1.getContext("2d");
ctx.clearRect(0, 0, 800, 600);
ctx.save();
ctx.translate(400, 300);
ctx.rotate(-Math.PI / 2);
// 绘制表盘
ctx.beginPath();
ctx.arc(0, 0, 200, 0, 2 * Math.PI);
ctx.strokeStyle = "darkgrey";
ctx.lineWidth = 10;
ctx.stroke();
ctx.closePath();

function renderClock() {
    ctx.clearRect(0, 0, 800, 600);
    ctx.save();
    ctx.translate(400, 300);
    ctx.rotate(-Math.PI / 2);
    // 绘制表盘
    ctx.beginPath();
    ctx.arc(0, 0, 200, 0, 2 * Math.PI);
    ctx.strokeStyle = "darkgrey";
    ctx.lineWidth = 10;
    ctx.stroke();
    ctx.closePath();

    function drawLine({ count, start, end, width, color }) {
        let i = 0;
        for (; i < count; i++) {
            ctx.beginPath();
            ctx.rotate((2 * Math.PI) / count);
            ctx.moveTo(start, 0);
            ctx.lineTo(end, 0);
            ctx.lineWidth = width;
            ctx.strokeStyle = color;
            ctx.stroke();
            ctx.closePath();
        }
    }
    // 绘制分针刻度线
    drawLine({
        count: 60,
        start: 180,
        end: 190,
        width: 2,
        color: "orangered",
    });
    // 绘制时针刻度线
    drawLine({
        count: 12,
        start: 180,
        end: 200,
        width: 10,
        color: "darkgrey",
    });

    let time = new Date();
    let min = time.getMinutes();
    let sec = time.getSeconds();
    let hour = time.getHours();
    hour = hour > 12 ? hour - 12 : hour;
    // 绘制秒针
    ctx.save();
    ctx.beginPath();
    ctx.rotate(((2 * Math.PI) / 60) * sec);
    ctx.moveTo(-30, 0);
    ctx.lineTo(170, 0);
    ctx.lineWidth = 2;
    ctx.strokeStyle = "red";
    ctx.stroke();
    ctx.closePath();
    ctx.restore();
    // 绘制分针
    ctx.save();
    ctx.beginPath();
    ctx.rotate(((2 * Math.PI) / 60) * min + ((2 * Math.PI) / 3600) * sec);
    ctx.moveTo(-20, 0);
    ctx.lineTo(150, 0);
    ctx.lineWidth = 4;
    ctx.strokeStyle = "darkblue";
    ctx.stroke();
    ctx.closePath();
    ctx.restore();
    // 绘制时针
    ctx.save();
    ctx.beginPath();
    ctx.rotate(
        ((2 * Math.PI) / 12) * hour +
            ((2 * Math.PI) / 12 / 60) * min +
            ((2 * Math.PI) / 12 / 60 / 60) * sec
    );
    ctx.moveTo(-10, 0);
    ctx.lineTo(130, 0);
    ctx.lineWidth = 6;
    ctx.strokeStyle = "darkgrey";
    ctx.stroke();
    ctx.closePath();
    ctx.restore();
    // 绘制表圈
    ctx.beginPath();
    ctx.arc(0, 0, 10, 0, 2 * Math.PI);
    ctx.fillStyle = "deepskyblue";
    ctx.fill();
    ctx.closePath();
    ctx.restore();
}
setInterval(function () {
    renderClock();
}, 1000);
```

## Canvas 绘图板

绘图板参考（附带效果）： [https://github.com/llwodexue/canvas_painting](https://github.com/llwodexue/canvas_painting)

![](https://gitee.com/lilyn/pic/raw/master/company-img/%E7%BB%98%E5%9B%BE%E6%9D%BF.gif)

## 参考

[Fira Code | 为写程序而生的字体](https://www.jianshu.com/p/266b4fa2c446)

[Canvas教程](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial)

[Detecting touch screen devices with Javascript](https://stackoverflow.com/questions/3974827/detecting-touch-screen-devices-with-javascript)

[HTML 5 Canvas 参考手册](https://www.w3school.com.cn/tags/html_ref_canvas.asp)

