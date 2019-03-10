---
title: 浅谈MySQL用户账号认证方式
date: 2018-01-17 15:33:00
comments: true
categories: [MySQL]
tags: [MySQL, 数据库]
---

为了有效控制数据库用户的访问权限，在MySQL数据库中创建了一个新用户，但使用刚创建的用户和密码却发现连接不了MySQL数据库，通过查看官网手册及《MySQL技术内幕》一书，才逐渐熟悉MySQL的用户账号认证方式，这里做一个简单总结。另外本篇文章基于MySQL5.5版本，如果需要测试文章内容，低于此版本[请先升级MySQL数据库](https://www.zerostopbits.com/how-to-upgrade-mysql-5-1-to-mysql-5-5-on-centos-6-7/)

<!-- more -->

MySQL数据库中有一个名称为`mysql`的系统数据库，`USE mysql;`进入该数据库，`SHOW TABLES`查看该数据库下的表，可以看到大概有二三十张表，本篇文章仅讨论`user`表

下面使用`CREATE`命令创建一个用户账号：

```
CREATE USER JING IDENTIFIED BY 'TESTPASSWORD';
```

每新创建的用户账号就会在`user`表下增加一条记录，使用`SELECT`命令查看`user`表可以找到刚刚创建的那个用户，用户名（User）为`JING`，密码（Password）已被加密：

![mark](http://imgblog.kuranado.com/blog/180117/l4kCke448F.png)

使用`Xshell`连接到CentOS服务器（我的MySQL数据库装在京东云的CentOS服务器上），输入`mysql -u JING -p`连接MySQL数据库服务器，输入密码后却拒绝访问，返回如下结果：

```
ERROR 1045 (28000): Access denied for user 'JING'@'localhost' (using password: YES)
```

这就非常奇怪了，明明使用正确的账号和密码，怎么就连接不了了呢，这种时候如果不了解MySQL的账户认证方式，可能会急得直跺脚吧！￣へ￣

下面就来简单介绍一下MySQL的用户账号认证方式：

首先MySQL的用户账号（account）由两部分组成：用户名（User）和主机名（Host）。格式为`'User'@'Host'`，简单举几个创建用户的例子：

```
# 用户名为Mary，密码为watermelon，必须从指定的主机地址192.168.128.3连接服务器
CREATE USER 'Mary'@'192.168.128.3' IDENTIFIED BY 'watermelon';
# 用户名为Jack，密码为pineapple，必须从C类子网192.168.128连接服务器
CREATE USER 'Jack'@'192.168.128.%' IDENTIFIED BY 'pineapple';
# 用户名为Jane，密码为pear，必须是test.com域名或其子域名下的主机连接服务器
CREATE USER 'Jane'@'%.test.com' IDENTIFIED BY 'pear';
# 用户名为Michael，密码为banana，只能从本地连接服务器
CREATE USER 'Michael'@'localhost' IDENTIFIED BY 'banana';
# 用户名为Bob@localhost，密码为jujube，可从任意主机连接服务器
CREATE USER 'Bob@localhost' IDENTIFIED BY 'jujube';
# 用户名为Smith，密码为orange，可从任意主机连接服务器
CREATE USER 'Smith'@'%' IDENTIFIED BY 'orange';
# 用户名为JING，密码为TESTPASSWORD，可从任意主机连接服务器
CREATE USER 'JING' IDENTIFIED BY 'TESTPASSWORD';
```

从这几个例子可以得出下面几条结论：

1.MySQL的账户认证除了需要用户名和密码正确之外，还必须从指定的主机连接服务器才能够成功访问MySQL数据库服务器
2.主机名可以使用`%`通配符
3.主机名可以是域名、IPv4地址、localhost，实际上还可以是IPv6地址、127.0.0.1等（localhost和127.0.0.1还是有一些区别的，不要误以为两者完全等价）
4.主机名可以省略，如果省略则默认主机名为`%`，表示任意主机，注意如果省略主机名，则`@`符号也需要省略，`'JING'@''`的主机名为空字符串并不是`%`
5.用户名和主机名需要用`@`符号隔开

除此之外，还有如下几点需要了解

6.用户名和主机名可以使用单引号括起来，也可以省略单引号，但如果包含像`%`、`_`等特殊字符，则单引号不可以省略
7.**MySQL服务器对用户的连接请求先根据主机地址（Host）匹配，再根据用户名（User）行匹配，最后才会匹配密码（Password）**

重点在于第7点，根据第7点写出下面的SQL语句：

```
SELECT Host, User, Password FROM user ORDER BY Host DESC, User DESC;
```

返回结果如下：

![mark](http://imgblog.kuranado.com/blog/180117/aj62D2Eeim.png)

回到刚开始的问题，为什么我使用正确的用户名和密码连接MySQL服务器会被拒绝访问呢？

这是因为我使用`CREATE USER JING IDENTIFIED BY 'TESTPASSWORD';`命令创建的用户`JING `可以通过任意主机访问MySQL服务器，而我通过Xshell登录京东云主机，然后使用`mysql -u JING -p`命令连接MySQL服务器，使用的主机地址（Host）为：`localhost`，用户名（User）为：`JING`，密码（Password）为：`TESTPASSWORD`。根据`SELECT Host, User, Password FROM user ORDER BY Host DESC, User DESC;`返回的结果，MySQL会从上往下逐个去校验匹配，最终匹配第2条记录，即Host为`localhost`，User为空，密码为空。而一旦成功匹配一条记录，便不会继续向下匹配了。

综合上面的分析，如果想要成功连接MySQL，只要执行`mysql -u JING -p`，当需要输入密码时，不用输入任何内容直接回车就可以了；如果我是通过我的Windows笔记本想要使用用户`JING`连接我的京东云MySQL数据库，只要打开`cmd`执行如下命令然后输入密码（此处需要输入密码是因为MySQL只能匹配到Host为`%`，User为`JING`，Password为`TESTPASSWORD`的那条记录）即可成功连接：

```
mysql -u JING -h 116.196.114.75 -p
```

![mark](http://imgblog.kuranado.com/blog/180117/hgd4Ce2JCi.png)

登录之后我们也可以通过如下命令查看当前是通过哪个账号（account）连接的，从而验证上面的结论：

```
SELECT CURRENT_USER();
```

Xshell登录京东云主机返回的结果：

![mark](http://imgblog.kuranado.com/blog/180117/a3h8CELi6b.png)

我的Windows命令行返回的结果：

![mark](http://imgblog.kuranado.com/blog/180117/H8KHgf7m89.png)