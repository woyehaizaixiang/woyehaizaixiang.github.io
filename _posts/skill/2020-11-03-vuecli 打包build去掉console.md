---
layout: post
title: vuecli 打包build去掉console
category: 技术
tags: 技术
keywords: vue webpack babel
description: vuecli 打包build去掉console
---

# 生产环境去掉console

1.安装依赖包
```
npm install babel-plugin-transform-remove-console --save-dev
```
2.配置babel.config.js
```
let transformRemoveConsolePlugin = []
if (process.env.NODE_ENV === 'production') {
  transformRemoveConsolePlugin = ['transform-remove-console']
}
module.exports = {
  plugins: [
    ...transformRemoveConsolePlugin,
  ],
}
```