---
title: Nginx学习记录(一)
date: 2018-10-30 23:10:00
comments: true
categories: [Nginx]
tags: [Linux,Nginx]
---

本文为慕课网[Nginx 入门到实践－Nginx 中间件](https://coding.imooc.com/class/121.html) 课程学习笔记，主要关于 Nginx 的安装配置和常用模块介绍

<!-- more -->

系统要求：
CPU >= 2 Core
内存 >= 256 MB
操作系统 >= CentOS 7.0 及以上版本，位数 X64

## 准备工作

安装基础依赖：

```
yum -y install gcc gcc-c++ autoconf pcre pcre-devel make automake
yum -y install wget httpd-tools
```

关闭 iptables 规则：
```
# 查看 iptables 规则
iptables -L
# 关闭 iptables 规则
iptables -F
```

关闭 SELinux：

```
# 查看 SELinux 状态
getenforce
# 设置 SELinux 状态为 Disabled，参考https://blog.csdn.net/tangsilian/article/details/80144112
https://www.cnblogs.com/tdcqma/p/5671299.html
```

创建一些工作目录：

```
cd /opt
mkdir app download logs work backup
```

## Nginx 的中间件架构

Nginx 是一个开源且高性能、可靠的 HTTP 中间件、代理服务。阿里的 Tengine 就是基于 Nginx 作二次开发

Nginx 特点：

- IO 多路复用
- 轻量级
- CPU 亲和
- sendfile

### IO 多路复用 epoll

多个描述符的 IO 操作都能在一个线程内并发交替的顺序完成，这就叫 IO 多路复用，这里的“复用”指的是复用同一个线程

#### select 模型

1. 能够监视文件描述符（FD）的数量存在最大限制
2. 线性扫描效率低下

#### epoll 模型

2.6 内核之后，出现 epoll 模型

1. 每当 FD 就绪，采用系统的回调函数直接将 FD 放入，效率更高
2. 最大连接无限制

### 轻量级

- 功能模块少
- 代码模块化

### CPU 亲和

通过把 CPU 核心和 Nginx 工作进程进行绑定的方式，把每个 worker 进程固定在一个 CPU 上执行，减少切换 CPU 的 cache miss，获得更好的性能

### sendfile

关于 Linux 森地了，参考：[sendfile 对 Nginx 性能的提升](http://blog.51cto.com/laoxu/1417294)

## 安装 Nginx

安装具体过程参考：[http://nginx.org/en/linux_packages.html#stable](http://nginx.org/en/linux_packages.html#stable)

配置 yum 源：

新建文件 `/etc/yum.repos.d/nginx.repo`，编辑内容如下：

```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/OS/OSRELEASE/$basearch/
gpgcheck=0
enabled=1
```

根据所使用系统的发布版本将 `OS` 替换成 `rhel` 或 `centos`，根据是 6.X 还是 7.X 系统将 `OSRELEASE` 替换成 `6` 或 `7`

安装 Nginx：

```
yum install nginx
```

查看 nginx 版本，此处安装的为 nginx 1.14.0 稳定版：

```
[root@localhost yum.repos.d]# nginx -v
nginx version: nginx/1.14.0
```

查看 Nginx 相关配置参数：

```
[root@localhost yum.repos.d]# nginx -V
nginx version: nginx/1.14.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_dav_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_ssl_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-mail --with-mail_ssl_module --with-stream --with-stream_realip_module --with-stream_ssl_module --with-stream_ssl_preread_module --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' --with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'
```

## 基本参数使用

### Nginx 启动、停止

- ./nginx 启动 Nginx
- nginx -s stop 强制停止
- nginx -s quit 优雅停止
- nginx -s reload 重新加载配置文件
- nginx -s reopen 重新打开日志文件

### 安装目录

使用 `rpm -ql nginx` 查看 nginx 的相关安装目录

Path | Type | Function
-|
`/etc/logrotate.d/nginx` | 配置文件 | Nginx 日志轮转，用于 logrotate 服务的日志切割
`/etc/nginx/conf.d` `/etc/nginx` `/etc/nginx/conf.d/default.conf` `/etc/nginx/nginx.conf`| 目录、配置文件 | Nginx 主配置文件
`/etc/nginx/fastcgi_params` `/etc/nginx/scgi_params` `/etc/nginx/uwsgi_params` | 配置文件 | cgi 相关，fastcgi 相关
`/etc/nginx/win-utf` `/etc/nginx/koi-utf` `/etc/nginx/koi-win` | 配置文件 | 编码转换映射转化文件
`/etc/nginx/mime.types` | 配置文件 | 设置 http 协议的 Content-Type 与扩展名的对应关系
`/usr/lib64/nginx` `/usr/lib64/nginx/modules` | 目录 | Nginx 模块目录
`/usr/sbin/nginx` `/usr/sbin/nginx-debug` | 命令 | Nginx 服务的启动管理的终端命令
`/var/cache/nginx` | 目录 | Nginx 的缓存目录（Nginx 不仅可以做代理，还可以做缓存）
`/var/log/nginx` | 目录 | Nginx 的日志目录

### Nginx 基本配置语法

打开 Nginx 的主配置文件：`/etc/nginx/nginx.conf`，内容如下：

```
# 设置 Nginx 服务的系统使用用户
user  nginx;
# 工作进程数
worker_processes  1;  

# Nginx 的错误日志
error_log  /var/log/nginx/error.log warn;
# Nginx 服务启动时候的 PID
pid        /var/run/nginx.pid;


events {
	# 每个进程允许的最大连接数，最大可设置到 65535
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on; 
    #tcp_nopush     on; 

	# 超时时间
    keepalive_timeout  65; 

    #gzip  on; 

	# 包含进其他配置文件
    include /etc/nginx/conf.d/*.conf;
}
```

因为 `/etc/nginx/conf.d` 目录下有一个 `default.conf` 文件，所以该文件中所做的配置也会被加载，该文件内容如下：

```
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

## Nginx 日志

通过 Nginx 的配置文件了解到，Nginx 的日志类型分为如下两种：

- error.log：用于记录服务器的错误日志
- access.log：用于记录 Nginx 接受的每次 HTTP 请求的各种信息

### access.log

日志的格式需要在 Nginx 的配置文件中的 `log_format` 模块下配置，可以使用 Nginx 的内置变量：

```
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
```

`curl -v http://127.0.0.1` 返回结果如下：

```
[root@localhost sbin]# curl -v http://127.0.0.1
* Rebuilt URL to: http://127.0.0.1/
*   Trying 127.0.0.1...
* Connected to 127.0.0.1 (127.0.0.1) port 80 (#0)
> GET / HTTP/1.1
> Host: 127.0.0.1
> User-Agent: curl/7.47.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.14.0
< Date: Mon, 03 Sep 2018 11:59:38 GMT
< Content-Type: text/html
< Content-Length: 612
< Last-Modified: Tue, 17 Apr 2018 15:48:00 GMT
< Connection: keep-alive
< ETag: "5ad61730-264"
< Accept-Ranges: bytes
< 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
* Connection #0 to host 127.0.0.1 left intact
```

此时 `tail -f /var/log/nginx/access.log` 打印内容：

```
127.0.0.1 - - [03/Sep/2018:07:25:26 -0400] "GET / HTTP/1.1" 200 612 "-" "curl/7.47.0" "-"
```

为 log_format 增加 `$http_user_agent` 内置变量：

```
log_format  main  '$http_user_agent - $remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
```

然后重新加载配置文件：

```
[root@localhost sbin]# ./nginx -tc /etc/nginx/nginx.conf -s reload
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

用到的选项含义：

- -t 检测配置文件是否有语法错误
- -c 指定配置文件位置，若不指定，则默认使用 `/etc/nginx/nginx.conf` 文件
- -s reload 让 Nginx 重新加载配置文件

再次使用 `curl` 命令访问 Nginx：

```
`curl -v http://127.0.0.1`
```

access.log 中的内容：

```
curl/7.47.0 - 127.0.0.1 - - [03/Sep/2018:07:28:11 -0400] "GET / HTTP/1.1" 200 612 "-" "curl/7.47.0" "-"
```

其中 `curl/7.47.0` 正是我们刚刚所添加 `$http_user_agent` 内置变量所表示的内容，更多 Nginx 内置变量参考 [ngx_http_core_module](http://nginx.org/en/docs/http/ngx_http_core_module.html)

## Nginx 模块

Nginx 模块分为官方模块和第三方模块

### 官方模块

[Nginx 官方模块索引](http://nginx.org/en/docs/)

#### [ngx_http_stub_status_module 模块](http://nginx.org/en/docs/http/ngx_http_stub_status_module.html)

该模块用于查看 Nginx 状态

在 `/etc/nginx/conf.d/default.conf` 文件中增加如下配置：

```
location /mystatus {
    stub_status;
}
```

重启 Nginx 后，浏览器访问如下图：

![mark](http://imgblog.kuranado.com/blog/180903/CgIc2I457j.png)

- Active connections：Nginx 正在处理的活动连接数为 2 个
- Server accepts handled requests：Nginx 从启动到现在共处理了 2 个连接，共成功创建了 2 次握手，共处理了 1 次请求
- Reading：Nginx 读取到客户端的 Header 信息数为 0 
- Writing：Nginx 返回给客户端的 Header 信息数为 1
- Waiting：开启 keep-alive 的情况下,这个值等于 active – (reading + writing),意思就是 Nginx 已经处理完成,正在等候下一次请求指令的驻留连接，此处为 1

#### [ngx_http_random_index_module 模块](http://nginx.org/en/docs/http/ngx_http_random_index_module.html)

模块处理以斜杠字符（' /'）结尾的请求，并选取目录中的随机文件作为索引文件

配置语法：

```
Syntax:	random_index on | off;
Default: random_index off;
Context: location
```

修改 `/etc/nginx/conf.d/default.conf` 如下：

```
location / {
    root   /usr/share/nginx/html;
    #index  index.html index.htm;
    random_index on;
}
```

重新加载配置文件后，每次刷新浏览器 Nginx 都会随机访问 `/usr/share/nginx/html` 目录下的文件

#### [ngx_http_sub_module](http://nginx.org/en/docs/http/ngx_http_sub_module.html)

该模块用于替换指定字符串（下面的示例如果想要测试成功，记得每次清理浏览器缓存）
修改 `/etc/nginx/conf.d/default.conf` 如下：

```
location / {
        root   /opt/app/;
        index  test_sub_module.html;
        sub_filter 'kuranado' 'KURANADO';
    }
```

`/opt/app/` 目录下新增 test_sub_module.html 文件内容如下：

```
<html>
<head>
<title>测试ngx_http_sub_module模块</title>
</head>
<body>
Hello, I'm kuranado, nice to meet you!
<br/>
Hello, kuranado. I'm Jack, nice to meet you too!
</body>
</html>
```

浏览器访问如下：

![mark](http://imgblog.kuranado.com/blog/181021/JH2ElGCHCm.png)

发现虽然替换了指定的字符串，但却只替换了第一次出现的字符串，如果想要替换所有，配置改为如下即可：

```
location / {
        root   /opt/app/;
        index  test_sub_module.html;
        sub_filter 'kuranado' 'KURANADO';
        sub_filter_once off;
    }
```

#### 请求限制 [ngx_http_limit_req_module](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html)

default.conf 做如下配置：

```
limit_conn_zone $binary_remote_addr zone=conn_zone:1m;
limit_req_zone $binary_remote_addr zone=req_zone:1m rate=1r/s;
server {
    listen       80; 
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / { 
        root   /opt/app/;
        index  test_sub_module.html;
        #sub_filter 'kuranado' 'KURANADO';
        #sub_filter_once off;
        limit_req zone=req_zone;
    }   

    location /mystatus {
        stub_status;
    }   

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #   
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }   
}

```

上面的配置表示请求限制被设置为 1 秒一次，在同一秒内只能有一个请求会成功，其他请求都会失败。重启 Nginx 后，使用 ab 命令做测试，发起 50 次请求，并发数为 20，结果表明 1 秒内完成了 50 次请求，只有一个成功，其余请求全部失败，也可以查看 `/var/log/nginx/error.log` 日志：

```
[root@localhost sbin]# ab -n 50 -c 20 -k http://127.0.0.1/
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 127.0.0.1 (be patient).....done


Server Software:        nginx/1.14.0
Server Hostname:        127.0.0.1
Server Port:            80

Document Path:          /
Document Length:        186 bytes

Concurrency Level:      20
Time taken for tests:   0.032 seconds
Complete requests:      50
Failed requests:        49
   (Connect: 0, Receive: 0, Length: 49, Exceptions: 0)
Write errors:           0
Non-2xx responses:      49
Keep-Alive requests:    50
Total transferred:      36487 bytes
HTML transferred:       26499 bytes
Requests per second:    1575.45 [#/sec] (mean)
Time per request:       12.695 [ms] (mean)
Time per request:       0.635 [ms] (mean, across all concurrent requests)
Transfer rate:          1122.72 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   1.0      0       3
Processing:     2    9   8.5      3      21
Waiting:        2    9   8.5      3      21
Total:          2   10   9.3      3      22
WARNING: The median and mean for the initial connection time are not within a normal deviation
        These results are probably not that reliable.

Percentage of the requests served within a certain time (ms)
  50%      3
  66%     21
  75%     21
  80%     21
  90%     22
  95%     22
  98%     22
  99%     22
 100%     22 (longest request)
```

把上面配置中的 `limit_req zone=req_zone;` 改为 `limit_req zone=req_zone burst=3 nodelay;`，表示在超过请求限制后，后面的 3 个请求将被放到下一秒执行且没有限制，再剩下的所有请求还是 1 秒内只能处理一个，`ab -n 50 -c 20 -k http://127.0.0.1/` 测试，查看 `/var/log/nginx/access.log` 如下：

```
"GET / HTTP/1.0" 200 186 "-" "ApacheBench/2.3" "-"
ApacheBench/2.3 - 127.0.0.1 - - [30/Oct/2018:20:09:39 +0800] "GET / HTTP/1.0" 200 186 "-" "ApacheBench/2.3" "-"
ApacheBench/2.3 - 127.0.0.1 - - [30/Oct/2018:20:09:39 +0800] "GET / HTTP/1.0" 200 186 "-" "ApacheBench/2.3" "-"
ApacheBench/2.3 - 127.0.0.1 - - [30/Oct/2018:20:09:39 +0800] "GET / HTTP/1.0" 200 186 "-" "ApacheBench/2.3" "-"
ApacheBench/2.3 - 127.0.0.1 - - [30/Oct/2018:20:09:39 +0800] "GET / HTTP/1.0" 503 537 "-" "ApacheBench/2.3" "-"
ApacheBench/2.3 - 127.0.0.1 - - [30/Oct/2018:20:09:39 +0800] "GET / HTTP/1.0" 503 537 "-" "ApacheBench/2.3" "-"
...
```

#### 连接限制 [ngx_http_limit_conn_module](https://nginx.org/en/docs/http/ngx_http_limit_conn_module.html) 

把上面配置中的 `limit_req zone=req_zone;` 改为 `limit_conn conn_zone 1;` 表示每秒只允许一个连接，重启 Nginx 后， 执行 `ab -n 50 -c 20 -k http://127.0.0.1/` 并查看日志，发现成功率与设置请求限制相比会高很多，这是因为在一次 HTTP 连接中可以进行多次请求导致的

#### Nginx 访问限制 

##### 基于 IP 的访问控制 [ngx_http_access_module](https://nginx.org/en/docs/http/ngx_http_access_module.html)

##### 基于用户的信任登录 [ngx_http_auth_basic_module](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html)

### 第三方模块

## 参考资料

[Nginx 官方文档](http://nginx.org/en/docs)
[Nginx 开启 stub_status 模块监控](http://hi.baidu.com/oncard/blog/item/f4422f0103f7e2137aec2cf5.html)