---
title: ELK之Logstash学习记录
date: 2018-08-25 17:28:00
comments: true
categories: [ELK]
tags: [ELK,Logstash,日志]
---

本篇文章为 Logstash 的学习记录，为了统一版本号，和上篇文章一样，使用 Logstash 5.6 版本。

<!-- more -->

## Logstash 基础

### 简介

Logstash 可以传输和处理你的日志，事务或其他数据
Logstash 是 Elasticsearch 的最佳数据管道
Logstash 是插件式管理模式，在输入、过滤、输出以及编码过程中都可以使用插件进行定制，Logstash 社区拥有超过 200 种插件

### 工作原理

Logstash 的配置文件有两个必要元素：`input` 和 `output`，一个可选元素：`filter`。这三个元素分别代表 Logstash 事件处理的三个阶段 ：输入 > 过滤器 > 输出：

![mark](http://imgblog.kuranado.com/blog/180825/DaBjDdDg3J.png)

打开公司生产环境下 Logstash 的配置文件，可以看到输入、过滤器和输出都不止一个，具体应根据业务来自由组合

#### Input

Input 用于抓取数据放入 Logstash，常用的 Input 插件有：

- **file：从文件读取数据，类似 `tail -0F` 命令**
- syslog：监听 `514` 端口上的 syslog 日志并解析为 `RFC3164` 格式
- redis：从 redis 服务器读取，使用 redis 通道和 redis 列表。redis 经常用作集中式 Logstash 安装中的代理，它将来自远程 Logstash "托运人"的 Logstash 事件排队
- beats：处理由 Filebeat 发送的事件

官网 Input 插件列表：[https://www.elastic.co/guide/en/logstash/current/input-plugins.html#input-plugins](https://www.elastic.co/guide/en/logstash/current/input-plugins.html#input-plugins)

#### Filters

Filter 在 Logstash 管道中作为中介处理，用于将数据修改为指定的内容或格式。常用的 Filter 插件有：

- date：用于解析字段中的日期
- grok：解析和组织任意文本。Grok 是当前能够解析非结构化的日志数据成为结构良好的并且可查询的最好的方式，拥有 120 中内置模式总有一款适合你
- mutate：对事件字段执行一段转换，可以重命名、删除、替换和修改事件中的字段
- drop：安全放弃一个事件，例如调试事件
- clone：制作一个事件的副本，可能会添加或删除字段
- geoip：添加有关 IP 地址的地理位置的信息


官网 Filter插件列表：[https://www.elastic.co/guide/en/logstash/current/filter-plugins.html#filter-plugins](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html#filter-plugins)

#### Outputs

Outputs 是 Logstash 管道的最后阶段，每个事件都可以拥有多个输出，一旦所有的输出过程都完成了，该事件也就执行完成了。常用 Output 插件有：

- **elasticsearch：发送事件数据到 Elasticsearch。如果希望保存数据高效、方便而且简单易查询，保存到 Elasticsearch 是非常理想的方案**
- **file：将事件数据写到文件或磁盘**
- graphite：发送事件数据到 `graphite`（一个开源的存储和绘制指标工具）

官网 Output 插件列表：[https://www.elastic.co/guide/en/logstash/current/output-plugins.html](https://www.elastic.co/guide/en/logstash/current/output-plugins.html)

#### Codec

Codec 用于格式化对应的内容，常用 Codec 插件：

- json：以 JSON 格式对数据进行编解码
- multiline：将多行文本事件（如 Java 异常和堆栈跟踪消息）合并为单个事件

官网 Input 插件列表：[https://www.elastic.co/guide/en/logstash/current/codec-plugins.html](https://www.elastic.co/guide/en/logstash/current/codec-plugins.html)

具体工作原理可以参考官网：[https://www.elastic.co/guide/en/logstash/current/pipeline.html](https://www.elastic.co/guide/en/logstash/current/pipeline.html)

## 安装 Logstash 5.X

```
[root@jing server]# cd /data/server
[root@jing server]# wget https://artifacts.elastic.co/downloads/logstash/logstash-5.6.2.tar.gz
[root@jing server]# tar -xvf logstash-5.6.2.tar.gz
```

### 配置文件

- LOGSTASH_HOME/config/logstash.yml：Logstash 的默认启动配置文件
- LOGSTASH_HOME/config/jvm.options：Logstash 的 JVM 配置文件

#### logstash.yml

具体请参考官网：[https://www.elastic.co/guide/en/logstash/current/logstash-settings-file.html](https://www.elastic.co/guide/en/logstash/current/logstash-settings-file.html)
此处有翻译版本：[http://www.cnblogs.com/jingmoxukong/p/8118791.html](http://www.cnblogs.com/jingmoxukong/p/8118791.html)

## 启动 Logstash

```
[root@jing server]# cd logstash-5.6.2/bin
# -f 选项指定启动时需要的配置文件
[root@jing server]# ./logstash -f /data/server/log.conf
```

可以在 `/etc/rc.local` 文件下增加如下命令实现 Logstash 开机自启：

```
/data/server/logstash-5.6.2/bin/logstash -f /data/server/log.conf &
```

下面以演示 `stdin` 标准输入和 `stdout` 标准输出插件为例启动 Logstash：

```
[root@jing server]# mkdir /data/server/conf
[root@jing server]# cd /data/server/conf
[root@jing conf]# vim logstash-input-stdin.conf
```

修改配置文件如下：

```
input {
	stdin {
	}
}

output {
	elasticsearch {
		hosts => ["127.0.0.1:9200"]
	}
	stdout {
		codec => rubydebug
	}
}
```

使用刚创建的配置文件启动 Logstash：

```
[root@jing conf]# /data/server/logstash-5.6.2/bin/logstash -f logstash-input-stdin.conf 
Sending Logstash's logs to /data/server/logstash-5.6.2/logs which is now configured via log4j2.properties
[2018-08-25T15:11:03,452][INFO ][logstash.modules.scaffold] Initializing module {:module_name=>"fb_apache", :directory=>"/data/server/logstash-5.6.2/modules/fb_apache/configuration"}
[2018-08-25T15:11:03,455][INFO ][logstash.modules.scaffold] Initializing module {:module_name=>"netflow", :directory=>"/data/server/logstash-5.6.2/modules/netflow/configuration"}
[2018-08-25T15:11:04,750][INFO ][logstash.outputs.elasticsearch] Elasticsearch pool URLs updated {:changes=>{:removed=>[], :added=>[http://127.0.0.1:9200/]}}
[2018-08-25T15:11:04,752][INFO ][logstash.outputs.elasticsearch] Running health check to see if an Elasticsearch connection is working {:healthcheck_url=>http://127.0.0.1:9200/, :path=>"/"}
[2018-08-25T15:11:04,885][WARN ][logstash.outputs.elasticsearch] Restored connection to ES instance {:url=>"http://127.0.0.1:9200/"}
[2018-08-25T15:11:04,886][INFO ][logstash.outputs.elasticsearch] Using mapping template from {:path=>nil}
[2018-08-25T15:11:04,961][INFO ][logstash.outputs.elasticsearch] Attempting to install template {:manage_template=>{"template"=>"logstash-*", "version"=>50001, "settings"=>{"index.refresh_interval"=>"5s"}, "mappings"=>{"_default_"=>{"_all"=>{"enabled"=>true, "norms"=>false}, "dynamic_templates"=>[{"message_field"=>{"path_match"=>"message", "match_mapping_type"=>"string", "mapping"=>{"type"=>"text", "norms"=>false}}}, {"string_fields"=>{"match"=>"*", "match_mapping_type"=>"string", "mapping"=>{"type"=>"text", "norms"=>false, "fields"=>{"keyword"=>{"type"=>"keyword", "ignore_above"=>256}}}}}], "properties"=>{"@timestamp"=>{"type"=>"date", "include_in_all"=>false}, "@version"=>{"type"=>"keyword", "include_in_all"=>false}, "geoip"=>{"dynamic"=>true, "properties"=>{"ip"=>{"type"=>"ip"}, "location"=>{"type"=>"geo_point"}, "latitude"=>{"type"=>"half_float"}, "longitude"=>{"type"=>"half_float"}}}}}}}}
[2018-08-25T15:11:04,968][INFO ][logstash.outputs.elasticsearch] Installing elasticsearch template to _template/logstash
[2018-08-25T15:11:05,113][INFO ][logstash.outputs.elasticsearch] New Elasticsearch output {:class=>"LogStash::Outputs::ElasticSearch", :hosts=>["//127.0.0.1:9200"]}
[2018-08-25T15:11:05,115][INFO ][logstash.pipeline        ] Starting pipeline {"id"=>"main", "pipeline.workers"=>4, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>5, "pipeline.max_inflight"=>500}
[2018-08-25T15:11:20,218][INFO ][logstash.pipeline        ] Pipeline main started
The stdin plugin is now waiting for input:
[2018-08-25T15:11:20,296][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
hahaha
{
      "@version" => "1",
          "host" => "0.0.0.0",
    "@timestamp" => 2018-08-25T07:11:35.494Z,
       "message" => "hahaha"
}
```

启动成功后在终端输入的内容会被再终端被打印出来，这是 stdin 和 stdout 插件的功能，输出格式为 `rubydebug` 格式，同时也可以看到 Elasticsearch 打印如下，这是输出插件 elasticsearch 的功劳：

```
[2018-08-25T15:11:35,537][INFO ][o.e.c.m.MetaDataCreateIndexService] [node-1] [logstash-2018.08.25] creating index, cause [auto(bulk api)], templates [logstash], shards [5]/[1], mappings [_default_]
[2018-08-25T15:11:35,689][INFO ][o.e.c.m.MetaDataMappingService] [node-1] [logstash-2018.08.25/3dvCpemyTpeClxxnh-_hPA] create_mapping [logs]
```

接下来看一段公司实际业务应用中的配置（敏感配置已被更改）：

```
input { 
    file {
        type => "core"
        path => ["/data/logs/core/prod/mylog*.log"]
        discover_interval => 5
        start_position => "beginning"
        sincedb_path => "/opt/logstash-5.6.2/config/.sincedb"
        sincedb_write_interval => 15
        codec => plain { 
            charset => "UTF-8" 
        }
    }   
    file {
        type => "kafka"
        path => ["/data/logs/kafka/prod/mylog*.log"]
        discover_interval => 5
        start_position => "beginning"
        sincedb_path => "/opt/logstash-5.6.2/config/.sincedb"
        sincedb_write_interval => 15
        codec => plain { 
            charset => "UTF-8" 
        }
    }
    file {
        type => "redis"
        path => ["/data/logs/redis/prod/mylog*.log"]
        discover_interval => 5
        start_position => "beginning"
        sincedb_path => "/opt/logstash-5.6.2/config/.sincedb"
        sincedb_write_interval => 15
        codec => plain { 
            charset => "UTF-8" 
        }
    }
}  

filter {
    date {
        locale => en
        match => ["logdate", "MMM dd yyyy HH:mm:ss"]
    }
}

output{
    if [type] == "core" {
        elasticsearch {
            hosts  => ["127.0.0.1:9200"]
            index => "core-log"
#           user => "elastic"
#           password => "changeme"
        } 
    }
    if [type] == "kafka" {
        elasticsearch {
            hosts  => ["127.0.0.1:9200"]
            index => "kafka-log"
#           user => "elastic"
#           password => "changeme"
        }
    }
    if [type] == "redis" {
        elasticsearch {
            hosts  => ["127.0.0.1:9200"]
            index => "redis-log"
#           user => "elastic"
#           password => "changeme"
        }
    }
}
```

### `file` 插件主要配置

Setting|Input type|Required|Default value|Detail
-|
type|string|No|No default value|定义 type 主要是为了给 filter 动作作判断
path|array|Yes|No default value|指定文件位置，**必须为绝对路径**。可以使用通配符，如 `/var/log/*.log`，`/var/log/**/*.log`(双星表示递归查找，此处为递归查找 `/var/log/` 目录下的所有后缀名为 `.log` 的文件)
discover_interval |number|No|15|该值用于检查 `path` 选项中的文件描述符所匹配的文件，如 `stat_interval` 选项的值为 500ms(该选项的默认值为 1s)，`discover_interval` 选项的值为 15，则 15 * 500ms = 7.5s 后检查 `path` 选项配置的文件通配符是否有新的文件
start_position|string, one of ["beginning", "end"]|No|end|监听的位置，默认是end，即一个文件如果没有记录它的读取信息，则从文件的末尾开始读取，也就是说，仅仅读取新添加的内容。对于一些更新的日志类型的监听，通常直接使用end就可以了；相反，beginning就会从一个文件的头开始读取。但是如果记录过文件的读取信息，这个配置也就失去作用了
sincedb_path|string|No|No default value|配置了默认的读取文件信息记录在哪个文件中，默认是按照文件的inode等信息自动生成。其中记录了inode、主设备号、次设备号以及读取的位置。因此，如果一个文件仅仅是重命名，那么它的inode以及其他信息就不会改变，因此也不会重新读取文件的任何信息。类似的，如果复制了一个文件，就相当于创建了一个新的inode，如果监听的是一个目录，就会读取该文件的所有信息
sincedb_write_interval|number or string_duration|No|"15 seconds"|设置多长时间写入读取的位置信息
codec|codec|No|"plain"|用于在数据进入 input 插件前解码你的输入数据，这样就可以省去在 Filter 中设置 codec 了

### `date` 插件主要配置

Setting|Input type|Required|Default value|Detail
-|
match|array|No|[]|`match => ["logdate", "MMM dd yyyy HH:mm:ss"]` 表示如果你有个字段为 `logdate`，并且对应的值类似 `Aug 13 2010 00:03:44`，将被这条配置匹配到

## 参考文章

- [Elastic 技术栈之 Logstash 基础](http://www.cnblogs.com/jingmoxukong/p/8118791.html)
- [logstash-input-file插件使用详解](https://www.cnblogs.com/xing901022/p/4805586.html)