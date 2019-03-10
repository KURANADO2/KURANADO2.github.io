---
title: Java基础面试题集锦(一)
date: 2018-06-8 17:14:00
comments: true
categories: [Java]
tags: [Java,面试]
---

通过整理一些 Java 面试题，帮助查漏补缺 Java 相关基础知识
本文使用 JDK1.8 进行测试

<!-- more -->

## Arrays.asList

经常会有将数组转换为 List 的需求，而常用的方法就是使用 `java.util.Arrays` 类的 `asList` 静态方法，所以常常有下面这段代码：

```
String[] arr = new String[]{"a", "b", "c"};
List<String> list = Arrays.asList(arr);
```

返回值为 `java.util.List`，于是就对返回值进行`add`、`addAll`、`remove`、`removeAll` 等操作：

```
list.add("d");
```

运行程序时却发现报错如下：

```
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.AbstractList.add(AbstractList.java:148)
	at java.util.AbstractList.add(AbstractList.java:108)
```

查看 `asList` 方法的 API 文档可以看到说明：

> Returns a fixed-size list backed by the specified array.

即 `asList` 方法返回的 List 为固定大小，所以当我们执行会改变 List 元素个数的操作时会抛出异常。如果要深究其原因的话，可以发现 `Arrays.asList()` 的返回值为一个名为 `ArraysList` 的静态内部类而不是 `java.util.ArrayList` 类，该静态内部类继承了 `AbstractList` 类，`AbstractList` 类的 `add` 方法定义如下：

```
public boolean add(E e) {
    add(size(), e);
    return true;
}
public void add(int index, E element) {
    throw new UnsupportedOperationException();
}
```

同理其 `addAll`、`remove`、`removeAll` 方法也都是抛出 `java.lang.UnsupportedOperationException` 异常，又因为`ArrayList` 静态内部类并没有重写 `AbstractList` 类的这些方法，所以会直接调用其父类的方法导致抛出异常。

如果要把数组转换为大小可变的 List 可以通过如下方法解决：

```
ArrayList<String> list = new ArrayList(Arrays.asList(arr));
list.add("d");
```

## 判断数组中是否包含某值的几种方法

### 使用 List

```
String[] arr = new String[]{"a", "b", "c", "d"};
return Arrays.asList(arr).contains("b");
```

### 使用 Set

```
String[] arr = new String[]{"a", "b", "c", "d"};
Set<String> set = new HashSet<>(Arrays.asList(arr));
return set.contains("b");
```

### 使用循环

```
String[] arr = new String[]{"a", "b", "c", "d"};
for(String str : arr) {
	if(str.equals("c")) {
		return true;
	}
}
```

### 使用 ArrayUtils 工具类

如果使用 Maven 的话需要先导入对应依赖如下：

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.3.2</version>
</dependency>
```

```
String[] arr = new String[]{"a", "b", "c", "d"};
return ArrayUtils.contains(arr, "c");
```

## 父类和子类的构造方法

```
class Super<T> {

    private T t;

    public Super(T t) {
        this.t = t;
        System.out.println(t);
    }


}
public class Sub extends Super {

    public Sub() {
        System.out.println("sub");
    }

    public Sub(String s) {
        System.out.println(s);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        Sub sub2 = new Sub("world");
    }

}
```

Sub 子类的两个构造方法都会出现编译错误，究其原因是因为：

> 子类的所有构造方法（无论是有参还是无参）都会调用父类的无参构造方法

此处的父类因为已经定义了有参构造方法，导致编译器不再为其提供默认的无参构造方法，所以会导致子类的构造方法报编译错误，可通过以下任意一种方法解决：

- 为父类显示定义无参构造方法，即 public Super() {}
- 在子类构造方法中通过 super 调用父类的指定构造方法（必须保证 super 出现在子类构造方法中的第一行），就像下面这样：

```
public Sub() {
    super(3);
    System.out.println("sub");
}

public Sub(String s) {
    super("hello");
    System.out.println(s);
}
```

## final 修饰的变量不可变是指引用不可变还是引用的对象不可变

```
final StringBuilder sb = new StringBuilder("hello");
sb.append("world");
```

程序可正常运行，可见 `final` 修饰的变量引用的对象可以改变，但引用不可变，所以下面的代码编译错误：

```
final StringBuilder sb = new StringBuilder("hello");
sb = new StringBuilder("hello world");
```

## 参考资料：
[Top 10 Mistakes Java Developers Make](https://www.programcreek.com/2014/05/top-10-mistakes-java-developers-make/)
[Java 面试宝典2017链接：https://pan.baidu.com/s/1rJE0V3vNHN410Bgq6rdC3g 密码：wu7x](https://pan.baidu.com/s/1rJE0V3vNHN410Bgq6rdC3g)