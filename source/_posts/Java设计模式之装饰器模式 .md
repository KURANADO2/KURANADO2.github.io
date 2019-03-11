---
title: Java 设计模式之装饰器模式
date: 2018-12-24 21:27:00
comments: true
categories: [Java]
tags: [Java, 设计模式, 装饰器模式]
---



<!-- more -->

装饰器模式也称为包装器模式，往往以 Decorator 或 Wrapper 结尾的类都是使用的装饰器模式

## 功能

- 动态的为一个对象增加新的功能
- 装饰模式是一种用于代替继承的技术，无须通过继承增加子类就能扩展对象的新功能。使用对象的关联关系代替继承关系，更加灵活，同时避免类型体系的快速膨胀

## 角色分类

- Component：抽象构件角色
> 如 IO 流中的 InputStream、OutputStream、Reader、Writer 等
- ConcreteComponent：具体构件角色
> 如 IO 流中的 FileInputStream、FileOutputStream 等
- Decorator：装饰角色，持有一个抽象构件的引用
> 如 IO 流中的 FilterInputStream、FilterOutputStream 等
- ConcreteDecorator：具体装饰角色，负责给构建角色对象增加新的功能
> 如 IO 流中的 BufferedInputStream、BufferedOutputStream 等

## 总结

装饰器模式降低了系统的耦合度，可以动态的增加或删除对象的职责，并使得需要装饰的具体构建类和具体装饰类可以独立变化，以便增加新的具体构建类和具体装饰类

### 优点

- 扩展对象功能，比继承灵活，不会导致类个数急剧增加
- 可以对一个对象进行多次装饰，创造出不同对象的组合，得到功能更加强大的对象
- 具体构建类和具体装饰类可以独立变化，用户可以根据需要自己增加新的具体构建子类和具体装饰子类

### 缺点

- 产生很多小对象，大量小对象占据内存，一定程度上影响性能