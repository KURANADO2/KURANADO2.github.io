---
title: CentOS安装Redis
date: 2017-11-03 23:16:00
comments: true
categories: [Redis]
tags: [Redis, 数据库, 缓存, CentOS]
---

几个月前做了个网上商城项目，其中商城首页用到了Redis做缓存，当时只是随便用了一下，并没有系统学习Redis。上个星期面试时被问到几个Redis相关问题，虽然勉强答上来了但却明显能看出我对Redis并不熟。最近在做黑马的淘淘商城时，又用到了搭建Redis集群，所以结合老师提供的文档和自己查找的资料整理了Redis相关知识，本篇文章记录如何安装Redis

<!-- more -->

## 安装Redis

- 因为Redis是使用C语言开发的，所以需要安装C语言编译环境，已有忽略

```
yum install gcc-c++
```

- 下载Redis压缩包[https://redis.io/](https://redis.io/)，并上传到云服务器上（若使用XShell，可通过`rz`命令上传文件）

```
tar xvf redis-4.0.2.tar.gz -C /usr/local
cd /usr/local/redis-4.0.2	
make	//编译
make install PREFIX=/usr/local/redis	//安装到指定目录下
```

## 启动Redis

### 前台启动：

如果窗口关闭，则Redis服务就会关闭
```
./redis-server
```

### 后台启动：

```
cp /usr/local/redis-4.0.2/redis.conf /usr/local/redis/bin
vim /usr/local/redis/bin/redis.conf		//将daemonize的值由no改为yes
cd /usr/local/redis/bin
./redis-server redis.conf	//后台启动
```

## 连接Redis服务

```
./redis-cli	//默认连接的是运行在本地（127.0.0.1）6379端口上的Redis服务
./redis-cli -h 116.196.154.75 -p 6379	//连接指定服务器指定端口上的Redis服务
```

## 断开Redis服务

```
./redis-cli shutdown
```

## Redis设置连接密码

博主一开始安装完Redis就没有设置连接密码，结果一个星期之后服务器就中了病毒，我的具体遭遇写在了这篇文章里：[清除挖矿工wnTKYg木马](http://www.kuranado.com/2017/11/09/%E6%B8%85%E9%99%A4%E6%8C%96%E7%9F%BF%E5%B7%A5wnTKYg%E6%9C%A8%E9%A9%AC/)，所以出于安全考虑一定要为Redis设置连接密码，具体设置方法如下：

```
vim redis.conf
```

取消requirepass的注释，然后设置密码即可：

![mark](http://imgblog.kuranado.com/blog/171109/9L22hhFdaC.png)

重启Redis服务，使用密码连接Redis：

```
./redis-cli shutdown
./redis-server redis.conf
./redis-cli -a YiZLD2bjzkDfdfanE74EKYWt    //此处为本地连接，远程连接还需加-h和-p参数（hostAddress和port）
```

关于Redis安全方面的更多应对方法可参考官方文档：[Redis Security](https://redis.io/topics/security)