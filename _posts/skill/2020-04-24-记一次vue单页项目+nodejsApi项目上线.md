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
vuecli3 vue@2.6.11 vue-router mode:history 开启gzip

vue.config.js
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
        javascriptEnabled: true,
      },
    },
  },
  devServer: {
    port: 3033,
    proxy: {
      '/api': {
        target: 'http://192.168.0.107:3000/api',
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
将项目打包后dist目录下文件放在nginx的html根目录下

# nodejs
nodejs项目直接部署在ngnix上，使用pm2守护进程(后台开启,防止nodejs掉线)
nodejs的api运行在3000端口，稍后会在nginx上配置 将项目中请求'/api'代理到3000端口上
1.安装pm2 ```npm/cnpm install pm2 -g```
2.cd 到nodejs项目目录 ```pm2 start bin/www``` 参考https://www.jianshu.com/p/ee935729f49c

# ngnix
nginx配置:
1.tryfiles(配合单页项目history路由模式)
2.开启gzip
3.代理/api到nodejs项目(3000端口)

nginx.conf
```
user root;

worker_processes 8;

events {
	worker_connections 5000;
}


http {
	log_format main '$remote_addr - $remote_user [$time_local] "$request" "$http_cookie" $http_host '
	'$status $body_bytes_sent "$http_referer" '
	'"$http_user_agent" "$http_x_forwarded_for" ';
	include mime.types;
	default_type application/octet-stream;
	sendfile on;
	keepalive_timeout 65;
	# nginx优化
	client_header_timeout 15;
	client_body_timeout 15;
	# 开启gzip
	gzip on;
	# 启用gzip压缩的最小文件，小于设置值的文件将不会压缩
	gzip_min_length 1k;
	# gzip 压缩级别，1-10，数字越大压缩的越好，也越占用CPU时间，后面会有详细说明
	gzip_comp_level 2;
	# 匹配压缩类型
	gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
	# 是否在http header中添加Vary: Accept-Encoding，建议开启
	gzip_vary on;
	# 禁用IE 6 gzip
	gzip_disable "MSIE [1-6]\.";
	#上传文件大小
	client_max_body_size 30M;
	access_log off;


	server {
		server_name dev_bus.11oi.com;
		listen 80;
		charset utf-8;
		root /usr/local/nginx/html/;
                index index.html;

          location /api/ {
			 proxy_pass http://localhost:3000;
		}
		location / {
			#index index.html;
			try_files $uri $uri/ /index.html;
			error_page 404 /index.html;
		}

	}

}
```
配置完成。重启nginx。

到此，一个完整的项目的部署完毕。总结一下：
1.前端工程上线：
将前端项目放在nginx的html根目录下
2.api接口上线
将node项目放在服务器任意位置，并安装pm2，使用pm2运行node项目
3.配置nginx.conf
①开启gzip ②ngnix try_files 重定向 ③反向代理/api