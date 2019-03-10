---
title: MySQL必知必会读书笔记(三)
date: 2017-12-19 22:10:00
comments: true
categories: [MySQL]
tags: [MySQL, 数据库]
---

主要总结视图、触发器、存储过程的使用方法，尤其是存储过程，在公司里用的会非常多

<!-- more -->

## VIEW-视图

记得之前面试一家小的创业公司时曾被面试官问道“用过视图和触发器吗”，因为只是上课时听老师讲过并没有实际使用过所以所以直接一句“没用过！”给带过去了，其实当时我是非常中意那家公司的，数了一下公司上下包括HR只有11个人，公司人少不会给人太多的约束感，而且面试官所问的问题都是关于目前比较流行的技术，小公司更倾向于使用新的技术，进去是肯定能学到很多东西的，可惜最后还是没有收到Offer

### 为什么使用视图

关于为什么使用视图，这里贴一个[Stack Overflow上的回答](https://stackoverflow.com/questions/1278521/why-do-you-create-a-view-in-a-database)：

- Views can hide complexity

If you have a query that requires joining several tables, or has complex logic or calculations, you can code all that logic into a view, then select from the view just like you would a table.

- Views can be used as a security mechanism

A view can select certain columns and/or rows from a table, and permissions set on the view instead of the underlying tables. This allows surfacing only the data that a user needs to see.

- Views can simplify supporting legacy code

If you need to refactor a table that would break a lot of code, you can replace the table with a view of the same name. The view provides the exact same schema as the original table, while the actual schema has changed. This keeps the legacy code that references the table from breaking, allowing you to change the legacy code at your leisure.

其中1、2两点是很多教材中都有提到过的，唯独第3点作为`VIEW`最大的特性却很少被提到。当然视图也有一些缺点：

- 使用视图更新基表数据有很多限制

视图本身是可以更新的（视图本身没有数据，对视图更新实际上是对其基表数据更新），但如果视图中包含了`GROUP BY`,`HAVING`,`JOIN`,`子查询`,`UNION`,`聚集函数`等，将会导致该视图无法更新基表数据，因此有很多人都觉得操作视图还不如直接操作表方便，但因为视图通常只用来做查询，所以我倒并不觉得这是一个缺点

- 视图作为对象存储在数据库中需要占用存储空间
- 因为视图依赖于表，如果表删除了，视图也将不可用
- 从视图中查找数据一般比直接从表中查找数据的效率要低

### CREATE VIEW-创建视图

**列出已订购了任意产品的所有客户：**

```
SELECT cust_name, cust_contact, prod_id
FROM customers, orders, orderitems
WHERE customers.cust_id = orders.cust_id AND orders.order_num = orderitems.order_num;
```

![mark](http://imgblog.kuranado.com/blog/171219/dkk2A92Eim.png)

显然每次需要查询这些信息时都需要写这么一大串`SELECT`语句是非常麻烦的，所以下边把这条SQL语句放在视图中：

```
CREATE VIEW productcustomers AS
SELECT cust_name, cust_contact, prod_id
FROM customers, orders, orderitems
WHERE customers.cust_id = orders.cust_id AND orders.order_num = orderitems.order_num;
```

然后直接从视图中取需要的值就会让查询变得非常简单： 

```
SELECT cust_name, prod_id
FROM productcustomers
WHERE prod_id = 'TNT2';
```

![mark](http://imgblog.kuranado.com/blog/171219/7i5J08C5a1.png)


**使用视图格式化数据：**

```
CREATE VIEW vendorlocations AS
SELECT CONCAT(TRIM(vend_name), ' (', TRIM(vend_country), ')') AS vend_title
FROM vendors
ORDER BY vend_name;

SELECT * FROM vendorlocations;
```

![mark](http://imgblog.kuranado.com/blog/171219/Ig9d195bB8.png)

**使用视图过滤数据：**

```
CREATE VIEW customeremaillist AS
SELECT cust_id, cust_name, cust_email
FROM customers
WHERE cust_email IS NOT NULL;

SELECT * FROM customeremaillist;
```

![mark](http://imgblog.kuranado.com/blog/171219/0ke73laIaK.png)

### SHOW CREATE VIEW-查看创建视图的语句

`SHOW CREATE TABLE tablename;`用来查看创建表的SQL语句，类似的，`SHOW CREATE VIEW viewname;`用来查看创建视图的SQL语句：

```
SHOW CREATE VIEW custoemremaillist;
```

### DROP-删除视图

`DROP TABLE tablename;`用来删除表；`DROP VIEW viewname;`用来删除视图：

```
DROP VIEW custoemremaillist;
```

### 更新视图

这里说的更新视图并不是指通过视图来更新基表中的数据，虽然MySQL可以这样做，但却是极不推荐的做法，这里的更新视图仅仅是指若视图内的语句写错了该如何更改：

```
CREATE VIEW productcustomers AS
SELECT cust_name, cust_contact, prod_id
FROM customers, orders, orderitems
WHERE customers.cust_id = orders.cust_id AND orders.order_num = orderitems.order_num;
```

现在需要为下面这个视图增加一个查询字段，可以用`DROP VIEW`先删除这个视图然后重新创建，或者使用`CREATE OR REPLACE VIEW`替换原来的视图：

```
CREATE OR REPLACE VIEW productcustomers AS
SELECT cust_name, cust_contact, cust_address, prod_id
FROM customers, orders, orderitems
WHERE customers.cust_id = orders.cust_id AND orders.order_num = orderitems.order_num;

SELECT cust_name, cust_address FROM productcustomers;
```

## 存储过程

### CREATE PROCEDURE-创建存储过程

#### 无参存储过程

```
DELIMITER //
CREATE PROCEDURE productpricing()
BEGIN
        SELECT AVG(prod_price) AS priceaverage
        FROM products;
END //
DELIMITER ;
```

![mark](http://imgblog.kuranado.com/blog/171219/ghCJgch6E7.png)

`DELIMITER`关键字用于用于更改命令行实用程序的语句结束符，其中第1行代码将语句结束符更改为`//`（也可更改为其他符号，比如`$$`等），这样第5行代码的`;`将被当做是一个普通符号，第5行代码敲完回车并不会立即执行而是继续等待用户输入，输入第6行代码的`//`回车后，才会执行这条创建存储过程的SQL语句，第7行代码用于把SQL语句的结束符重新更改为`;`，如果是在`DataGrip`,`Navicat`等图形化界面下执行SQL语句则可以省略`DELIMITER`关键字，就像下面这样：

```
CREATE PROCEDURE productpricing()
BEGIN
        SELECT AVG(prod_price) AS priceaverage
        FROM products;
END;
```

另外创建的存储过程即使没有传入任何参数，圆括号也是不可以省略的

#### 有参存储过程

```
CREATE PROCEDURE productpricing(
	OUT pl DECIMAL(8, 2),
	OUT pa DECIMAL(8, 2),
	OUT ph DECIMAL(8, 2)
)
BEGIN
	SELECT MIN(prod_price) INTO pl FROM products;
	SELECT AVG(prod_price) INTO pa FROM products;
	SELECT MAX(prod_price) INTO ph FROM products;
END;
```

### CALL-调用存储过程

#### 调用无参存储过程

```
CALL productpricing();
```

![mark](http://imgblog.kuranado.com/blog/171219/3m3L5bGJja.png)

#### 调用有参存储过程

执行`CALL`语句时还不会输出任何内容：

```
CALL productpricing(@pricelow, @priceaverage, @pricehigh);
SELECT @pricelow;
SELECT @pricelow, @pricehigh, @priceaverage;
```

![mark](http://imgblog.kuranado.com/blog/171219/eccidL3m0e.png)

> MySQL中调用存储过程时的所有变量都必须以`@`开头

### DROP PROCEDURE-删除存储过程

```
DROP PROCEDURE IF EXISTS productpricing;
```

`IF EXISTS`关键字表示仅在存储过程存在时删除，该关键字可以省略

### IN&OUT&INTO

```
CREATE PROCEDURE ordertotal(
	IN onumber INT,
	OUT ototal DECIMAL(8, 2)
)
BEGIN
	SELECT SUM(quantity * item_price) FROM orderitems
	WHERE order_num = onumber
	INTO ototal;
END;
```

`IN`表示onumber将作为参数传入存储过程，`OUT`表示ototal将作为存储过程的计算结果返回，`INTO`表示将`SELECT`的计算结果放到`OUT`或`DECLARE`关键字后的变量内

下面调用这个存储过程：

```
CALL ordertotal(20005, @total);
SELECT(@total);
```

![mark](http://imgblog.kuranado.com/blog/171219/6HfDE0KH0c.png)

另外还有`INOUT`关键字既可以表示输入，又可以表示输出，相当于`IN`和`OUT`的结合，但我却并不推荐大家使用这个关键字

### IF-带有条件判断的存储过程

```
CREATE PROCEDURE ordertotal(
	IN onumber INT,
	IN taxable BOOLEAN,
	OUT ototal DECIMAL(8, 2)
)
BEGIN
	DECLARE tmptotal DECIMAL(8, 2);
	DECLARE taxrate INT DEFAULT 6;
	
	SELECT SUM(quantity * item_price) FROM orderitems
	WHERE order_num = onumber
	INTO tmptotal;

	IF taxable THEN
		SELECT tmptotal + (tmptotal / 100 * taxrate) INTO tmptotal;
	END IF;

	SELECT tmptotal INTO ototal;
END;
```

调用该存储过程：

```
//传入0表示收税
CALL ordertotal(20005, 0, @ototal);
SELECT @ototal;
//传入1表示不收税
CALL ordertotal(20005, 1, @ototal);
SELECT @ototal;
```

![mark](http://imgblog.kuranado.com/blog/171219/E0AmFA7CD9.png)

当然存储过程中也可以使用`ELSE`、`WHEN`、`CASE`等进行条件判断，简单用法可参考这篇博客：[Learn About MySQL Stored Procedures](http://www.codewebber.com/blog/learn-mysql-stored-procedures/)

关于存储过程看过《阿里巴巴Java开发手册》的同学可能会注意到下面这条强制规约：

![mark](http://imgblog.kuranado.com/blog/171220/kjJkffc4d7.png)

关于这个问题，也有人在知乎上提问过：[为什么阿里巴巴Java开发手册里要求禁止使用存储过程？](https://www.zhihu.com/question/57545650/answer/238513606)手册的主要作者[孤尽](https://www.zhihu.com/people/gujin159/activities)也对此做出了回答：

> 曾经写过近1200行的存储过程，没有办法断点，下层数据结构只是稍微变动，根本无法找到出错点，只是提示一下说：ERROR:1064啥的。在数据库迁移的时候，由于数据库版本变更，居然存储过程无法执行。另外，业务上需要扩展一下，那就是灾难性的啊。

存储过程1行代码可能相当于十几行的Java代码，而且性能会更优，但因为将业务逻辑写在存储过程中对数据库的压力会更大，除了Java代码还必须单独维护这部分的业务逻辑，随着业务扩展，存储过程中的代码势必会越来越多，难以调试（并不是指无法像Java代码那样对存储过程进行调试，借助一些可视化工具比如`DbVisualizer`是可以调试存储过程的）和维护，所以很多公司都禁止使用存储过程。不过，我司出于性能的考虑目前还是有在大量使用存储过程的，总之，这种东西就是`double-edged sword`，如何使用还是要视具体情况而定的

### SHOW CREATE PROCEDURE-查看创建存储过程的语句

```
SHOW CREATE PROCEDURE ordertotal;
```

### SHOW PROCEDURE STATUS-列出数据库中的所有存储过程

列出当前用户有权限访问的所有数据库中的所有存储过程的基本信息：

```
SHOW PROCEDURE STATUS;
```

![mark](http://imgblog.kuranado.com/blog/171226/H0di1d0aAa.png)

当然也可以加上`WHERE`条件：

```
SHOW PROCEDURE STATUS WHERE db LIKE '%course';
SHOW PROCEDURE STATUS WHERE name = 'ordertotal';
```


## TRIGGER-触发器

- 只有表才支持触发器，视图和临时表都不支持触发器
- `INSERT`,`UPDATE`,`DELETE`支持触发器，其他MySQL语句均不支持触发器

### CREATE TRIGGER-创建触发器

创建触发器的语法格式为：

```
CREATE TRIGGER trigger_name trigger_time trigger_event
ON table_name
FOR EACH ROW
BEGIN
...
END;
```

其中trigger_event(触发事件)有以下6种（MySQL5.7.2版本之前，5.7.2之后的版本可以使用更多的触发器）：

- `BEFORE INSERT` - 数据插入表前触发
- `AFTER INSERT` - 数据插入表后触发
- `BEFORE UPDATE` - 数据更新前触发
- `AFTER UPDATE` - 数据更新后触发
- `BEFORE DELETE` - 数据删除前触发
- `AFTER DELETE` - 数据删除后触发

MySQL必知必会中作者举的创建触发器的例子如下：

```
CREATE TRIGGER newproduct AFTER INSERT
ON products
FOR EACH ROW
SELECT 'Product added';
```

但实际执行这条语句时会报错：`ERROR 1415 (0A000):Not allowed to return a result set from a trigger` ，因为在触发器中允许调用存储过程、执行`INSERT`,`UPDATE`,`DELETE`等操作，但却唯独不允许返回任何结果，如果存储过程中返回结果却又被触发器调用也会报同样的错误，就像下面这样：

```
DELIMITER $
CREATE PROCEDURE emp_procedure()
BEGIN
  SELECT 'New product added';
END $
DELIMITER ;

DELIMITER $
CREATE TRIGGER emp_trigger AFTER INSERT
ON emp
FOR EACH ROW
BEGIN
CALL emp_procedure();
END $
DELIMITER ;

INSERT INTO emp VALUES (NULL, 'Huang', 'Taiwan', 8000);
```

上面的代码期望每向员工表中插入一条记录就返回`New product added`：当执行`INSERT`语句后会触发触发器定义的事件调用`emp_procedure`存储过程，但是因为该存储过程调用了`SELECT`返回结果，所以会返回`ERROR 1415 (0A000):Not allowed to return a result set from a trigger`错误，而不是预期结果

关于书中的这个错误，我发了封邮件给作者，当天晚上10点多收到了作者的回复（此处请忽略我蹩脚的英语，作为完全不懂语法的英语渣已经很努力把这些单词凑到一块了）：

![mark](http://imgblog.kuranado.com/blog/171226/hFmcGbkbGK.png)

之后我也在[MySQL官网找到了相应的明文规定](https://dev.mysql.com/doc/mysql-reslimits-excerpt/5.7/en/stored-program-restrictions.html)：

> **The RETURN statement is not permitted in triggers, which cannot return a value**. To exit a trigger immediately, use the LEAVE statement. 

### OLD虚拟表

`OLD`是MySQL中的虚拟表，`OLD`表中的值只允许读，不允许更新

使用`OLD`实现保存将要删除的行到一个备份表（为此需要先创建一个备份表）：

```
DELIMITER $
CREATE TRIGGER delete_order BEFORE DELETE
ON orders
FOR EACH ROW
BEGIN
INSERT INTO backup_orders (order_num, order_date, cust_id)
VALUES (OLD.order_num, OLD.order_date, OLD.cust_id);
END $
DELIMITER ;

#因为存在外键约束，执行下面这条DELETE语句时，需要先删除orderitems表中对应的记录
DELETE FROM orders WHERE cust_id = '1003';

SELECT * FROM backup_orders;
```

![mark](http://imgblog.kuranado.com/blog/180103/bk3EjHKgdI.png)

### DROP TRIGGER-删除触发器

```
DROP TRIGGER emp_trigger;
```

### SHOW CREATE TRIGGER-查看创建触发器的SQL语句

语法格式为：

```
SHOW CREATE TRIGGER trigger_name;
```

### SHOW TRIGGERS-查看触发器

查看指定数据库下的触发器，若不指定数据库名，则默认查找当前数据库下的所有触发器（SQL语句的定义方法中`[]`用来表示可选项，`{}`表示必填项，`|`表示或者）：

```
SHOW TRIGGERS [{FROM | IN} db_name];
```

例如：

```
SHOW TRIGGERS FROM crashcourse;
```

![mark](http://imgblog.kuranado.com/blog/180103/8LB075h7LA.png)

当然也可以使用`LIKE/WHERE`条件：

```
SHOW TRIGGERS FROM crashcourse LIKE '%order%';
SHOW TRIGGERS FROM crashcourse WHERE `table` = 'orders';
```

此处需要注意的有两点：
1. `LIKE`条件匹配的是触发器所作用的表名，而不是触发器的名称
2. `WHERE`条件后的`table`条件需要使用`backtick(反引号&#96;)`(就是标准键盘Tab键上面那个键)而不是`'`括起来

## TRANSACTION-事务

`MySQL`中使用事务有一点需要明确：`MySQL`只有`InnoDB`存储引擎支持明确的事务处理

事务相关基本命令：

- `START TRANSACTION` 开启事务
`BEGIN`和`BEGIN WORK`是`START TRANSACTION`的别名，同样也可以开启事务
- `ROLLBACK` 回滚当前事务
关于事务回滚需要了解3点：
	1. 只针对当前事务起作用
	2. 执行的`DDL`语句（如创建、删除数据库，创建、删除、修改表，存储过程等）是不能回滚的
	3. 已执行`COMMIT`命令的事务不能回滚
- `COMMMIT` 提交事务
对于之前执行过的任何单条`SQL`语句，执行结果都会持久化到数据库中，这是因为MySQL默认启用`自动提交`功能，当执行`START TRANSACTION`手动开启事务时，会关闭`自动提交`功能，当执行`ROLLBACK`或`COMMIT`后，`自动提交`功能会再次开启。如果要手动关闭当前连接（会话）的`自动提交`功能，只要执行`SET autocommit=0;`即可

```
START TRANSACTION;
DELETE FROM orderitems WHERE order_num = '20008';
#COMMIT;执行COMMIT后事务无法回滚
ROLLBACK ;
SELECT * FROM orderitems;
```

![mark](http://imgblog.kuranado.com/blog/180107/kJbJDm69jI.png)