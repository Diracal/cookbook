# 为MinIO Server设置Nginx代理 [![Slack](https://slack.min.io/slack?type=svg)](https://slack.min.io)

Nginx是一个开源的Web服务器和反向代理服务器。  

在本文中，我们将学习如何给MinIO Server设置Nginx代理。

## 1. 前提条件

从[这里](https://docs.min.io/docs/minio-quickstart-guide)下载并安装MinIO Server。

## 2. 安装

从[这里](http://nginx.org/en/download.html)安装Nginx。

## 3. 配置

### 代理全部请求
在文件``/etc/nginx/sites-enabled``中添加下面的内容，例如``/etc/nginx/sites-enables/minio``，同时删除同一个目录中现有的``default``文件。

```sh
server {
 listen 80;
 server_name example.com;
 # 准许header中的特殊字符
 ignore_invalid_headers off;
 # 准许任意大小的文件都可以上传
 # 把值设置为像1000m等来将文件大小限制为特定值
 client_max_body_size 0;
 # 禁用buffering
 proxy_buffering off;
 location / {
   proxy_set_header Host $http_host;
   proxy_pass http://localhost:9000;
   health_check uri=/minio/health/ready;
 }
}
```

注意:

* 用你自己的主机名替换example.com。
* 用你自己的服务名替换``http://localhost:9000``。
* 为了能够上传大文件，在``http``上下文中添加``client_max_body_size 1000m;``——简单地按你的需求调整该值。默认值是`1m`，这对大多数场景来说太低了。为了禁用对客户端请求body大小的检查，将``client_max_body_size``的大小设置为`0`。
* Nginx缓冲在默认情况下会响应。为了禁用Nginx对temp文件的MinIO响应进行缓冲，设置`proxy_buffering off;`,这将会提升客户端请求的响应能力（time-to-first-byte）。
* Nginx在默认情况下不允许特殊的字符。设置``ignore_invaild_headers off;``来允许header使用特殊的字符。

### 代理基于存储桶的请求

如果你想从同一个Nginx端口提供web应用和MinIO，那么你可以代理基于存储桶名字的MinIO请求。
当需要非root配置时，按如下方式修改location：

```sh
 # 代理运行在9000端口的MinIO服务器存储桶"photos"的请求
 location /photos/ {
   proxy_buffering off;
   proxy_set_header Host $http_host;
   proxy_pass http://localhost:9000;
 }
 # 代理运行在9001端口的应用服务器的任何请求
 location / {
   proxy_buffering off;
   proxy_set_header Host $http_host;
   proxy_pass http://localhost:9001;
 }
```

## 4. 步骤

### 第一步: 启动MinIO Server

```sh
minio server /mydatadir
```

### 第二步: 重启Nginx server

```sh
sudo service nginx restart
```

## 了解更多

参考[这里](https://www.nginx.com/blog/enterprise-grade-cloud-storage-nginx-plus-minio/)了解更多的MinIO和Nginx配置选项。
