---
title: Linux进程管理
date: 2018-2-26 16:14:00
comments: true
categories: [Linux]
tags: [Linux]
---

慕课网[Linux系统管理课程笔记（一）](https://www.imooc.com/learn/583)

<!-- more -->

## 查看进程

### PS

`ps` 命令常用选项：

- a:显示一个终端的所有进程
- u:显示进程的归属用户及内存的使用情况
- x:显示没有控制终端的进程

```
[root@JDu4e00u53f7 ~]# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  19236  1588 ?        Ss   Feb15   0:02 /sbin/init
root         2  0.0  0.0      0     0 ?        S    Feb15   0:00 [kthreadd]
...
root     26041  0.0  0.0 110248  1176 pts/0    R+   10:47   0:00 ps aux
daemon   32410  0.0  0.0  27352  1772 ?        S    Feb18   0:00 /usr/local/apache2/bin/httpd -k start
daemon   32411  0.0  0.0  27352  1784 ?        S    Feb18   0:00 /usr/local/apache2/bin/httpd -k start
daemon   32412  0.0  0.0  27352  1796 ?        S    Feb18   0:00 /usr/local/apache2/bin/httpd -k start
```

`ps` 命令输出字段的含义：

字段|含义
-|
USER|该进程是由哪个用户产生的
PID|进程的 ID 号
%CPU|进程占用 CPU 资源的百分比
%MEM|进程占用物理内存的百分比
VSZ|进程占用虚拟内存大小，单位 KB
RSS|进程占用实际物理内存大小，单位 KB
TTY|进程是在哪个终端中运行的，tty1-tty6 是本地的字符界面终端，tty7 是本地图形界面终端，pts/0-255 代表远程终端
START|进程的启动时间
TIME|进程占用 CPU 的运算时间
COMMAND|产生此进程的命令名

另外 `ps` 命令的输出结果中还有一个 `STAT` 字段，它表示进程的状态，常见的状态有以下几种：

STAT|含义
-|
R| 正在运行或可运行的进程（在运行队列中）
S| 处于休眠状态的可中断进程
D| 不可中断的休眠进程（通常为IO进程）
T| 停止或被追踪进程
Z| 僵尸进程
<| 优先级高的进程
N| 优先级低的进程
s| 包含子进程
+| 位于后台的进程
l| 多进程的


### pstree

`pstree` 命令显示进程树，可以看到 `init` 进程(/sbin/init)是所有进程的父进程

```
[root@JDu4e00u53f7 ~]# pstree
init─┬─AgentMonitor─┬─jcloudhids───25*[{jcloudhids}]
     │              ├─jcloudhidsupdat───{jcloudhidsupda}
     │              └─3*[{AgentMonitor}]
     ├─agetty
     ├─auditd───{auditd}
     ├─crond
     ├─dhclient
     ├─httpd───9*[httpd]
     ├─6*[mingetty]
     ├─mysqld_safe───mysqld───9*[{mysqld}]
     ├─nscd───6*[{nscd}]
     ├─ntpd
     ├─qemu-ga
     ├─rsyslogd───3*[{rsyslogd}]
     ├─sshd───sshd───bash───pstree
     ├─udevd───2*[udevd]
     └─xinetd

```

可以看到 `pstree` 命令会使用 `*` 表示重叠的子进程，使用 `-p` 选项可以显示所有进程的 PID，从而展开被折叠的进程，使用 `-u`选项可以显示进程的所属用户

### top

`top` 命令用于判断系统的健康状态，输出结果默认每 3 秒刷新一次，允许以交互模式输入命令，`top` 命令常用选项如下：

- d:秒数，指定 `top` 命令输出结果每隔几秒更新，默认是 3 秒
- b:使用批处理模式输出，一般和-n选项一起使用
- n:指定top命令的执行次数，一般和-b选项一个使用

`top` 命令交互模式中常用命令：

- ? 或 h:显示交互模式的帮助
- P:以 CPU 使用频率排序（默认选项）
- M:以内存使用率排序
- N:以 PID 排序
- q:退出 top 命令

```
top - 11:32:15 up 10 days, 17:22,  1 user,  load average: 0.77, 0.45, 0.25
Tasks:  88 total,   1 running,  87 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.0%us,  0.3%sy,  0.0%ni, 99.7%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   1922272k total,  1102484k used,   819788k free,    80884k buffers
Swap:        0k total,        0k used,        0k free,   709376k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                                                                                                          
 1321 mysql     20   0  499m  28m 6304 S  0.3  1.5   7:37.34 mysqld                                                                                                                            
11777 root      20   0  107m 3192 2540 S  0.3  0.2   7:33.12 AgentMonitor                                                                                                                      
27818 root      20   0 15024 1320 1012 R  0.3  0.1   0:01.14 top                                                                                                                               
    1 root      20   0 19236 1588 1296 S  0.0  0.1   0:02.70 init                                                                                                                              
...                                                                                                                     
   33 root      20   0     0    0    0 S  0.0  0.0   0:00.27 khungtaskd                                             
```

`top` 命令的第一行中有这样一行数据：`load average: 0.77, 0.45, 0.25`，表示系统在之前1分钟，5分钟，15分钟的平均负载，对于单核 CPU，这 3 个值都不应该超过1，否则表示系统已经超出负荷，同理对于多核系统则不应该大于其对应的核数

`top` 命令的第三行为 CPU 信息：

内容|说明
-|
us|用户模式占用的 CPU 百分比
sy|系统模式占用的 CPU 百分比
ni|改变过优先级的用户进程占用的 CPU 百分比
id|空闲 CPU 的 CPU 百分比
wa|等待输入/输出的进程占用的 CPU 百分比
hi|硬中断请求服务占用的 CPU 百分比
si|软中断请求服务占用的 CPU 百分比
st|当有虚拟机时，虚拟 CPU 等待实际 CPU 的时间百分比

`top` 命令的第四行和第五行分别表示物理内存和交换分区（虚拟内存）信息。这里需要区分开`buffer(缓冲)`和 `cache(缓存)`的区别：cache用于加速数据的读取，而buffer用于加速数据的写入，不要把二者混为一谈

你可能会发现 `top` 命令并不能列出所有的进程信息，此时可以结合 `-b` 和 `-n` 选项将 `top` 命令的输出重定向到文件中，就可以查看到所有进程的信息了：

```
top -b -n 1 > top.log
```

当然你可以把数字 1 更改为其他数字，表示执行指定次数的 `top` 命令：

```
top -b -n 3 > top.log
```

这条命令会在 6 秒钟后执行完毕，查看 top.log 文件，将看到 3 组输出结果

我觉得 `top` 命令算是一个非常有趣的命令，支持很多交互式操作，更多关于 `top` 命令的相关介绍可参考[How To Use The Linux Top Command To Show Running Processes
](https://www.lifewire.com/linux-top-command-2201163)

## 杀死进程

### kill

`kill -l` 列出系统中所有的信号名称，共计 64 个：

```
[root@JDu4e00u53f7 conf]# kill -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
...
63) SIGRTMAX-1	64) SIGRTMAX
```

常用信号说明(其他信号说明参考[Linux下信号种类以及特殊信号的含义](http://blog.csdn.net/yusiguyuan/article/details/43272225))：

代号|信号名称|说明
-|-|-
1|SIGHUP|该信号让进程立即关闭，重新读取配置文件后重启
2|SIGINT|程序终止信号，用于终止前台进程，相当于 Ctrl+C 快捷键
9|SIGKILL|用来立即结束程序的运行，本信号不能被阻塞、处理和忽略。一般用于强制终止进程
14|SIGALRM|时钟定时信号，计算的是实际的时间或时钟时间。alarm 函数使用该信号
15|SIGTERM|正常结束进程的信号，kill 命令的默认信号。但如果进程已经发生问题，该信号是无法正常终止进程的，所以才会有 SIGKILL，即信号 9 的存在
18|SIGCONT|该信号可以让暂停的进程恢复执行，该信号不能被阻断
19|SIGSTOP|该信号可以暂停前台进程，相当于 Ctrl+Z 快捷键，该信号不能被阻断

例如我现在需要重启 `httpd` 服务：

先使用 `ps` 命令查看进程 ID（当然 `pgrep` 命令也可以查看到进程ID，但是因为返回的信息太少，很可能找错进程 ID）：

![mark](http://imgblog.kuranado.com/blog/180226/Bi6i851IDH.png)

然后使用 `kill -1` 或 `kill -HUP` 命令重启进程，这样就能在修改了 `/usr/local/apache2/bin/conf/httpd.conf` 配置文件后重启服务使配置生效，同时不会让已连接到服务器的用户断开连接

```
[root@JDu4e00u53f7 ~]# kill -1 5740
[root@JDu4e00u53f7 ~]# kill -HUP 5740
```

而如果需要强制杀死进程则使用 `kill -9` 或 `kill -kill` 命令：

```
[root@JDu4e00u53f7 conf]# kill -kill 5740
[root@JDu4e00u53f7 conf]# kill -9 5740
```

### killall

很明显 `kill` 命令因为一次只能杀死一个进程，对于源码包安装的 `httpd` 服务，一启动就会默认产生 6 个进程，那如何才能一次性杀死全部 `httpd` 服务相关进程呢？这时候就需要 `killall` 登场了：

```
[root@JDu4e00u53f7 conf]# pgrep httpd
6683
6684
6685
6686
6687
6688
[root@JDu4e00u53f7 conf]# killall httpd
[root@JDu4e00u53f7 conf]# pgrep httpd
[root@JDu4e00u53f7 conf]# 
```

`killall` 常用选项：

- i:交互式询问是否要杀死某个进程
- I:忽略进程名的大小写

### pkill

和 `killall` 一样，`pkill` 命令同样可以一次性杀死一组进程，用法和 `killall` 完全相同，但 `pkill` 命令还支持 `-9 -t` 选项用于按**终端号踢出其它登录用户**：

首先我使用 Xshell 远程登录两个 root 用户，然后在京东云控制台点击远程连接本地登录一个 root 用户：

![mark](http://imgblog.kuranado.com/blog/180226/7gg8aBbeai.png)

之后在其中一个 Xshell 窗口中执行 `w` 命令：

```
[root@JDu4e00u53f7 conf]# w
 15:56:28 up 10 days, 21:46,  3 users,  load average: 0.14, 0.20, 0.17
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
root     tty1     -                15:56    6.00s  0.02s  0.02s -bash
root     pts/0    116.231.145.42   11:21    0.00s  0.31s  0.00s w
root     pts/1    116.231.145.42   15:55   43.00s  0.02s  0.02s -bash
```

我们知道tty1-tty6 是本地的字符界面终端，tty7 是本地图形界面终端，而pts/0-255 代表远程终端，所以我们现在可以看到有一个 root 用户通过本地登录，另外两个用户通过远程终端登录，此时通过 `pkill` 命令加上 `-9 -t` 选项就可以根据终端号踢出已登录用户：

```
[root@JDu4e00u53f7 conf]# pkill -9 -t tty1
[root@JDu4e00u53f7 conf]# pkill -9 -t pts/1
[root@JDu4e00u53f7 conf]# w
 16:09:58 up 10 days, 22:00,  1 user,  load average: 0.12, 0.16, 0.15
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    116.231.145.42   11:21    0.00s  0.35s  0.00s w
[root@JDu4e00u53f7 conf]# 
```

注意选项 `-9 -t`的顺序不可以颠倒，因为`-t -9` 并不能踢出已登录用户

## 修改进程优先级

使用 `ps -le` 命令查看进程：

```
[root@JDu4e00u53f7 conf]# ps -le
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0     1     0  0  80   0 -  4809 poll_s ?        00:00:02 init
1 S     0     2     0  0  80   0 -     0 kthrea ?        00:00:00 kthreadd
1 S     0     3     2  0 -40   - -     0 migrat ?        00:00:00 migration/0
1 S     0     4     2  0  80   0 -     0 ksofti ?        00:00:02 ksoftirqd/0
1 S     0     5     2  0 -40   - -     0 cpu_st ?        00:00:00 stopper/0
...
4 S     0 27530 27528  0  80   0 - 27106 wait   pts/0    00:00:00 bash
```

通过修改 `NI` 字段的值，可以改变进程的优先级，在修改 `NI` 字段值之前，先来了解几个关于进程优先级的基本规则：

- 系统根据 PRI 值的大小决定进程优先级，PRI 值越小，进程优先级越高，将越先被 CPU 执行
- 用户只能修改 NI 值，不能直接修改 PRI 值
- PRI（最终值） = PRI（原始值） + NI
- NI 的值的范围是[-20,19]
- 普通用户调整 NI 值的范围是[0,19]，而且只能调整自己的进程
- 普通用户只能调高 NI 值，而不能降低 NI 值，如原本 NI 值为 0，则只能调整为大于0
- root 用户才能设定 NI 值为负值，而且可以调整任何用户的进程
- 闲着没事不要随便乱改进程优先级，因为 CPU 的运算速度很快，即使改了也很难看出变化

### nice

`nice` 命令语法格式：

```
nice [选项] 命令
```
`nice` 命令可以给新执行的命令直接赋予 NI 值，但是却不能修改已经存在的进程的 NI 值

常用选项：

- n:给命令赋予NI值

```
[root@JDu4e00u53f7 conf]# ps -le | grep httpd
1 S     0  9325     1  0  80   0 -  6804 poll_s ?        00:00:00 httpd
5 S     2  9326  9325  0  80   0 -  6804 inet_c ?        00:00:00 httpd
5 S     2  9327  9325  0  80   0 -  6804 inet_c ?        00:00:00 httpd
5 S     2  9328  9325  0  80   0 -  6804 inet_c ?        00:00:00 httpd
5 S     2  9329  9325  0  80   0 -  6804 inet_c ?        00:00:00 httpd
5 S     2  9330  9325  0  80   0 -  6804 inet_c ?        00:00:00 httpd
[root@JDu4e00u53f7 conf]# service apachectl stop  # 必须要先停止服务，因为nice命令不能直接修改已经存在的进程的 NI 值
[root@JDu4e00u53f7 conf]# nice -n -5 service apachectl start  # 修改 httpd 服务相关进程的 NI 值为 -5
[root@JDu4e00u53f7 conf]# ps -le | grep httpd
1 S     0  9398     1  0  75  -5 -  6804 poll_s ?        00:00:00 httpd
5 S     2  9399  9398  0  75  -5 -  6804 inet_c ?        00:00:00 httpd
5 S     2  9400  9398  0  75  -5 -  6804 inet_c ?        00:00:00 httpd
5 S     2  9401  9398  0  75  -5 -  6804 inet_c ?        00:00:00 httpd
5 S     2  9402  9398  0  75  -5 -  6804 inet_c ?        00:00:00 httpd
5 S     2  9403  9398  0  75  -5 -  6804 inet_c ?        00:00:00 httpd
```

可以看到进程的 PRI 值都已经由 80 变成了 75（80 - 5 = 75），成功提高了进程的优先级

### renice

`renice` 命令语法格式：

```
renice [NI 值] PID
```

相比较 `nice`  命令， `renice` 命令的优点在于其可以修改已经存在的进程的 NI 值，这样我们就省去了先关闭进程，再修改 NI 值的麻烦。继续上面的例子：

```
[root@JDu4e00u53f7 conf]# renice -10 9400
9400: old priority -5, new priority -10
[root@JDu4e00u53f7 conf]# ps -le | grep httpd
1 S     0  9398     1  0  75  -5 -  6804 poll_s ?        00:00:00 httpd
5 S     2  9399  9398  0  75  -5 -  6804 inet_c ?        00:00:00 httpd
5 S     2  9400  9398  0  70 -10 -  6804 inet_c ?        00:00:00 httpd
5 S     2  9401  9398  0  75  -5 -  6804 inet_c ?        00:00:00 httpd
5 S     2  9402  9398  0  75  -5 -  6804 inet_c ?        00:00:00 httpd
5 S     2  9403  9398  0  75  -5 -  6804 inet_c ?        00:00:00 httpd
```
看到进程 ID 为 9400 的进程的 PRI 值由原来的 80 变成了 70 （80 - 10 = 70）。