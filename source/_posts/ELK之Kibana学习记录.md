---
title: ELK之Kibana学习记录
date: 2018-08-26 18:42:00
comments: true
categories: [ELK]
tags: [ELK,Kibana,日志]
---

本篇文章为 Kibana的学习记录，为了统一版本号，和前两篇篇文章一样，使用 Logstash 5.6 版本。Kibana 实际上就是 Elasticsearch 的图像化，我司使用 Kibana 主要就是为了方便测试人员查看日志

<!-- more -->

## Kibana 主要功能

- 可以与 Elasticsearch 无缝集成。Kibana 架构为 Elasticsearch 定制，可以将任何结构化和非结构化数据加入 Elasticsearch 索引。Kibana 还充分利用了 Elasticsearch 强大的搜索和分析功能
- 整合数据。Kibana 能够更好地处理海量数据，并据此创建柱形图、折线图、散点图、饼图和地图等
- 能够进行复杂数据分析。Kibana 提升了 Elasticsearch 的分析能力，能够更加智能地分析数据，执行数学转换，并且根据要求对数据切割分块
- 让更多团队成员收益。强大的数据库可视化接口让各业务岗位都能够从数据集合收益
- 接口灵活，分享更容易。使用 Kibana 可以更方便地创建、保存、分享数据。

## 安装 Kibana 5.X

```
[root@jing server]# wget https://artifacts.elastic.co/downloads/kibana/kibana-5.6.2-linux-x86_64.tar.gz
[root@jing server]# tar -xvf kibana-5.6.2-linux-x86_64.tar.gz
```

## 配置文件

修改 kibana 配置文件：

```
[root@jing server]# cd kibana-5.6.2-linux-x86_64
[root@jing kibana-5.6.2-linux-x86_64]# vim conf/kibana.yml
```

取消如下三行注释并更改配置：

```
server.port: 5601
# 设置为本机 IP
server.host: "192.168.163.128"
# 设置需要连接的 Elasticsearch 的 url
elasticsearch.url: "http://192.168.163.128:9200"
```

## 启动 Kibana

- **Kibana 启动必须是在 root 用户下**，否则将会启动失败
- 启动 Kibana 前必须先启动 Elasticsearch

```
[root@jing kibana-5.6.2-linux-x86_64]# ./bin/kibana
  log   [12:52:19.363] [info][status][plugin:kibana@5.6.2] Status changed from uninitialized to green - Ready
  log   [12:52:19.461] [info][status][plugin:elasticsearch@5.6.2] Status changed from uninitialized to yellow - Waiting for Elasticsearch
  log   [12:52:19.536] [info][status][plugin:console@5.6.2] Status changed from uninitialized to green - Ready
  log   [12:52:19.567] [info][status][plugin:metrics@5.6.2] Status changed from uninitialized to green - Ready
  log   [12:52:19.890] [info][status][plugin:timelion@5.6.2] Status changed from uninitialized to green - Ready
  log   [12:52:19.909] [info][listening] Server running at http://192.168.163.128:5601
  log   [12:52:19.912] [info][status][ui settings] Status changed from uninitialized to yellow - Elasticsearch plugin is yellow
  log   [12:52:20.051] [info][status][plugin:elasticsearch@5.6.2] Status changed from yellow to green - Kibana index ready
  log   [12:52:20.052] [info][status][ui settings] Status changed from yellow to green - Ready
```

其中 `  log   [12:52:19.909] [info][listening] Server running at http://192.168.163.128:5601` 表示 Kibana 正在监听 5601 端口

浏览器访问 Kibana：

```
http://192.168.163.128:5601
```

![mark](http://imgblog.kuranado.com/blog/180825/8J41lJjEj7.png)

## 安装 X-pack 插件

- X-pack 监控组件可以通过 Kibana 轻松的监控 Elasticsearch，可以实时查看集群的健康和性能，以及分析过去的集群、索引和节点度量。此外还可以监视 Kibanna 本身性能。
- X-pack 会进行二次认证

首先需要停掉 Elasticsearch、Logstash 和 Kibanna 服务，然后才能开始安装

### Elasticsearch 中安装 x-pack

更详细的安装步骤参考官网：[https://www.elastic.co/guide/en/elasticsearch/reference/5.6/installing-xpack-es.html](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/installing-xpack-es.html)

```
[eser@jing ~]$ cd /data/server/elasticsearch-5.6.2/bin/
[eser@jing bin]$ ./elasticsearch-plugin install x-pack
```

此处我安装失败，抛出了 `javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException` 异常，所以使用 `wgets` 命令下载压缩包后再进行安装：

```
[root@jing bin]# cd /data/server/
[root@jing server]# wget https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-5.6.2.zip  --no-check-certificate
[root@jing server]# cd /data/server/elasticsearch-5.6.2/bin/
[root@jing bin]# ./elasticsearch-plugin install file:///data/server/x-pack-5.6.2.zip 
-> Downloading file:///data/server/x-pack-5.6.2.zip
[=================================================] 100%   
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@     WARNING: plugin requires additional permissions     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
* java.io.FilePermission \\.\pipe\* read,write
* java.lang.RuntimePermission accessClassInPackage.com.sun.activation.registries
* java.lang.RuntimePermission getClassLoader
* java.lang.RuntimePermission setContextClassLoader
* java.lang.RuntimePermission setFactory
* java.security.SecurityPermission createPolicy.JavaPolicy
* java.security.SecurityPermission getPolicy
* java.security.SecurityPermission putProviderProperty.BC
* java.security.SecurityPermission setPolicy
* java.util.PropertyPermission * read,write
* java.util.PropertyPermission sun.nio.ch.bugLevel write
* javax.net.ssl.SSLPermission setHostnameVerifier
See http://docs.oracle.com/javase/8/docs/technotes/guides/security/permissions.html
for descriptions of what these permissions allow and the associated risks.

Continue with installation? [y/N]y
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@        WARNING: plugin forks a native controller        @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
This plugin launches a native controller that is not subject to the Java
security manager nor to system call filters.

Continue with installation? [y/N]y
-> Installed x-pack
```

安装完成后切换到普通用户启动 Elasticsearch即可，浏览器访问如下：

![mark](http://imgblog.kuranado.com/blog/180826/iBkadF7a1j.png)

说明安装 x-pack 后，访问 Elasticsearch 增加了一个安全验证，默认的用户名和密码分别为 `elastic` 和 `changeme`。输入用户名密码后即可看到返回的 Json 数据：

```
{
  "name" : "node-1",
  "cluster_name" : "my-elk",
  "cluster_uuid" : "meEf4_2zTz6TVPkta631gw",
  "version" : {
    "number" : "5.6.2",
    "build_hash" : "57e20f3",
    "build_date" : "2017-09-23T13:16:45.703Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```

### Kibana 中安装 x-pack

更详细的安装步骤参考官网：[https://www.elastic.co/guide/en/kibana/5.6/installing-xpack-kb.html](https://www.elastic.co/guide/en/kibana/5.6/installing-xpack-kb.html)

```
[root@jing ~]# cd /data/server/kibana-5.6.2-linux-x86_64/bin/
[root@jing bin]# ./kibana-plugin install file:///data/server/x-pack-5.6.2.zip 
Attempting to transfer from file:///data/server/x-pack-5.6.2.zip
Transferring 160300067 bytes....................
Transfer complete
Retrieving metadata from plugin archive
Extracting plugin archive
Extraction complete
Optimizing and caching browser bundles...
Plugin installation complete
```

安装完成后启动 Kibana，浏览器访问如下：

![mark](http://imgblog.kuranado.com/blog/180826/B5IjdjgFiE.png)

和在 Elasticsearch 中安装 x-pack 一样，默认的用户名和密码分别为 `elastic` 和 `changeme`

## ELK 整合

编辑一个 Logstash 配置文件：

```
[root@jing ~]# cd /data/server
[root@jing server]# vim conf/acc.conf
```

配置如下：

```
input {
    file {
        path => ["/data/logs/mylog*.log"]
        type => "acc.log"
        start_position => "beginning"
        sincedb_path => "/data/server/conf/.sincedb"
        sincedb_write_interval => 15
    }
    stdin {
    }
}

output {
    elasticsearch {
        hosts => "192.168.163.128:9200"
        index => "acc_index"
        # 因为安装了 x-pack 插件，Logstash 将数据写入到 Elasticsearch 需要用户名和密码
        user => "elastic"
        password => "changeme"
    }
    stdout {
        codec => rubydebug
    }
}
```

之后启动 Logstash：

```
[root@jing server]# ./logstash-5.6.2/bin/logstash -f /data/server/conf/acc.conf
```

Logstash 启动成功后会将 `/data/logs/mylog*.log` 日志写到 Elasticsearch 中，访问 Kibana 时需要创建 `acc.conf` 配置文件所配置的索引（此处配置的索引为 `acc_index`）即可成功在 Kibana 中展示数据：

![mark](http://imgblog.kuranado.com/blog/180826/KfeKHAH1J5.png)