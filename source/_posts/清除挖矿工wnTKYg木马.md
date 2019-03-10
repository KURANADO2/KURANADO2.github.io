---
title: 清除挖矿工wnTKYg木马
date: 2017-11-09 22:18:00
comments: true
categories: [Redis]
tags: [木马, Redis, 数据库, 缓存]
---

今天打开京东云控制台时发现服务器CPU使用率飚到100%，这显然是中毒的迹象，结果就发现了wnTKYg挖矿工木马，究其原因，原来是因为我没有为Redis设置连接密码，而Redis又是使用root权限安装的，病毒通过远程连接Redis在服务器中放置定时任务做着挖矿的事情

<!-- more -->

## 清除木马

打开京东云控制台的监控页面是这样的：

![mark](http://imgblog.kuranado.com/blog/171109/GI2Fajj2bm.png)

`top`查看：

![mark](http://imgblog.kuranado.com/blog/171109/09L2eDjE3f.png)

发现有个名称为wnTKYg的进程消耗着99.9%的CPU，于是赶紧把它`kill -9`掉了，可是没过多会这个进程又重新启动了：

![mark](http://imgblog.kuranado.com/blog/171109/eAFCFhj7fb.png)

上网查了一下才发现这个wnTKYg木马是用来挖币的，要想删除该木马除需要kill掉它所对应的进程外还必须kill它的守护进程ddg，首先查看这两个进程的源文件位置：

```
ps -ef | grep wnTKYg    //查看源文件位置还可以通过top命令之后按c键来查看
ps -ef | grep ddg
kill -9 进程ID1 进程ID2
```

发现位置都是`/tmp`目录，进入到`/tmp`删除这两个文件之后，然后删除`/var/spool/cron`目录就可以清除该病毒了

## Redis设置连接密码

为了防止再次中毒，为Redis设置连接密码

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