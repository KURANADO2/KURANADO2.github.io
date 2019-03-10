---
title: Java多线程之内存可见性
date: 2018-3-27 10:26:00
comments: true
categories: [Java]
tags: [Java,多线程,Synchronized,Volatile]
---

本篇文章为慕课网课程[细说Java多线程之内存可见性](https://www.imooc.com/learn/352)学习笔记

<!-- more -->


## JMM

`JMM(Java Memory Model)` 即 Java 内存模型，描述了 Java 程序中各种变量（线程共享变量）的访问规则，以及在 `JVM`中将变量存储到内存和从内存中读取出变量这样的底层细节，JMM 有如下几条规则：
- 所有的变量都存储在`主内存`中
- 每个线程都有自己独立的`工作内存`，里面保存该线程使用到的变量的副本（`主内存`中该变量的一份拷贝）
- 线程对`共享变量`的所有操作都必须在自己的`工作内存`中进行，不能直接从`主内存`中读写
- 不同线程之间无法直接访问其他线程`工作内存`中的变量，线程间变量值的传递需要通过`主内存`来完成

![mark](http://imgblog.kuranado.com/blog/180624/ekJ0691708.png)

根据上图也可以发现：线程不能直接与`主内存`进行交互，`工作内存`负责与线程和`主内存`进行交互

## 共享变量

如果一个变量在多个线程的`工作内存`中都存在副本，那么这个变量就是这几个线程的`共享变量`

## 可见性

一个线程对`共享变量`值的修改，能够及时被其他线程看到则称这个`共享变量`在线程之间是可见的

## 重排序

代码书写的顺序与实际执行的顺序不同,指令重排序是编译器或处理器为了提高程序性能而做的优化

### as-if-serial语义

无论如何重排序，程序执行的结果应该与代码顺序执行的结果一致，Java 编译器、运行时和处理器都会保证 Java 在**单线程下**遵循 as-if-serial 语义，所以重排序不会给单线程带来内存可见性问题，但是在多线程中程序交错执行时，重排序可能会造成内存可见性问题

```
int num1 = 3;
int num2 = 5;
int sum = num1 + num2;
```

在单线程中，第 1 行和第 2 行代码可以进行重排序，但第 3 行代码不可以进行重排序，也就是说代码实际执行顺序可能是1 -> 2 -> 3 或 2 -> 1 -> 3，但绝不可能是3 -> 1 -> 2 或 3 -> 2 -> 1，因为这和代码顺序执行的结果不一致，不满足 as-if-serial 语义

## 实现共享变量可见性

要实现共享变量的可见性，必须保证两点：

1. 线程修改后的`共享变量`值能够及时从该线程的`工作内存`刷新到`主内存`中
2. 其他线程能够及时把`共享变量`的最新值从`主内存`更新到自己的`工作内存`中

而 Java 保证`共享变量`可见性主要通过 `synchronized` 或 `volatile` 关键字实现：

### synchronized
`synchronized` 能够实现：
- 原子性（同步）
- 内存可见性

`JMM` 关于 `synchronized` 的两条规定：

- 线程解锁前，必须把`共享变量`的最新值刷新到`主内存`中
- 线程加锁时，将清空`工作内存`中共享变量的值，从而使用`共享变量`时，需要从`主内存`中重新读取最新的值（注意：加锁与解锁需要是同一把锁）

可见 `synchronized` 的这两条规定刚好满足要实现`共享变量`可见性所必须要保证的两点

`synchronized` 实现可见性过程如下：

1. 获得互斥锁
2. 清空`工作内存`
3. 从`主内存`拷贝变量的最新副本到`工作内存`
4. 执行代码
5. 将更改后的`共享变量`的值刷新到`主内存`
6. 释放互斥锁

下面上一段代码：

```
public class Synchronpublic class SynchronizedDemo {
    // 共享变量
    private boolean ready = false;
    private int result = 0;
    private int number = 1;
    // 写操作
    public void write(){
        ready = true;                           // 1.1
        number = 2;                             // 1.2
    }
    // 读操作
    public void read(){
        if(ready){                              // 2.1
            result = number * 3;                // 2.2
        }
        System.out.println("result的值为：" + result);
    }

    // 内部线程类
    private class ReadWriteThread extends Thread {
        // 根据构造方法中传入的flag参数，确定线程执行读操作还是写操作
        private boolean flag;
        public ReadWriteThread(boolean flag){
            this.flag = flag;
        }
        @Override
        public void run() {
            if(flag){
                // 构造方法中传入true，执行写操作
                write();
            }else{
                // 构造方法中传入false，执行读操作
                read();
            }
        }
    }

    public static void main(String[] args)  {
        SynchronizedDemo synDemo = new SynchronizedDemo();
        // 启动线程执行写操作
        synDemo.new ReadWriteThread(true).start();

        // 启动线程执行读操作
        synDemo.new ReadWriteThread(false).start();
    }
}
```

执行这段代码输出结果可能为 6，可能为 0，也可能为 3，而不论哪种结果，都可能有多种执行顺序 

result = 6  1.1->1.2->2.1->2.2
result = 6 1.1->2.1->1.2->2.2
result = 6  1.2->1.1->2.1->2.2
result = 3  1.1->2.1->2.2->1.2
result = 0  1.2->2.1->2.2->1.1   （1.1 和 1.2 进行了重排序，先执行了 1.2，然后写线程让出 CPU 资源执行读线程 ）
result = 0 1.2->2.2->2.1->1.1    （1.1 和 1.2 进行了重排序，2.1 和 2.2 也进行了重排序）
...

注：代码 2.1 和代码 2.2 也可以进行重排序，因为在单线程中，2.1 和 2.2 无论谁先执行，都不会影响 result 的值

上面的例子简单说明了导致`共享变量`在线程中不可见的原因可能是线程的交叉执行或重排序，通过 `synchronized` 可以解决：

导致共享变量在线程间不可见的原因|synchronized解决方案
-|
线程的交叉执行（比如先执行 1.1 后执行 2.1）|原子性，synchronized保证了锁内部代码的原子性，避免了锁内部代码在线程之间交叉执行
重排序结合线程交叉执行|原子性
共享变量更新后的值没有在工作内存与主内存之间及时更新|可见性

`synchroized` 实现可见性代码：

```
//写操作
public synchronized void write(){
    ready = true;	      			//1.1
    number = 2;		                //1.2
}
//读操作
public synchronized void read(){
    if(ready){				//2.1
        result = number * 3;	 	//2.2
    }
    System.out.println("result的值为：" + result);
}
```

这样程序的输出结果将总是 6

### volatile

`volatile` 关键字：

- 能够保证 volatile 变量的可见性
- 在 JDK 1.5 之后，volatile 变量能够禁止指令重排序
- 不能保证 volatile 变量复合操作的原子性

`volatile` 如何实现内存可见性：

volatile 变量在每次被线程访问时，都强迫从主内存中重读该变量的值，而当该变量发生变化时，又会强迫线程将最新的值刷新到主内存，这样任何时刻，不同的线程总能看到该变量的最新值

线程写 `volatile` 变量的过程:

1. 改变线程工作内存中 `volatile` 变量副本的值
2. 将改变后的副本的值从工作内存刷新到主内存

线程读 `volatile` 变量的过程:

1. 从主内存中读取 `volatile` 变量的最新值到线程的工作内存中
2. 从工作内存中读取 `volatile` 变量的副本

上代码：

```
public class VolatileDemo {

    private volatile int number = 0;
    private Lock lock = new ReentrantLock();

    public void increase() {
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.number++;
    }

    public int getNumber() {
        return this.number;
    }

    public static void main(String[] args) {
        VolatileDemo volatileDemo = new VolatileDemo();
        for (int i = 0; i < 500; i ++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    volatileDemo.increase();
                }
            }).start();
        }
        //如果还有子线程在运行，主线程就让出CPU资源，
        //直到所有子线程都运行完了，主线程再继续往下执行
        while (Thread.activeCount() > 2) {  //在IDEA中设置大于2，在Eclipse中设置为大于1即可，因为IDEA除了主线程之外还会有一个监视线程在运行
            Thread.yield();
        }
        System.out.println("number:" + volatileDemo.getNumber());
    }

}
```

这段代码的执行结果可能为 500，也可能小于 500，而问题就出在 `number ++;` 上，因为 `number ++` 实际要分为如下 3 步执行：

1. 读取 number 的值
2. 将 number 的值加 1
3. 写入最新的 number 值

而 `volatile` 虽然能够保证`共享变量`的内存可见性，但却不能保证复合操作的原子性，假设有两个线程 A 和 B，volatile int number = 5，线程 A 和 B 并发执行 `number ++;` 操作时就可能产生下面的执行顺序：

1. 线程 A 从主内存读取 number 的值
2. 线程 B 从主内存读取 number 的值
3. 线程 B 执行加 1 操作
4. 线程 B 向主内存写入最新的 number 值
5. 线程 A 执行加  1 操作
6. 线程 A 向主内存写入最新的 number 值

两个线程都执行了 `number ++;`，但主内存中共享变量 `number` 的值却是 6 而不是 7

解决办法有两种，一是通过 `synchronized` 加锁保证自增操作原子性，二是通过 `ReentrantLock` 对象加锁保证自增操作原子性 

1. 使用 synchronized 关键字保证 number 自增操作的原子性
```
public synchronized void increase() {
    try {
        Thread.sleep(100);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    this.number++;
}
```

当然加锁的范围应该尽量更小一些：

```
public void increase() {
    try {
        Thread.sleep(100);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    synchronized (this) {
        this.number++;
    }
}
```

2. 使用 `ReentrantLock` 保证 number 自增操作的原子性

```
public void increase() {
    try {
        Thread.sleep(100);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    lock.lock();    // 加锁
    try {
        this.number ++;
    } finally {
        lock.unlock();  // 释放锁
    }
}
```

使用上面两种方法的任意一种输出结果将总是 500

`synchronized` 和 `volatile` 比较：

- `volatile` 不需要加锁，比 `synchronized` 更轻量级，不会阻塞线
程，执行效率更高
- `synchronized` 既能保证`共享变量`的可见性，又能保证`共享变量`的原子性，而 `volatile`只能保证`共享变量`的可见性，无法保证`共享变量`的原子性。

最后，我们知道 Java 中 long 和 double 都是64 位的数据类型，而 `JMM` 允许 JVM 将没有被 `volatile` 修饰的 64 位数据类型的读写操作划分为两次 32 位的读写操作来进行，这就可能会导致读取到“半个变量”的情况，为了预防这种情况，最好为 long 和 double 类型的变量加上 `volatile` 关键字