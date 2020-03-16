---
layout: wiki
title: Php Laravel 笔记
description: Laravel 可以让你从面条一样杂乱的代码中解脱出来；它可以帮你构建一个完美的网络APP，而且每行代码都可以简洁、富于表达力。
date: 2020-02-27
categories: Php
prism: [php, bash, yaml, markup]
---

* TOC
{:toc}

## Blade 模板相关

### php 变量输出为 html 代码

```php
{!! $content !!}
```

### 获取当前路由路径

```php
{% raw %}{{ Request::path() }}
{{ Route::currentRouteName() }}{% endraw %}
```

### 获取当前路由参数

```php
{% raw %}{{ app('request')->input('param') }}
{{ Request::query('param') }}         // laravel 5.6
{{ request()->param }}                // laravel 5.8{% endraw %}
```

### 分页传入路由参数

```php
{% raw %}{{ $data->appends(['param' => request()->param])->links('path_to_pagination_view') }}{% endraw %}
```

### 将视图转化为字符串

```php
$view = view('emails.index')->with(['lang' => $lang, 'name' => $name]);
$viewStr = response($view)->getContent();
```

## Eloquent 模型相关

### ORM 查询指定列记录

* 按 id 取单条记录

    ```php
    $data = Model::find($id, ['column1', 'column2']);
    ```

* 取集合首条记录

    ```php
    $data = Model::first(['column1', 'column2']);
    ```

* 取集合全部记录

    ```php
    $data = Model::all(['column1', 'column2']);
    ```

* 取集合多条记录

    ```php
    $data = ModelA::where(...)->get(['column1', 'column2']); 
    ```

## Redis 相关

### 使用 Redis 发布与订阅消息队列

1. 安装 predis 客户端

    ```php
    composer require predis/predis
    ```

2. 发布者 Command

    ```php
    try {
        Redis::publish($QUEUE_NAME, $message);
    } catch (Exception $e) {
        this->info('$e');
    }
    ```

3. 订阅者 Command

    ```php
    Redis::subscribe([$QUEUE_NAME], function($message) {
        this->info($message);
    });
```

### 使用 Redis 异步任务队列

1. 安装 predis 客户端

    ```bash
    composer require predis/predis
    ```

2. 修改环境变量 QUEUE_DRIVER

    ```yaml
    QUEUE_CONNECTION=redis
    ```

3. 创建失败任务迁移文件

    ```bash
    php artisan queue:failed-table
    ```

4. 生成失败任务表

    ```bash
    php artisan migrate
    ```

5. 生成任务类

    ```bash
    php artisan make:job SendEmailJob
    ```

6. 编写任务执行逻辑

    ```php
    namespace App\Jobs;

    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;

    class SendEmailJob implements ShouldQueue {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        protected $injectionObject;

        public function __construct(Object $object) {
            $this->injectionObject = $object;
        }

        public function handle() {
            // TODO: 任务执行方法体
        }
    }
    ```

7. 发送任务到任务队列

    ```php
    public function work() {
        $job = new SendEmailJob($obj);
        dispatch(job);                  // 分发任务
    }
    ```

8. 监听队列

    ```bash
    php artisan queue:work --daemon --quiet --queue=default --delay=3 --sleep=3 --tries=3
    ```

9. 仪表盘监控队列（仅 Linux）

    ```bash
    composer require "laravel/horizon:~1.3"
    php artisan vendor:publish --provider="Laravel\Horizon\HorizonServiceProvider"
    php artisan horizon
    ```

    访问 `http://localhost/horizon` 即可访问仪表盘


### 使用 Redis 缓存

1. 修改配置文件

```php
// config/cache.php
'redis' => [
    'driver'     => 'redis',
    'connection' => 'default',
],
```

```php
// config/database.php
'redis' => [
    'client' => 'predis',

    'default' => [
        'host'     => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD', null),
        'port'     => env('REDIS_PORT', 6379),
        'database' => 0,
    ],
],
```

2. 使用 Redis 缓存

```php
if (Cache::has($key)) {
    $value = Cache::get($key);
    // use $value
} else {
    Cache::put($key, $value, $expire_time);
}
```

## HTTP 相关

### HTTP 客户端

* php-curl-class

    ```php
    use \Curl\Curl;

    $curl = new Curl();
    $curl->get('http://www.example.com/');

    if ($curl->error) {
        echo 'Error: ' . $curl->errorCode . ': ' . $curl->errorMessage;
    } else {
        echo $curl->response;
    }
    ```

* GuzzleHttp\Client

    ```php
    use GuzzleHttp\Client;

    $http = new Client();
    $url = 'http://www.baidu.com';
    $response = $http->get($url);
    $data = json_decode((string)$response->getBody(), true);
    ```

## MVC 相关

### 返回默认的 error 视图

```php
view()->replaceNamespace('errors', [
    resource_path('views/errors'),
    __DIR__.'/views',
]);
return response()->view("errors::{$status}";
```

### 返回 Json 数据

```php
return response()->json(['code' => '1', 'msg' => 'Subscribe successfully']);
```

## Composer 相关

### Composer 换源

```bash
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

## 部署相关

### Nginx

1. 将网站目录指向 public 文件夹

    `root   "D:/WORKSPACE/PhpStorm/up_server_admin/public";`

2. 设置网站入口为 index.php

    `index index.php index.html;`

3. 设置所有资源入口为 index.php

    `try_files $uri $uri/ /index.php?$query_string;`