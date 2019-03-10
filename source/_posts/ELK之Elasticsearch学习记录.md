---
title: ELK之Elasticsearch学习记录
date: 2018-08-20 12:30:00
comments: true
categories: [ELK]
tags: [ELK,Elasticsearch,日志]
---

公司使用 ELK 搭建日志系统，遂找教程学习了 ELK，此处记录。教程链接[https://edu.aqniu.com/course/6388](https://edu.aqniu.com/course/6388)。这套教程我觉得还是挺贵的，当时是因为做活动，所以获得了免费学习资格。
本文使用 CentOS 6.7 进行测试，硬件配置最好内存 8G，处理器 2核

<!-- more -->

## 什么是 Elasticsearch

基于 Lucene 的搜索服务器，开始使用 Java 开发，后使用 Ruby 开发，现使用 Node js 开发

## 为什么使用 Elasticsearch

日志量大，难以查询，Elasticsearch 可以解决这些问题

## Elasticsearch 2.x 安装

### 安装步骤

首先需要确保机器已经安装 JDK 1.8 及以上版本，可直接使用 `yum install -y java` 命令安装，如果默认安装的不是 JDK 1.8，则配置 yum 源即可

```
[root@jing server]# java -version
openjdk version "1.8.0_181"
OpenJDK Runtime Environment (build 1.8.0_181-b13)
OpenJDK 64-Bit Server VM (build 25.181-b13, mixed mode)
```

前往 Elasticsearch 的官网下载指定版本的 tar 包上传到服务器，或直接复制下载链接使用 wget 命令下载，我这里下载的是 Elasticsearch 2.3.1 版本：

```
[root@jing ~]mkdir -p /data/server
[root@jing ~]cd /data/server
[root@jing server]wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.3.1/elasticsearch-2.3.1.tar.gz
tar -xvf elasticsearch-2.3.1.tar.gz
```

### 修改配置文件

```
[root@jing server]# vim elasticsearch-2.3.1/config/elasticsearch.yml
```

简单修改内容如下：

```
# 指定 Elasticsearch 集群的名称
cluster.name: cluster
# 指定 Elasticsearch 当前节点的名称
node.name: cluster-node1
# 指定 IP 地址为当前机器的 IP 地址，ifconfig 命令可查看
network.host: 192.168.163.128
# 指定服务的端口号
http.port: 9200
```

### 启动 Elasticsearch

因为 Elasticsearch 不能使用 `root` 身份启动，强行启动会报对应的错误：

```
[root@jing server]# cd elasticsearch-2.3.1/bin
root@jing bin]# ./elasticsearch
Exception in thread "main" java.lang.RuntimeException: don't run elasticsearch as root.
	at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:93)
	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:144)
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:270)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:35)
Refer to the log for complete error details.
```

所以需要使用其他用户来操作 Elasticsearch，这里新增加一个用户 `eser`，并把 Elasticsearch 目录的所有者和所属组都改为 eser

```
[root@jing bin]# useradd eser
[root@jing bin]# chown -R eser.eser /data/server/elasticsearch-2.3.1
# 关闭防火墙（CentOS6.7）
[root@jing bin]# service iptables stop
[root@jing bin]# su eser
[eser@jing bin]# ./elasticsearch
# 查看 9200 端口上的服务是否已经启动，若没有 netstat 命令，则执行 yum install -y net-tools 命令即可安装 netstat 命令
[eser@jing bin]# netstat -tlunp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
...     
tcp        0      0 ::ffff:192.168.163.128:9200 :::*                        LISTEN      9253/java           
tcp        0      0 ::ffff:192.168.163.128:9300 :::*                        LISTEN      9253/java           
...
```

![mark](http://imgblog.kuranado.com/blog/180816/Ihajif0JKF.png)

### 插件安装

```
# 由于网络原因该插件可能会安装失败
[eser@jing bin]# ./plugin install lmenezes/elasticsearch-kopf
# head 插件的 GitHub 地址为：https://github.com/mobz/elasticsearch-head，可以看到 5.x 版本不支持 head 插件
[eser@jing bin]# ./plugin install mobz/elasticsearch-head
```

安装完 kopf 插件，浏览器访问如下：

![mark](http://imgblog.kuranado.com/blog/180816/2jl2f92g13.png)

head 插件：

![mark](http://imgblog.kuranado.com/blog/180816/711iIaKd54.png)

## Elasticsearch 5.X 安装

相比 Elasticsearch 2.X，Elasticsearch 5.X 的启动过程会报更多的错误，需要一一排查解决，本文安装的是 Elasticsearch 5.6 版本，后续文章的 LogStash 和 Kibana 也都将安装 5.6 版本

### 安装步骤

同样要求 JDK 版本为 1.8

```
[root@jing ~]# cd /data/server
[root@jing server]# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.2.tar.gz
[root@jing server]# tar -xvf elasticsearch-5.6.2.tar.gz
[root@jing server]# vim elasticsearch-5.6.2/config/elasticsearch.yml
```

### 修改配置文件

修改文件如下：

```
cluster.name: my-elk
node.name: node-1
# 公网上的机器为了让任何主机都都能访问 Elasticsearch 服务，所以将其设置为 0.0.0.0，如果只是在虚拟机上进行测试，请设置为虚拟机的 IP 地址
network.host: 192.168.163.128
http.port: 9200
```

### 启动 Elasticsearch

同样不能以 root 用户启动 Elasticsearch：

```
[root@jing server]# chown -R eser.eser elasticsearch-5.6.2
[root@jing server]# su eser
[eser@jing server]$ cd elasticsearch-5.6.2/bin/
# 尝试以普通用户启动 Elasticsearch
[eser@jing bin]$ ./elasticsearch
```

打印日志如下：

```
[2018-08-17T00:56:58,275][WARN ][o.e.b.JNANatives         ] unable to install syscall filter: 
java.lang.UnsupportedOperationException: seccomp unavailable: CONFIG_SECCOMP not compiled into kernel, CONFIG_SECCOMP and CONFIG_SECCOMP_FILTER are needed
	at org.elasticsearch.bootstrap.SystemCallFilter.linuxImpl(SystemCallFilter.java:364) ~[elasticsearch-5.6.2.jar:5.6.2]
	at org.elasticsearch.bootstrap.SystemCallFilter.init(SystemCallFilter.java:639) ~[elasticsearch-5.6.2.jar:5.6.2]
	at org.elasticsearch.bootstrap.JNANatives.tryInstallSystemCallFilter(JNANatives.java:258) [elasticsearch-5.6.2.jar:5.6.2]
	at org.elasticsearch.bootstrap.Natives.tryInstallSystemCallFilter(Natives.java:113) [elasticsearch-5.6.2.jar:5.6.2]
	at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:111) [elasticsearch-5.6.2.jar:5.6.2]
	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:195) [elasticsearch-5.6.2.jar:5.6.2]
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:342) [elasticsearch-5.6.2.jar:5.6.2]
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:132) [elasticsearch-5.6.2.jar:5.6.2]
	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:123) [elasticsearch-5.6.2.jar:5.6.2]
	at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:67) [elasticsearch-5.6.2.jar:5.6.2]
	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:134) [elasticsearch-5.6.2.jar:5.6.2]
	at org.elasticsearch.cli.Command.main(Command.java:90) [elasticsearch-5.6.2.jar:5.6.2]
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:91) [elasticsearch-5.6.2.jar:5.6.2]
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:84) [elasticsearch-5.6.2.jar:5.6.2]
[2018-08-17T00:57:11,592][INFO ][o.e.n.Node               ] [node-1] initializing ...
[2018-08-17T00:57:11,757][INFO ][o.e.e.NodeEnvironment    ] [node-1] using [1] data paths, mounts [[/ (/dev/mapper/vg_jing-lv_root)]], net usable_space [14gb], net total_space [17.1gb], spins? [possibly], types [ext4]
[2018-08-17T00:57:11,758][INFO ][o.e.e.NodeEnvironment    ] [node-1] heap size [1.9gb], compressed ordinary object pointers [true]
[2018-08-17T00:57:11,760][INFO ][o.e.n.Node               ] [node-1] node name [node-1], node ID [6jO7gjdgQ2C6mlsdCcZ1hw]
[2018-08-17T00:57:11,760][INFO ][o.e.n.Node               ] [node-1] version[5.6.2], pid[9978], build[57e20f3/2017-09-23T13:16:45.703Z], OS[Linux/2.6.32-573.el6.x86_64/amd64], JVM[Oracle Corporation/OpenJDK 64-Bit Server VM/1.8.0_181/25.181-b13]
[2018-08-17T00:57:11,760][INFO ][o.e.n.Node               ] [node-1] JVM arguments [-Xms2g, -Xmx2g, -XX:+UseConcMarkSweepGC, -XX:CMSInitiatingOccupancyFraction=75, -XX:+UseCMSInitiatingOccupancyOnly, -XX:+AlwaysPreTouch, -Xss1m, -Djava.awt.headless=true, -Dfile.encoding=UTF-8, -Djna.nosys=true, -Djdk.io.permissionsUseCanonicalPath=true, -Dio.netty.noUnsafe=true, -Dio.netty.noKeySetOptimization=true, -Dio.netty.recycler.maxCapacityPerThread=0, -Dlog4j.shutdownHookEnabled=false, -Dlog4j2.disable.jmx=true, -Dlog4j.skipJansi=true, -XX:+HeapDumpOnOutOfMemoryError, -Des.path.home=/data/server/elasticsearch-5.6.2]
[2018-08-17T00:57:13,837][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [aggs-matrix-stats]
[2018-08-17T00:57:13,837][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [ingest-common]
[2018-08-17T00:57:13,838][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [lang-expression]
[2018-08-17T00:57:13,838][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [lang-groovy]
[2018-08-17T00:57:13,840][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [lang-mustache]
[2018-08-17T00:57:13,840][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [lang-painless]
[2018-08-17T00:57:13,840][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [parent-join]
[2018-08-17T00:57:13,841][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [percolator]
[2018-08-17T00:57:13,841][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [reindex]
[2018-08-17T00:57:13,841][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [transport-netty3]
[2018-08-17T00:57:13,841][INFO ][o.e.p.PluginsService     ] [node-1] loaded module [transport-netty4]
[2018-08-17T00:57:13,842][INFO ][o.e.p.PluginsService     ] [node-1] no plugins loaded
[2018-08-17T00:57:16,784][INFO ][o.e.d.DiscoveryModule    ] [node-1] using discovery type [zen]
[2018-08-17T00:57:18,170][INFO ][o.e.n.Node               ] [node-1] initialized
[2018-08-17T00:57:18,170][INFO ][o.e.n.Node               ] [node-1] starting ...
[2018-08-17T00:57:31,576][INFO ][o.e.t.TransportService   ] [node-1] publish_address {192.168.163.128:9300}, bound_addresses {[::]:9300}
[2018-08-17T00:57:31,592][INFO ][o.e.b.BootstrapChecks    ] [node-1] bound or publishing to a non-loopback or non-link-local address, enforcing bootstrap checks
ERROR: [4] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
[2]: max number of threads [1024] for user [eser] is too low, increase to at least [2048]
[3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
[4]: system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk
[2018-08-17T00:57:31,619][INFO ][o.e.n.Node               ] [node-1] stopping ...
[2018-08-17T00:57:31,893][INFO ][o.e.n.Node               ] [node-1] stopped
[2018-08-17T00:57:31,894][INFO ][o.e.n.Node               ] [node-1] closing ...
[2018-08-17T00:57:31,911][INFO ][o.e.n.Node               ] [node-1] closed
```

### 启动报错解决方案

启动过程中发现报了很多错误，针对这些错误的解决方案如下：

> [1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]

对于 Elasticsearch 进程来说，最大文件描述符 4096 太小了，应该增大到至少 65536
可以使用 `ulimit -n` 命令来查看系统最大文件描述符大小：

```
[eser@jing server]# ulimit -n
4096
```

更改 `etc/security/limits.conf` 文件，在文件最后添加如下两行代码，表示为 eser 用户设置最大文件描述符（最大文件描述符即最大文件打开数）为 65536，保存之后，eser **用户需要退出 (`exit`) 重新登录 (`su eser`) 才能使用 `ulimit -n` 命令查看到更新后的值**：

```
# nofile 表示最大文件描述符，soft 表示软件限制，hard 表示硬件限制
eser soft nofile 65536
eser hard nofile 65536
```

如果要为所有用户都设置最大文件描述符，则语法如下：

```
* soft nofile 65536
* hard nofile 65536
```

> [2]: max number of threads [1024] for user [eser] is too low, increase to at least [2048]

对于 eser 用户来说最大进程数 1024 太小，至少应该增加至 2048，在 `etc/security/limits.conf` 文件末尾增加如下配置：

```
eser soft nproc 2048
eser hard nproc 2048
```

使用 `ulimit -u` 命令查看当前用户的最大进程数：

```
[eser@jing server]$ ulimit -u
2048
```

> [3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

该数值用于表示一个进程可以拥有的虚拟内存区域(Virtual Memory Area)的数量

修改 `/etc/sysctl.conf` 文件，添加如下代码即可：

```
vm.max_map_count = 262144
```

同时可以使用 `sysctl -p` 命令查看 `/etc/sysctl.conf` 文件配置：

```
[root@jing server]# sysctl -p
net.ipv4.ip_forward = 0
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
kernel.sysrq = 0
kernel.core_uses_pid = 1
net.ipv4.tcp_syncookies = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.shmmax = 68719476736
kernel.shmall = 4294967296
vm.max_map_count = 262144
```

> [4]: system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk

在 `/data/server/elasticsearch-5.6.2/config/elasticsearch.yml` 文件中添加如下配置：

```
bootstrap.system_call_filter: false
```

关闭防火墙后重新以 `eser` 用户启动 Elasticsearch 可以成功启动，在浏览器访问如下：

![mark](http://imgblog.kuranado.com/blog/180819/iB5JGKeIBJ.png)

### 安装 head 插件

#### 下载插件

首先保证机器上已经安装 git，可参考这里：[https://blog.csdn.net/qq_35213388/article/details/80507245
](https://blog.csdn.net/qq_35213388/article/details/80507245)

然后使用 git 克隆 head 插件在 GitHub 上的项目，如使用 SSH 方式克隆失败，请尝试使用 HTTPS 方式进行克隆：

```
[root@jing server]# git clone https://github.com/mobz/elasticsearch-head.git
```

#### 安装 Node.js

```
[root@jing server]# wget https://nodejs.org/download/release/v4.6.1/node-v4.6.1-linux-x64.tar.gz
# 配置环境变量
[root@jing server]# export PATH=/data/server/node-v4.6.1-linux-x64/bin:$PATH
[root@jing server]# node --version
v4.6.1
```

#### 安装 head 插件

- 查看 elasticsearch-head 目录下是否有 `node_module/grunt` 目录，没有则执行如下命令创建：

```
[root@jing elasticsearch-head]# npm install grunt --save
```

- 正式安装 head 插件：

```
[root@jing elasticsearch-head]# npm install
```

- 安装 grunt

```
[root@jing elasticsearch-head]# npm install -g grunt-cli
```

- 编辑 Gruntfile.js

93 行处添加如下代码：

```
# 注意不要漏掉单引号
hostname: '0.0.0.0',
```

![mark](http://imgblog.kuranado.com/blog/180819/7K9Al14La2.png)

- 检查 elasticsearch-head 目录下是否有 `base` 文件夹，若没有，则将 `_site` 下的 base 文件夹拷贝到 elasticsearch-head 目录下：

```
[root@jing elasticsearch-head]# cp -r  _site/base/ .
```

- 启动 grunt server：

```
[root@jing elasticsearch-head]# grunt server -d
```

启动成功：

![mark](http://imgblog.kuranado.com/blog/180819/bJ69eF8ILg.png)


浏览器访问 head 插件效果如下：

![mark](http://imgblog.kuranado.com/blog/180819/j0E6eDKJE9.png)