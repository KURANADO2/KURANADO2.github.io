---
title: asmtools 工具的简单使用
date: 2019-01-13 20:12:00
comments: true
categories: [Java]
tags: [Java, JVM]
---

在极客时间[《深入拆解 JVM 虚拟机这门课程中》](https://time.geekbang.org/column/108)经常会用到一个叫做 asmtools.jar 的工具，可以用来修改 .class 文件。为了完成每节课的课后小作业,必须了解其简单使用方法

<!-- more -->

## 前言

AsmTools 有两种表示 .class 文件的语法

- jasm:用类似 java 本身的语法来定义类和函数，字节码指令则很像传统的汇编
- jcod:整个 .class 用容器的方式来表示，可以很清楚表示类文件的结构

本文主要使用 jasm 语法

## 生成 asmtools.jar

如果你的机器没有安装 `Mercurial`，请先安装(Mericurial 类似 Git 和 SVN，是一个分布式版本控制系统，使用 Python 编写，OpenJDK 就托管在 Mercurial 平台上)

然后执行如下命令：

```
hg clone http://hg.openjdk.java.net/code-tools/asmtools
cd asmtools/build
# 使用 ant 编译
ant
```

然后就会得到 asmtools.jar 文件。当然，如果你和我一样懒，可以到这里直接下载已经生成的 asmtools.jar 文件:[https://github.com/hengyunabc/hengyunabc.github.io/files/2188258/asmtools-7.0.zip](https://github.com/hengyunabc/hengyunabc.github.io/files/2188258/asmtools-7.0.zip)

## 准备工作

有如下 Foo.java 文件：

```
public class Foo {
 public static void main(String[] args) {
  boolean flag = true;
  if (flag) System.out.println("Hello, Java!");
  if (flag == true) System.out.println("Hello, JVM!");
 }
}
```

`javac Foo.java` 命令生成 Foo.class 文件，使用 JD-GUI 打开内容如下：

```
import java.io.PrintStream;

public class Foo
{
  public static void main(String[] paramArrayOfString)
  {
    int i = 1;
    if (i != 0) {
      System.out.println("Hello, Java!");
    }
    if (i == 1) {
      System.out.println("Hello, JVM!");
    }
  }
}
```

`java Foo` 命令运行 Foo.class 文件输出结果：

```
Hello, Java!
Hello, JVM!
```

## 由 class 文件生成 jasm 文件

如下命令将 class 文件中的内容转换为对应的 jasm 语法：

```
java -jar asmtools.jar jdis Foo.class
```

上面输出的内容会直接打印在终端，当然你也可以把它输入到文件中：

```
java -jar asmtools.jar jdis Foo.class > Foo.jasm
```

Foo.jasm 文件内容如下：

```
super public class Foo
    version 52:0
{


public Method "<init>":"()V"
    stack 1 locals 1
{
        aload_0;
        invokespecial   Method java/lang/Object."<init>":"()V";
        return;
}

public static Method main:"([Ljava/lang/String;)V"
    stack 2 locals 2
{
        iconst_1;
        istore_1;
        iload_1;
        ifeq    L14;
        getstatic   Field java/lang/System.out:"Ljava/io/PrintStream;";
        ldc String "Hello, Java!";
        invokevirtual   Method java/io/PrintStream.println:"(Ljava/lang/String;)V";
    L14:    stack_frame_type append;
        locals_map int;
        iload_1;
        iconst_1;
        if_icmpne   L27;
        getstatic   Field java/lang/System.out:"Ljava/io/PrintStream;";
        ldc String "Hello, JVM!";
        invokevirtual   Method java/io/PrintStream.println:"(Ljava/lang/String;)V";
    L27:    stack_frame_type same;
        return;
}

} // end Class Foo
```

为了将 Foo.class 中的 `int i = 1;` 修改成 `int i = 2;`，我们需要替换 Foo.jasm 文件中的 `iconst_1` 为 `iconst_2`(这里暂且不讨论 jasm 的语法含义，先照着做就好了)。你可以使用 `awk` 命令修改，但我还是觉得直接用 VIM 更简单

## 由 jasm 文件生成 class 文件

执行如下命令：

```
java -jar asmtools.jar jasm Foo.jasm
```

这时再用 JD-GUI 打开 Foo.class 文件，内容如下：

```
import java.io.PrintStream;

public class Foo
{
  public static void main(String[] paramArrayOfString)
  {
    int i = 2;
    if (i != 0) {
      System.out.println("Hello, Java!");
    }
    if (i == 1) {
      System.out.println("Hello, JVM!");
    }
  }
}
```

看到 i 的确由 1 变成 2 了。

`java Foo` 输出内容如下：

```
Hello, Java!
```

## 总结

其实就两个命令：

- 由 class 文件生成 jasm 文件：`java -jar asmtools.jar jdis Foo.class > Foo.jasm`
- 由 jasm 文件生成 class 文件：`java -jar asmtools.jar jasm Foo.jasm`

## 参考资料

- [OpenJDK 里的 AsmTools 简介](http://hengyunabc.github.io/openjdk-asmtools/)
- [https://wiki.openjdk.java.net/display/CodeTools/Chapter+2#Chapter2-Jasm.1](https://wiki.openjdk.java.net/display/CodeTools/Chapter+2#Chapter2-Jasm.1)
- [asmtools 修改 class 文件](https://www.jianshu.com/p/f258664ba8ba)
