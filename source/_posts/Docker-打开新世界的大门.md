---
title: Docker-打开新世界的大门
date: 2018-06-12 00:07:00
comments: true
categories: [Docker]
tags: [Docker]
---

![mark](http://imgblog.kuranado.com/blog/180611/bCb1Ed16Al.png)

<!-- more -->

记得去年在 QQ 上[学长](http://mrdear.cn/)提到过 Docker 才是自动化部署的未来，但当时对 Docker 是闻所未闻，只觉得这是技术大牛才会用到的东西吧。作为小白最近在混进的几个大牛群里经常有人提到 Docker 相关问题，才知道 Docker 是现在非常流行的一项技术，所以拨开云层，通过学习 Docker 这一技术，为今后的项目部署提供一种高效的解决方案，也是提升自身技术实力的一种实现方式。

本篇博客主要参考[Docker 从入门到实践](https://legacy.gitbook.com/book/yeasy/docker_practice/details)，虽然其中讲到的很多概念对 Linux 小白来说不太好理解，但选择性跳过并不影响学习 Docker 的基本操作，强烈推荐小伙伴们阅读。

本文使用 CentOS 7 安装 Docker 18.05.0-ce 版进行测试。

## 什么是 Docker

Linux 容器是一种对进程进行封装隔离，属于操作系统层面虚拟化技术。因为隔离的进程独立于宿主机（Host）和其他隔离的进程，所以将其称之为容器。而 Docker 直译是码头工人或容器的意思，使用 Google 公司推出的 Go 语言实现，Docker 在 Linux 容器的基础上又做了进一步的封装，从文件系统、网络互连到进程隔离等，极大的简化了 Linux 容器的创建和维护，使得 Docker 技术比传统虚拟机技术更为轻便、快捷。

## 为什么使用 Docker

- Docker 容器应用直接运行于宿主内核，无需启动完整的操作系统，所以应用启动的时间可以达到秒级甚至毫秒级。
- 容器不需要进行硬件虚拟及运行完整操作系统等额外开销，所以 Docker 对系统资源的利用率更高，比传统虚拟机技术更高效。
- 容器只打包其运行所需要的组件即可，所以体积相比虚拟机更小。
- Docker 可以确保运行环境的一致性，使得在一个平台上运行的应用很方便的迁移到另一个平台上，不需要担心运行环境的变化导致应用无法正常运行。

## Docker 安装

Linux 安装 Docker 过程十分简单，可参考以下链接：

[CentOS 安装 Docker 社区版](https://yeasy.gitbooks.io/docker_practice/content/install/centos.html)
[Docker 镜像加速器配置](https://yeasy.gitbooks.io/docker_practice/content/install/mirror.html)

需要注意的是 CentOS 安装 Docker 需要 CentOS 7 或其以上版本才能够使用 Docker。

## Registry

了解 Maven 的都知道在 Maven 的 pom.xml 文件中定义依赖坐标后，Maven 会首先到本地仓库查找是否有对应的依赖，如果没有再从远程仓库中在线下载所需的依赖。和 Maven 类似的是，Docker 也存在本地或远程下载镜像的服务，这就是 Docker Registry。只不过 Docker Registry 的名称定义和 Maven 略有不同。

Docker 中一个 Docker Registry 可以包含多个 Repository（仓库），而每个仓库又可以包含多个 TAG（标签），每个 TAG 对应一个 Docker 镜像。比如官方的 Docker Registry 地址为：[Docker Hub](https://hub.docker.com/)，点击进入该 Docker Registry，以 `Nginx` 作为关键词搜索 Nginx 相关仓库结果如下：

![mark](http://imgblog.kuranado.com/blog/180611/3i8g51IEFf.png)

搜索结果中第一个 nginx 仓库为 Nginx 官方上传的仓库，下面的仓库都是其他开发者上传的，仓库名格式为两段式，类似于 `jing/nginx` 这样的格式，但并不绝对。拿 nginx 官方仓库为例，点击进入该仓库，并切换到 Tags 标签：

![mark](http://imgblog.kuranado.com/blog/180611/Je9baJc6KF.png)

可以看到同一个 nginx 仓库下包含多个 TAG，每一个 TAG 对应一个 Docker 镜像。

## Image 镜像

Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置文件外，还包含了一些为运行时准备的一些配置参数。镜像本身并不包含任何动态数据，其内容在构建之后也不会被改变。Docker 镜像采用`分层存储`架构设计，所以 Docker 镜像由多层文件系统联合组成。正是由于 Docker 的`分层存储`，是的镜像的复用、定制变得非常容易。

### 拉取镜像

拉取 Docker 镜像的命令格式为（本文中括号`[]`表示可选值）

```
docker pull [option] [Docker Registry Address[:port]/]仓库名[:TAG]
```

默认拉取镜像地址为 Docker Hub，可以不填。因为国内下载 Docker Hub 镜像速度较慢，所以安装 Docker 以后最好配置镜像加速器，这样 `docker pull` 命令会自动从配置的加速器地址拉取镜像

以拉取 Nginx 镜像为例：

```
[JING@localhost mynginx]$ docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
f2aa67a397c4: Pull complete 
1cd0975d4f45: Pull complete 
72fd2d3be09a: Pull complete 
Digest: sha256:3e2ffcf0edca2a4e9b24ca442d227baea7b7f0e33ad654ef1eb806fbd9bedcf0
Status: Downloaded newer image for nginx:latest
```

此处省略了 Docker Registry Address:port，则默认从 `https://hub.docker.com:80` 地址拉取镜像，若配置了镜像加速器，比如我在 `/etc/docker/daemon.json` 文件中配置镜像加速器地址为：`https://registry.docker-cn.com`，则默认从 `https://registry.docker-cn.com:80` 地址拉取 Nginx 镜像；当然也可以手动指定 Docker Registry 地址：

```
docker pull hub.c.163.com/library/nginx:latest
```

另外例子中省略了 TAG，会使用 `latest` 作为默认 TAG，`latest` 表示最新版镜像，所以上面的例子的执行效果和和下面这条命令的效果完全相同：

```
docker pull nginx:latest
```

当然也可以通过 TAG 指定拉取哪个版本的镜像：

```
docker pull nginx:1.14
```

### 列出镜像

`docker imaegs(或 docker image ls)` 可以查看本地的所有`顶层镜像`：

```
[JING@localhost mynginx]$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              16.04               5e8b97a2a082        5 days ago          114MB
nginx               latest              cd5239a0906a        5 days ago          109MB
redis               alpine              494c839f5bb5        2 weeks ago         27.8MB
nginx               stable              f759510436c8        4 weeks ago         109MB
hello-world         latest              e38bc07ac18e        2 months ago        1.85kB
```

各字段含义：

- REPOSITORY 镜像仓库名称
- TAG 标签
- IMAGE ID 镜像 ID
- CREATED 镜像创建时间
- SIZE 镜像大小（细心的小伙伴可能已经发现本地镜像的体积比在 Docker Hub 上要大不少，这是因为 Docker Hub 上的镜像是经过压缩的，拉取到本地后，镜像会被展开，所以体积会稍大一点）

列出部分镜像：

```
[JING@localhost mynginx]$ docker images nginx
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              cd5239a0906a        5 days ago          109MB
nginx               stable              f759510436c8        4 weeks ago         109MB
```

列出指定镜像：

```
[JING@localhost mynginx]$ docker images nginx:stable
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               stable              f759510436c8        4 weeks ago         109MB
```

只列出镜像 id：

```
[JING@localhost mynginx]$ docker images -q
5e8b97a2a082
cd5239a0906a
494c839f5bb5
f759510436c8
e38bc07ac18e
```

列出镜像摘要：

```
[JING@localhost mynginx]$ docker images --digests
REPOSITORY          TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
ubuntu              16.04               sha256:b050c1822d37a4463c01ceda24d0fc4c679b0dd3c43e742730e2884d3c582e3a   5e8b97a2a082        5 days ago          114MB
nginx               latest              sha256:3e2ffcf0edca2a4e9b24ca442d227baea7b7f0e33ad654ef1eb806fbd9bedcf0   cd5239a0906a        5 days ago          109MB
redis               alpine              sha256:8a2bec06c9f2aa9105b45f04792729452fc582cf90d8540a14543687e0a63ea0   494c839f5bb5        2 weeks ago         27.8MB
nginx               stable              sha256:85ab7c44474df01422fe8fdbf9c28e497df427e8a67ce6d47ba027c49be4bdc6   f759510436c8        4 weeks ago         109MB
hello-world         latest              sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77   e38bc07ac18e        2 months ago        1.85kB
```

列出在 nginx:stable 镜像之前创建的镜像：

```
[JING@localhost mynginx]$ docker images -f before=nginx:stable
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              e38bc07ac18e        2 months ago        1.85kB
```

列出在 nginx:stable 镜像之后创建的镜像：

```
[JING@localhost mynginx]$ docker images -f since=nginx:stable
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              16.04               5e8b97a2a082        5 days ago          114MB
nginx               latest              cd5239a0906a        5 days ago          109MB
redis               alpine              494c839f5bb5        2 weeks ago         27.8MB
```



### 删除镜像

删除镜像的命令格式为：

```
docker image rm [选项] <镜像1> [<镜像2> ...]
```

通过镜像 id 删除镜像，因为镜像 id 很长，只要取能够唯一区别本地其他镜像的前几位即可，一般取前三位：

先查看镜像：

```
[JING@localhost mynginx]$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              16.04               5e8b97a2a082        5 days ago          114MB
nginx               latest              cd5239a0906a        5 days ago          109MB
redis               alpine              494c839f5bb5        2 weeks ago         27.8MB
nginx               stable              f759510436c8        4 weeks ago         109MB
hello-world         latest              e38bc07ac18e        2 months ago        1.85kB
```

通过镜像 id 删除 redis:alpine 镜像：

```
[JING@localhost mynginx]$ docker image rm 494
Untagged: redis:alpine
Deleted: sha256:494c839f5bb5d9e4f7b50b096c6b317e0ac9114155858e48acf0dd9bfe93ef7c
Deleted: sha256:dafb9d2e1d4b58bc1e70fbc7f79f7866eb6d4222db6a9e351984d19d89279da0
Deleted: sha256:ab5db7501e74998d48c587be367949667e384190c5326903d74194dbc66d5047
Deleted: sha256:4b63248f36687f4a40b22daeb3eb6ee2156cc7f41204fdf3e730d42822b94b97
Deleted: sha256:053a9dfd3a9dfaec430cdf7725e9a41be894266ac9439e68c5dd7322630605f4
Deleted: sha256:e9a28fc52cd61f76c44c469c488f95306f89f741ef51e173ac7aadb68e29fd52
Deleted: sha256:cd7100a72410606589a54b932cabd804a17f9ae5b42a1882bd56d263e02b6215
```

通过镜像名称删除镜像：

```
[JING@localhost mynginx]$ docker image rm hello-world
Untagged: hello-world:latest
Untagged: hello-world@sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77
Deleted: sha256:e38bc07ac18ee64e6d59cf2eafcdddf9cec2364dfe129fe0af75f1b0194e0c96
Deleted: sha256:2b8cbd0846c5aeaa7265323e7cf085779eaf244ccbdd982c4931aef9be0d2faf
```

通过镜像摘要删除镜像，如：

```
[JING@localhost mynginx]$ docker image rm ubuntu@sha256:b050c1822d37a4463c01ceda24d0fc4c679b0dd3c43e742730e2884d3c582e3a
Untagged: ubuntu@sha256:b050c1822d37a4463c01ceda24d0fc4c679b0dd3c43e742730e2884d3c582e3a
```

删除所有仓库名为 redis 的镜像：

```
docker image rm $(docker images -q reids)
```

细心的小伙伴会发现执行删除镜像操作时终端会打印 `Untagged` 或 `Deleted`，Untagged 表示取消该镜像的标签，Deleted 表示真正删除该镜像

## Container 容器

Docker 容器是 Docker 镜像运行时的实体，容器可以被创建、启动、暂停、停止、删除等。容器的实质是进程，但与宿主机（Host）中的进程不同，容器进程拥有属于自己的独立的命名空间，因此容器可以拥有自己的 root 文件系统，自己的网络配置等。

上面提到镜像采用`分层存储`设计，容器同样也是如此。容器运行时需要以镜像作为基础层，在基础层上创建一个当前容器的存储层，用于容器运行时进行读写操作，我们称该存储层为`容器存储层`。需要注意的是，`容器存储层`的生命周期和容器的生命周期相同，也就是说删除容器后，`容器存储层`也将会被随之删除。既然容器存储层不能持久化数据，必然会存在其他解决方案，`Volume（数据卷）`就是用于解决数据持久化而存在的，关于 `数据卷` 以后的文章中会提到，这里暂时跳过。

### 启动容器

`docker run` 命令用于基于镜像创建容器，执行该命令实际 Docker 在后台所做的一系列操作过程如下：

1. 检查本地是否存在指定的镜像，不存在则从共有仓库下载
2. 利用镜像创建并启动一个容器
3. 分配一个文件项系统，并在只读的镜像层外面挂载一层可读写层
4. 从宿主机（Host）配置的网桥接口中桥接一个虚拟接口道容器中去
5. 从地址池配置一个 ip 地址给容器执行用户指定的应用程序
6. 执行完毕后容器被终止

#### 基于镜像新建容器并启动容器

基于 ubuntu:16.04 镜像启动一个容器，用于使用Ubuntu 16.04 的 echo 命令输出 “Hello Docker”：

```
[JING@localhost mynginx]$ docker run ubuntu:16.04 /bin/echo 'Hello Docker'
Hello Docker
```

成功输出  ”Hello Docker“ 后，容器会被自动终止

基于 ubuntu:16.04 镜像启动一个 bash 终端容器，这样容器成功启动后，可在 bash 中输入 Linux 命令：

```
[JING@localhost mynginx]$ docker run -t -i ubuntu:16.04 /bin/bash
root@3fe29326c786:/# whoami
root
root@3fe29326c786:/# cat /etc/issue
Ubuntu 16.04.4 LTS \n \l
root@3fe29326c786:/# exit
exit
[JING@localhost mynginx]$ 
```

选项含义如下：

- t 表示让 Docker 分配一个伪终端并绑定到标准输入上
- i 让 Docker 容器的标准输入保持打开

#### 启动已终止容器

启动已终止容器使用 `docker container start [container id]` 实现：

```
docker container start 3fe
```

#### 重新启动运行中的容器

```
docker container restart 3fe
```

### 列出容器

查看运行中的容器：

```
[JING@localhost mynginx]$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

查看所有容器，包括已终止的进程：

```
[JING@localhost mynginx]$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
3fe29326c786        ubuntu:16.04        "/bin/bash"              14 minutes ago      Exited (1) 8 minutes ago                        optimistic_chandrasekhar
be52bb583d7b        ubuntu:16.04        "/bin/echo 'Hello Do…"   20 minutes ago      Exited (0) 20 minutes ago                       heuristic_wright
```

### 后台运行容器

在后台运行容器和 Linux 的守护进程类似：

前台运行容器，打印的内容会直接输出到宿主机上：

```
[JING@localhost mynginx]$ docker run ubuntu:16.04 /bin/sh -c "while true; do echo hello docker; sleep 1; done"
hello docker
hello docker
hello docker
hello docker
^C[JING@localhost mynginx]$
```

使用 `-d` 选项让容器在后台运行，打印的内容将不会再直接出现在宿主机上：

```
[JING@localhost mynginx]$ docker run -d ubuntu:16.04 /bin/sh -c "while true; do echo hello docker; sleep 1; done"
a7c0540f4ad703a41b1ce06d4245980f5d96c01b7aedd6d2a2188425396cc535
```

`docker container logs [container id]` 查看容器打印输出：

```
[JING@localhost mynginx]$ docker container logs a7c
hello docker
hello docker
...
```

### 终止容器

容器执行完毕后会自动终止，但有些容器是会一直运行下去的，比如上面死循环打印 hello docker 的容器永远不会执行完毕，如果想要终止这类容器，使用 `doker container stop [container id]` 实现：

```
[JING@localhost mynginx]$ docker container stop a7c
a7c
```

### 删除容器

`docker container rm [container id]` 用于删除处于终止状态的容器，`docker container rm -f [container id]` 用于强制删除处于运行状态的容器：

```
# 列出所有容器包括已终止容器
[JING@localhost mynginx]$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                         PORTS               NAMES
a7c0540f4ad7        ubuntu:16.04        "/bin/sh -c 'while t…"   13 minutes ago      Exited (137) 5 minutes ago                         peaceful_perlman
e54e8f5dece9        ubuntu:16.04        "/bin/sh -c 'while t…"   14 minutes ago      Exited (0) 14 minutes ago                          pedantic_turing
3fe29326c786        ubuntu:16.04        "/bin/bash"              43 minutes ago      Exited (0) 2 minutes ago                           optimistic_chandrasekhar
be52bb583d7b        ubuntu:16.04        "/bin/echo 'Hello Do…"   About an hour ago   Exited (0) About an hour ago                       heuristic_wright
# 根据容器 id 删除一个已终止容器
[JING@localhost mynginx]$ docker container rm a7c
a7c
[JING@localhost mynginx]$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                         PORTS               NAMES
e54e8f5dece9        ubuntu:16.04        "/bin/sh -c 'while t…"   15 minutes ago      Exited (0) 15 minutes ago                          pedantic_turing
3fe29326c786        ubuntu:16.04        "/bin/bash"              44 minutes ago      Exited (0) 3 minutes ago                           optimistic_chandrasekhar
be52bb583d7b        ubuntu:16.04        "/bin/echo 'Hello Do…"   About an hour ago   Exited (0) About an hour ago                       heuristic_wright
# 启动一个容器
[JING@localhost mynginx]$ docker container start 3fe
3fe
# 列出正在运行中的容器
[JING@localhost mynginx]$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
3fe29326c786        ubuntu:16.04        "/bin/bash"         About an hour ago   Up 28 seconds                           optimistic_chandrasekhar
# 强制删除正在运行中的容器
[JING@localhost mynginx]$ docker container rm -f 3fe
3fe
# 列出所有容器
[JING@localhost mynginx]$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                          PORTS               NAMES
e54e8f5dece9        ubuntu:16.04        "/bin/sh -c 'while t…"   20 minutes ago      Exited (0) 20 minutes ago                           pedantic_turing
be52bb583d7b        ubuntu:16.04        "/bin/echo 'Hello Do…"   About an hour ago   Exited (0) About a minute ago                       heuristic_wright
```

## Docker 架构图

![mark](http://imgblog.kuranado.com/blog/180611/kD98L8aG0a.png)

客户端输入的 Docker 命令交给 Docker daemon 守护程序来执行具体操作，Docker daemon 根据镜像创建容器，镜像来源于 Docker Registry。

## 参考资料

[慕课网 Docker 入门教程](https://www.imooc.com/learn/867)
[阮一峰 Docker 入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)
[Docker - 通往新世界的大门](https://juejin.im/post/5b04e7946fb9a07ab7749082)
[A Look At Docker & Containers Technology](https://www.esds.co.in/blog/docker-and-containers-technology/#sthash.LRJkkrTw.dpbs)
[Docker 从入门到实践](https://legacy.gitbook.com/book/yeasy/docker_practice/details)