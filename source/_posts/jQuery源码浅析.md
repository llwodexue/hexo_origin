---
title: jQuery 源码浅析
tags:
  - JavaScript
  - jQuery
  - 闭包应用
  - 冲突处理
categories: JavaScript
description: >-
  从闭包应用和冲突处理两方面浅析jQuery源码
author: LiLyn
copyright: ture
abbrlink: 6362a5
---

JS 代码执行的环境：

- 浏览器：PC端、移动端【webkit、gecko、trident、blink...】
- Hybrid 混合APP开发，把H5页面嵌入native app（IOS/安卓）的webview中【webkit】
- node，一个基于V8引擎，渲染和解析JS的环境。没有window，全局对象global
- 小程序

### 判断环境（闭包应用）

> 为什么 jQuery 即能在浏览器中运行也能在 webpack 下运行

- 形参 A 检测大概是什么环境执行

  如果 `A===window` 说明：浏览器、webview中运行

  如果 `A!==window`  说明：在Node环境下运行，不过 A 可能是Global，也可能是当前模块

- 形参 B 返回 jQuery

什么时候执行 B 函数呢？检测环境的时候执行函数 B（浏览器/Node环境）

- **Node** 应用由模块组成，采用 CommonJS 模块规范，可以用 `module && module.exports` 来检测

  - 如果支持 CommonJS 规范，需要再检测一下是否有 `window.document` ，比如：webpack 工程化环境

  - webpack 可能通过 import 导入，也可能通过 require 导入

    `import $ from 'jquery'` `let $ = require('jquery')` 

  - `module.exports` 导出的是 `factory(global, true)`（函数 B）

    所以：$->jQuery 

- else 那一块，返回 window

  浏览器导入jQuery：`<script src='jquery.min.js'></script>` ，执行 `factory(global)` （函数 B），因为 `noGlobal === "undefined"` ，执行 `window.jQuery = window.$ = jQuery` ，所以在 window 下可以使用 jQuery 和 $

```js
var A = typeof window !== "undefined" ? window : this;
var B = function (window, noGlobal) {
    // 浏览器环境下执行这个函数
    //  window -> window    noGlobal -> undefined
    // webpack环境下导入执行
    //  window -> window    noGlobal -> true
    "use strict";
    var version = "3.5.1";
    var jQuery = function (selector, context) {
        return new jQuery.fn.init(selector, context);
    };
    if (typeof noGlobal === "undefined") {
        // 浏览器直接导入
        window.jQuery = window.$ = jQuery;
    }
    // ...
    return jQuery;
};
(function (global, factory) {
    // 支持 CommonJS 模块规范[node环境]
    if (typeof module === "object" && typeof module.exports === "object") {
        module.exports = global.document ?
            // 有 window
            factory(global, true) :
            // 没有 window
            function (w) {
                if (!w.document) {
                    throw new Error("jQuery requires a window with a document");
                }
                return factory(w);
            };

    } else {
        // 不支持 CommonJS 规范的[浏览器环境]
        // global->window
        factory(global)
    }

})(A, B);
```

- **应用：暴露API**（支持浏览器直接导入也支持webpack CommonJS模块导入）

  如果你写了一个很好的库，既想在浏览器中使用，还想在 Node 中支持导入

```js
if (typeof window !== "undefined") {
    window.utils = utils;
}

if (typeof module === "object" && typeof module.exports === "object") {
    module.exports = utils;
}
```

### 冲突处理

> 有些 JavaScript 的库会使用 `$` 作为变量名（Zepto/jQuery），如果我们同时引用两个使用 `$` 作为变量名，或者引入两个不同版本的 jQuery 时，就可能会出现 `$` 冲突问题

```html
<script src="node_modules/zepto/dist/zepto.min.js"></script>
<!-- 
    $ -> Zepto
    Zepto -> Zepto
-->
<script src="node_modules/jquery/dist/jquery.min.js"></script>
<!-- 
    $ -> jQuery
    jQuery -> jQuery
-->
```

- 先看一下 Zepto 的冲突处理，先看一下全局 `$` 有没有被占用，如果没有被占用 `window.$ = Zepto`

```js
window.Zepto = Zepto
window.$ === undefined && (window.$ = Zepto)
```

- jQuery 可以使用 `jQuery.noConflict()` 来解决

```js
function factory(window, noGlobal) {
    "use strict";
    var jQuery;
    // ...
    // 在没有暴露给 window之前，把当前 window的$存一下
    var _jQuery = window.jQuery,
        _$ = window.$;

    jQuery.noConflict = function (deep) {
        if (window.$ === jQuery) {
            // 转让使用权给 Zepto
            window.$ = _$;
        }
        if (deep && window.jQuery === jQuery) {
            window.jQuery = _jQuery;
        }
        return jQuery;
    }

    if (typeof noGlobal === "undefined") {
        // 暴露给 window
        window.jQuery = window.$ = jQuery;
    }
    return jQuery;
}
```
