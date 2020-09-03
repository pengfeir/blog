---
title: husky
date: 2020-09-03 15:39:49
description: lint-staged,prettier,eslint,husky,commitizen
tags: git,,prettier,husky
categories: git,,prettier,husky
---

#### 记录一波 husky 配置提交代码校验的过程

最近做项目的时候人比较多，老是因为提交代码格式等问题产生冲突，为解决类似问题，配置了提交前校验，记录下配置过程。

1. 首先安装所需要的依赖

```
yarn add husky lint-staged commitizen cz-conventional-changelog eslint prettier eslint-config-prettier eslint-plugin-prettier --dev
```

2. 然后找到 package.json,加上提交前校验

```
"config": {
    "commitizen": {
      "path": "node_modules/cz-conventional-changelog"
    }
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "src/**/*.{jsx,tsx,ts,js,vue}": [
      "prettier --write",
      "eslint --fix"
    ],
    "src/**/*.{css,md,less,scss}": [
      "prettier --write"
    ]
  },
```

3. 自定义 prettier 规则，新建.prettierrc.js

```
module.exports = {
  // 最大长度80个字符
  printWidth: 80,
  // 行末分号
  semi: true,
  // 单引号
  singleQuote: true,
  // JSX双引号
  jsxSingleQuote: false,
  // 在对象文字中打印括号之间的空格。
  bracketSpacing: true,
  // > 标签放在最后一行的末尾，而不是单独放在下一行
  jsxBracketSameLine: false,
  // 箭头圆括号
  arrowParens: 'avoid',
  // 在文件顶部插入一个特殊的 @format 标记，指定文件格式需要被格式化。
  insertPragma: false,
  // 缩进
  tabWidth: 2,
  // 使用tab还是空格
  useTabs: false,
  // 行尾换行格式
  endOfLine: 'auto',
  HTMLWhitespaceSensitivity: 'ignore',
};
```

4. 提交代码

   git add .
   git cz
   ![avatar](/images/2.png)
   选择一个 commit 类型填写 message
   按规则格式话代码，修复
   ![avatar](/images/1.png)
   git push
