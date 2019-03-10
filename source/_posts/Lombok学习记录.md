---
title: Lombok学习记录
date: 2017-09-22 19:23:00
comments: true
categories: [Java]
tags: [Java, Lombok, Maven]
---

今天在读学长分配的项目任务源码时，其中用到了一些Lombok注解，虽说对Lombok常用注解有一定的了解，但是学长的源码中还是用到了一些从没见过的Lombok注解，所以这里做一个Lombok的总结，整理了部分常用注解，后续用到其它注解也会继续更新。

<!-- more -->

> 首先是Lombok插件的官网：[https://projectlombok.org/](https://projectlombok.org/)

## Data注解

> Data注解编译时自动添加Setter、Getter、toString、equals、hashCode

- bean实体类：

```
package com.kuranado.bean;

import lombok.Data;

import java.util.Date;

/**
 * Data注解编译时自动添加**Setter、Getter、toString、equals、hashCode**
 *
 * Created by JING on 2017/9/22.
 */
@Data
public class User {

    private Integer id;
    private String username;
    private Integer age;
    private String sex;
    private Date birthday;
    private String address;
    private String pro;

}
```

- 测试类：

```
package com.kuranado.bean;

import org.junit.Test;

import java.util.Date;

/**
 * Created by JING on 2017/9/22.
 */
public class UserTest {

    @Test
    public void testData() {
        User user = new User();
        user.setId(1);
        user.setUsername("葛二蛋");
        user.setAge(18);
        user.setSex("男");
        user.setBirthday(new Date());
        user.setAddress("安徽省淮南市");
        user.setPro("班长");
        User user2 = new User();
        user2.setId(1);
        user2.setUsername("葛二蛋");
        user2.setAge(18);
        user2.setSex("男");
        user2.setBirthday(new Date());
        user2.setAddress("安徽省淮南市");
        user2.setPro("班长");
        //user2.setPro("团长");

        //test Getter
        System.out.println(user.getUsername());
        //test toString
        System.out.println(user);
        //test equals
        System.out.println(user.equals(user2));
        //test hashCode
        System.out.println(user.hashCode());

    }

}
```

- 测试结果：

![](http://res.cloudinary.com/code-clannad-site/image/upload/v1506080660/d1a8e681-4722-4c40-8dbb-3de9474ffa83_s5jgvf.png)

## Value注解

> Value注解编译时自动添加**Getter、toString、equals、hashCode以及一个全参的构造器**

- bean实体类：

将上例中的@Data注解改为@Value

- 测试类：

测试类中添加如下方法

```
@Test
    public void testValue() {
        //全参的构造器（参数顺序和bean中定义字段的顺序相同，IDEA中可通过Ctrl+P提示参数）
        User user = new User(2, "久美子", 16, "女", new Date(), "京都府宇治市", "上低音号");
        User user2 = new User(2, "久美子", 16, "女", new Date(), "京都府宇治市", "上低音号");
        //test Getter
        System.out.println(user.getUsername());
        //test toString
        System.out.println(user.toString());
        //test equals
        System.out.println(user.equals(user2));
        //test hashCode
        System.out.println(user.hashCode());
    }
```

- 测试结果：

![](http://res.cloudinary.com/code-clannad-site/image/upload/v1506080820/9bc68bf8-6902-416e-8148-382334e1d458_vcu6ak.png)

## Getter

> 编译时自动添加Getter方法

## Setter

> 编译时自动添加Setter方法

## ToString注解

> ToString注解编译时自动添加toString方法

## EqualsAndHashCode

> 编译时自动添加equals方法和hashCode方法

## Builder注解

> Builder注解编译时增加了一个Builder内部类和全参的构造器

- bean实体类：

在bean实体类上添加@ToString和@Builder注解

- 测试类：

```
 @Test
    public void testBuilder() {
        //全参的构造器
        User user = new User(3, "久美子", 16, "女", new Date(), "京都府宇治市", "上低音号");
        //Builder内部类
        User user2 = User.builder().id(4).username("utsuha").age(14).sex("女").birthday(new Date()
        ).address("京都府").pro("student").build();
        System.out.println(user);
        System.out.println(user2);
    }
```

- 测试结果：

![](http://res.cloudinary.com/code-clannad-site/image/upload/v1506080881/d7307c76-7726-4bd5-9aa4-266b5cb01374_ytqqz0.png)

## 构造器注解：

> Lombok提供了三个构造器注解：
> 1. AllArgsConstructor 增加全参构造器
> 2. NoArgsConstructor 增加无参构造
> 3. RequiredArgsConstructor 增加必选参数构造器

对于这三个构造器注解可以同时使用，以增加不同的构造器，其中**RequiredArgsConstructor**注解需要配合**NonNull**注解使用，也就是说只有标记了NonNull注解的field才会被纳入RequiredArgsConstructor添加的构造器中

- bean实体类：

```
@RequiredArgsConstructor
public class User {

    @NonNull
    private Integer id;
    private String username;
    @NonNull
    private Integer age;
    private String sex;
    private Date birthday;
    private String address;
    private String pro;

}
```

- 测试类：

测试类中添加如下方法：

```
@Test
    public void testRequiredArgsConstructor() {
        User user = new User(5, 18);//只有此构造器可以使用
        System.out.println(user);
    }
```

- 测试结果：略

## Log注解

> 添加日志注解的类会隐式的定义一个名为log的日志对象。Lombok提供了很多日志相关的注解，具体参考：[Lombok官网日志注解](https://projectlombok.org/features/log)

```
@CommonsLog
    Creates private static final org.apache.commons.logging.Log log = org.apache.commons.logging.LogFactory.getLog(LogExample.class);

@JBossLog
    Creates private static final org.jboss.logging.Logger log = org.jboss.logging.Logger.getLogger(LogExample.class);

@Log
    Creates private static final java.util.logging.Logger log = java.util.logging.Logger.getLogger(LogExample.class.getName());

@Log4j
    Creates private static final org.apache.log4j.Logger log = org.apache.log4j.Logger.getLogger(LogExample.class);

@Log4j2
    Creates private static final org.apache.logging.log4j.Logger log = org.apache.logging.log4j.LogManager.getLogger(LogExample.class);

@Slf4j
    Creates private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);

@XSlf4j
    Creates private static final org.slf4j.ext.XLogger log = org.slf4j.ext.XLoggerFactory.getXLogger(LogExample.class);
```

- 测试类

在测试类上添加Log注解，然后添加如下测试方法：

```
@Test
    public void testLog() {
        log.info("info");
        log.warning("warning");
    }
```

- 测试结果：

![](file:///D:/Program%20Files/WizNote/data/temp/editor_md_temp/a148ed14-bd28-43bc-a4a5-24ba9b6335e4/index_files/5fc0f0b1-a12f-4a1b-8b0a-c921396d4491.png)

## Getter(lazy=true) 懒加载

将Getter(lazy=true)用于指定field上，当对象初始化时，该field并不会真正的初始化，而是第一次访问该field时才进行初始化field的操作

- 测试类：
```
@Data
public class GetterLazyExample {
    @Getter(lazy = true)
    private final int[] cached = expensive();
    private Integer id;

    private int[] expensive() {
        int[] result = new int[100];
        for (int i = 0; i < result.length; i++) {
            result[i] = i;
            System.out.println(i);
        }
        System.out.println("cached 初始化完成。");
        return result;
    }
    public static void main(String[] args) {
        GetterLazyExample obj=new GetterLazyExample();
        obj.setId(1001);
        System.out.println("打印id："+obj.getId());
        System.out.println("cached 还没有初始化哟。");
        // obj.getCached();
    }
}
```
