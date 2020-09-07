---
title: postcss
date: 2020-09-07 11:31:52
description: postcss,移动端适配方案
tags: vw移动端适配方案
categories: vw移动端适配方案
---

#### vw 移动端适配方案

最近做了几个 h5 项目，适配采用 postcss-px-to-viewport，效果还可以。

1. 首先安装依赖

```
yarn add postcss-px-to-viewport --dev
```

postcss-px-to-viewport 是基于 postcss 的一个插件，将 px 转 viewport 的。
2. 然后创建postcssrc.config.js,将自己设计稿的宽度等一系列配置写进去。
拿vant举例子，vant的设计稿是375的需要特殊处理。

```
module.exports = ({ file }) => {
  const designWidth = file.dirname.includes('node_modules/vant') ? 375 : 750;
  return {
    plugins: {
      "postcss-px-to-viewport": {
        viewportWidth: designWidth, // (Number) The width of the viewport.
        unitPrecision: 3, // (Number) The decimal numbers to allow the REM units to grow to.
        viewportUnit: "vw", // (String) Expected units.
        selectorBlackList: [".ignore", ".hairlines"], // (Array) The selectors to ignore and leave as px.
        minPixelValue: 1, // (Number) Set the minimum pixel value to replace.
        mediaQuery: false, // (Boolean) Allow px to be converted in media queries.
      },
    },
  }
};

```
其中，有两个疑惑点。一是postcssrc.config.js，eslint.config.js这些类似插件的配置是怎么做的，为什么写个[插件名+config.js]或者.插件名.js就可以当作配置，最后在postcss-load-config里找到了答案-(cosmiconfig)[https://github.com/davidtheclark/cosmiconfig#readme],二就是module.exports可以接收函数了-(postcss-load-config)[https://github.com/michael-ciniawsky/postcss-load-config],
