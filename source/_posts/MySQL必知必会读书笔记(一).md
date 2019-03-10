---
title: MySQL必知必会读书笔记(一)
date: 2017-11-11 20:35:00
comments: true
categories: [MySQL]
tags: [MySQL, 数据库]
---

《MySQL必知必会》的确是一本非常好的MySQL查阅手册，大三时买的这本书，在图书馆两天就看完了，因为书非常小（32开）也非常薄只有200多页，但看完和会用还是有很大区别的，面试时被要求写条不是很难的SQL语句，看起来挺简单，真正提笔去写时却怎么都想不起来具体要用哪个关键字，所以今天重新翻了一遍这本书，顺便做一个读书笔记，系统的整理一下MySQL基本知识，另外推荐小伙伴们看一下慕课网的那套[与MySQL的零距离接触](http://www.imooc.com/learn/122)教程，老师讲的非常好。

<!-- more -->

- 书中sql文件下载地址：[http://forta.com/books/0672327120/](http://forta.com/books/0672327120/)，如果你要学习这本书请务必导入sql文件亲自测试每一条命令。

## 连接MySQL

关于MySQL的安装不再赘述，不知如何安装的请看[京东云CentOS6.8安装JDK1.8+Tomcat7+MySQL5.6](http://www.kuranado.com/2017/09/26/%E4%BA%AC%E4%B8%9C%E4%BA%91CentOS6.8%E5%AE%89%E8%A3%85JDK1.8+Tomcat7+MySQL5.6/#安装MySQL)

![mark](http://imgblog.kuranado.com/blog/171111/0aH6mK9gFF.png)

本地连接直接使用:

```
mysql -u username -p  //然后输入密码即可
```

## SHOW

```
SHOW DATABASES;
USE crashcourse;
SHOW TABLES;
SHOW COLUMNS FROM customers;  //查看表列
DESCRIBE customers;  //查看表列
DESC customers;  //查看表列
SHOW CREATE DATABASE databasename;  //显示创建指定数据库的sql语句
SHOW CREATE TABLE tablename;  //显示创建指定表的sql语句
HELP SHOW;
```

## SELECT

### 检测单个列

```
SELECT prod_name FROM products;
```

![mark](http://imgblog.kuranado.com/blog/171111/KHJeLh7EF4.png)

这里需要注意返回数据的顺序可能是数据被添加到表中的顺序，也可能不是，在没有明确使用`ORDERY BY`关键字进行排序的情况下，返回的数据顺序是没有如何意义的。因为如果不排序，数据一般将以它在底层表中出现的顺序显示，这可以是数据最初添加到表中的顺序。但是，如果数据后来进行过更新或删除，**则此顺序将会受到MySQL重用回收存储空间的影响**，所以每个人执行这条查询语句返回的数据顺序可能会不一样。

### 检索多个列

```
SELECT prod_id, prod_name, prod_price FROM products;
```

### *-检索所有列

```
SELECT * FROM products;
```

除非确定需要表中的每个列，否则最好不要使用`*`通配符，检索不必要的列通常会降低检索和应用程序的性能

### DISTINCT-检索不同的行

```
SELECT vend_id FROM products;
```

![mark](http://imgblog.kuranado.com/blog/171111/CDBjAeFj5e.png)

```
SELECT DISTINCT vend_id FROM products;
```

![mark](http://imgblog.kuranado.com/blog/171111/fHJlHgI7KD.png)

`DISTINCT`关键字必须直接放在列名的前面

### LIMIT-分页

使用LIMIT之前需要明确一点：MySQL检索出的行**从0开始而不是从1开始**

```
SELECT prod_name FROM products LIMIT 1;  //查看前1条记录，也就是第0行
SELECT prod_name FROM products LIMIT 2;  //查看前2条记录（第0行和第1行）
SELECT prod_name FROM products LIMIT 4, 5;  //从第4行开始查找5行记录
```

其中第1个参数表示从第几行开始，第2个参数表示查找几条记录，所以上面的语句返回结果为从第4行开始显示5条记录；一般LIMIT关键字常用来做分页查询，如果用作分页查询，那么：

- 第2个参数表示每页显示m条记录
- 第1个参数=(当前是第n页 - 1) * m

所以当要求每页显示3条记录，查找第2页的数据时应该使用下面的SQL语句：

```
SELECT prod_name FROM products LIMIT 3, 3;
```

### 完全限定列名&完全限定表名

以下三条语句等价：

```
SELECT prod_name FROM products;
SELECT products.prod_name FROM products;  //完全限定列名
SELECT products.prod_name FROM crashcourse.products;  //完全限定表名
```

## ORDER BY

### 根据单个列排序

```
SELECT prod_name, prod_price FROM products ORDER BY prod_price;
```

![mark](http://imgblog.kuranado.com/blog/171111/3jC3jBJc1K.png)

### 根据多个列排序

先按价格排序（默认数字升序），价格相等时再按名称排序（默认字典序升序）

```
SELECT prod_id, prod_name, prod_price FROM products ORDER BY prod_price, prod_name;
```

![mark](http://imgblog.kuranado.com/blog/171111/gebHd3l6f7.png)

### 指定排序方向

`ASC`升序，`DESC`降序

```
SELECT prod_id, prod_price, prod_name FROM products ORDER BY prod_price DESC;  //按照价格降序
SELECT prod_id, prod_price, prod_name FROM products ORDER BY prod_price DESC, prod_name ASC;  //按照价格降序，价格相等时按照名称升序，ASC为默认，因此可省略
```

需要注意的是，大多数DBMS都把大写字母和小写字母视为相同，MySQL也是如此。

ORDER BY和LIMIT组合可用来查一列中的最大值或最小值

```
SELECT prod_price FROM products ORDER BY prod_price DESC LIMIT 1;
```

![mark](http://imgblog.kuranado.com/blog/171111/E3EfGG66g7.png)

`LIMIT`子句和`ORDER BY`子句同时出现时，`LIMIT`位于`ORDER BY`之后

## WHERE

`WHERE`子句用于过滤数据，说道过滤数据有两种方式，一种是应用过滤，一种是SQL过滤（推荐，当然`WHERE`属于这种）。而应用过滤有两个缺点：

1. 让客户机应用（或开发语言）处理数据库的工作将会极大的影响应用的性能，并且使所创建的应用完全不具备可伸缩性
2. 如果在客户机上过滤数据，服务器不得不通过网络发送多余的数据，这将导致网络带宽的浪费

### 检查单个值

```
SELECT prod_name, prod_price FROM products WHERE prod_name = 'Fuses';
SELECT prod_name, prod_price FROM products WHERE prod_price < 10;
SELECT prod_name, prod_price FROM products WHERE prod_price <= 10;
```

### <>-不匹配检查

`<>`和`!=`等价

```
SELECT vend_id, prod_name FROM products WHERE vend_id <> 1003;  //不是由供应商1003制造的所有商品
SELECT vend_id, prod_name FROM products WHERE vend_id != 1003;
```

### BETWEENT-范围值检查

```
SELECT prod_name, prod_price FROM products WHERE prod_price BETWEEN 5 AND 10;  //5 <= cust_price <= 10
```

### 空值检查

`NULL`表示为空值，它与字段包含0、空字符串或仅仅包含空格不同

```
SELECT cust_id FROM customers WHERE cust_email IS NULL;  //从customers表中查找没有email的客户id
```

`WHERE`子句和`ORDER BY`子句同时出现时，`ORDER BY`位于`WHERE`之后

## 数据过滤

### AND

检索由供应商1003制造且价格小于等于10美元的所有产品的名称和价格：

```
SELECT prod_id, prod_price, prod_name FROM products WHERE vend_id = 1003 AND prod_price <= 10;
```

### OR

检索由任意一个指定供应商制造的所有产品的产品名称和价格：

```
SELECT prod_name, prod_price FROM products WHERE vend_id = 1002 OR vend_id = 1003;
```

### ()-优先级

先来看需求：列出价格为10美元以上且由1002或1003制造的所有产品，于是SQL语句如下：

```
SELECT prod_name, prod_price FROM products WHERE vend_id = 1002 OR vend_id = 1003 AND prod_price >= 10;
```

![mark](http://imgblog.kuranado.com/blog/171112/jgljD7k3L8.png)

检索结果中竟然有价格低于10美元的产品，这是因为SQL在处理`OR`操作符前，会优先处理`AND`操作符，所以上面那条SQL语句被理解为：由供应商1003制造的任何价格为10美元以上的产品，或由供应商1002制造的任何产品。为了解决这种优先级问题，可以使用圆括号明确的分组相应的操作符。

```
SELECT prod_name, prod_price FROM products WHERE (vend_id = 1002 OR vend_id = 1003) AND prod_price >= 10;
```

![mark](http://imgblog.kuranado.com/blog/171112/j3E4C2J037.png)

任何时候具有`AND`和`OR`操作符的`WHERE`子句，**都应该使用圆括号明确的分组操作符**，不要过分依赖默认计算次序，即使它确实是你想要的东西。

### IN

下面两条SQL语句的含义完全相同：

```
SELECT prod_name, prod_price FROM products WHERE vend_id IN (1002, 1003);
SELECT prod_name, prod_price FROM products WHERE vend_id = 1002 OR vend_id = 1003;
```

通常会使用`IN`操作符代替`OR`操作符，因为`IN`相比`OR`以下几个优点：

1. `IN`操作符的语法更清楚更直观
2. 在使用`IN`时，计算的次序更容易管理
3. **`IN`操作符一般比`OR`操作符执行的更快**
4. `IN`最大的优点在于它可以包含其他`SELECT`语句，即子查询

### NOT

`NOT`操作符用于否定它之后所跟的任何条件

列出除1002和1003之外的所有供应商制造的产品：

```
SELECT prod_name, prod_price FROM products WHERE vend_id NOT IN (1002, 1003) ORDER BY prod_name;
```

MySQL支持`NOT`对`IN`、`BETWEEN`、`EXISTS`子句取反

## LIKE-通配符过滤

### %

`%`表示任何字符出现任意次数

```
SELECT prod_id, prod_name FROM products WHERE prod_name LIKE 'jet%';  //检索所有以jet起头的产品（不区分大小写，下同）
SELECT prod_id, prod_name FROM products WHERE prod_name LIKE '%anvil%';  //匹配任何位置包含文本anvil的值，不论它之前或之后出现什么字符
SELECT prod_name FROM products WHERE prod_name LIKE 's%e';  //检索以s起头e结尾的所有产品
```

- MySQL检索的时候不区分大小写，如果需要区分大小写，需更改MySQL的配置文件。
- 尾空格可能会干扰通配符匹配，例如在保存anvil时，如果它后面有一个或多个空格，则子句`WHERE prod_name LIKE '%anvil'`将因为空格的存在而不会匹配它们，最好的解决办法是使用`TRIM`函数去掉首尾空格
- `%`用于匹配任何字符出现任意次数，并不能用来匹配`NULL`，所以语句`SELECT cust_name, cust_email FROM customers WHERE cust_email LIKE '%';`表示检索除`NULL`值以外的所有行

![mark](http://imgblog.kuranado.com/blog/171112/2GHkID8i7l.png)

![mark](http://imgblog.kuranado.com/blog/171112/CHlk5H6I0k.png)

### _（下划线）

`_`表示只匹配单个字符而不是多个字符

```
SELECT prod_id, prod_name FROM products WHERE prod_name LIKE '_ ton anvil';
```

![mark](http://imgblog.kuranado.com/blog/171112/df47jK5Ig2.png)

### 通配符使用技巧

- 因为通配符的代价很大，所以不要过度使用通配符，如果其他操作符能达到相同的效果，应该使用其他操作符
- 如果将通配符放在搜索模式的开始处，搜索起来是最慢的，除非绝对有必要，否则不要把通配符放在搜索模式的开始处

## REGEXP-正则表达式

不论哪种语言的正则表达式都大同小异，实际上MySQL也只支持多数正则表达式实现的一个很小的子集，即便这样但却已经足够我们使用了，尤其是对于熟悉正则表达式的同学来说MySQL的正则怕是再简单不过了！

### 基本字符匹配

```
SELECT prod_name FROM products WHERE prod_name REGEXP '1000' ORDER BY prod_name;
```

![mark](http://imgblog.kuranado.com/blog/171112/dGEeBdDbE6.png)

`.`用于匹配任意一个字符：

```
SELECT prod_name FROM products WHERE prod_name REGEXP '.000' ORDER BY prod_name;
```

![mark](http://imgblog.kuranado.com/blog/171112/G3A8d9Bi96.png)

`LIKE`不加通配符和`=`没有区别：

```
SELECT prod_name FROM products WHERE prod_name LIKE '1000' ORDER BY prod_name;
```

![mark](http://imgblog.kuranado.com/blog/171112/H96Kfh4gKd.png)

MySQL的正则匹配同样是不区分大小写的，如要区分大小写需要在`REGEXP`后加上`BINARY`关键字：

```
SELECT prod_name FROM products WHERE prod_name REGEXP 'jetpack .000' ORDER BY prod_name;
```

![mark](http://imgblog.kuranado.com/blog/171112/0FaLJd5a63.png)

加上`BINARY`之后区分大小写，没有检索到匹配项，返回empty set

```
SELECT prod_name FROM products WHERE prod_name REGEXP BINARY 'jetpack .000' ORDER BY prod_name;
```

![mark](http://imgblog.kuranado.com/blog/171112/bA10Bh7keI.png)

### |-OR匹配

`|`是正则中的`OR`操作符，表示匹配其中之一

```
SELECT prod_name FROM products WHERE prod_name REGEXP '1000|2000' ORDER BY prod_name;
```

![mark](http://imgblog.kuranado.com/blog/171113/0df4h6EK02.png)

很多人不论在什么编程语言中都喜欢操作符和其两边的字符各留有一个空格，博主同样也有这种习惯，但对于SQL语句中单引号内操作符两边是否该保留空格需要仔细看清，不然就检索不到想要的结果了。比如下面这条语句多加了两个空格就变成了：检索prod_name包含`1000空格`或`空格2000`的产品了

```
SELECT prod_name FROM products WHERE prod_name REGEXP '1000 | 2000' ORDER BY prod_name;
```

![mark](http://imgblog.kuranado.com/blog/171113/jCB4FcDd89.png)

### []-匹配几个字符之一

```
SELECT prod_name FROM products WHERE prod_name REGEXP '[123] Ton' ORDER BY prod_name;
```

![mark](http://imgblog.kuranado.com/blog/171113/f2AgHmdj8H.png)

`[123]`表示匹配`1`或`2`或`3`，也就是说`[]`是`OR`的另一种形式，因此和下面的语句等价：

```
SELECT prod_name FROM products WHERE prod_name REGEXP '(1|2|3) Ton' ORDER BY prod_name;
```

另外正如前面提到的，如果不用`()`明确计算次序，你可能会犯下面的错误：

```
SELECT prod_name FROM products WHERE prod_name REGEXP '1|2|3 Ton' ORDER BY prod_name;
```

![mark](http://imgblog.kuranado.com/blog/171113/AeDBKK4Daa.png)

### -匹配范围

`[0123456789]`等价于`[0-9]`，`[a-z]`匹配任意字母字符

```
SELECT prod_name FROM products WHERE prod_name REGEXP '[1-5] Ton' ORDER BY prod_name;
```

![mark](http://imgblog.kuranado.com/blog/171113/cDhAC06ImE.png)

### 转义-匹配特殊字符

MySQL使用**`\\`**转义`|.[]\^)(`等特殊字符

检索供应商名称中包含`.`的供应商

```
SELECT vend_name FROM vendors WHERE vend_name REGEXP '\\.' ORDER BY  vend_name;
```

![mark](http://imgblog.kuranado.com/blog/171113/CD84f32I20.png)

### 匹配字符类

<center>字符类</center>

类|说明
-|
[:alnum:]|任意字母和数字（同`[a-zA-Z0-9]`）
[:alpha:]|任意字符（同`[a-zA-Z]`）
[:digit:]|任意数字（同`[0-9]`）
[:print:]|任意可打印字符
[:graph:]|与`[:print:]`相同，但不包括空格
[:lower:]|任意小写字母（同`[a-z]`）
[:upper:]|任意大写字母（同[A-Z]）
[:xdigit:]|任意十六进制数字（同[a-fA-F0-9]）
[:blank:]|空格和制表（同`[\\t]`）
[:cntrl:]|ASCII控制字符
[:space:]|包括空格在内的任意空白字符（同`[\\f\\t\\r\\v\\n]`）

### 匹配多个实例

<center>重复元字符</center>

元字符|说明
-|
*|0个或多个匹配
+|1个或多个匹配(等价于`{1,}`)
?|0个或1个匹配(等价于`{0,1}`)
{n}|指定数目的匹配
{n,}|不少于指定数目的匹配(>=n)
{n,m}|匹配数目的范围(大于等于n且小于等于m,且m<=255)

上表和JDK API中的一模一样，所以还是非常好记的

```
SELECT prod_name FROM products WHERE prod_name REGEXP '\\([0-9] sticks?)' ORDER BY prod_name;
```

![mark](http://imgblog.kuranado.com/blog/171113/a5bKEcHCLJ.png)

下面的SQL语句匹配连在一起的4位数字，其中`[:digit:]`匹配任意数字，等价于`[0-9]`，`{4}`要求它前面的字符出现4次，所以`[[:digit:]]{4}`匹配连在一起的4位数字：

```
SELECT prod_name FROM products WHERE prod_name REGEXP '[[:digit:]]{4}' ORDER BY prod_name;
```

![mark](http://imgblog.kuranado.com/blog/171113/a0lfc8cjEe.png)

上面这个例子还可以这样实现，检索结果是完全相同的：

```
SELECT prod_name FROM products WHERE prod_name REGEXP '[0-9][0-9][0-9][0-9]' ORDER BY prod_name;
```

### 定位符

 <center>定位元字符</center>
 
元字符|说明
-|
^|文本的开始
&|文本的结尾
[[:<:]]|词的开始
[[:>:]]|词的结尾

检索出产品名以一个数字开始或以`.`开始的产品

```
SELECT prod_name FROM products WHERE prod_name REGEXP '^[0-9\\.]' ORDER BY prod_name;
```

![mark](http://imgblog.kuranado.com/blog/171113/c9I2ga92Hk.png)

### 正则测试技巧

可以在不使用数据表的情况下使用`SELECT`测试正则表达式，返回`1`表示匹配，返回`0`不匹配

```
SELECT 'hello 2333' REGEXP '[0-9]';
```

![mark](http://imgblog.kuranado.com/blog/171113/8JK41kkFJA.png)

## 创建计算字段

### 拼接字段

MySQL使用`CONCAT()`函数拼接列，各个串之间使用`,`分隔，这一点和大多数DBMS使用`+`或`||`不同：

```
SELECT CONCAT(vend_name, ' (', vend_country, ')') FROM vendors ORDER BY vend_name;
```

![mark](http://imgblog.kuranado.com/blog/171120/ccBfaG4aIB.png)

最近工作上用到了 `CONCAT()` 函数拼接字符串，但却发现了 CONCAT 有一个值的注意的属性：

```
mysql> SELECT CONCAT('How', '-', 'old', '-', 'are', '-', 'you');
+---------------------------------------------------+
| CONCAT('How', '-', 'old', '-', 'are', '-', 'you') |
+---------------------------------------------------+
| How-old-are-you                                   |
+---------------------------------------------------+
1 row in set (0.00 sec)

mysql> SELECT CONCAT('How', '-', 'old', '-', 'are', '-', 'you', NULL);
+---------------------------------------------------------+
| CONCAT('How', '-', 'old', '-', 'are', '-', 'you', NULL) |
+---------------------------------------------------------+
| NULL                                                    |
+---------------------------------------------------------+
1 row in set (0.00 sec)
```

发现问题了没有？
那就是 CONCAT 函数中只要有任意一个参数为 NULL，则整个返回结果为 NULL，然而很多时候这并不是我们所期望的。通常都会认为为 NULL 的字段应该自动被忽略，而不应该是让整个返回值都为 NULL。所幸 MySQL 中有 `IFNULL()` 函数：

```
mysql> SELECT CONCAT('How', '-', 'old', '-', 'are', '-', 'you', IFNULL(NULL, ''));
+---------------------------------------------------------------------+
| CONCAT('How', '-', 'old', '-', 'are', '-', 'you', IFNULL(NULL, '')) |
+---------------------------------------------------------------------+
| How-old-are-you                                                     |
+---------------------------------------------------------------------+
1 row in set (0.00 sec)
```

IFNULL 函数当第一个参数为 NULL 时，会自动用第二个参数代替第一个参数返回

除此之外 `CONCAT_WS()` 函数也是一个很有用的函数：

```
mysql> SELECT CONCAT_WS('-', 'How', 'old', 'are', 'you');
+--------------------------------------------+
| CONCAT_WS('-', 'How', 'old', 'are', 'you') |
+--------------------------------------------+
| How-old-are-you                            |
+--------------------------------------------+
1 row in set (0.00 sec)

mysql> SELECT CONCAT_WS('-', 'How', 'old', 'are', 'you', NULL);
+--------------------------------------------------+
| CONCAT_WS('-', 'How', 'old', 'are', 'you', NULL) |
+--------------------------------------------------+
| How-old-are-you                                  |
+--------------------------------------------------+
1 row in set (0.00 sec)
```

可见 CONCAT_WS 函数使用第一个参数作为连接符连接其他参数，同时会自动忽略为 NULL 的字段


### AS-别名

`AS`用于给检索结果起别名

```
SELECT CONCAT(TRIM(vend_name), ' (', TRIM(vend_country), ')') AS vend_title FROM vendors ORDER BY vend_name;
```

![mark](http://imgblog.kuranado.com/blog/171120/eD5e85kEbC.png)

其中`TRIM()`用于去除串两边的空格，`LTRIM()`和`RTRIM()`分别用于去除串左边和右边的空格

### 执行算术计算

```
SELECT prod_id, quantity, item_price, quantity * item_price AS expanded_price FROM orderitems WHERE order_num = 20005;
```

## 数据处理函数

### 文本处理函数

<center>常用文本处理函数</center>

函数|说明
-|
LEFT()|返回串左边的字符（需要加参数指定返回几个字符）
RIGHT()|返回串右边的字符（需要加参数指定返回几个字符）
TRIM()|去除串两边的空格
LTRIM()|去除串左边的空格
RTRIM()|去除串右边的空格
LOWER()|将串转为小写
UPPER()|将串转为大写
LENGTH()|返回串长度
LOCATE()|找出子串substr在串str中的索引
SUBSTRING()|返回子串的字符

其中`LEFT()`函数举例如下：

```
SELECT LEFT(vend_name, 2) FROM vendors;
```

![mark](http://imgblog.kuranado.com/blog/171120/jdmejEiJbd.png)

`LOCATE`函数在[官方手册中的声明](https://dev.mysql.com/doc/refman/5.7/en/string-functions.html#function_locate)为:`LOCATE(substr,str), LOCATE(substr,str,pos)`，其中第一个用于查找substr在str中第一次出现的索引（此处索引从1开始数），第二个为从`pos`后开始substr在str中第一次出现的索引（同样从1开始计数），若找不到返回0，substr或str为`NULL`则返回NULL

```
SELECT LOCATE('bar', 'foobarbar');  //返回4
SELECT LOCATE('bar', 'foobarbar', 7);  //返回7
SELECT LOCATE('bar', 'foobarbar', 8);  //返回0
SELECT LOCATE('bar', NULL);  //返回NULL
```

### 日期和时间处理函数

使用日期和时间处理函数使请尽量使用`yyyy-MM-dd HH:mm:ss`格式

<center>常用日期时间处理函数</center>

函数|说明
-|
ADDDATE()|增加一个日期
ADDTIME()|增加一个时间
CURDATE()|返回当前日期
CURTIME()|返回当前时间
NOW()|返回当前日期和时间
DATE()|返回日期时间的日期部分
TIME()|返回日期时间的时间部分
YEAR()|返回日期的年部分
MONTH()|返回日期的月份部分
DAY()|返回一个日期的天数部分
HOUR()|返回时间的小时部分
MINUTE()|返回时间的分钟部分
SECOND()|返回时间的秒部分
DATEDIFF()|计算两个日期之差(第一个日期减第二个日期)
DATE_ADD()|高度灵活的日期运算函数
DATE_FORMAT()|返回一个格式化的日期或时间串
DAYOFWEEK()|返回日期对应星期几

检索出2005年9月的所有订单如下两条语句都可以：

```
SELECT cust_id, order_date FROM orders WHERE DATE(order_date) BETWEEN '2005-09-01' AND '2005-09-30';
SELECT cust_id, order_date FROM orders WHERE YEAR(order_date) = 2005 ADN MONTH(order_date) = 9;
```

### 数值处理函数

<center>常用数值处理函数</center>

函数|说明
-|
ABS()|绝对值
COS()|余弦
SIN()|正弦
TAN()|正切
MOD()|求余
SQRT()|平方根
PI()|圆周率

MySQL内置函数有很多，具体可查阅MySQL官方文档中的函数手册：[https://dev.mysql.com/doc/refman/5.7/en/func-op-summary-ref.html](https://dev.mysql.com/doc/refman/5.7/en/func-op-summary-ref.html)

## Appendix

- MySQL以`\g`结束等价于以`;`结束
- 并不是所有的MySQL语句都需要以`;`结束，但建议总是加上`;`
- SQL语句虽然不区分大小写（MySQL4.1.1之后版本），但建议SQL关键字使用大写，而对所有的列和表名使用小写
- 将较长的SQL语句分成多行便于阅读和调试
- SQL语句一般返回原始的无格式的数据，所以表示（如用货币符号表示查询出的金额）一般在显示该数据的应用程序中规定
- MySQL中多个子句同时出现时，顺序依次为：`FROM`子句,`WHERE`子句,`ORDER BY`子句,`LIMIT子句`
- 任何时候具有`AND`和`OR`操作符的`WHERE`子句，**都应该使用圆括号明确的分组操作符**，不要过分依赖默认计算次序，即使它确实是你想要的东西。
- `LIKE`与`REGEXP`的不同在于`LIKE`匹配整个串而`REGEXP`匹配子串，当然如果正则使用`^`开始每个表达式，`$`结束每个表达式，则`REGEXP`和`LIKE`相同