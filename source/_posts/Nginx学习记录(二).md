---
title: Nginx学习记录(二)
date: 2018-10-30 23:10:00
comments: true
categories: [Nginx]
tags: [Linux,Nginx]
---

本文为慕课网 [Nginx 入门到实践－Nginx 中间件](https://coding.imooc.com/class/121.html) 课程学习笔记，主要关于 Nginx 的实际应用场景

<!-- more -->

## 静态资源服务场景-CDN

### gzip 压缩

- sendfile：
- tcp_nopush：在 sendfile 开启的情况下，提高网络包的传输速率
- tcp_nodelay：在 keepalive 连接下，提高网络包的传输实时性

#### [ngx_http_gzip_module 模块](http://nginx.org/en/docs/http/ngx_http_gzip_module.html)
 
- gzip：压缩传输
- gzip_comp_level：压缩比，数值越大，压缩比越大，但同时也需要消耗更多的时间和 CPU
- gzip_http_version：gzip http 版本，主流使用 1.1 版本

#### [ngx_http_gzip_static_module 模块](http://nginx.org/en/docs/http/ngx_http_gzip_static_module.html)

- http_gzip_static_module：gzip 静态压缩。普通的 gzip 压缩在每次接收到客户端请求后进行，这样每次请求都需要重新压缩，费时费力，所以有了 http_gzip_static_module 模块可以预先压缩好文件，提高效率，但同时也需要更多的磁盘空间存储压缩后的文件
- http_gunzip_module：应用支持 gunzip 的压缩方式（很少使用）

![mark](http://imgblog.kuranado.com/blog/181105/jbHCJa9889.png)

在服务端进行压缩，浏览器端进行解压，减少服务器带宽消耗

具体举例，在 conf.d 目录下创建 static_access.conf 文件，该文件配置如下：

```
server {
    listen 80;
    server_name 172.16.20.8;

    sendfile on;
    access_log /var/log/nginx/log/static_access.log;

    location ~ .*\.(jpg|gif|png)$ {
        #gzip on;
        #gzip_http_version 1.1;
        #gzip_comp_level 2;
        #gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
        root /opt/app/code/images;
    }

    location ~ .*\.(txt|html)$ {
        #gzip on;
        #gzip_http_version 1.1;
        #gzip_comp_level 2;
        #gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
        root /opt/app/code/doc;
    }

    location ~ ^/download {
        #gzip_static on;
        #tcp_nopush on;
        root /opt/app/code;
    }
}
```

上面的配置所有的 gzip 压缩功能都没有启用，浏览器访问效果如下：

![](http://imgblog.kuranado.com/1544351170.png)

看到一个文本文件的原始大小为 17.4 KB

下面启用 gzip 功能，修改部分配置如下：

```
location ~ .*\.(txt|html)$ {
        gzip on;
        gzip_http_version 1.1;
        gzip_comp_level 2;
        gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
        root /opt/app/code/doc;
}
```

然后重新加载配置文件，浏览器访问效果如下：

![](http://imgblog.kuranado.com/1544351188.png)

此时文件的大小已变成 181 B，说明 gzip 的确对文件进行压缩了

另外为了防止浏览器缓存干扰测试，最好勾选上 `Disable Cache`

![](http://imgblog.kuranado.com/1544444945.png)

### 跨域访问问题

浏览器出于安全的考虑（如 CRSF 共计），默认会禁止跨域访问。但是当服务器返回给浏览器的 response 头中包含 `Access-Control-Allow-Origin`，则表示服务器允许浏览器进行跨域访问。通过 Nginx 的 `add_header` 可以实现这种需求。

### 防盗链问题

基本的防盗链问题可以通过 Nginx 的 [ngx_http_referer_module](http://nginx.org/en/docs/http/ngx_http_referer_module.html) 模块实现。

nginx 做如下配置：

```
location ~ .*\.(jpg|gif|png)$ {
        #gzip on;
        #gzip_http_version 1.1;
        #gzip_comp_level 2;
        #gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
        valid_referers none blocked 172.16.20.8;
        if ($invalid_referer) {
            return 403; # 403 用来表示服务器理解请求，但拒绝处理请求
        }
        root /opt/app/code/images;
    }
```

其中 `valid_referers` 表示允许哪些来源可以访问，当一个来自没有被 `valid_referers` 所列出来的请求访问 Nginx 时，`$invalid_referer` 变量的值会变成 1，通过判断该变量的值是否非 0，从而做到拦截的效果。

- none: 没有带 refer 信息
- blocked：没有带协议信息，如没有带 `http://`
- server_name：只允许特定 server 访问，如可以指定 IP，也可以设置为 *，表示允许所有 server

Nginx `log_format` 配置如下：

```
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                       '$status $body_bytes_sent "$http_referer" '
                       '"$http_user_agent" "$http_x_forwarded_for"';
```

变量 `$http-referer` 表示请求来自哪里，`$status` 表示返回状态码

重启 Nginx 使用 `curl` 命令做 3 个小测试：

#### 测试 1

```
curl -I http://172.16.20.8/wechat.png
```

查看 Nginx 访问日志打印如下：

```
172.16.20.8 - - [12/Dec/2018:20:58:12 +0800] "HEAD /wechat.png HTTP/1.1" 200 0 "-" "curl/7.54.0"
```

`-` 表示不是从其他来源过来的请求

#### 测试 2

```
# -e 设置 refer
curl -e 172.16.20.8 -I http://172.16.20.8/wechat.png
```

Nginx 访问日志打印如下：

```
172.16.20.8 - - [12/Dec/2018:20:58:30 +0800] "HEAD /wechat.png HTTP/1.1" 403 0 "http://www.baidu.com" "curl/7.54.0"
```

可以看到对于 refer 为 http://www.baidu.com 的请求，Nginx 返回了 403

#### 测试 3

```
curl -e 172.16.20.8 -I http://172.16.20.8/wechat.png
```

Nginx 访问日志打印如下：

```
172.16.20.8 - - [12/Dec/2018:21:00:14 +0800] "HEAD /wechat.png HTTP/1.1" 200 0 "172.16.20.8" "curl/7.54.0"
```

对于 refer 为 `172.16.20.8` 的请求，Nginx 返回了 200，而该 IP 正式在 `valid_referers` 中指定的 server_name，Nginx 不会对其进行拦截

从上面的小例子中总结 Nginx 是通过判断 refer 实现防盗链。但实际上也有很多网站或黑客会通过伪装 HTTP Referer 头信息来绕过 Nginx 的这种防盗链，所以 Nginx 的这种防盗链技术并不是绝对可靠的。

### 代理服务

Nginx 既可以做正向代理，又可以做反向代理。简单了解一下什么是正反向代理：

- 正向代理：正向代理需要使用指定的中间服务器代替客户端访问目标服务器，称之为正向代理，客户端需要配置中间代理服务器的 IP 或域名。例如通过 SS 访问 Google
- 反向代理：反向代理不需要客户端做任何设置，直接访问目标 IP 或域名，但中间会存在代理服务器判断将请求发往何处，客户端并不知道最终访问的是哪台目标服务器。Nginx 被用来作为反向代理服务器。

#### Nginx 作为反向代理

conf.d 目录下新建如下两个文件：

realserver.conf，开放了 8080 端口:

```
server {
    listen 8080;
    server_name 172.16.20.8;

    charset utf-8;
    access_log /var/log/nginx/server_access.log;

    location / {
        root /opt/app/code;
        index index.html index.htm;
    }
}
```

ft_proxy.conf，开放了 80 端口:

```
server {
    listen 80;
    server_name 172.16.20.8;

    charset utf-8;
    access_log /var/log/nginx/proxy_access.log;

    location ~ /test_proxy.html$ {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

/opt/app/code 目录下新建 test_proxy.html 文件:

```
<html>
<head>
<title>测试反向代理</title>
<script src="http://libs.baidu.com/jquery/2.1.4/jquery.min.js"></script>
</head>
<body>
    <h1>测试 Nginx 反向代理</h1>
</body>
</html>
```

重启 Nginx 后浏览器访问 `http://172.16.20.8/test_proxy.html` 如下：

![](http://imgblog.kuranado.com/1544791391.png)

虽然访问的是 80 端口，但是最终请求被 Nginx 反向代理到 8080 端口了，而客户端并不需要知道这些。事实上，类似的场景在实际公司服务部署中也是经常用到的，例如我们编写的 Java 服务需要 18088 端口，但是服务器却只向外开放了特定的几个端口（如 80 等），没有开放 18088 端口，这个时候就可以通过 Nginx 作为反向代理，从而让客户端能够成功访问到 18088 端口上的服务。

### 负载均衡

conf.d 目录下新建 server1.conf、server2.conf 和 server3.conf 分别监听 8001、8002 和 8003 端口，如 server1.conf 配置如下：

```
server {
    listen 8001;
    server_name 172.16.20.8;

    charset utf-8;
    access_log /var/log/nginx/server1_access.log main;

    location / {
        root /opt/app/code1;
        index index.html index.htm;
    }
}
```

同时在 /opt/app 目录下新建 code1、code2、code3 目录，每个目录下都有一个 index.html 文件，如 /opt/app/code1/index.html 内容如下：

```
<html>
<head>
<title>Server1</title>
</head>
<body>
    <h1>Server1</h1>
</body>
</html>
```

最后在 conf.d 目录下新建 test_upstream.conf 文件，配置如下：

```
upstream kuranado {
    server 172.16.20.8:8001;
    server 172.16.20.8:8002;
    server 172.16.20.8:8003;
}

server {
    listen 80;
    server_name 172.16.20.8;
    charset utf-8;

    location / {
        proxy_pass http://kuranado;
        include proxy_params;
    }
}
```

上面的配置表示，当请求 172.16.20.8 机器的 80 端口时，Nginx 会将请求代理到一个 upstream 所指定的服务列表中，均衡的让该列表中的服务提供最终服务，也就是所谓的负载均衡。重启 Nginx 后，浏览器访问 http://172.16.20.8，不断刷新页面，每次都会由不同的服务提供服务。另外如果此时服务列表中的任何一个服务挂掉了（如利用 iptables 规则，将所有访问 8002 端口的请求都抛弃掉：`iptables -I INPUT -p tcp --dport 8002 -j DROP`），Nginx 会自动检测到，只将请求发到剩下的有效的服务中

常用的 `upstream` 配置例子：

```
upstream kuranado {
   	server image.kuranado.com weight=3;
   	server blog.kuranado.com:8080;
   	server unix:/tmp/kuranado;
   	
   	server image.kuranado.com backup;
   	server blog.kuranado.com:8080 backup;
}
```

每个服务在负载均衡的调度中可以有如下几种状态：

状态|说明
-|
down|当前的 server 暂时不参与负载均衡，即不向外提供服务
backup|预留的备份服务，主服务停掉之后，备份服务才会向外提供服务
max_fails|允许请求失败的次数
fail_timeout|经过 max_fails 失败后，服务暂停的时间
max_conns|限制最大接收的连接数

修改 test_upstream.conf 如下：

```
upstream kuranado {
    server 172.16.20.8:8001 down;
    server 172.16.20.8:8002 backup;
    server 172.16.20.8:8003 max_fails=1 fail_timeout=10s;
}

server {
    listen 80;
    server_name 172.16.20.8;
    charset utf-8;

    location / {
        proxy_pass http://kuranado;
        include proxy_params;
    }
}
```

上面的配置表示 8001 端口上的服务不对外提供服务；8002 端口上的服务为备份服务；8003 端口正常提供服务，而且经过一次失败后，会暂停服务 10 秒。浏览器访问 http://172.16.20.8，将只会出现如下页面：

![](http://imgblog.kuranado.com/1544875228.png)

此时如果执行 `iptables -I INPUT -p tcp --dport 8003 -j DROP` 命令，将关掉 8003 端口上的服务，浏览器访问 http://172.16.20.8，此时 8002 端口上的备份服务将开始提供服务：

![](http://imgblog.kuranado.com/1544875422.png)

执行 `iptables -F` 清除 iptables 规则，浏览器访问 http://172.16.20.8，8003 服务重新变为主服务，8002 回归备份服务身份：

![](http://imgblog.kuranado.com/1544875228.png)

### Nginx 负载均衡轮询策略

Nginx 之所以能够保证负载均衡，全在于其内部的轮询调度算法。Nginx 有以下几种调度算法：

算法|描述
-|
轮询|默认调度算法。按时间顺序逐一分配到不同的后端服务器
加权轮询|weight 值越大，分配到的访问几率越高
least_conn|最少连接数。哪台机器连接数最少，就将请求分发到哪台机器
ip_hash|每个请求按访问 IP 的 Hash 结果分配请求，这样来自同一个 IP 的访问将会固定访问同一个后端服务器
url_hash|按照访问的 URL 的 Hash 结果分配请求，这样来自同一个 URL 的访问将会固定访问同一个后端服务器
hash 关键数值|Hash 自定义的 Key
