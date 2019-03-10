---
title: Java设计模式之外观模式
date: 2018-08-21 17:14:00
comments: true
categories: [Java]
tags: [Java,设计模式,外观模式]
---

最近在看《研磨设计模式》，看到外观模式时觉得在 23 种设计模式中应该算得上最简单的模式了。本文以香油制作流程为例讲解外观模式。

<!-- more -->

首先来看看传统的香油制作办法：

1. 淘洗芝麻，去除芝麻里的沙土
2. 将芝麻放到旋转的炒锅里炒至出烟，约 40 分钟左右
3. 炒好的芝麻中筛选掉劣质的芝麻
4. 芝麻放到石墨上进行磨酱，磨好的酱倒进大锅里
5. 将开水倒进有香油酱的大锅里进行搅拌出油
6. 把香油装在桶里进行沉淀，准备出售

总结一下上面的制作流程，主要流程就是炒芝麻 -> 磨酱 -> 搅拌，用程序表示如下：

```
/**
 * @Author: Xinling Jing
 * @Date: 2018/8/20 0020 下午 6:54
 */
public interface SesameOil {

	void execute();

}
```

```
/**
 * 炒芝麻
 * @Author: Xinling Jing
 * @Date: 2018/8/20 0020 下午 6:54
 */
public class Fry implements SesameOil {

	@Override
	public void execute() {
		System.out.println("将芝麻放到旋转的炒锅里炒至出烟，约 40 分钟左右");
	}
}
```

```
/**
 * 磨酱
 * @Author: Xinling Jing
 * @Date: 2018/8/20 0020 下午 6:55
 */
public class Grind implements SesameOil {

	@Override
	public void execute() {
		System.out.println("芝麻放到石墨上进行磨酱，磨好的酱倒进大锅里");
	}
}
```

```
/**
 * 搅拌
 * @Author: Xinling Jing
 * @Date: 2018/8/20 0020 下午 6:56
 */
public class Stir implements SesameOil {

	@Override
	public void execute() {
		System.out.println("将开水倒进有香油酱的大锅里进行搅拌出油");
	}
}
```

客户端调用如下：

```
/**
 * @Author: Xinling Jing
 * @Date: 2018/8/20 0020 下午 6:54
 */
public class Client {

	public static void main(String[] args) {
		new Fry().execute();
		new Grind().execute();
		new Stir().execute();
	}
}
```

程序打印结果：

```
将芝麻放到旋转的炒锅里炒上 40 分钟，直至芝麻出烟
芝麻放到石墨上进行磨酱，磨好的酱倒进大锅里
将开水倒进有香油酱的大锅里进行搅拌出油
```

但对于一个外行来说自己制作香油还需要了解这么多道工序显然是非常麻烦的，如果有一台机器只要把芝麻倒进机器中，就可以自动制作出香油就好了，此时外观模式就应运而生了

我们定义一个 `Facade` 类如下：

```
/**
 * @Author: Xinling Jing
 * @Date: 2018/8/21 0021 上午 10:53
 */
public class Facade {

	public void powerOn() {
		new Fry().execute();
		new Grind().execute();
		new Stir().execute();
	}
}
```
该 `Facade` 类就是一个外观类，所谓外观类就是对复杂的业务逻辑进行封装，外观类与客户端直接交互使客户端只需要调用外观类的特定方法就能完成一系列复杂的业务操作，而客户端完全不需要知道外观类的执行过程。换句话说，这里的外观类就是一台可以自动制作香油的机器，只要按下电源键，这台机器就可以代替人工自动进行炒芝麻、磨酱和搅拌等一系列工序，而自己完全不需要知道这台机器是如何工作的

客户端调用如下：

```
/**
 * @Author: Xinling Jing
 * @Date: 2018/8/20 0020 下午 6:54
 */
public class Client {

	public static void main(String[] args) {
		// 打开开关开始制作香油
		new Facade().powerOn();
	}

}
```

此时结构如图：


![mark](http://imgblog.kuranado.com/blog/180821/j50g3JLah5.png)

到此相信你已经完全了解外观模式了，是不是很简单？下面对外观模式做一个简单的总结

## 总结

### 外观模式的目的

- 外观模式的目的不是给子系统添加新的功能接口，而是为了让外部减少与子系统内多个模块的交互，松散耦合，让外部能够更简单的使用子系统。虽然完全可以在外观类中增加新功能，但不建议这么做，因为外观模式就是用来对已有的功能进行组合、包装，而不是用来添加新的实现

### 外观模式调用示意

- 编写了外观模式并不代表只能使用外观模式，当只需要使用外观模式封装的其中某个模块时，完全可以单独调用该模块，而不需要经过外观模式

![mark](http://imgblog.kuranado.com/blog/180821/IbJB6095kd.png)

### 外观模式的实现

- 将外观类中的方法实现为静态方法，将外观类当成一个辅助工具类使用

```
public class Facade {

	private Facade() {}

	public static void powerOn() {
		new Fry().execute();
		new Grind().execute();
		new Stir().execute();
	}

}
```

###外观模式的优缺点

- 松散耦合：外观模式松散了客户端与子系统的耦合关系，让子系统内部的模块更容易扩展和维护
- 简单易用：外观模式让子系统更加易用，客户端不需要了解子系统的内部实现，也不需要跟众多子系统内部的模块进行交互，只需要跟外观模式交互就可以了，相当于外观类为客户端使用子系统提供了一条龙服务
- 更好地划分访问的层次：合理的使用外观模式，可以更好的划分子系统的访问层次，把需要暴露给外部的功能集中到外观中，既方便客户端调用，也隐藏了内部的实现细节
- 过多或者不合理的使用外观模式容易让人迷惑，不知是使用外观模式好还是直接调用模块好

### 对设计原则的体现

外观模式是`迪米特法则`（也称最少知识原则）的体现，即不和陌生人说话，只和你的直接朋友通信

### 相关模式

#### 外观模式与单例模式

通常一个子系统只需要一个外观实例，所以可以将外观模式和单例模式组合，将 Facade 类实现成为单例。当然前面介绍的将 Facade 类的方法设置为 `static`，并将 Facade 的构造方法私有化也可以当作为简单的单例