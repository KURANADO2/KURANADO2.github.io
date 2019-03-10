---
title: MySQL必知必会读书笔记(二)
date: 2017-12-10 22:10:00
comments: true 
categories: [MySQL]
tags: [MySQL, 数据库]
---

这段时间有点懒！

<!-- more -->

## 聚集函数

### AVG

`AVG`函数用于求平均值

```
SELECT AVG(prod_price) avg_price FROM products;  //返回所有产品的平均价格
SELECT AVG(prod_price) FROM products WHERE vend_id = '1003';  //返回1003供应商所有产品的平均价格
```

![mark](http://imgblog.kuranado.com/blog/171210/gHcGakg3BJ.png)

需要注意的是，`AVG`函数会**忽略值为`NULL`的行**

### COUNT

`COUNT`函数用于计数，可用于确定表中的记录数（记录数即行数）或符合条件的记录数

`COUNT`函数有两种使用方式：

1. 使用`COUNT(*)`对表中行的数据进行计数，`不管表列中包含的是否是NULL`
2. 使用`COUNT(columnname)`对特定列中具有值的行计数，也就是说会忽略`NULL`值

```
SELECT COUNT(*) AS num_cust FROM customers;  //返回所有客户数
SELECT COUNT(cust_email) AS num_cust FROM customers;  //返回拥有email的客户数
```

![mark](http://imgblog.kuranado.com/blog/171210/8kegImEg83.png)

因为曾经碰到过用`SELECT COUNT(1)`代替`SELECT COUNT(*)`的情况，所以当时上网查了一下它们的区别并做了笔记：

- 如果表没有主键，那么`COUNT(1)`比`COUNT(*)`快
- 如果表有主键的话，那么`COUNT(PrimaryKey)`最快
- 如果表中只有一个字段，那么`COUNT(*)`最快
- `COUNT(1)`和`COUNT(*)`都包括对NULL值的统计

### MAX&MIN

`MAX`要求指定列名，用于返回指定列中的最大值，与之对应的为`MIN`函数，用于返回指定列的最小值，**`MAX`和`MIN`均会忽略值为`NULL`的行**

```
SELECT MAX(prod_price) AS max_price FROM products;
SELECT MIN(prod_price) AS min_price FROM products;
```

### SUM

`SUM`函数用于返回指定列值的总和，**`SUM`函数会忽略值为`NULL`的行**

```
SELECT SUM(quantity) AS items_ordered FROM orderitems WHERE order_num = '20005';  //返回指定订单下所购物品总数
```

![mark](http://imgblog.kuranado.com/blog/171210/j1bhc7Akge.png)

```
SELECT SUM(quantity * item_price) AS total_price FROM orderitems WHERE order_num = '20005';  //计算订单金额
```

![mark](http://imgblog.kuranado.com/blog/171210/Gc80lF4Fah.png)

### DISTINCT-聚集不同值

```
SELECT AVG(DISTINCT prod_price) AS avg_price FROM products WHERE vend_id = '1003';  //返回指定供应商的产品平均价格，不计算重复的价格
```

### 组合聚集函数

```
SELECT COUNT(prod_id) AS num_items,
       MIN(prod_price) AS min_price,
       MAX(prod_price) AS max_price,
       AVG(prod_price) AS avg_price
       FROM products;
```

![mark](http://imgblog.kuranado.com/blog/171210/E0j4jdeLAH.png)

## 分组数据

### GROUP BY-创建分组

```
SELECT vend_id, COUNT(*) AS num_products
       FROM products
       GROUP BY vend_id;
```

![mark](http://imgblog.kuranado.com/blog/171210/Ag4Ef5hIka.png)

### HAVING-过滤分组

`WHERE`只能过滤指定的是行而不能过滤分组，如果想要过滤分组需要使用`HAVING`关键字

```
SELECT cust_id, COUNT(*) AS orders FROM orders GROUP BY cust_id HAVING COUNT(*) >= 2;
```

![mark](http://imgblog.kuranado.com/blog/171210/6kjE8eaDlg.png)

当`LIMIT`，`ORDER BY`，`HAVING`，`WHERE`和`GROUP BY`同时出现时，前后顺序依次为：`WHERE`，`GROUP BY`，`HAVING`，`ORDER BY`，`LIMIT`

下面为列出具有2个以上，价格为10以上的产品的供应商的SQL语句，其中同时出现了`WHERE`和`HAVING`

```
SELECT vend_id, COUNT(*) AS num_prods FROM products WHERE prod_price >= 10 GROUP BY vend_id HAVING COUNT(*) >=2;
```

![mark](http://imgblog.kuranado.com/blog/171210/IE4K2bBLK9.png)

按总计订单价格排序输出，同时出现`GROUP BY`，`HAVING`和`ORDER BY`的SQL语句：

```
SELECT order_num, SUM(quantity * item_price) AS ordertotal 
       FROM orderitems 
       GROUP BY order_num 
       HAVING ordertotal >= 50 
       ORDER BY ordertotal;
```

![mark](http://imgblog.kuranado.com/blog/171210/JHA4jaIL3h.png)

## 子查询

列出订购物品TNT2的所有客户过程如下：

```
SELECT order_num FROM orderitems WHERE prod_id = 'TNT2';
```

![mark](http://imgblog.kuranado.com/blog/171210/BgLEJ26L57.png)

```
SELECT cust_id FROM orders WHERE order_num IN (20005, 20007);
```

![mark](http://imgblog.kuranado.com/blog/171210/968EI58IlA.png)

```
SELECT cust_name, cust_contact FROM customers WHERE cust_id IN (10001, 10004);
```

![mark](http://imgblog.kuranado.com/blog/171210/1lImgj8aF0.png)

使用子查询实现如下：

```
SELECT cust_name, cust_contact
FROM customers
WHERE cust_id IN (SELECT cust_id
                  FROM orders
                  WHERE order_num IN (SELECT order_num
                                     FROM orderitems
                                     WHERE prod_id = 'TNT2'));
```

![mark](http://imgblog.kuranado.com/blog/171210/gakL0J43c6.png)

MySQL对子查询的嵌套数目没有限制，但是因为会影响性能，所以不建议嵌套太多。

统计customers表中每个客户的订单总数：

```
SELECT cust_name, cust_statE, (
       SELECT COUNT(*)
       FROM orders
       WHERE orders.cust_id = customers.cust_id)AS orders
       FROM customers
       ORDER BY cust_name;
```

![mark](http://imgblog.kuranado.com/blog/171212/dFKJJH60jj.png)

## 联结表

### INNER JOIN-内联结

内联结分为显式内联结和隐式内联结：

隐式内联结：

```
SELECT vend_name, prod_name, prod_price
FROM vendors, products
WHERE vendors.vend_id = products.vend_id;
```

显式内联结：

```
SELECT vend_name, prod_name, prod_price
FROM vendors INNER JOIN products
ON vendors.vend_id = products.vend_id;
```

这两种内联结的写法不同，但效果是等价的，`ANSI SQL`规范中推荐使用显式内联结，因为显示内联结的语法不容易让人忘记连接条件，但就我目前所接触过的SQL语句中还是隐式内联结用的更多一些。

另外小伙伴们可能会发现，如果把上面的`INNER JOIN`替换成`CROSS JOIN（叉联结）`，返回结果是完全相同的，需要提醒的是在标准SQL中，`CROSS JOIN`和`INNNER JOIN`并不等价，标准SQL中，`CROSS JOIN`就表示笛卡尔积，不能加连接条件，但MySQL为了增加SQL语句的容错能力，允许`CROSS JOIN`加上连接条件进行数据过滤。

### 自联结

发现某产品（其ID为DTNTR）存在问题，想查出该物品的供应商及该供应商生产的其他产品是否也存在问题，可以使用子查询实现如下：

```
SELECT prod_id, prod_name
FROM products
WHERE vend_id = (SELECT vend_id
	             FROM products
	             WHERE prod_id = 'DTNTR');
```

![mark](http://imgblog.kuranado.com/blog/171216/Hig52iElDB.png)

上面的子查询执行过程是先找出生产DTNTR产品的供应商，再根据供应商的ID查找出该供应商生产的所有产品

下面使用自联结的方式实现同样的需求：

```
SELECT p1.prod_id, p1.prod_name
FROM products AS p1, products AS p2
WHERE p2.prod_id = 'DTNTR' AND p2.vend_id = p1.vend_id;
//或者
SELECT p1.prod_id, p1.prod_name
FROM products AS p1 INNER JOIN products AS p2
ON p2.prod_id = 'DTNTR' AND p2.vend_id = p1.vend_id;
```

可以看到自联结可以实现和子查询一样的效果，但是因为MySQL处理联结的速度一般比处理子查询的速度快的多，所以像上面这种情况推荐小白优先使用自联结，在项目中则最好是分别测试两种方法以确定哪种性能更优

### LEFT OUTER JOIN-左外连接&RIGHT OUTER JOIN-右外连接

`A LEFT OUT JOIN B` 会选择A表中的所有行，对应的，`A RIGHT OUT JOIN B`则会选择B表中的所有行，`OUT`关键字是可以省略的

检索所有客户及每个客户所下的订单数：

```
SELECT c.cust_id, c.cust_name, COUNT(o.order_num) AS ord_num
FROM customers c LEFT OUTER JOIN orders o
ON c.cust_id = o.cust_id
GROUP BY c.cust_id;
```

![mark](http://imgblog.kuranado.com/blog/171216/1F4J1egmbl.png)

可以看到Mouse House的订单数为0，即在orders表中没有该客户，如果直接使用内联结，就不会查到该用户，但使用LEFT OUTER JOIN把customers表放在左边或使用RIGHT OUTER JOIN把customers表放在右边，就能够查到所有的客户

## 组合查询

### UNION

`UNION`关键字用来合并多条`SELECT`语句的查询结果

查出价格小于等于5或者是由供应商1001或1002生产的所有物品

首先想到的是直接使用`OR`关键字：

```
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <=5 OR vend_id IN (1001, 1002);
```

![mark](http://imgblog.kuranado.com/blog/171216/79mk0EGK8k.png)

为了使条件逻辑更清晰，使用`UNION`关键字实现同样效果：

```
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
UNION
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001, 1002);
```

很明显使用`UNION`关键字比直接在`WHERE`后写完所有条件要麻烦很多，但优点是条件判断一目了然，设想如果需要查询的是多张表或者`WHERE`后有很多的条件，这种情况下很容易写出错误的SQL语句，而使用`UNION`就会使出错率大大降低

### UNION ALL

上面的SQL语句如果拆成两部分分别执行，会发现一个返回4条记录，一个返回5条记录，而使用`UNION`返回的结果是8条，这就是因为`UNION`自动为我们去除了重复的记录，如果把`UNION`替换成`UNION ALL`则执行结果如下：

![mark](http://imgblog.kuranado.com/blog/171216/m4Agb3D6Lc.png)

`UNION`关键字会从查询结果集中去除重复的行，但`UNION ALL`会返回所有匹配行，最近看了篇关于[SQL的面试题](https://www.toptal.com/sql/interview-questions)，其中就有关于`UNION`和`UNION ALL`区别的问题，其中有提到因为`UNION ALL`不需要像`UNION`那样需要做去除重复行的处理，所以执行效率上`UNION ALL`是优于`UNION`的，另外最近我发现公司的数据库表中保存的所有业务SQL语句全部使用的是`UNION ALL`而不是`UNION`

## INSERT-插入数据

### 插入多个行

```
INSERT INTO customers(
cust_name, 
cust_address) VALUES(
'Pep E. LaPew',
'100 Main Street');

INSERT INTO customers(
cust_name, 
cust_address) VALUES(
'M. Martian',
'42 Galaxy Way');
```

MySQL用单条`INSERT`语句处理多个插入比使用多条`INSERT`语句快：

```
INSERT INTO customers(
cust_name, 
cust_address) VALUES(
'Pep E. LaPew',
'100 Main Street'), (
'M. Martian',
'42 Galaxy Way');
```

### 插入检索出的值

```
INSERT INTO customers(
cust_id, 
cust_contact, 
cust_email, 
cust_name, 
cust_address, 
cust_city, 
cust_state, 
cust_zip, 
cust_country) SELECT 
cust_id, 
cust_contact, 
cust_email, 
cust_name, 
cust_address, 
cust_city, 
cust_state, 
cust_zip, 
cust_country 
FROM custnew;
```

这里`SEELCT`后的列名和`INSERT`后的列名保持一致，实际上MySQL并不在意列名是否能对的上，MySQL只会将`SELECT`后的第一列的值（不管它的列名是什么）插入到`INSERT`关键字后的第一个列中，以此类推，另外此处使用custnew表需要自己创建并添加数据，注意不要让两张表的主键产生冲突

## UPDATE&DELETE-更新和删除数据

说到`UPDATE`和`DELETE`，执行前可是要非常小心的，这点我深有体会：在操作公司数据库时，我就特智障的写了条`UPDATE`语句，没想清楚就回车执行了，结果错误更新表中的四千多条记录，当时可把我吓的不轻，脑海中还闪现出要不要立刻辞职的念头，不过所幸公司用的是Oracle数据库，在网上查了资料之后，得知Oracle的[Flashback技术](https://docs.oracle.com/cd/B28359_01/appdev.111/b28424/adfns_flashback.htm#i1009447)就是用来恢复数据的，所以很快就把数据恢复到了之前的状态，翻了下之前保存的脚本，找到了我当时执行的那条错误的更新语句：

```
UPDATE GP_BM_DATA_DIC SET data_type_no = '210050399' WHERE data_no_len = '5';
```

没错，就是把GP_BM_DATA_DIC这张表下所有data_no_len等于5的data_type_no字段改成了同一个值，而实际上我需要更新的只有1条记录，结果却更新了四千多条，都是因为我写错了`WHERE`条件导致的，查了资料之后我执行了下面这条语句，也就是把20分钟前的数据更新到现在的表中：

```
UPDATE GP_BM_DATA_DIC t1 
SET t1.data_type_no = (SELECT t2.data_type_no 
FROM GP_BM_DATA_DIC t2 AS OF TIMESTAMP SYSDATE - 20 / 1440 
WHERE t1.data_id = t2.data_id)
```

这件事告诉我在执行`UPDATE`和`DELETE`之前是需要非常谨慎的，上面是我在Oracle中恢复数据的经验，类似的特性在MySQL中也有，但这种特性有时并不是万能的，所以执行前还是先用`SELECT`测试一下看看是不是自己想要操作的数据，避免一些不必要的麻烦

### UPDATE

更新多个列的值：

```
UPDATE customers
SET cust_name = 'The Fudds',
	cust_email = 'elmer@fudd.com'
WHERE cust_id = 10005;
```

### DELETE

删除表中指定数据：

```
DELETE FROM customers WHERE cust_id = 10006;  //删除客户id为10006的记录
```

删除表中所有数据：

```
TRUNCATE customers;
```

这里使用了`TRUNCATE`而不是`DELETE`是因为前者实际上是直接删除原来的表然后重新创建一张表，而后者则是逐行删除表中的数据，显然在表中数据量很大时，前者的执行效率更高。

使用`DELETE`删除整表数据语法如下：

```
DELETE FROM customers;
```

可见在写`DELETE`语句时，一旦忘记加上`WHERE`条件就会导致整张表的数据都被删除掉！

## 创建和操纵表

### CREATE-创建表

```
CREATE TABLE IF NOT EXISTS emp (
        emp_id INT NOT NULL AUTO_INCREMENT,
        emp_name CHAR(50) NOT NULL,
        emp_address CHAR(50) NOT NULL DEFAULT 'New York',
        emp_salary DECIMAL(8, 2) NULL,
        PRIMARY KEY(emp_id)
) ENGINE = InnoDB;
```

`IF NOT EXISTS`用于判断该表在数据库中是否已经存在，若已经存在，则不会再创建，可以省略；`AUTO INCREMENT`	自动增量，如果手动增加一条记录，并赋值一个ID（不要与已有的记录ID重复），则以后再添加记录时，将以该ID作为起始值递增，`SELECT LAST_INSERT_ID();`可以获得最后插入的那条记录的id；`DEFAULT`用于当该列值为NULL空时为其赋默认值；PRIMARY KEY用于指定主键列，若是联合主键，则多个列之间用逗号分隔；`ENGINE`关键字用于指定存储引擎，关于MySQL中各存储引擎的简单区别可参照下图（图片来源于慕课网）：

![mark](http://imgblog.kuranado.com/blog/171218/E9BGBkmJ7a.png)

其中最常用的就是`MyISAM`和`InnoDB`

### ALTER-修改表

为vendors表新增一列并指定其数据类型：

```
ALTER TABLE vendors ADD vend_phone CHAR(20);
```

添加外键（此代码在下载下来的create.sql文件中有）：

```
ALTER TABLE orderitems ADD CONSTRAINT fk_orderitems_orders FOREIGN KEY (order_num) REFERENCES orders (order_num);
ALTER TABLE orderitems ADD CONSTRAINT fk_orderitems_products FOREIGN KEY (prod_id) REFERENCES products (prod_id);
ALTER TABLE orders ADD CONSTRAINT fk_orders_customers FOREIGN KEY (cust_id) REFERENCES customers (cust_id);
ALTER TABLE products ADD CONSTRAINT fk_products_vendors FOREIGN KEY (vend_id) REFERENCES vendors (vend_id);
```

### DROP-删除表

删除emp表：

```
DROP TABLE emp;  
```

### RENAME-重命名表

重命名单张表：

```
//将customers表重命名为customers2;
RENAME TABLE customers TO customers2;
```

重命名多张表：
 
```
RENAME TABLE customers TO customers2,
			 vendors TO vendors2,
			 products TO products2;
```