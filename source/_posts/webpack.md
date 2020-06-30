---
title: 初识webpack
date: 2020-06-26 10:24:38
description: webpack基础
categories: webpack
tags: webpack
---

### 基础

#### webpack 是什么

webpack 可以分析你的项目结构，找到 JavaScript 模块以及其它的一些浏览器不能直接运行的拓展语言（Scss，TypeScript 等），并将其打包为合适的格式以供浏览器使用

#### webpack 学习路线
![alt webpack路线](/images/webpack.jpg "图")

#### webpack 基础用法，概念

##### entry

entry 用来配置入口文件的地址，支持三种写法

1.字符串

```
entry: path.resolve(__dirname, "src/index.js")
```

打包后生成 main.js 
2.对象

```
entry: {
    main: path.resolve(__dirname, "src/index.js"),
    index: path.resolve(__dirname, "src/main.js"),
  }
```

打包后生成 main.js，index.js

3.数组

```
entry: [
    path.resolve(__dirname, "src/index.js"),
    path.resolve(__dirname, "src/main.js"),
  ]
```

打包后生成 main.js
如果key不指定名称默认 main.js，其中数组写法等价对象 key-value

##### output

output 用来配置出口文件的地址

```
output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name].[hash].js",
  },
```

##### mode

mode 支持三种种模式：分别是 production，development，none
- mode为production时，会启用各种性能优化的功能，包括构建结果优化以及 webpack 运行性能优化
- mode为development时，则会开启 debug 工具，运行时打印详细的错误信息，以及更加快速的增量编译构建

##### status

status 用来控制命令行展示的信息[地址](https://webpack.docschina.org/configuration/stats/)
```
stats: {
    modules: false,
    children: false,
    chunks: false,
    chunkModules: false,
}
```
我配置的modules，children，chunks，chunkModules都不显示看起来信息没那么多
如图![status](/images/status.jpg)

##### optimization

##### performance
展示性能提示
```
performance: {
    // false | "error" | "warning"
    hints: "warning",
    // 根据入口起点的最大体积，控制webpack何时生成性能*提示,整数类型,以字节为单位
    maxEntrypointSize: 5000000,
    // 最大5m
    maxAssetSize: 1024 * 1024 * 5,
}
```

##### resolve解析
常用到的两个
1.extensions
指定extension之后可以不用在require或是import的时候加文件扩展名,会依次尝试添加扩展名进行匹配
```
extensions: [".js", "vue", ".jsx", ".json", ".css"]
```
2.alias
设置别名
```
alias: {
      "@": path.resolve(__dirname, "../src"), // 这样配置后 @ 可以指向 src 目录
}
```

##### module

import，require，define ，css/sass/less 文件中的 @import==引过来的

##### 文件指纹

###### 1.hash

hash 是以项目为入口生成的，每次编译都会重新生成

###### 2.chunckhash

chunckhash 根据不同的入口文件(Entry)进行依赖文件解析、构建对应的 chunk，生成对应的哈希值

###### 3.contenthash

contenthash 主要是为了应对 js 引入 css 文件，js 改变 css 的 hash 也跟着改变这种情况的，contenthash 可以在 js 中引入 css,只改变 js 的时候不改变 css 的 hash

##### sourcemap

sourcemap 是为了解决运行浏览器运行时帮助我们 debug 到原始开发代码
[devtool](https://www.webpackjs.com/configuration/devtool/#devtool)
<!-- ##### devServer -->
##### plugin

- 常用 plugin
  copy-webpack-plugin
  clean-webpack-plugin
  webpack-bundle-analyzer

##### loader

###### 1.loader 是干什么的

loader 是转换器，可以要把不同的文件都转成浏览器认识的 js,css==

###### 2.loader 用法

- loader

```
rules: [
            {
                test: /\.css/,
                loader:['style-loader','css-loader']
            }
        ]
```

- use

```
rules: [
            {
                test: /\.css/,
                use:['style-loader','css-loader']
            }
        ]
```

- use+loader

```
rules: [
            {
                test: /\.css/,
                include: path.resolve(__dirname,'src'),
                use: [{
                    loader: 'style-loader',
                    options: {
                        insert:'top'
                    }
                },'css-loader']
            }
        ]
```

<!-- #### webpack 优化 -->

##### externals

打包成 umd，结构目录

```
! function (e, o) {
    if ("object" == typeof exports && "object" == typeof module) module.exports = o(require("jQuery"));
    else if ("function" == typeof define && define.amd) define(["jQuery"], o);
    else {
        var n = "object" == typeof exports ? o(require("jQuery")) : o(e.jQuery);
        for (var t in n)("object" == typeof exports ? exports : e)[t] = n[t]
    }
}(window, (function (e) {
    return (window.webpackJsonp = window.webpackJsonp || []).push([[0], [function (o, n) {
        o.exports = e
    }, function (e, o, n) {}, function (e, o, n) {
        "use strict";
        n.r(o), o.default =
            "data:image/png;base64,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"
    }, , , , function (e, o, n) {
        "use strict";
        n.r(o);
        n(0), n(1);
        var t = function (e, o) {
            return e + o
        };
        t(1, 22), n(3).defaults({
            a: 1
        }, {
            a: 3,
            b: 2
        }), t(1, 2);
        var A = document.getElementById("app"),
            a = document.createElement("div"),
            s = document.createDocumentFragment(),
            u = new Image;
        u.className = "pidan", u.src = n(2).default, a.className = "dog", s.appendChild(a), s.appendChild(
            u), A.appendChild(s)
    }], [[6, 1, 2]]])
}));
```
其中
```
if ("object" == typeof exports && "object" == typeof module) module.exports = o(require("jQuery"));
    else if ("function" == typeof define && define.amd) define(["jQuery"], o);
    else {
        var n = "object" == typeof exports ? o(require("jQuery")) : o(e.jQuery);
        for (var t in n)("object" == typeof exports ? exports : e)[t] = n[t]
}
```
主要用来区分运行时用的哪种规范
#### webpack 打包后简化后结果

```
(function (modules) {
  // webpackBootstrap
  var installedModules = {};
  // The require function
  function __webpack_require__(moduleId) {
    // Check if module is in cache
    if (installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }
    // Create a new module (and put it into the cache)
    var module = (installedModules[moduleId] = {
      i: moduleId,
      l: false,
      exports: {},
    });
    // Execute the module function
    modules[moduleId].call(
      module.exports,
      module,
      module.exports,
      __webpack_require__
    );
    // Flag the module as loaded
    module.l = true;
    // Return the exports of the module
    return module.exports;
  }
})({
  "./src/js/a.js": function (module, exports) {
    console.log("a");
  },

  "./src/js/b.js": function (module, exports) {
    console.log("b");
  },

  "./src/main.js": function (module, __webpack_exports__, __webpack_require__) {
    __webpack_require__.r(__webpack_exports__);
    var _js_a__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(
      "./src/js/a.js"
    );
    var _js_b__WEBPACK_IMPORTED_MODULE_1__ = __webpack_require__(
      "./src/js/b.js"
    );
  },
});

```
附一份自己的webpack配置
[传送门](https://github.com/pengfeir/webpack)

