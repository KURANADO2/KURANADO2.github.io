---
title: Linux后台运行进程
date: 2018-01-30 09:40:00
comments: true
categories: [Linux]
tags: [Linux]
---

当执行一个耗时任务时，为了能够继续做其它事情，你可能会需要重新打开一个终端窗口，又或者是，你需要在服务器挂上一个时刻运行的脚本，你不希望自己退出登录或关闭终端时，这个脚本就停止运行了。为了解决这些问题，可以通过Linux命令把进程放到后台执行，这样我们就可以在当前终端窗口继续工作而不受耗时任务的影响，当然需要时，你也可把后台进程重新拿到前台执行。

<!-- more -->

本篇文章主要参考：
[How to run commands in the background in Linux](https://linuxaria.com/article/how-to-run-commands-background-nohup-disown)
[Running Bash Commands in the Background the Right Way](https://www.maketecheasier.com/run-bash-commands-background-linux/)

## &

通过在所要执行的任务最后加上 `&` 参数，就可以把这个进程放到后台执行，下面这条命令如果没有加上 `&` 参数，你需要等待 100 秒后才能继续在终端输入命令

```
sleep 100 &
```

![mark](http://imgblog.kuranado.com/blog/180129/HJ6e34Chhh.png)

为了确定进程是不是真的在后台运行着，你可以使用 `ps aux | grep sleep` 查看，但有种更简便的方法是使用 `jobs` 命令，它将列出所有正在后台执行的任务

![mark](http://imgblog.kuranado.com/blog/180129/jiD8mKAeE7.png)

可以发现正在后台运行的任务，其状态显示为 `Running`，并且前面有一个编号 `[1]`,这代表后台任务的编号，如果连续放了几个任务到后台运行，它们的编号则会依次递增。

好了，过了 100 秒之后，再次执行 `jobs` 命令，发现刚刚的任务已经完成了

![mark](http://imgblog.kuranado.com/blog/180129/JI7JD943ee.png)

如果你忘记在命令后加 `&` 参数也没关系，这时候，只需要按 `Ctrl+Z`键，即可将该进程暂停（其状态为 `Stopped` ）然后输入 `bg` 命令，即可将该进程放到后台继续运行

![mark](http://imgblog.kuranado.com/blog/180129/ElhBDhbig2.png)

## nohup

事实上 `nohup` 命令并不能把进程放到后台执行，要想把进程放到后台，你仍然需要加上 `&` 参数或者按 `Ctrl+Z` 后输入 `bg` 命令，而`nohup`命令可以帮我们处理输出问题，看下面这个Python脚本：

```
#/usr/bin/env python3

import time

count = 0
while count != 100:
	print('Hello! %d' % count)
	time.sleep(1)
	count += 1
```

这段脚本每隔 1 秒钟打印输出一次‘Hello!’，打印 100 次后退出，像上面一样，在命令末尾加上 `&` 参数：

```
python nohuptest.py &
```

![mark](http://imgblog.kuranado.com/blog/180129/K33dC5miJ4.gif)

可以看到，虽然脚本已经被放到了后台运行，但其打印的内容，仍然会占据终端窗口，解决办法就是使用 `nohup` 命令：

```
nohup python nohuptest.py &
```

使用 `jobs` 命令看到脚本已经在后台运行了，而且终端窗口也没有继续打印输出，此时在当前目录可以找到一个 `nohup.out` 文件，这是调用 `nohup` 命令时系统自动为我们创建的文件，打开该文件发现脚本打印的内容都已经存到这个文件里了，你也可以像下面这样指定文件名称：

```
nohup python nohuptest.py > nohuptest.txt &
```

这样打印输出的内容就会被存储在 `nohuptest.out` 文件当中，且系统不会帮我们创建默认的 `nohup.out` 文件，如果不想把输出结果保存到文件，则使用下面的命令：

```
nohup python nohuptest.py > /dev/null &
```

## fg

`fg` 命令用于把正在后台运行的进程拿到前台执行，把对应的编号传给 `fg` 就可以了，编号可使用 `jobs` 命令查看：

```
# 把编号为1的后台进程拿到前台执行
fg 1
```