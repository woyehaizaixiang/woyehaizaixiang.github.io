---
layout: post
title: 记一次vue单页项目+nodejsApi项目上线
category: 技术
tags: 技术
keywords: vue nodejs nignx
description: 记一次vue单页项目+nodejsApi项目上线
---

# vue单页

项目参数具体如下：

1.vuecli3 2.vue@2.6.11 3.vue-router mode:history 4.开启gzip

``` javascript
const path = require('path')
const CompressionPlugin = require("compression-webpack-plugin")
module.exports = {
  css: {
    loaderOptions: {
      sass: {
        // sass 全局变量自动导入
        prependData: `
          @import "@/assets/style/common.scss";
        `
      },
      less: {
        // 定制主题 -- 变量覆盖
        modifyVars: {
          'primary-color': '#FE4128', // 基础色
          'link-color': '#1DA57A',
          'border-radius-base': '6px',
          'success-color': '#3995ff', // 成功色
          'warning-color': '#f26b24', // 警告色
          'error-color': '#fe4128', // 错误色
          'font-size-base': '14px', // 主字号
          'heading-color': '#333333', // 标题色
          'text-color': '#333333', // 主文本色
          'text-color-secondary': '#333333', // 次文本色
          'disabled-color': '#666666', // 失效色
          'border-radius-base': '4px', // 组件/浮层圆角
          'border-color-base': '#FE4128', // 边框色
          'box-shadow-base': 'none', // 浮层阴影
        },
        javascriptEnabled: true,
      },
    },
  },
  devServer: {
    port: 3033,
    proxy: {
      '/api': {
        target: 'http://192.168.0.107:3000/api',  // 后台接口域名 http://192.168.0.107:3000 本地 http://139.129.240.51:3000 蚁淘研发
        // ws: true,        //如果要代理 websockets，配置这个参数
        // secure: false,  // 如果是https接口，需要配置这个参数
        changeOrigin: true,  //是否跨域
        pathRewrite: {
          '^/api': ''
        }
      }
    }
  },
  configureWebpack: {
    plugins: [
      new CompressionPlugin({ // 开启gzip压缩
        test: /\.js$|\.html$|.\css/, //匹配文件名
        threshold: 10240,//对超过10k的数据压缩
        deleteOriginalAssets: false //不删除源文件
      })
    ]
  }
};
```
