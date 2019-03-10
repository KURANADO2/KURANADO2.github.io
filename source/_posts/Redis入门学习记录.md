---
title: Redis入门学习记录
date: 2017-11-06 23:52:00
comments: true
categories: [Redis]
tags: [Redis, 数据库, 缓存]
---

结合[Redis官方文档](https://redis.io/documentation)和黑马的视频教程整理了一些Redis常用知识，其中列举的一些命令的使用场景是我直接根据官方文档翻译的。另外随着以后对Redis的深入，这篇文章也会不定期更新！
<!-- more -->

## 数据类型

Data type|Explanation
-|
**Binary-safe strings**|字符串，是二进制安全的，也就是说可以用来存储图片文件、CSS文件等
**Hashes**|哈希
**Lists**|基于链表的集合，元素可重复，有序
**Sets**|元素唯一，无序
**Sorted sets**|在保证元素唯一性的同时根据元素score为元素排序
Bit arrays|
HyperLogLogs|

其中最常用的五种数据类型分别为:String,Hash,List,Set和Sorted set

### String

```
SET key1 1
GET key1    //获得的结果为字符串"1"
INCR key1   //Redis会自动尝试将字符串转换为integer并加1，如不能成功转换会报错
DECR key1    //将字符串转换为integer并减1
KEYS *    //查看当前数据库中的所有key
DEL key1    //删除key1
```

上面的SET命令设置key的value时，如果key不存在则会新建key并存储value，如果key存在，则会用新的value覆盖旧的value，其实如果key已经存在Redis可以让`SET`命令执行失败，或者相反的，即使key存在也让`SET`命令成功执行，下面是官方文档给出的示例：

```
SET mykey val nx    //此时因为mykey不存在，所以返回OK，成功将val赋给mykey
SET mykey newval nx    //因为mykey已经存在，所以赋值失败，返回(nil)
SET mykey newval2 xx    //不管mykey是否存在都将newval2赋给mykey，事实上这个xx参数加不加和单纯的SET功能完全一样
```

### Hash

```
HSET hkey1 field1 a
HSET hkey1 field2 3
HSET hkey1 field2 b
HKEYS hkey1
HVALS hkey1
HGETALL hkey1
HDEL hkey1 field1
DEL hkey1
KEYS *
```

### List

Reids中List底层数据结构是链表，而Python的list底层数据结构是数组，链表和数组各有优缺点如下：

- 链表：插入删除数据方便，查询数据麻烦
- 数组：查询数据方便，插入删除数据麻烦

```
LPUSH mylist 1 2 3 4 5 6    //LPUSH命令每次都是从最左边插入新数据，类似链表操作的头插法
LRANGE mylist 0 -1    //取出的结果为6 5 4 3 2 1，取数据时下标0代表第一个数据，-1代表最后一个数据，-2为倒数第二个数据，以此类推，很多语言都具有这种特性，Redis中并没有对应的RRange命令
RPUSH mylist a b c d e f    //每次从最右边放入数据，类似链表的尾插法
LPOP mylist    //弹出（删除）最左边的数据(6)，如果mylist为空，则返回(nil)
RPOP mylist    //弹出（删除）最右边的数据(f)
LLEN mylist    //返回(integer)10
```

场景一：在你的Twitter个人主页上，想要显示上传的最新10张照片，我们可以这样做
1. 每上传一张照片时就把照片id通过`LPUSH`放到photolist中
2. 当用户访问你的主页时，通过`LRANGE photolist 0 9`取出最新的10张照片的id，然后显示照片

场景二：有时我们希望删除List中非常非常旧的数据，比如日志数据，因为非常老的数据对我们来说可能没什么意义，这时就可以通过`LTRIM`命令删除旧数据

```
LPUSH loglist
LTRIM loglist 0 999999    //每次调用该命令只获取最新的1000000条数据，后面的数据会自动删除，和LRANGE命令的唯一区别就是LRANGE不会删除任何数据
```

### Set

数据和插入顺序无关，且不能插入重复数据

```
SADD set1 a b b a c d    //成功添加4个元素
SREM set1 a    //移除元素a（Remove）
SMEMBERS set1    //查看set1中的所有元素
KEYS *
DEL set1
SADD seta a b c d e f
SADD setb c d e f g h
SDIFF seta setb    //求差集，输出a b
SDIFF setb seta    //输出g h
SINTER seta setb    //取交集（Intersection），输出c d e f
SUNION seta setb    //取并集，输出a b c d e f g h
SISMEMBER seta f    //返回1，判断f是否是seta的成员
SISMEMBER seta z    //返回0
SCARD seta    //返回6，用于查询set中的元素个数
SPOP seta    //从seta中随机删除一个元素并输出该元素
SRANDMEMBER seta    //从seta中随机取一个元素并输出，和SPOP的区别是该命令不会删除元素
```

接下来用Set简单模拟纸牌的发牌过程：

一副纸牌除去大小王剩下分别为Club(梅花)A-K，Diamond(方块)A-K，Heart(红桃)A-K和Spade(黑桃)A-K共计52张，用CA表示梅花A，C2表示梅花2，以此类推，现在把这52张牌放到一个set中：

```
sadd deck C1 C2 C3 C4 C5 C6 C7 C8 C9 C10 CJ CQ CK
   D1 D2 D3 D4 D5 D6 D7 D8 D9 D10 DJ DQ DK H1 H2 H3
   H4 H5 H6 H7 H8 H9 H10 HJ HQ HK S1 S2 S3 S4 S5 S6
   S7 S8 S9 S10 SJ SQ SK
```

假设有4个玩家，每一轮的发牌就可以使用`SPOP`命令从set中随机取出4个元素然后发给各个玩家：

```
SPOP deck
SPOP deck
SPOP deck
SPOP deck
```

一局游戏结束后，因为set中的元素已经全被删除了，如果想要再多玩几局时，就必须重新调用`SADD`命令将52张牌放到set中，太麻烦了，所幸Redis提供了`SUNIONSTORE` 命令，可以复制set的命令，这样就可以在第一局发牌前将deck复制一份到另一个set：game:1:deck中，而发牌则从game:1:deck中取牌而不从deck中取，第二局游戏时就将deck复制到game:2:deck中......

```
SUNIONSTORE game:1:deck deck
SPOP game:1:deck
SPOP game:1:deck
SPOP game:1:deck
SPOP game:1:deck
...
```

### Sorted set

在Set基础上增加了为元素排序功能，当然需要消耗的资源是要大于Set的，这和JDK中的SortedSet接口的特性一样，实际上Redis的这五种常用数据类型在JDK中都能找到对应的接口

另外Sorted set中判断元素是否重复以及排序是根据元素的score来确定的，如果有两个元素a，但score不同，Sorted set则认为这两个a是不同的元素，都能够成功插入到Sorted set，默认按照score从小到大为元素排序

```
zadd hackers 1940 "Alan Kay"    //如果中间有空格需要使用引号括起来，否则Err
zadd hackers 1957 "Sophie Wilson"
zadd hackers 1953 "Richard Stallman"
zadd hackers 1949 "Anita Borg"
zadd hackers 1965 "Yukihiro Matsumoto"
zadd hackers 1914 "Hedy Lamarr"
zadd hackers 1916 "Claude Shannon"
zadd hackers 1969 "Linus Torvalds"
zadd hackers 1912 "Alan Turing"
ZRANGE hackres 0 -1
ZRANGE hackres 0 -1 WITHSCORES
ZREVRANGE hackers 0 -1
ZRANGEBYSCORE hackers -INF 1950    //获得所有小于等于1950的元素
ZREMRANGEBYSCORE hackers 1940 1960    //从hackers中删除score为1940-1960的元素
ZRANK hackers "Linus Torvalds"    //输出4，返回元素在hackers中的下标（从0开始）
```

## Expire-生存时间

EXPIRE命令用来设置key的生存时间，以秒为单位

- key存在返回1
- key不存在返回0

```
EXPIRE mykey 10    //返回(integer)1mykey10s后将被删除
TTL mykey    //查看mykey还有多少秒将被删除
```

通过`SET`命令加ex参数也可以设置key的生存时间：

```
SET mykey hello ex 10
TTL mykey    //查看mykey的剩余时间
```

另外任何修改key对应value值的命令都无法清除为key设置的生存时间，比如`SET,HSET,LPUSH,DEL,GETSET,INCR`等，如果要清除为key设置的生存时间，可以在key**过期前**，使用`PERSIST`命令清除key的生存时间

```
PERSIST mykey
TTL mykey    //返回-1，表示该key没有设置生存时间
```

在Redis2.4中，设置的生存时间会存在0-1秒的误差，在Redis2.6中，误差被减小到0-1ms，另外Redis销毁key有两种方式：

- 消极方式
	1. 当用户访问该key时Redis才去判断该key是否已经过期
- 积极方式（基于概率性的算法）
	每秒钟执行10次下面的步骤
	1. 随机测试20个已经被设置生存时间的key
	2. 删除所有被发现的已经过期的key
	3. 如果超过5个key已经过期则重新执行第1步直到已经过期的key少于5个

## Persistence-持久化

Redis中的数据都放在内存中，这也是Redis读写效率远远高于其他关系型数据库的原因之一，但数据放在内存中，如果遇到断电、kill -9等意外情况，内存中的数据就会丢失，为此Redis提供了两种将数据持久化到硬盘的方案

### RDB



### AOF(Append-only file)



## Transaction-事务

`MULTI`开启事务队列，`EXEC`执行事务，事务中的每条命令正常情况下将返回QUEUED
```
MULTI    //返回OK
INCR foo    //返回QUEUED
INCR bar    //返回QUEUED
EXEC    //返回(integer)1 (integer)1
```

需要注意的是**Redis并不支持事务回滚**，也就是说即使事务中某条命令返回error，事务队列中的其他命令仍然会被执行：

```
MULTI    //返回OK
SET a 3    //返回QUEUED
LPOP a    //返回QUEUED
EXEC    //返回 1)OK 2)(error) WRONGTYPE Operation against a key holding the wrong kind of value
GET a    //返回3，说明SET a 3并没有被回滚
```

`DISCARD`命令抛弃事务

```
SET foo 1  //OK
MULTI  //OK
INCR foo  //QUEUED
DISCARD  //OK,抛弃事务
GET foo  //"1"
```

## 常用命令

### PING

若返回PONG表示Redis服务正常

### INCRBY&DECRBY

与其对应的为`DECRBY`

```
SET count 100
INCR count    //101
INCRBY count 20    //121，一次加20
```

### GETSET

设置新的值并返回旧的值，设想这样一个场景：你有一个网站，你想每隔1小时获得这1小时内网站的访问人数，我们可以这样做：创建一个visitor key，每来1个访客就通过`INCR`命令让visitor加1，一小时后再通过`GETSET`获得1小时内访客数并将visitor重置为0

```
GETSET visitor 0
```

### MSET&MGET

为了减少网络传输次数，`MSET`/`MGET`(Multiple)可以一次设置/获得多个key

```
MSET a 10 b b c 30
MGET a b c    //输出10 b 30
```

### EXISTS

判断key是否存在，1存在，0不存在，实际上`DEL`是否能删除key就是根据`EXISTS`的返回值来决定的

```
SET mykey hello
EXISTS mykey    //返回(integer)1
DEL mykey
EXISTS mykey    //返回(integer)0
```

### TYPE

```
SET mykey hello
TYPE mykey    //返回string
SET mykey 23    //返回string
DEL mykey
TYPE mykey    //返回none
```

### TTL&PTTL

`TTL`以s为单位，`PTTL`以ms为单位

- 返回正数表示该key还有多少s/ms将被删除
- 返回-1表示该key没有设置生存时间
- 返回-2表示该key不存在

## Redis常识

- key不要超过1024字节，value不要超过512MB
- Redis中key区分大小写但命令并不区分大小写，通常所有的命令都使用大写字母，这和我在MySQL里的习惯一样，可以有效的区分命令和key/value