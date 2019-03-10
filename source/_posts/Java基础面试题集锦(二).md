---
title: Java基础面试题集锦(二)
date: 2018-06-17 16:41:00
comments: true
categories: [Java]
tags: [Java,面试]
---

本文使用 JDK1.8 进行测试

<!-- more -->

## `==` 和 `equals` 的区别

`==` 在基本类型中比较的是值是否相同；在引用类型中，`==` 和 `equals` 一样比较的都是对象的地址是否相 同。究其源码，Object 类中的 `equals` 方法定义如下：

```
public boolean equals(Object obj) {
    return (this == obj);
}
```

所以当一个类没有重写继承自Object 类的 `equals` 方法时，默认比较的都是对象地址是否相同，和 `==` 的效果完全一致。JDK 中已经重写了 `equals` 方法的类有 String、Integer、Float 等。如 String 类的 `equals` 方法如下：

```
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String) anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n--!=0) {
                if (v1[i] != v2[i]) return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

## `equals` 和 `hashCode` 的区别

我们知道 Java 中的 Set 集合可以保证存储元素的唯一性，即如果试图向 Set 中添加重复元素会添加失败。而 Set 保证元素唯一性的根据是什么，可能有人会说通过重写 `equals` 方法判断元素是否相同不就好了。但如果随着 Set 中存放的元素越来越多，每次都通过 `equals`方法判断效率会大大降低，所以才有了 `hashCode` 方法的存在。通过重写 Object 类的 `hashCode` 方法，直接计算出对象将在内存中的存放地址，若该地址没有元素，则将对象存放到该处，如果该地址已经有对象存在，才会调用 `equals` 方法判断该元素与 Set 中的任意元素是否相同，若存在相同的元素，则添加失败，若不存在，则将该元素散列到其他位置。

比如 String 类重写 hashCode 方法如下：

```
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

所以 String 类计算 hash 值的算法为：s[0] * 31 ^ (n - 1) + s[1] * 31 ^ (n - 2) + ... + s[n - 1]

当 `String str = "abc";` 时，打印 `str.hashCode()` 值为 96354，即 97 * 31 ^ 2 + 98 * 31 ^ 1 + 99 * 31 ^ 0 = 96354

综上，可以得出以下两个结论：

- 两个对象相同，`equals` 返回值一定相同，`hashCode` 返回值一定相同
- 两个对象不同，`equals`  返回值一定不同，`hashCode` 返回值可能相同

所以当需要重写 `equals` 时，总会同时重写 `hashCode` 方法，这样才能够减少 `equals` 方法的调用，而 `hashCode` 方法的重写应该尽可能做到对于不同的对象，返回的 hash 值不同

此外当重写完 `equals` 方法时请检查是否符合以下 5 点要求：

1.  自反性：对任意引用值X，x.equals(x)的返回值一定为true. 
2.  对称性：对于任何引用值x,y,当且仅当y.equals(x)返回值为true时，x.equals(y)的返回值一定为true; 
3.  传递性：如果x.equals(y)=true, y.equals(z)=true,则x.equals(z)=true 
4.  一致性：如果参与比较的对象没任何改变，则对象比较的结果也不应该有任何改变 
5.  非空性：任何非空的引用值X，x.equals(null)的返回值一定为false 

## 创建对象的几种方式

- `new` 关键字可以调用有参和无参的构造函数
- `Class` 类的 `newInstance` 方法：

```
Student s = (Student)Class.forName("com.kuranado.Student").newInstance();
```

或：

```
Student s = Student.class.newInstance();
```

- `Constructor` 类的 `newInstance` 方法：

```
Constructor<Student> constructor = Student.class.getConstructor();
Student s = constructor.newInstance();
```

事实上，java.lang.Class 的 newInstance 方法内部调用的就是 java.lang.reflect.Constructor 类的 newInstance 方法，Spring 框架使用的就是 Constructor 类的 newInstance 方法创建对象

- 使用 `clone` 方法：

```
Student s = new Student();
Student s2 = (Student) s.clone();
```

需要注意要使用 clone 方法需要 Student 类实现 `Cloneable` 接口，否则 Student 对象调用该方法时会抛出 `CloneNotSupportedException` 异常。另外 `clone` 方法为一个 `native` 方法，其底层使用 C 语言实现，所以按照惯例重写该方法时，需要通过 `super.clone()` 调用 Object 中的该方法。

## native 关键字

相信大家在读 JDK 源码的时候，会经常看到 `native` 关键字修饰的方法，比如 Object 类中有一半方法都使用了 `native` 关键字：

```
private static native void registerNatives();
public final native Class<?> getClass();
public native int hashCode();
protected native Object clone() throws CloneNotSupportedException;
public final native void notify();
public final native void notifyAll();
```

`native method` 译为`本地方法`，本地方法是指一个 Java 调用非 Java 代码的接口，也就是说该方法并不是用 Java 语言实现的，而在 Java 中则是指 C 语言。因为这些方法需要与操作系统底层直接交互，直接使用 Java 实现会很困难，而且性能较低，所以使用其它更适合于操作系统交互的语言来编写这些方法是提升方法性能的一种方式。

Java 中 native 方法的编写需要使用 C，至于如何自己编写 native 方法可参考这里：[Java的native关键字](https://blog.csdn.net/jiakw_1981/article/details/3073613)
