---
title: JVM判断对象是否可回收
date: 2018-3-31 10:24:30
comments: true
categories: [Java]
tags: [Java,JVM]
---

最近看了《深入理解Java虚拟机》一书，其中 3.2 节介绍了 Java 虚拟机如何判断对象是否应该被 GC 回收的`可达性分析算法`，特在此做个笔记

<!-- more -->

## 引用计数算法

提到`可达性分析算法`之前，先谈谈`引用计数算法`，所谓引用计数算法就是给对象添加一个引用计数器，每当有一个地方引用它时，计数器值就加 1；当引用失效时，计数器值就减 1；任何时刻计数器值为 0 的对象都是不可能再被使用的。引用计数算法实现简单，而且效率很高，但主流的 JVM 中**并没有使用引用计数算法**，其中最主要的原因就是引用计数算法存在对象之间`循环引用`的缺陷

![mark](http://imgblog.kuranado.com/blog/180329/bbD7j0gC6l.png)
图片来源[极客学院 JVM 自动内存管理](http://www.jikexueyuan.com/course/2098.html)

举例代码如下：

```
public class ReferenceCountAlgo {

    private Object instancce = null;

    public static void main(String[] args) {
        ReferenceCountAlgo a = new ReferenceCountAlgo();
        ReferenceCountAlgo b = new ReferenceCountAlgo();

        a.instancce = b;
        b.instancce = a;

        a = null;
        b = null;

        System.gc();
    }

}
```

如果 JVM 采用引用计数算法，那么因为 a 和 b 之间存在循环引用的关系，垃圾回收算法将无法回收对象 a 和 b，然而事实是当我们打印出 GC 日志时可以发现对象已经被 GC 回收，从而证明主流的 JVM 并没有采用引用计数算法判断对象是否应该被 GC 回收。

上面程序打印出的 GC 日志如下：

![mark](http://imgblog.kuranado.com/blog/180329/k27F6E3lL6.png)

关于 IDEA 如何打印 GC 日志及日志中各项的含义可参考：[idea打印gc日志的2种方法](https://blog.csdn.net/bear_lam/article/details/79648701)

## 可达性分析算法

可达性分析算法的基本思路是通过一系列的称为 `GC Roots` 的对象作为起始点，从这些起始点开始向下搜索，搜索所走过的路径称为`引用链（Reference Chain）`，当一个对象到 GC Roots 没有任何引用链相连时，则证明此对象是不可用的，将会被判定为是可回收对象。

![mark](http://imgblog.kuranado.com/blog/180329/IaBcj6Ac66.png)

Java 语言中可作为 GC Roots 的有：

- 虚拟机栈（栈帧中的本地变量表）中引用的对象
- 方法区中的类静态属性引用的对象
- 方法区中的常量引用的对象
- 本地方法栈中 JNI（即一般说法的 Native 方法）引用的对象

关于虚拟机栈、本地方法栈、方法区的简单介绍如下：

### Java 虚拟机栈：

Java 虚拟机栈是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是**Java 方法执行的内存模型**：每个方法在执行的同时都会创建一个`栈帧`（Stack Frame，栈帧是方法运行时的基础数据结构）用于存储**局部变量表**、操作数栈、动态链接、方法出口等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。

局部变量表存放了编译器可知的各种基本数据类型（boolean、char、byte、short、int、long、float、double）、对象引用和returnAddress类型（返回地址类型指向了一条字节码指令的地址）。局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小（其中 64 位长度的 long 和 double 类型的数据会占用 2 个局部变量空间，其余的数据类型只占用 1 个）

Java 虚拟机规范规定了 Java 虚拟机栈的两种异常情况：

1. 如果线程请求的栈深度大于虚拟机所允许的深度，将抛出 `StackOverflowError` 异常
2. 如果虚拟机栈可以动态扩展（当前大部分的 Java 虚拟机都支持动态扩展，只不过 Java 虚拟机规范中也允许固定长度的虚拟机栈），但在扩展时无法申请到足够的内存，就会抛出 `OutOfMemoryError` 异常

### 本地方法栈：

本地方法栈（Native Method Stack）与虚拟机栈所发挥的作用是非常相似的，他们之间的区别不过是虚拟机栈为虚拟机执行 Java 方法服务，而本地方法栈则为虚拟机执行使用到的 Native 方法服务。在虚拟机规范中对本地方法栈中方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机（譬如 Sun HotSpot 虚拟机）直接就把虚拟机栈和本地方法栈合二为一。与虚拟机栈一样，本地方法栈区域也会抛出 `StackOverflowError`  和 `OutOfMemoryError` 异常

### 方法区：

方法区（Method Area）与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

## 总结

主流的 JVM 使用可达性分析算法而不是引用计数法判断对象是否可回收