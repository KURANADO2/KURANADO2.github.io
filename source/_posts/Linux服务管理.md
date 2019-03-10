---
title: Linux服务管理
date: 2018-2-17 19:16:00
comments: true
categories: [Linux]
tags: [Linux]
---

看了[慕课网Tony老师](https://www.imooc.com/u/279399/courses?sort=publish)关于[Linux服务管理](https://www.imooc.com/learn/537)的课程，学到了很多，在此做个笔记，这里也强烈推荐Linux小白们能够认真学习这套课程，绝对会让你受益匪浅

<!-- more -->

**参考文章：**
[chkconfig命令](http://man.linuxde.net/chkconfig)
[7 Linux chkconfig Command Examples – Add, Remove, View, Change Services](https://www.thegeekstuff.com/2011/06/chkconfig-examples/)
**测试系统：CentOS6.8**

## runlevel

首先需要知道 `Linux` 系统有7个运行级别分别如下（这7个运行级别各自所代表的含义也可以在 `/etc/inittab` 文件中查看到）：

```
0 - halt (Do NOT set initdefault to this) # 关机
1 - Single user mode # 单用户模式，类似于Windows的安全模式，主要用于系统修复
2 - Multiuser, without NFS (The same as 3, if you do not have networking) # 无网络连接的多用户命令行模式
3 - Full multiuser mode # 有网络连接的多用户命令行模式，也被称为标准模式
4 - unused # 不可用
5 - X11 # 带图形界面的多用户模式（前提是安装了图形界面）
6 - reboot (Do NOT set initdefault to this) # 重新启动
```

`runlevel` 命令可以查看系统当前的运行级别：

```
[root@JDu4e00u53f7 httpd-2.4.29]# runlevel
N 3
```

看到系统当前的运行级别为 `3` ，即有网络连接的多用户命令行模式，这也是服务器版的 `CentOS` 的默认运行级别，关于默认运行级别，可以通过 `/etc/inittab` 文件进行修改：在 `/etc/inittab` 文件中，可以找到这样一行代码：

```
id:3:initdefault:
```

只要更改数字即可在以后的开机或重启时自动进入指定的运行级别，如更改为 `id:5:initdefault:` ，则在以后每次开机或重启时自动进入图形化界面。需要注意的是，千万不要把这个数字更改为 `0` 或 `1` ，否则系统一打开就自动关机或自动重启，事实上关于运行级别 `0` 和 `1` ，`/etc/inittab` 文件中也明确的提示了我们：`Do NOT set initdefault to this`

`init` 命令可以进入相应的运行级别：

```
init 5 # 进入图形界面，注意前提是已经安装了图形界面，否则执行init 5会报错（在诸如XShell、Secure CRT等远程连接工具下并不能看到报错信息）
```

再次运行 `runlevel` 命令：

```
[root@JDu4e00u53f7 etc]# runlevel
3 5
```

其中前一个数字表示之前的运行级别，后一个数字表示系统现在的运行级别

都知道 `shutdown` 和 `reboot` 命令分别用来关机和重启，有了 `init` 命令，结合运行级别，同样可以完成关机和重启的功能：

```
init 0 # 关机
init 6 # 重启
```

只是 `init 0` 和 `init 6` 有时并不能保证正确的关机/重启 ，所以还是建议使用 `shutdown` 和 `reboot` 命令

## chkconfig

`chkconfig` 命令用于检查和设置系统的各种服务：

```
[root@JDu4e00u53f7 etc]# chkconfig --list
agentboot      	0:off	1:off	2:on	3:on	4:on	5:on	6:off
auditd         	0:off	1:off	2:on	3:on	4:on	5:on	6:off
blk-availability	0:off	1:on	2:on	3:on	4:on	5:on	6:off
crond          	0:off	1:off	2:on	3:on	4:on	5:on	6:off
htcacheclean   	0:off	1:off	2:off	3:off	4:off	5:off	6:off
httpd          	0:off	1:off	2:on	3:on	4:on	5:on	6:off
...
xinetd         	0:off	1:off	2:off	3:on	4:on	5:on	6:off

xinetd based services: # 在centos6中该服务默认没有安装，如要使用需要自己安装（yum install -y xinetd），xinetd服务正在逐渐被淘汰，基于xinetd的服务已经越来越少，到CentOS7已经完全淘汰xinetd了
	chargen-dgram: 	off
	chargen-stream:	off
	daytime-dgram: 	off
	...
	time-stream:   	off
```

`chkconfig --list` 命令返回结果表示系统中各个独立服务在以后开机进入各个运行级别后的开启/关闭状态，而不是表示服务当前的开启/关闭状态，拿 `msyqld` 服务来说：

```
[root@JDu4e00u53f7 etc]# chkconfig --list mysqld
mysqld         	0:off	1:off	2:on	3:on	4:on	5:on	6:off
```

其返回结果表示下次开机进入 `2345` 任一级别，`mysqld` 服务都会自动启动，假设系统现在处于运行级别 `3` ，并不能代表 `mysqld` 服务现在已处于启动状态

`chkconfig` 结合 `grep` 命令还可以进行结果筛选，如 `chkconfig --list | grep 3:on`、`chkconfig --list | grep ip6`等

`chkconfig` 其它常用选项：

```
chkconfig --add httpd # 添加httpd服务
chkconfig --del httpd # 删除httpd服务
chkconfig --level 2345 httpd on 一般2345放在一块使用，执行这条命令后只是代表在下次开机进入2345任一级别时都启动httpd服务，并不代表httpd服务现在已经启动
chkconfig mysqld on  #设置mysqld服务2345等级为on
```

`chkconfig` 命令不能查看到源码包安装的服务，除此之外，像service、ntsysv等命令也不能操作通过源码包安装的服务

## service

`service` 命令用于查看服务状态，开启/关闭服务，常用命令如下：

```
service service_name status # 查看服务状态
service --status-all # 查看所有系统服务状态
service service_name start # 启动服务
service service_name stop # 关闭服务
service service_name restart # 重启服务
```

而 `service` 命令可以操作的服务都必须是 `/etc/rc.d/init.d/` 目录下存在的服务,查看 `init.d` 目录下的文件如下：

```
[root@JDu4e00u53f7 init.d]# ll /etc/rc.d/init.d
total 200
-rwxr-xr-x  1 root root   250 Nov 13 11:32 agentboot
-rwxr-xr-x. 1 root root  3580 May 11  2016 auditd
-r-xr-xr-x. 1 root root  1343 May 11  2016 blk-availability
...
-rwxr-xr-x. 1 root root  4621 May 11  2016 sshd
-rwxr-xr-x. 1 root root  2294 May 11  2016 udev-post
-rwxr-xr-x  1 root root  3555 May 11  2016 xinetd

```


 
## 通过RPM包安装的服务和通过源码包安装的服务

我们知道 `Linux` 上安装一个服务主要通过两种方式来实现，一个是通过RPM包安装，一个是通过下载源码包安装，这两种方式都各有优缺点，而通过源码包安装的服务最大的缺点就是：`chkconfig` 命令不能查看到源码包安装的服务，除此之外，像`service`、`ntsysv`等命令也不能操作通过源码包安装的服务，这就意味着，想要启动一个服务会更麻烦，想要把一个服务添加到开机自动启动也会更麻烦

这里就详细说说如何让 `chkconfig` 和 `service` 命令也能够操作通过源码包安装的服务，这里以 `apache` 的 `httpd` 服务为例

首先到 [apache 官网下载apache源码包](http://httpd.apache.org/download.cgi)，看到目前官网最新的版本为 `2.4.29` 版本，这里并不推荐下载这个最新版本，因为这个版本在执行 `./configure --prefix=PREFIX` 命令时会报错，虽然[网上有对应的解决办法](https://stackoverflow.com/questions/9436860/apache-httpd-setup-and-installation/9436971)，但还是推荐大家下载不会报错的 `2.2.34` 版本。将源码包上传到 `CentOS` 上，解压之后进入解压后的目录依次执行以下命令：

```
./configure --prefix=/usr/local/apache2
make
make install
```

关于为什么这三条命令就是安装步骤，完全可以通过查看解压后目录中的 `INSTALL` 文件得知

好了，现在我们已经通过源码包安装了 `apache` ，同样通过查看 `INSTALL` 文件，可以知道想要启动 `apache` 服务需要执行下面这条命令：

```
/usr/local/apache2/bin/apachectl start
```

因为通过源码包安装的 `apache` 服务和通过 `rpm` 包安装的 `apache` 服务都需要占用 `80` 端口，所以要想 `/usr/local/apache2/bin/apachectl start` 执行成功，请先调用 `servcie httpd stop` 命令关掉通过 `rpm` 包安装的 `apache` 服务

现在通过在浏览器中输入服务器的IP地址，看到页面上显示如下，则说明通过源码包安装的 `apache` 服务已经成功启动：

![mark](http://imgblog.kuranado.com/blog/180217/AJ4BLC5dDm.png)

当然也可以通过 `netstat` 命令查看到 `80` 端口被占用证明服务已经启动：

```
[root@JDu4e00u53f7 rc.d]# netstat -tlunp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
tcp        0      0 0.0.0.0:3306                0.0.0.0:*                   LISTEN      1321/mysqld         
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      3447/sshd           
tcp        0      0 :::80                       :::*                        LISTEN      30178/httpd         
...      
udp        0      0 ::1:123                     :::*                                    1182/ntpd           
udp        0      0 :::123                      :::*                                    1182/ntpd 
```

 我们知道 `service` 命令只能操作 `/etc/rc.d/init.d/` 目录下存在的服务，所以只要在 `init.d` 目录下添加一个链接指向通过源码包安装的服务就可以使用 `service` 命令操作该服务了，这也使大多数情况下遇到 `unrecognized service` 报错的常见解决方法：

```
[root@JDu4e00u53f7 init.d]# service apachectl start
apachectl: unrecognized service
[root@JDu4e00u53f7 init.d]# ln -s /usr/local/apache2/bin/apachectl /etc/rc.d/init.d/
[root@JDu4e00u53f7 init.d]# service apachectl start
[root@JDu4e00u53f7 init.d]# netstat -tlunp | grep httpd
tcp        0      0 :::80                       :::*                        LISTEN      937/httpd 
```

那如何通过 `chkconfig` 命令将通过源码包安装的服务添加到开机启动呢？
首先需要知道 `/etc/rc.d/` 目录下有 `rc0` ~ `rc6` 7个目录，因为我的系统现在处于运行级别 `3`，所以我进入 `rc3.d` 目录下查看文件：

```
[root@JDu4e00u53f7 init.d]# ll /etc/rc.d/rc3.d
total 0
lrwxrwxrwx. 1 root root 19 Jun  6  2017 K10saslauthd -> ../init.d/saslauthd
lrwxrwxrwx  1 root root 22 Feb 14 12:21 K15htcacheclean -> ../init.d/htcacheclean
lrwxrwxrwx  1 root root 17 Jun 26  2017 K30postfix -> ../init.d/postfix
lrwxrwxrwx  1 root root 17 Jun  6  2017 K75ntpdate -> ../init.d/ntpdate
lrwxrwxrwx. 1 root root 20 Jun  6  2017 K87multipathd -> ../init.d/multipathd
...
lrwxrwxrwx  1 root root 15 Feb 16 16:28 S85httpd -> ../init.d/httpd
lrwxrwxrwx. 1 root root 15 Jun  6  2017 S90crond -> ../init.d/crond
lrwxrwxrwx. 1 root root 11 Jun  6  2017 S99local -> ../rc.local

```

该目录下全都是指向 `/etc/rc.d/init.d/` 目录下服务的链接，而每个链接要么以 `K` 开头要么以 `S` 开头，其后跟两个数字， 其含义为 `K-kill`关闭，`S-start` 启动，即进入运行级别 `3` 时，依次启动以 `S` 开头的链接所指向的服务，比如`S90crond` 比 `S99local`所指向的服务先启动，同理当退出系统时，则依次执行以 `K` 开头的链接所指向的服务，同样数字越小的越先执行

在服务脚本中添加代码：

```
vim /etc/rc.d/init.d/apachectl
```

![mark](http://imgblog.kuranado.com/blog/180217/BmAH696Bgi.png)

第一组数字表示运行级别，第二组数字表示启动顺序，第三组数字表示关闭顺序，注意第二组、第三组数字不要和对应的rcN.d目录下的数字重复；描述内容随意

最后把服务添加到 `chkconfig`下开机启动即可：

```
[root@JDu4e00u53f7 init.d]# chkconfig --list apachectl
service apachectl supports chkconfig, but is not referenced in any runlevel (run 'chkconfig --add apachectl')
[root@JDu4e00u53f7 init.d]# chkconfig --add apachectl
[root@JDu4e00u53f7 init.d]# chkconfig apachectl on # 添加开机启动
[root@JDu4e00u53f7 init.d]# chkconfig --list apachectl
apachectl      	0:off	1:off	2:on	3:on	4:on	5:on	6:off
```

## 修改/etc/rc.d/rc.local文件添加开机自启动

通过上面的介绍可以了解到rpm包安装的服务可以很方便的使用 `chkconfig` 命令设置开机自启动，而对于源码包安装的服务虽然也可以使用 `chkconfig` 命令设置开机自启，但过程毕竟是麻烦了点，那有没有更简单的方法呢？答案是肯定的。经过前面的一系列操作，不知道大家有没有注意到 `/etc/rc.d/` 目录下有一个名为 `rc.local` 的文件呢，打开该文件，内容如下：

```
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.

touch /var/lock/subsys/local
```

这个文件明确告诉我们该文件会在所有的初始化脚本执行完毕后执行，所以我们完全可以在这个文件中添加自己的脚本，这样就能在每次开机后自动执行了，比如在文件末尾添加以下代码：

```
/usr/local/apache2/bin/apachectl start
service mysqld start
```

这样在下次开机后就能自动启动通过源码包安装的 `apache` 和通过rpm包安装的 `mysqld` 服务了，是不是非常简单方便呢！