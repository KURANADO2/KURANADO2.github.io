---
title: Java 设计模式之装饰器模式
date: 2019-07-21 20:44:00
comments: true
categories: [Java]
tags: [Java, 设计模式, 装饰器模式]
---

装饰器模式学习记录

<!-- more -->

装饰器模式也称为包装器模式，往往以 Decorator 或 Wrapper 结尾的类都是使用的装饰器模式

## 什么是装饰器模式？

> 装饰器模式又称装饰模式、包装模式。用于动态的给一个对象添加一些额外的职责。就增加功能来说，装饰器模式比生成子类更为灵活

## 装饰模式结构

![](http://imgblog.kuranado.com/1563712999.png?imageMogr2/thumbnail/!70p)


## 装饰器模式实现简单示例

### 1. 定义组件接口

```java
import lombok.Data;

/**
 * @Author: Xinling Jing
 * @Date: 2019-07-21 15:59
 */
@Data
public abstract class BasePerson {

    private String name;

    public abstract String show();
}
```

### 2. 定义具体组件类，该组件类的对象将被装饰器装饰

所有人开始都是一丝不挂

```java
/**
 * @Author: Xinling Jing
 * @Date: 2019-07-21 16:01
 */
public class ConcretPerson extends BasePerson {

    @Override
    public String show() {
        return this.getName() + "裸体";
    }
}
```

### 3. 定义装饰器接口，需要和被装饰的对象实现同样的接口

```java
/**
 * @Author: Xinling Jing
 * @Date: 2019-07-21 16:01
 */
public abstract class BaseDecorator extends BasePerson {

    private BasePerson person;

    public BaseDecorator(BasePerson person) {
        this.person = person;
    }

    @Override
    public String show() {
        return person.show();
    }
}
```

### 4. 定义具体装饰器类

#### 穿内裤的装饰器

```java
/**
 * @Author: Xinling Jing
 * @Date: 2019-07-21 16:01
 */
public class ConcretPerson extends BasePerson {

    ConcretPerson(String name) {
        this.setName(name);
    }

    @Override
    public String show() {
        return this.getName() + "裸体";
    }
}
```

#### 穿裤子的装饰器

```java
/**
 * @Author: Xinling Jing
 * @Date: 2019-07-21 16:21
 */
public class TrousersDecorator extends BaseDecorator {

    public TrousersDecorator(BasePerson person) {
        super(person);
    }

    @Override
    public String show() {
        return String.join(" -> ", super.show(), "穿上了裤子");
    }
}
```

#### 穿 T 恤的装饰器

```java
/**
 * @Author: Xinling Jing
 * @Date: 2019-07-21 16:23
 */
public class TShirtDecorator extends BaseDecorator {

    public TShirtDecorator(BasePerson person) {
        super(person);
    }

    @Override
    public String show() {
        return String.join(" -> ", super.show(), "穿上了 T 恤");
    }
}
```

#### 梳头发的装饰器

```java
/**
 * @Author: Xinling Jing
 * @Date: 2019-07-21 16:25
 */
public class HairstyleDecorator extends BaseDecorator {

    public HairstyleDecorator(BasePerson person) {
        super(person);
    }

    @Override
    public String show() {
        return String.join(" -> ", super.show(), "梳了一个性感的发型");
    }
}
```

### 5. 客户端调用

客户端首先创建被装饰的组件对象，然后创建一种或多种装饰器对象，然后把装饰器对象组合起来：

```java
/**
 * @Author: Xinling Jing
 * @Date: 2019-07-21 16:26
 */
public class Client {

    public static void main(String[] args) {
        
        // 小明家很穷，出门只能穿得起一条内裤
        PantsDecorator decorator = new PantsDecorator(new ConcretPerson("小明"));
        System.out.println(decorator.show());

        // 康康不仅穿了内裤，还穿了裤子和 T 恤
        TShirtDecorator decorator2 = new TShirtDecorator(new TrousersDecorator(new PantsDecorator(new ConcretPerson(
            "康康"))));
        System.out.println(decorator2.show());

        // 西瓜个头比较小，只需要穿条内裤，然后梳理下发型就可以了
        HairstyleDecorator decorator3 = new HairstyleDecorator(new PantsDecorator(new ConcretPerson("西瓜")));
        System.out.println(decorator3.show());
    }
}
```

输出结果如下：

```
小明裸体 -> 穿上了红胖次
康康裸体 -> 穿上了红胖次 -> 穿上了裤子 -> 穿上了 T 恤
西瓜裸体 -> 穿上了红胖次 -> 梳了一个性感的发型
```

类图如下：

![](http://imgblog.kuranado.com/1563709868.png?imageMogr2/thumbnail/!70p)

## 实际业务示例

<del>上面使用了非常简单的小栗子展示了装饰器模式的用法，放到实际工作中，实现原理也是完全一样的。比如。。。 （有空再写）</del>

## 常见装饰器模式实现

### IO 流

类图如下：

![](http://imgblog.kuranado.com/1563709992.png?imageMogr2/thumbnail/!70p)

各个类在装饰器中扮演的角色如下：

- `InputStream` 对应 `Component`
- `FileInputStream` 对应 `ConcretComponent`
- `FilterInputStream` 对应 `Decorator`
- `DataInputStream`、`BufferedInputStream`、`PushbackInputStream` 均是具体的装饰器实现

示例代码：

```java
/**
 * @Author: Xinling Jing
 * @Date: 2019-07-21 18:21
 */
public class IOTest {

    public static void main(String[] args) {

        DataInputStream dataInputStream = null;
        try {
            dataInputStream = new DataInputStream(new BufferedInputStream(new FileInputStream("/Users/jing/Desktop" +
                "/IOTest.txt")));
            byte[] buff = new byte[dataInputStream.available()];
            dataInputStream.read(buff);
            System.out.println(new String(buff));
        } catch (IOException e) {
             e.printStackTrace();
        } finally {
            try {
                if (dataInputStream != null) {
                    dataInputStream.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## 总结

下面开始装饰器模式的技术总结：

- 装饰器模式的本质是**动态组合**
- 装饰器用来装饰组件，装饰器一定要实现和组件类一致的接口，保证它们是同一个类型，并且具有同一个外观，这样组合完成的装饰才能递归调用下去
- 各个装饰器之间最好是完全独立的功能，不要有依赖，以在装饰组合的时候，可以随心所欲，不受先后顺序的限制，也就是说先装饰谁和后装饰谁都应该是一样的，否则会大大降低装饰器组合的灵活性。虽然在实际应用中，可以根据具体的功能要求而有顺序的限制，但应该尽量避免这种情况，以免影响口感

### 优点

- 装饰模式比继承更灵活。继承是静态的，一旦继承，所有子类都有一样的功能，然而有时候有些子类可能并不需要这些继承而来的功能；装饰模式把功能分离到多个装饰器中，然后通过对象组合的方式，在运行时动态的组合功能，每个被装饰的对象最终拥有哪些功能，是由运行时期动态组合的功能来决定的
- 功能容易复用。一个对象可以使用多个装饰器，一个装饰器也可以用来装饰多个对象，从而实现功能复用

### 缺点

- 会产生很多细粒度的对象。装饰模式是把一系列复杂的功能分散到多个装饰器当中，一般每个装饰器只实现一个功能，这样会产生很多细粒度的对象，而且功能越复杂，需要的细粒度对象也就越多，大量小对象占据内存，一定程度上会影响性能

## 参考资料

- 《研磨设计模式》

## 源码

- [https://github.com/KURANADO2/DesignPatterns](https://github.com/KURANADO2/DesignPatterns)