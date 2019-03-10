---
title: Java设计模式之单例模式
date: 2018-06-24 22:38:00
comments: true
categories: [Java]
tags: [Java,设计模式,单例模式,volatile,多线程,懒加载]
---

臃肿的业务代码往往都是由于不懂得如何使用设计模式造成的，且设计模式的实现原理涉及到很多 Java 语言特性，学习 Java 设计模式可以优化项目业务代码同时也能够更深入了解 Java。本文以最简单的单例模式作为开篇总结 Java 设计模式。说到单例模式，最早是在大二的机器人仿真救援比赛中接触单例模式的饱汉式和饥汉式，当时只知道单例模式的这两种写法，没有深入了解过有没有更好的实现方法，本文将总结单例模式的多种写法及其实现原理。

<!-- more -->

## 什么是单例模式

所谓单例模式即单例类只能有一个实例，并向外部提供一个访问该实例的全局访问点。

单例模式优点：

- 可以避免类的频繁创建与销毁
- 只创建一个实例，节省系统资源

## 饱汉式-单线程

饱汉式又称懒汉式（吃饱了可不就懒得动弹了嘛）：

```
public class Test {

    public static void main(String[] args) {
        Singleton singleton = Singleton.getSingleton();
        Singleton singleton2 = Singleton.getSingleton();
        // 输出 true
        System.out.println(singleton == singleton2);
    }

}
class Singleton {

    private static Singleton singleton;

    private Singleton () {}

    public static Singleton getSingleton() {

        if (singleton == null) {
            singleton = new Singleton();
        }

        return singleton;
    }
}
```

这就是最简单的单例模式，为了防止外部通过 `new` 创建多个 Singleton 类的对象，将构造方法设为 `private`；向外部提供 `getSingleton` 方法，通过 `if` 判断，即便外部多次调用该方法也只会创建一次实例；因为 new 关键字已被禁用，无法通过 `new Singleton().getSingleton()` 的形式创建实例，所以该方法为静态方法，可以直接通过 `Singleton.getSingleton()` 调用该方法创建实例；因为静态方法不能访问普通变量，所以 Singleton 变量也使用 `static` 修饰。

## 饱汉式-多线程

饱汉式-单线程的代码在单线程下的执行效率很高，同时也实现了懒加载（外部调用 getInstance() 方法时才创建 Singleton 类的实例），但缺点也很明显，那就是在多线程情况下，并不能保证只创建一次实例，如两个线程 A 和 B，A 执行完 `if (singleton == null)` 后，线程 B 获得处理器资源也执行到 `if (singleton == null)` ，这样线程 A 和 B 将都会创建一次实例。解决办法也很简单，使用 `synchronized` 修饰 `getSingleton` 方法即可：

```
public static synchronized Singleton getSingleton() {

    if (singleton == null) {
        singleton = new Singleton();
    }

    return singleton;
}
```

这样即可保证多线程下也只会创建一次实例，但同样存在很大的缺陷：加锁会影响效率，实际应该在创建完第一个实例后就解锁，否则每个线程调用 `getSingleton` 时都要上锁阻塞其他线程，导致执行效率极低。

## 双重检验锁(Double Checked Locking)

双重检验锁是对饱汉式-多线程的优化，做到一旦创建完第一个实例后就不再加锁的效果：

```
class Singleton {

    private volatile static Singleton singleton;
    
    private Singleton () {}

    public static Singleton getSingleton() {
        // 可能会有多个线程都进入了此 if
        if (singleton == null) {  // 第一次检查
            // 加锁
            synchronized (Singleton.class) {
                // 第一个进入锁内的线程才会进入此 if
                if (singleton == null) {  // 第二次检查
                    singleton = new Singleton();
                }
            }
        }

        return singleton;
    }
}
```

关键点在于 `volatile` 关键字的使用，此处为何要使用 `volatile` 关键字呢？

问题在于 `singleton = new Singleton();` 这行代码上，这行代码在底层可以粗略的分为以下几步执行：

1. 栈内存开辟空间给 singleton 引用
2. 堆内存开辟空间准备初始化对象
3. 初始化对象
4. 栈中引用指向这个堆内存空间地址

因为指令重排序的原因，这行代码的执行顺序可能是 1 -> 2 -> 3 -> 4，也可能是 1 -> 2 -> 4 -> 3。在某个时刻，确实可以保证只有一个线程进入同步代码块，如果进入同步代码块的线程刚好执行到 1 -> 2 -> 4，并没有执行到 3，但此时 singleton 已经非空，如果这时还有一个线程抢占资源调用 getInstance() 方法，则该线程执行到第一个 `if` 判断时，由于 singleton 非空，直接返回该 singleton，而实际上，该 singleton 所指向的堆内存空间地址并没有存放初始化后的对象，造成我们并没有拿到正确的对象实例。

`volatile` 刚好可以解决上述问题，我们知道 `volatile` 有 3 个特点：

1. 能够保证 `volatile` 变量的可见性
2. 在 JDK 1.5 之后，`volatile` 变量能够禁止指令重排序
3. 不能保证 `volatile` 变量复合操作的原子性。

其中禁止指令重排序的特性正是我们所需要的。

## 饥汉式

饥汉式又称饿汉式，实现代码最为简单：

```
class Singleton {

    private static Singleton singleton = new Singleton();

    private Singleton () {}

    public static Singleton getSingleton() {
        return singleton;
    }
}
```

饥汉式一上来就在类加载时创建好对象，由于 Java 的类加载机制避免了多线程的同步问题（类的加载方式是按需加载，且只加载一次，因为这个类在整个生命周期中只会被加载一次，所以只会创建一个实例），所以执行效率非常高。但饥汉式也存在如下两个缺点：

1. 没有实现懒加载，即便根本没有人主动调用 getSingleton 方法，不管三七二十一，也会在类加载时就创建 Singleton 类实例。假设 Singleton 类实例的创建非常消耗系统资源的话，则会造成系统资源浪费。
2. 像 Spring 等框架的设计中也都用到了单例模式，但这些框架常常需要通过参数进行配置，如果直接像饥汉式一样 `private static Singleton singleton = new Singleton();` 将创建对象写死，将无法传入配置参数。

## 静态内部类

```
class Singleton {

    private Singleton() {}
    
    // 私有静态内部类，用到时才加载，所以时懒加载
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getSingleton() {
        return SingletonHolder.INSTANCE;
    }
}
```

静态内部类的加载不需要依附外部类，在使用到静态内部类时才加载，所以实现了懒加载。同时和饥汉式一样，类加载时就创建好对象，Java 的类加载机制也避免了多线程的同步问题，区别只在于这里是内部类

## 枚举

常有人说实现单例模式的最佳方法是使用枚举，这是因为枚举拥有以下特性，且代码实现简洁：

- 枚举类的构造器只能使用 `private` 修饰，若省略 `private`，则默认也是使用 `private` 修饰，如果省略构造器，默认也会提供一个 `private` 修饰的构造器，这和我们前面 5 种写法完全吻合
- 枚举类的每个实例系统都会自动为其添加 `public static final` 修饰，保证了枚举中的实例都只会被实例化一次
- 线程安全

```
public class Test {

    public static void main(String[] args) {
        Singleton singleton = Singleton.SINGLETON;
        Singleton singleton2 = Singleton.SINGLETON;
        // 输出 true
        System.out.println(singleton == singleton2);
    }

}

enum Singleton {
    SINGLETON
}
```

遗憾的是枚举类加载时就开始加载枚举实例，所以并没有实现懒加载。

## 总结

各种写法特性总结如下：

s|饱汉式-单线程|饱汉式-多线程|双重检验锁|饥汉式|静态内部类|枚举
-|
支持多线程|✘|✔|✔|✔|✔|✔
懒加载|✔|✔|✔|✘|✔|✘
效率|高|低|高|高|高|高

综合来说我更倾向于使用双重检验锁方式，但每种写法各有其优缺点，在开发中应该根据需求选择，引用一段话：

> 既应当考虑到需求可能出现的扩展与变化，也应该避免无谓的提升设计、实现复杂度，最终反而会带来工期、性能和稳定性的损失，设计不足与设计过度都是危害，正所谓：没有最好的单例模式，只有最合适的单例模式。

## 参考资料

- 《Java 疯狂讲义》
- [菜鸟教程单例模式](http://www.runoob.com/design-pattern/singleton-pattern.html)
- [如何正确地写出单例模式](http://wuchong.me/blog/2014/08/28/how-to-correctly-write-singleton-pattern/)
- [Java并发：volatile内存可见性和指令重排](http://www.importnew.com/23535.html)
- [Java多线程之内存可见性](http://www.kuranado.com/2018/03/27/Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E4%B9%8B%E5%86%85%E5%AD%98%E5%8F%AF%E8%A7%81%E6%80%A7/)
- [The "Double-Checked Locking is Broken" Declaration](https://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html)
- [Java枚举enum以及应用：枚举实现单例模式](https://www.cnblogs.com/cielosun/p/6596475.html)