---
title:	Shadowsocks+VPS实现科学上网
date: 2017-10-16 00:20:32
comments: true
categories: [Shadowsocks]
tags: [Shadowsocks, VPS, DigitalOcean]
---

一直在用天路加速器，但最近YouTube播放视频非常卡，速度明显大大不如以前，登录官网切换服务IP却发现官网已经上不去了，这难不成真像学长说的那样是作者跑路了吧，心疼我的软妹币啊，让我先哭一会......于是，索性自己买了DigitalOcean的云主机折腾一下，做一个记录分享

<!-- more -->

## VPS

> 所谓VPS(Virtual Private Server)即虚拟专用服务器，通过购买海外的VPS使我们能够流畅的访问国外的网站，诸多的VPS提供商中本来打算用搬瓦工，既便宜又有很多的中文介绍方便选购，最便宜的KVM架构一年也只需要29.99美元，非常适合学生党，但手慢无发现目前已是out of stock，不得已我选择了DigitalOcean，后来发现DigitalOcean性价比也挺高的，不仅所有的主机都是KVM架构，而且随时可以更换数据中心，商家的口碑也很好。说DigitalOcean性价比高其实并不是因为它便宜，相对于搬瓦工来说DigitalOcean还是非常昂贵的，其中最便宜的主机一个月都需要5美元，但是如果通过别人的邀请链接注册账户就可以获得10刀的账户余额，再加上GitHub上验证学生身份后可以再增加50刀的余额，这样总共就可以拿到60美元的余额，刚好可以用一年，单单安装Shadowsocks用来~~看小电影~~(▰˘◡˘▰)科学上网或放点其他的挂机程序是不是很爽!

### 1. 注册DigitalOcean

通过别人的邀请链接注册DigitalOcean账号，注意这里一定要通过别人的邀请链接注册，这样注册成功后才能够获得10美元的账户余额，如果直接访问官网注册，注册成功后是不会有那10美元的哦！

这是官网地址：[https://www.digitalocean.com/](https://www.digitalocean.com/)
这是我的邀请链接：[https://m.do.co/c/74f5c2db875c](https://m.do.co/c/74f5c2db875c)

该点哪一个都清楚了吧！

当然小伙伴们也可以到网上随便找一个邀请链接注册，效果都是一样的，只不过点我的链接可以帮我省点早饭钱。

注册的过程非常简单，稍需提醒的是注册时需先支付5美元以防账户欺诈，放心支付即可，支付的方式有两种，一个是使用信用卡，但通常因为有可能被盗刷所以不建议，另一种方式是使用PayPal支付，推荐使用这种方式支付。没有PayPal账号的同学最好注册一个，会经常用到。当然这5美元会被放到自己的账户余额中，也就是说邀请链接10刀+学生礼包50刀+自己支付的5刀，账户中最后将会有65刀的余额。PayPal支付成功后，工作人员会审核你的账户，我的是晚上10点注册，第二天早上6点才审核通过，所以需要耐心等待。

### 2. Create Droplet

账户审核通过后，我们点击Create>Droplets，选择云主机配置及数据中心:

![mark](http://imgblog.kuranado.com/blog/171015/Ed4Ji36Eem.png)

[Shadowsocks使用说明](https://github.com/shadowsocks/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8
%E8%AF%B4%E6%98%8E)建议选择Ubuntu系统，所以我们这里也选择Ubuntu，配置我们选择最低配置足够，土豪随意。另外关于这里的`$5/mo $0.007/hour`是指如果创建了一个Droplet则按照0.007美元每小时收费，一个月合计5美元，如果销毁这个Droplet则不继续计费，同理如果创建多个Droplet则每个都将按照0.007每小时收费。所以如果打算长时间不用可以销毁Droplet,用时再重新配置即可，前提是你不嫌折腾，服务器里也没有重要的数据仅仅是为了翻墙上网。

![mark](http://imgblog.kuranado.com/blog/171015/1AlLB5F3E2.png)
![mark](http://imgblog.kuranado.com/blog/171015/kh0AiaALkH.png)

至于该选择那个数据中心，我们可以访问[各数据中心测速页面](http://speedtest-fra1.digitalocean.com/)根据自己实际情况选择。

之后点击Create按钮即创建成功！

## Shadowsocks

> Shadowsocks是由中国的[clowwindy](https://twitter.com/clowwindy?utm_source=textarea.com&utm_medium=textarea.com&utm_campaign=article)开发的，因为使用自定义的代理协议，流量特征不明显，所以不容易被GFW拦截

### 1. Ubuntu服务器安装Shadowsocks

```
apt-get update	//更新源 包列表
apt-get install python-pip	//安装pip
pip install --upgrade pip	//更新pip
apt-get install git
pip install git+https://github.com/shadowsocks/shadowsocks.git@master	//安装Shadowsocks
ssserver --version	//查看安装的Shadowsocks版本号
```

在/etc/目录下创建Shadowsocks配置文件：shadowsocks.json

```
touch /etc/shadowsocks.json
```

配置文件做如下配置：

```
{
 "server":"104.131.137.71",
 "server_port":8388,
 "local_address": "127.0.0.1",
 "local_port":1080, 
 "password":"yourpassword",
 "timeout":300, 
 "method":"aes-256-cfb", 
 "fast_open": false
}
```

server填写自己的服务器IP，server_port端口号不要冲突就行，password为你在Windows或Mac上的Shadowsocks连接需要的密码，关于method加密方式填aes-256-cfb，这也是默认值，更多加密方式参考[Supported Ciphers](https://github.com/shadowsocks/shadowsocks/wiki/Encryption)

关于各参数详细含义参考[Configuration via Config File](https://github.com/shadowsocks/shadowsocks/wiki/Configuration-via-Config-File)

配置完成后，启动Shadowsocks：

```
ssserver -c /etc/shadowsocks.json -d start	//启动Ss
ssserver -c /etc/shadowsocks.json -d stop	//关闭Ss（一般不需要关闭）
```

配置文件为`/var/log/shadowsocks.log`，可通过`tail -f 50 /var/log/shadowsocks.log`实时查看

### 2. 各平台下载安装Shadowsocks

- [Mac](https://github.com/shadowsocks/ShadowsocksX-NG/releases?utm_source=textarea.com&utm_medium=textarea.com&utm_campaign=article)
- [Windows](https://github.com/shadowsocks/shadowsocks-windows/releases)
- [Android](https://github.com/shadowsocks/shadowsocks-android/releases?utm_source=textarea.com&utm_medium=textarea.com&utm_campaign=article)

直接运行填写服务器配置即可：

![mark](http://imgblog.kuranado.com/blog/171015/IglE1hl6Il.png)

勾选对应服务器，启用系统代理后就可以流畅的访问Google、Youtube了，最重要的是速！度！快！便！宜！几！乎！免！费！

## GitHub学生教育礼包

**最后关于GitHub学生礼包请点击这里：**[https://education.github.com/pack](https://education.github.com/pack)

![mark](http://imgblog.kuranado.com/blog/171015/kfBhhIjhHH.png)

可以看到这个页面的所有礼包只要验证了学生资格就都可以获取，其中就包括DigitalOcean的50刀（其实以前是$100，现在只有50了）。通过学生验证需要你提供一个.edu结尾的教育邮箱，如果没有.edu的邮箱只要上传像学生证之类能够证明你是在校学生的照片（需满足13岁以上）也是可以的，只不过前一种方式会更快通过审核，后一种方式稍慢一点，其实博主所在的大学也是不为学生提供edu邮箱的，所以我上传了自己的学生证和成绩表照片，然后描述了下自己的实际情况，大概十几个小时之后便收到了GitHub发来的邮件告知我审核已通过，成功拿到了丰厚的教育礼包，点击其中的DigitalOcean会获得一个优惠码，登录自己的DigitalOcean账号后，选择Settings>Billing>Promo code，填入优惠码后就可以成功增加50刀的余额了。

![mark](http://imgblog.kuranado.com/blog/171017/JJBHFh5fme.png)


## 总结

整个搭建过程还是非常简单的，如果小伙伴们遇到问题实在解决不了欢迎评论留言交流或尝试一键搭建：[http://www.52yunwei.net/620.html](http://www.52yunwei.net/620.html)