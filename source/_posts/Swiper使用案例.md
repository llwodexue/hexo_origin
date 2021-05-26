---
title: Swiper使用案例
tags:
  - Swiper
  - CSS
categories: 前端插件
author: LiLyn
copyright: ture
abbrlink: 47fd9c19
---

官网链接：[https://www.swiper.com.cn/](https://www.swiper.com.cn/)

文档链接：[Swiper使用方法](https://www.swiper.com.cn/usage/index.html) 、[Swiper的API文档](https://www.swiper.com.cn/api/index.html)

<!--more-->

### 引用Swiper

[下载 swiper](https://www.swiper.com.cn/download/index.html) 或 [使用CDN](https://www.swiper.com.cn/cdn/index.html)

下面演示的案例使用的是 swiper4.3.3

```html
<link rel="stylesheet" type="text/css" href="./swiper/css/swiper.css" />
<script src="./swiper/js/swiper.js"></script>

<!-- swiper6 -->
<link rel="stylesheet" href="https://unpkg.com/swiper/swiper-bundle.min.css"> 
<script src="https://unpkg.com/swiper/swiper-bundle.min.js"> </script>
<!-- swiper4.3.3 -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/Swiper/4.3.3/css/swiper.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/Swiper/4.3.3/js/swiper.min.js"></script>
```

几点说明：

- 为了让CSS代码层级关系更加明显，下面代码案例使用的是 Less

- 如果想清除浏览器默认效果，可以使用 [CSS Tools: Reset CSS](https://sourl.cn/MiNzfr)

  但是这个可能并不太好，可以参考 [CSS reset的重新审视 – 避免样式重置](http://www.zhangxinxu.com/wordpress/?p=758)

### 轮播图

- [图片下载链接（觅元素）](https://sourl.cn/qSAcHh)

- HTML 代码

```html
<div class="swiper-banner">
    <div class="swiper-wrapper" id="swiper-1">
        <div class="swiper-slide"><img src="img/1.jpg" /></div>
        <div class="swiper-slide"><img src="img/2.jpg" /></div>
        <div class="swiper-slide"><img src="img/3.jpg" /></div>
    </div>
</div>
```

- Less 代码

```less
@imgBannerWidth: 1920px;
@imgBannerHeight: 600px;
.img-banner {
    width: @imgBannerWidth / 2;
    height: @imgBannerHeight / 2;
}
.swiper-banner {
    &:extend(.img-banner);
    overflow: hidden;
    #swiper-1 {
        img {
            &:extend(.img-banner);
        }
    }
}
```

- JS 代码

```js
//初始化一个Swiper
new Swiper(".swiper-banner", {
    speed: 600, //切换速度
    loop: true, //循环模式
    effect: "cube", //切换效果：方块
    autoplay: {
        delay: 1000, //自动播放
    },
});
```

- 效果如下：

![效果1](https://img-blog.csdnimg.cn/20200826152012303.gif#pic_center)



### 横屏滑动

- [图片下载链接（iconfont）](https://sourl.cn/L8Vem4)

- HTML 代码

```html
<div class="swiper-nav">
    <div class="swiper-wrapper" id="swiper-2">
        <div class="swiper-slide"><img src="./img/凯旋门.png" /><p>凯旋门</p></div>
        <div class="swiper-slide"><img src="./img/古罗马斗兽场.png" /><p>古罗马斗兽场</p></div>
        <div class="swiper-slide"><img src="./img/大本钟.png" /><p>大本钟</p></div>
        <div class="swiper-slide"><img src="./img/天坛.png" /><p>天坛</p></div>
        <div class="swiper-slide"><img src="./img/巴黎圣母院.png" /><p>巴黎圣母院</p></div>
        <div class="swiper-slide"><img src="./img/悉尼歌剧院.png" /><p>悉尼歌剧院</p></div>
        <div class="swiper-slide"><img src="./img/比萨斜塔.png" /><p>比萨斜塔</p></div>
        <div class="swiper-slide"><img src="./img/泰姬陵.png" /><p>泰姬陵</p></div>
    </div>
</div>
```

- Less 代码

```less
.flex-center {
    display: flex;
    justify-content: center;
    align-items: center;
}
.swiper-nav {
    #swiper-2 {
        .swiper-slide {
            &:extend(.flex-center);
            flex-direction: column;
        }
        p {
            text-align: center;
        }
    }
}
```

- JS 代码

```js
new Swiper(".swiper-nav", {
	slidesPerView: 4, //设置slider容器能够同时显示的slides数量
});
```

- 效果如下：

![效果2](https://img-blog.csdnimg.cn/20200826153547747.gif#pic_center)



### 数字下标（两种形式）

- HTML 代码

```html
<div class="swiper-img">
    <div class="swiper-wrapper" id="swiper-3">
        <div class="swiper-slide"><img src="./img/凯旋门.png" /></div>
        <div class="swiper-slide"><img src="./img/泰姬陵.png" /></div>
        <div class="swiper-slide"><img src="./img/大本钟.png" /></div>
        <div class="swiper-slide"><img src="./img/天坛.png" /></div>
    </div>
    <!-- 分页器 -->
    <div class="swiper-pagination"></div>
</div>
<div class="swiper-img-click">
    <div class="swiper-wrapper" id="swiper-3">
        <div class="swiper-slide"><img src="./img/凯旋门.png" /></div>
        <div class="swiper-slide"><img src="./img/泰姬陵.png" /></div>
        <div class="swiper-slide"><img src="./img/大本钟.png" /></div>
        <div class="swiper-slide"><img src="./img/天坛.png" /></div>
    </div>
    <!-- 分页器 -->
    <div class="swiper-pagination"></div>
</div>
```

- Less 代码

```less
@bgColorHover: #ff2832;
@bgColor: #646464;
@bgColorFont: #fff;
@imgIconWidth: 200px;
@imgIconHeight: 200px;

.img-icon {
    width: @imgIconWidth;
    height: @imgIconHeight;
}

.swiper-img,
.swiper-img-click {
    width: @imgIconWidth + 40px;
    height: @imgIconHeight + 40px;
    background: lighten(@bgColor, 40%);
    overflow: hidden;
    #swiper-3 {
        .swiper-slide {
            &:extend(.flex-center);
            &:extend(.img-icon);
        }
    }
    .swiper-pagination {
        position: relative;
        bottom: 40px;
        .swiper-pagination-bullet {
            height: 20px;
            width: 20px;
            line-height: 20px;
            color: @bgColor;
        }
        .swiper-pagination-bullet-active {
            background: @bgColorHover;
        }
    }
}
```

- JS 代码

```js
new Swiper(".swiper-img", {
    loop: true,
    autoplay: {
        delay: 1000,
    },
    pagination: {
        el: ".swiper-pagination",
        type: "fraction", //分式
    },
});
new Swiper(".swiper-img-click", {
    loop: true,
    autoplay: {
        delay: 1000,
    },
    pagination: {
        el: ".swiper-pagination",
        clickable: true, //点击分页器的指示点分页器会控制Swiper切换
        renderBullet: function(index, className) {
            return '<span class="' + className + '">' + (index + 1) + '</span>';
        }, //渲染分页器小点
    },
});
```

- 效果如下：

![效果三](https://img-blog.csdnimg.cn/20200826155510164.gif#pic_center)



### 缩略图显示（开启焦距功能）

- [图片下载链接（官网案例）](https://sourl.cn/APX8pS)

- HTML 代码

```html
<div id="banner">
    <div class="swiper-focus">
        <div class="swiper-wrapper" id="swiper-4">
            <div class="swiper-slide"><div class="swiper-zoom-container"><img src="img/01.jpg" /></div></div>
            <div class="swiper-slide"><div class="swiper-zoom-container"><img src="img/02.jpg" /></div></div>
            <div class="swiper-slide"><div class="swiper-zoom-container"><img src="img/03.jpg" /></div></div>
            <div class="swiper-slide"><div class="swiper-zoom-container"><img src="img/04.jpg" /></div></div>
        </div>
    </div>
    <div class="swiper-pagination"></div>
</div>
```

- Less 代码

```less
#banner {
    max-width: 1200px;
    overflow: hidden;
    .swiper-focus{
        .swiper-slide {
            width: 100%;
            img {
                width: 100%;
            }
        }
    }
    .swiper-pagination {
        position: relative;
        width: 100%;
        .swiper-pagination-bullet {
            width: 25%;
            height: auto;
            background: none;
            img {
                font-size: 0;
                width: 100%;
            }
        }
    }
}
```

- JS 代码

```js
new Swiper('.swiper-focus', {
    loop: true,
    pagination: {
        el: '.swiper-pagination',
        clickable: true,
        renderBullet: function(index, className) {
            return '<span class="' + className + '"><image src="img/0' + (index + 1) + '.jpg"></span>';
        },
    },
    zoom: true, // 开启焦距功能：双击slide会放大/缩小
});
```

- 效果如下：

![效果4](https://img-blog.csdnimg.cn/2020082616193980.gif#pic_center)

