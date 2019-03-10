---
title: Quartz 定时任务
date: 2018-10-08 23:00:00
comments: true
categories: [Java]
tags: [quartz,Java,定时]
---

最近自己负责的公司项目中业务上需要用到定时任务，所以对定时任务做一个总结。

<!-- more -->

## 问题分析

有这样一个业务逻辑：用户登录上传一个加密文件，后端代码解析文件后从服务器打包几个文件放到一个 `pkg/$SESSIONID` 文件夹中，用户在浏览器端点击“下载”按钮，将服务器上 `pkg/$SESSIONID` 中的文件下载到本地，之后删除 `$SESSIONID` 文件夹。另外如果用户在点击“下载”按钮之前就点击了“退出登录”按钮，则也应该清空 `$SESSIONID` 文件夹。

就是这么简单的一个业务逻辑，但却发现存在一个 Bug，那就是当用户登录提交加密文件后，服务器上创建了 `$SESSIONID` 文件夹，用户却既没有点击“下载”按钮，也没有点击“退出登录”按钮，那么当此次会话自动过期时，该会话对应的临时目录将永远得不到删除。如果这种现象经常出现，将会导致服务器上积累大量临时文件占用磁盘资源。

我想到的解决办法是使用定时任务，定时扫描那些存在时间过长的文件并将其删除。另外更简便的方法是编写 Shell 脚本，定期删除文件。此处我使用 `Quartz` 定时任务完成这一目的。

## 引入 quartz 依赖

```
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.2.1</version>
</dependency>
```

## 编写 quartz 配置文件

总的来说在该配置文件中就做三件事：
1. 创建任务，指定执行任务的目标对象和目标方法
2. 创建触发器，绑定任务，并指定该触发器何时被触发，从而执行任务
3. 创建调度器

`spring-quartz.xml`：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
                            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

    <context:component-scan base-package="com.adups.controller"/>

    <!-- 创建任务 -->
    <bean class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean" id="cleanFileJobDetail">
        <!-- 目标对象 -->
        <property name="targetObject" ref="cleanController"/>
        <!-- 目标方法 -->
        <property name="targetMethod" value="cleanFile"/>
    </bean>

    <!-- 创建SimpleTrigger触发器 -->
    <!--<bean class="org.springframework.scheduling.quartz.SimpleTriggerFactoryBean" id="simpleTrigger">-->
    <!-- 引用任务 -->
    <!--<property name="jobDetail" ref="jobDetail"/>-->
    <!-- 指定循环时间，以秒为单位 -->
    <!--<property name="repeatInterval" value="10000"/>-->
    <!--</bean>-->

    <!-- 创建 CronTrigger 触发器 -->
    <bean class="org.springframework.scheduling.quartz.CronTriggerFactoryBean" id="cleanFileCronTrigger">
        <!-- 引用任务 -->
        <property name="jobDetail" ref="cleanFileJobDetail"/>
         <!--指定 Cron 表达式-每天凌晨 3 点执行一次 -->
        <property name="cronExpression" value="0 0 3 * * ?"/>
    </bean>

    <!-- 创建调度器 -->
    <bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean" id="stdScheduler" lazy-init="false">
        <property name="triggers">
            <list>
                <!--<ref bean="simpleTrigger"/>-->
                <ref bean="cleanFileCronTrigger"/>
            </list>
        </property>
    </bean>

</beans>
```

关于 Cron 表达式下面的例子足够全面:

表达式|含义
-|
&#42;/5 &#42; &#42; &#42; &#42; ?|每隔5秒执行一次
0 &#42;/1 &#42; &#42; &#42; ?|每隔1分钟执行一次
0 0 &#42;/1 &#42; &#42; ?|每隔 1 小时执行一次
0 0 23 &#42; &#42; ?|每天23点执行一次
0 0 23 ? &#42; &#42;|每天23点执行一次
0 0 23 &#42; &#42; ? &#42;|每天23点执行一次
0 0 23 &#42; &#42; ? 2016|2016年每天23点执行一次
0 0 1 &#42; &#42; ?|每天1点执行一次
0 0 1 1 &#42; ?|每月1号1点执行一次
0 &#42; 14 &#42; &#42; ?|每天14:00点到14:59期间，每隔1分钟执行一次
0 0-5 14 &#42; &#42; ?|每天14:00点到14:05期间，每隔1分钟执行一次
0 0 23 L &#42; ?|每月最后一天23点执行一次
0 0 1 ? &#42; L|每周星期六1点执行一次
0 26,29,33 &#42; &#42; &#42; ?|在26分、29分、33分执行一次
0 0 0,13,18,21 &#42; &#42; ?	|每天的0点、13点、18点、21点都执行一次
0 0 0 ? &#42; 6#3|每月第3个星期六

如果你想了解这些例子的语法规则，参考下面两张表格，但我觉得根本没必要，直接使用上面表格中的例子反而来的简单直接

字段|可选值|特殊字符
-|
秒 | 0 - 59 | , - * / 
分 | 0 - 59 | , - * / 
时 | 0 - 23 | , - * / 
日 | 1 - 31 | , - * / ? L W 
月 | 1 - 12 | , - * / 
周 | 1（Sun） - 7（Sat） | , - * / ? L # 
年（可选） | 1970 - 2099 | , - * / 

字符|说明
-|
, | 指定多个数值 
- | 指定一个范围 
* | 代表整个时间段 
/ | 多长时间执行一次 
? | 不确定的值 
L | 每月最后一天（日）/每月最后一个星期六（周） 
W | 最近的工作日 
&#35; | 每月第N个工作日 

## 启动类引入配置文件

```
@SpringBootApplication // = @Configuration + @EnableAutoConfiguration + @ComponentScan
@ImportResource("classpath:spring-quartz.xml")
@MapperScan("com.adups.**.dao")
public class App {
    public static void main( String[] args ) {
        ApplicationContext context = SpringApplication.run(App.class, args);
        // 启动时加载（其实不用手动调用，项目启动后，定时任务就会被立即执行一次，所以下面两行代码可以注释）
        CleanController cleanController = context.getBean("cleanController", CleanController.class);
        cleanController.cleanFile();
    }
}
```

## 编写要执行的任务代码

```
/**
 * @Author: Xinling Jing
 * @Date: 2018/10/8 0008 16:19
 */
@Controller
public class CleanController {

    private Logger logger = LoggerFactory.getLogger(CleanController.class);

    @Autowired
    private UDisk uDisk;

    public void cleanFile() {
        File pkg = new File(uDisk.getPkgPath());
        Long currentTime = System.currentTimeMillis();
        File[] files = pkg.listFiles(pathname -> {
            Long modifiedTime = pathname.lastModified();
            // 过滤超过 24 小时的文件放到 files 数组中
            if (((currentTime - modifiedTime) * 1.0 / (1000 * 60 * 60)) > 24) {
                return true;
            }
            return false;
        });

        if (files != null) {
            String[] path = new String[files.length];
            for (int i = 0; i < files.length; i++) {
                path[i] = files[i].getPath();
            }
            // 删除文件夹
            FileUtil.delFolder(path);
        }
        logger.info("定时任务执行，删除 {} 目录下超过 24 小时的文件夹", uDisk.getPkgPath());
    }

}
```

## 总结

定时任务非常简单，但却非常实用，公司很多项目中都用到了 Quartz 定时任务，不得不会。另外编写脚本执行定时任务也是不错的解决方法，根据实际情况自行选择。

## 参考资料

[Java - Quartz 定时任务-简书](https://www.jianshu.com/p/13623119cb5b)
[spring-Quartz定时任务-CSDN](https://blog.csdn.net/zhemeban/article/details/76022134)
