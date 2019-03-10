---
title:	京东云CentOS6.8安装JDK1.8+Tomcat7+MySQL
date: 2017-09-26 18:23:00
comments: true
categories: [Linux]
tags: [Linux, CentOS, JDK, Tomcat, MySQL]
---

之前一直在用阿里云的云主机，学生的话一个月10块钱，非常便宜。但是这个月初学长推荐了京东云主机，价格比阿里云的要便宜很多，完成学生认证的话一年只需88，没有学生资格一年也只需要180块左右。主机买完之后这段时间都没有做过任何配置，今天因为打算把WEB项目部署上去所以做了一些基本的安装配置，在这里做一个记录。

<!-- more -->

## 安装JDK1.8

JDK的安装我是先从官网下载的.tar.gz文件然后传到CentOS上的。另外如果服务器上已经预安装了JDK（通过`rpm -qa | grep java`查看）则应该先卸载掉预安装的JDK（通过`rpm -e --nodeps 要卸载的JDK`）

### 下载安装包

官网下载[jdk-8u144-linux-x64.tar.gz](http://www.oracle.com/technetwork/java/javase/downloads/index.html)到windows本地，然后通过SSH Secure Shell传到服务器

### 解压安装包

解压安装包到指定目录：
```
tar -xvf jdk-8u144-linux-x64.tar.gz -C /usr/local/jdk
```

### 配置环境变量

在/etc/profile文件末尾添加如下配置：
```
JAVA_HOME=/usr/local/jdk/jdk1.8.0_144
  JRE_HOME=$JAVA_HOME/jre
  PATH=$PATH:$JAVA_HOME/bin
  CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
  export JAVA_HOME
  export JRE_HOME
  export PATH
  export CLASSPATH
```

### 刷新配置

```
source /etc/pfofile
```

### 验证配置是否成功

如果正确输出版本信息则配置成功
```
java -version
```

## 安装Tomcat7

### 下载安装包

官网下载[Tomcat7](http://tomcat.apache.org/download-70.cgi)到windows本地，然后通过SSH Secure Shell传到服务器

### 解压安装包

```
tar -xvf apache-tomcat-7.0.81.tar.gz -C /usr/local
```

### 开放Linux的对外访问端口8080

```
/sbin/iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
//下面这条命令执行完如果看到命令行显示OK则表明成功
/etc/rc.d/init.d/iptables save
```

### 启动/关闭Tomcat

进入tomcat bin目录：

```
//启动Tomcat
./startup.sh
//关闭Tomcat
./shutdown.sh
```

### 验证Tomcat是否启动成功

执行如下命令若看到`Server startup in xxxx ms`则证明启动成功，通过在浏览器中访问http://YourIpAddress:8080将可以看到汤姆猫
```
tail -f logs/catalina.out
```

## 安装MySQL

和安装JDK时一样需要先查看服务器端是否已经安装了MySQL(`rpm -qa | grep mysql`)，如果有需要先卸载(rpm -e --nodeps 要卸载的预装mysql)

### 查看yum库中的MySQL

```
yum list | grep mysql
```

### 选择需要的MySQL进行安装

```
yum install mysql mysql-server mysql-devel -y
```

### 启动/关闭MySQL服务

```
service mysqld start	//启动MySQL服务
service mysqld stop		//停止MySQL服务
service mysqld restart	//重启MySQL服务
```

### 进入MySQL

启动MySQL服务后，可以直接通过如下命令登录MySQL，然后修改登录密码：

```
mysql -uroot
SET PASSWORD = PASSWORD('YourPassword');
//修改完密码后下次登录MySQL需要输入：
mysql -uroot -p
YourPassword(命令行不不显示)
```

### 添加MySQL到系统服务中并开启开机启动

```
chkconfig --add mysqld	//加入到系统服务
chkconfig mysqld on		//开机启动
```

### 开放远程登录权限

登录MySQL后输入如下命令：

```
GRANT ALL PRIVILEGES ON *.* TO 'root' @'%' IDENTIFIED BY 'YourPassword';		//需要注意的是YourPassword不要设置的太简单，否则远程连接会失败（比如密码设置为123456就会远程连接失败）！另外如果是阿里云的CentOS开放远程登录权限后还必须在阿里云控制台配置安全组才可以成功远程连接

FLUSH PRIVILEGES;	//刷新权限
```

上面两条命令可以合成一条：

```
GRANT ALL PRIVILEGES ON *.* TO 'root' @'%' IDENTIFIED BY 'YourPassword' WITH FLUSH PRIVILEGES;
```

### 开放Linux的对外访问端口3306

```
/sbin/iptables -I INPUT -p tcp --dport 3306 -j ACCEPT
/etc/rc.d/init.d/iptables save		//将修改永久保存到防火墙中,执行完如果看到命令行显示OK则表示成功
```

另外也可以直接关闭防火墙（不推荐）：

```
service iptables stop	//关闭防火墙
chkconfig iptables off	//设置开机不启动
```