---
title:	BBR实现TCP加速
date: 2017-10-26 20:23:00
comments: true
categories: [Shadowsocks]
tags: [Shadowsocks, Ubuntu, BBR, DigitalOcean, VPS]
---

自上篇文章[Shadowsocks+VPS实现科学上网](http://www.kuranado.com/2017/10/16/Shadowsocks+VPS%E5%AE%9E%E7%8E%B0%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91/)之后，发现速度上还是不太理想，播放视频打开`详细统计信息`发现速度只有800K左右，于是寻找提速方案，发现方法有很多，本篇文章使用的是BBR加速，最终效果可以提速到2500K左右，对于博主的20兆宽带来说算是不错的结果了

<!-- more -->

## 使用BBR加速前提

关于什么是BBR可以看这篇论文：[BBR: Congestion-Based Congestion Control](https://queue.acm.org/detail.cfm?id=3022184)，英文不太好可以看这个：[Linux Kernel 4.9 中的 BBR 算法与之前的 TCP 拥塞控制相比有什么优势？](https://www.zhihu.com/question/53559433)，大概就是在有一定丢包率的情况下BBR算法可以抢占更多的公网带宽从而实现加速效果，而要使用BBR实现加速需要满足以下条件：

- VPS必须是KVM架构
- 系统内核>=4.9

## 查看VPS是否是KVM架构

因为DigitalOcean的主机全都是KVM架构的，所以使用DO主机的小伙伴可以省略这一步，如果不确定自己所购买的主机到底是OpenVZ还是KVM可以使用如下命令查看：

Ubuntu：

```
apt-get install virt-what
virt-what
```

同理CentOS：

```
yum install virt-what
virt-what
```

## 查看内核版本

使用如下命令查看内核版本号，如果版本号大于等于4.9则可以直接开启BBR加速，否则需要先升级内核版本

```
uname -r	//只显示内核版本号
```

或

```
uname -a	//显示内核版本号和其他信息
```

## 升级内核版本

如果版本号大于等于4.9则可以省略此步直接开启BBR加速，否则需要先升级内核版本

### 下载内核

```
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.13/linux-image-4.13.0-041300-generic_4.13.0-041300.201709031731_i386.deb
```

注意上面下载的版本是4.13，如果要下载其他版本，请查看：[http://kernel.ubuntu.com/~kernel-ppa/mainline](http://kernel.ubuntu.com/~kernel-ppa/mainline) ，然后选择想要的版本即可，另外选择好版本后还需要根据主机的CPU架构选择对应的包，我这里因为CPU是i386架构所以我选择'linux-image-4.13.0-041300-generic_4.13.0-041300.201709031731_i386.deb'，如下图，最终把url拼在一块使用`wget`命令下载即可

![mark](http://imgblog.kuranado.com/blog/171026/e0ceiDh356.png)

### 安装

```
dpkg -i linux-image-4.13.0-041300-generic_4.13.0-041300.201709031731_i386.deb	//安装
update-grub		//更新引导文件
reboot		//重启
uname -r	//查看内核版本发现已经升级到4.13
```

## 开启BBR加速

```
//修改系统配置
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
//生效配置
sysctl -p
//下面这条命令执行结果如果为：net.ipv4.tcp_available_congestion_control = bbr cubic reno则配置成功生效
sysctl net.ipv4.tcp_available_congestion_control
```

---

**更多加速方法参考：[https://doub.io/ss-jc26/#三、优化Shadowsocks](https://doub.io/ss-jc26/#三、优化Shadowsocks)**
