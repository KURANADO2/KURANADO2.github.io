---
title: Java泛型总结
date: 2018-06-9 00:10:00
comments: true
categories: [Java]
tags: [Java,泛型]
---

前段时间做毕设项目时，想通过泛型进一步优化项目代码，但苦于自己对泛型的了解只限于类似 `List<String> list = new ArrayList();` 来约束 List 中只允许放入指定类型的值的程度，本篇文章通过深入了解泛型从而达到在编译期间就可以及时发现问题所在的效果，保证代码质量，减少代码出现 bug 的概率。

<!-- more -->

指定 T 的具体类型，该类型可以是类、接口、数组等，但不能是基本类型

## 泛型类

我们知道使用泛型可以使类型错误在编译时就被检测到，从而能够增加程序的健壮性。

### 定义泛型类

```
public class Generic<T> {

    private T key;

    public Generic(T key) {
        this.key = key;
    }

    public T getKey() {
        return key;
    }

    public void setKey(T key) {
        this.key = key;
    }
}
```

### 实例化泛型类

```
Generic<String> genericString = new Generic<>("abc");
Generic<Integer> genericInteger = new Generic<>(123);
System.out.println(genericString.getKey());
System.out.println(genericInteger.getKey());
```

需要注意的是泛型的类型参数必须是`引用类型`（类、接口、数组等都是 `引用类型`）而不能是简单类型，如 `Generic<int> generic = new Generic<>(123);` 是不允许的。

当然和 List 等一样，实例化泛型类可以传入任意类型而并不一定非要传入泛型类实参，只不过既然我们将类定义为泛型类，其目的就是希望开发者们能够传入确定的类型实参，以增加程序健壮性：

```
Generic generic = new Generic("abc");
Generic generic2 = new Generic(123);
Generic generic3 = new Generic(true);
```

## 泛型接口

### 定义泛型接口

```
public interface Generator<T> {
    public T fun();
}
```

### 实现泛型接口

```
public class PersonGenerator<T> implements Generator<T> {
    @Override
    public T fun() {
        return null;
    }
}
```

可见当类实现泛型接口时若没有传入泛型实参，则需要将泛型也加到类的定义中，否则像下面的代码将会出现编译错误：

```
public class PersonGenerator implements Generator<T> {
    @Override
    public T fun() {
        return null;
    }
}
```

如果实现泛型接口时传入了泛型实参，则该类中所有使用泛型的地方都要替换成传入的泛型实参：

```
public class PersonGenerator implements Generator<String> {
    @Override
    public String fun() {
        return null;
    }
}
```

## 泛型方法

为了判断数组中是否包含某值写了如下两个重载方法：

```
public static void main(String[] args) {
    Integer[] integers = new Integer[]{1, 2, 3};
    String[] strings = new String[]{"a", "b", "c"};
    System.out.println(contains(2, integers));
    System.out.println(contains("b", strings));
}
public static boolean contains(Integer integer, Integer[] integers) {
    return Arrays.asList(integers).contains(integer);
}
public static boolean contains(String s, String[] strings) {
    return Arrays.asList(strings).contains(s);
}
```

但如果还想要判断 Float 类型的数组中是否包含某个值就有需要编写一个重载方法，好在我们可以通过泛型方法有效的避免这些冗余的方法：

```
public static void main(String[] args) {
    Integer[] integers = new Integer[]{1, 2, 3};
    String[] strings = new String[]{"a", "b", "c"};
    Float[] floats = new Float[]{0.1f, 0.2f, 0.3f};
    System.out.println(contains(2, integers));
    System.out.println(contains("b", strings));
    System.out.println(contains(0.2f, floats));
}
public static <T> boolean contains(T t, T[] ts) {
    return Arrays.asList(ts).contains(t);
}
```

需要注意的是方法返回值前需要包含形式参数，如 `<T>`， **否则该方法不能被称为泛型方法**，编译也将出错。
值得一提的是，如果同时保留以下两个方法：

```
public static <T> boolean contains(T t, T[] ts) {
    return Arrays.asList(ts).contains(t);
}
public static boolean contains(Integer integer, Integer[] integers) {
    return Arrays.asList(integers).contains(integer);
}
```

`contains("b", strings)` 会自动匹配泛型方法，而 `contains(2, integers)` 匹配的是普通方法而不是泛型方法。

## 泛型通配符

### 无限定通配符-Unbounded Wildcard

我们知道 Integer、Double、Float 等都是 Number 类的子类，所以下面的代码完全没问题：

```
public static void main(String[] args) {
    printMsg(123);
}
public static void printMsg(Number number) {
    System.out.println(number);
}
```

基本类型 123 被自动装箱成 Integer 类型，而 Integer 又是 Number 类的子类，所以可以作为 printMsg 方法的实参。

但泛型类 `Generic<Number>` 和 `Generic<Integer>` 可以认为是两个完全没有关联的新类型，两者之间不具有任何继承关系，所以下面的代码会出现编译错误：

```
public static void main(String[] args) {
    Generic<Number> genericNumber = new Generic<>(123);
    Generic<Integer> genericInteger = new Generic<>(123);
    printMsg(genericNumber);  // 编译通过
    printMsg(genericInteger); // 编译出错，因为 Generic<Integer> 和 Generic<Number> 二者之间没有任何继承关系
}
public static void printMsg(Generic<Number> generic) {
    System.out.println(generic.getKey());
}
```

而如果就是希望 printMsg 方法既能接收 `Generic<Number>` 又能够接收 `Generic<Integer>`类型，甚至是能够接收传入了任意实参类型的 `Generic` 泛型类（如 `Generic<String>`、`Generic<Random>`等），则需要用到泛型通配符 `?` 了：

```
public static void main(String[] args) {
    Generic<Number> genericNumber = new Generic<>(123);
    Generic<Integer> genericInteger = new Generic<>(123);
    printMsg(genericNumber);  // 编译通过
    printMsg(genericInteger); // 编译通过
}
public static void printMsg(Generic<?> generic) {
    System.out.println(generic.getKey());
}
```

### 上限通配符-Upper Bounded Wildcard

为泛型添加上边界，即传入的类型实参必须是指定类型或指定类型的子类。使用 `extends` 指定上限通配符

```
public static void main(String[] args) {
    Generic<Number> genericNumber = new Generic<>(123);
    Generic<Integer> genericInteger = new Generic<>(123);
    Generic<Float> genericFloat = new Generic<>(0.5f);
    Generic<String> genericString = new Generic<>("Hello");
    printMsg(genericNumber);  // 编译通过
    printMsg(genericInteger); // 编译通过
    printMsg(genericFloat);   // 编译通过
    printMsg(genericString);  // 编译出错
}
public static void printMsg(Generic<? extends Number> generic) {
    System.out.println(generic.getKey());
}
```

因为 `Generic<? extends Number> generic` 指定了传入的类型实参必须是 Number 类或 Number 类的子类，所以`printMsg(genericString);` 出错，因为 String 不是 Number 的子类

### 下限通配符-Lower Bounded Wildcard

和上限通配符类似，下限通配符使用 `super` 关键字实现：

```
public static void main(String[] args) {
    Generic<Number> genericNumber = new Generic<>(123);
    Generic<Integer> genericInteger = new Generic<>(123);
    Generic<Float> genericFloat = new Generic<>(0.5f);
    Generic<String> genericString = new Generic<>("Hello");
    printMsg(genericNumber);  // 编译通过
    printMsg(genericInteger); // 编译通过
    printMsg(genericFloat);   // 编译出错
    printMsg(genericString);  // 编译出错
}
public static void printMsg(Generic<? super Integer> generic) {
    System.out.println(generic.getKey());
}
```

因为 `Generic<? super Integer> generic` 指定了传入的类型实参必须是 Integer 类或 Integer 类的父类，所以 `printMsg(genericFloat);` 和 `printMsg(genericString);` 出现编译错误，因为 Float 和 String 都不是 Integer 类的父类

## 类型擦除-Type Erasure

Java 的泛型只在编译阶段有效，编译过程中正确检验泛型结果后，会将泛型相关信息擦除，并且在对象进入和离开方法的边界处添加类型检查和类型转换的方法，即泛型信息不回进入运行时阶段：

```
public static void main(String[] args) {
    Generic<Integer> genericInteger = new Generic<>(123);
    Generic<String> genericString= new Generic<>("Hello");
    System.out.println(genericInteger.getClass() == genericString.getClass());  // 返回 true
}
```

结果返回 `true` ，说明虽然编译时 `Generic<Integer>` 和 `Generic<String>` 是不同的类型，但因为泛型的类型擦除，所以编译后 `genericInteger` 和 `genericString` 为相同的类型

## 命名规则

JDK 中文档经常能看到 `T`、`K`、`V`、`E`、`N` 等类型参数，而我们在编写泛型相关代码时，这些符号都可以随意使用，实际上还可以使用 JDK 文档中从来没用过的符号，如 `A`、`B`、`C` 等，但却极力不推荐这样做

JDK 文档中各符号的含义：

- T：类型
- K：键
- V：值
- E：元素（如集合类中的元素全部用该符号表示）
- N：Number

我们应该遵循 JDK 中已有的规范

## 参考资料

[java泛型详解](https://blog.csdn.net/u012152619/article/details/47253811)
[java 泛型详解-绝对是对泛型方法讲解最详细的，没有之一](https://blog.csdn.net/s10461/article/details/53941091/)