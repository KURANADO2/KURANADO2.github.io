---
title:	Java 8 学习总结 - Lambda 表达式（一）
date: 2019-05-04 16:17:00
comments: true
categories: [Java 8]
tags: [Java 8, Lambda]
---

前段时间花功夫重构了公司部分项目代码，把原有杂乱冗余的逻辑重写了一遍，但实际完成后，却觉得自己重构后的代码并没有较之前好到哪里去，仍然是普通的遍历、查找、筛选，代码无脑且又长又丑。所以最近在钻研 Java 8，如何写出优雅、高挑的代码，以改善目前这种情况。

<!-- more -->

## 筛选苹果

苹果类：

```java
@Data
static class Apple {
    private int weight;
    private String color;
 
    Apple(int weight, String color) {
        this.weight = weight;
        this.color = color;
    }
}
```

### 最普通的做法

分别筛选出绿苹果和红苹果：

```java
public static void main(String[] args) {
    List<Apple> inventory = Arrays.asList(new Apple(80, "green"), new Apple(155, "green"), new Apple(120,
            "red"));
    List<Apple> greenApples = filterGreenApples(inventory);
    System.out.println(greenApples);
    List<Apple> redApples = filterRedApples(inventory);
    System.out.println(redApples);
}

private static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if ("green".equals(apple.getColor())) {
            result.add(apple);
        }
    }
    return result;
}

private static List<Apple> filterRedApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if ("red".equals(apple.getColor())) {
            result.add(apple);
        }
    }
    return result;
}
```

### 稍微改进一下

如果再要求筛选出黑苹果、白苹果、紫苹果......那是不是需要为每种颜色都创建一个方法呢？你很聪明，显然不会那么傻乎乎的去一顿复制粘贴，所以你写了下面这样的方法：

```java
private static List<Apple> filterApplesByColor(List<Apple> inventory, String color) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (color.equals(apple.getColor())) {
            result.add(apple);
        }
    }
    return result;
}
```

这样只用一个方法就可以分别筛选出指定颜色的苹果了：

```java
List<Apple> redApples = filterApplesByColor(inventory, "red");
System.out.println(redApples);
```

### 再改进一下

同理如果突然又说需要筛选出较重的苹果，只需要照葫芦画瓢写出下面的代码：

```java
private static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (apple.getWeight() > weight) {
            result.add(apple);
        }
    }
    return result;
}
```

```java
List<Apple> heavyApples = filterApplesByWeight(inventory, 150);
System.out.println(heavyApples);
```

## 换一种实现方式

上面的方法实在太简单、直白、繁琐了，为了把代码写的更简洁，你决定换一种方式实现：

### 定义一个函数式接口

```java
@FunctionalInterface
interface ApplePredicate {

    boolean test(Apple apple);
}
```

### 增加实现类

```java
static class AppleGreenColorPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
        return "green".equals(apple.getColor());
    }

}

static class AppleHeavyWeightPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
        return apple.getWeight() > 150;
    }
}

static class AppleRedAndHeavyPredicate implements ApplePredicate {

    @Override
    public boolean test(Apple apple) {
        return "red".equals(apple.getColor()) && apple.getWeight() > 150;
    }
}
```

### 实现筛选苹果的方法

该方法接受苹果列表和 `ApplePredicate` 接口，调用该方法时传入对应的实现类，就可以在该方法中使用该实现类的方法了

```java
private static List<Apple> filterApples(List<Apple> inventory, ApplePredicate applePredicate) {
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (applePredicate.test(apple)) {
            result.add(apple);
        }
    }
    return result;
}
```

```java
List<Apple> greenApples = filterApples(inventory, new AppleGreenColorPredicate());
System.out.println(greenApples);

List<Apple> heavyApples = filterApples(inventory, new AppleHeavyWeightPredicate());
System.out.println(heavyApples);

List<Apple> redAndHeavyApples = filterApples(inventory, new AppleRedAndHeavyPredicate());
System.out.println(redAndHeavyApples);
```

## 代码真的更简洁了吗？

上面的面向接口编程的确让代码扩展性提高了一个层次，但想想还是太过繁琐了，虽然每增加一种筛选方式，不需要再为其创建对应的 filter 方法，但却需要再定义一个新的实现类，和之前相比，代码依然没有变简洁。好在 Java 中提供了匿名内部类，所以你可以直接使用匿名内部类：

```java
List<Apple> greenApples = filterApples(inventory, new ApplePredicate() {
    @Override
    public boolean test(Apple apple) {
        return "green".equals(apple.getColor());
    }
});
System.out.println(greenApples);
```

## 匿名内部类包含的模板代码是否可以去除

匿名内部类写法中除了 `TODO` 部分，剩下的都是模板代码，而每增加一种筛选方式，都需要创建一个匿名内部类，然后书写这部分模板代码：

```java
new ApplePredicate() {
    @Override
    public boolean test(Apple apple) {
        // TODO
    }
}
```

## Lambda 表达式

几经周折，最终使出了 Lambda 表达式绝招

```java
List<Apple> greenApples = filterApples(inventory, (Apple apple) -> "green".equals(apple.getColor()));
System.out.println(greenApples);
```

## 何为 Lambda 表达式

Lambda 表达式是可传递的匿名函数，有以下特点:

- 匿名：不需要像普通方法那样提供明确的方法名称
- 函数：和方法的区别是，Lambda 函数不属于某个特定的类，但同样有参数列表、函数主体、返回类型以及可以抛出异常列表
- 传递：Lambda 表达式可以作为参数传递给方法或存储在变量中
- 简洁：不需要像匿名内部类那样需要写很多模板代码

使用 Lambda 表达式之前写法：

```java
ApplePredicate predicate = new ApplePredicate() {
    @Override
    public boolean test(Apple apple) {
        return "green".equals(apple.getColor());
    }
};
```

使用 Lambda：

```java
(Apple apple) -> "green".equals(apple.getColor())
```

看到 Lambda 表达式真正的只用到了自己需要的部分，大量的模板代码全都被省略掉了，代码看起来一下子简洁了很多

Lambda 表达式的三个部分：

- Lambda 参数列表：参数列表
- 箭头：把 Lambda 参数列表与 Lambda 主体隔开
- Lambda 主体：具体的行为

![](http://imgblog.kuranado.com/1553784979.png?imageMogr2/thumbnail/!70p)

### 有效的 Lambda 表达式

```java
// 接收一个 String 类型参数，返回 int 类型结果
(String s) -> s.length()
// 接收一个 Apple 类型参数，返回 boolean 类型结果
(Apple a) -> a.getWeight() > 150
// 接收一个 Apple 类型参数，返回 boolean 类型结果，如前面函数式接口 ApplePredicate 下的 test 抽象方法
(Apple apple) -> "green".equals(apple.getColor())
// 接收两个 Apple 类型参数，返回 int 类型结果
(Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())
// 接收两个 int 类型参数，没有返回值（返回 void）,Lambda 表达式可以有多行
(int x, int y) -> {
    System.out.println("Result:");
    System.out.println(x+y);
}
// 不接收任何参数，返回 int 类型结果
() -> 42
// 不接收任何参数，不返回结果，如函数式接口 Runnable 下的 run 方法
() -> {}
// 不接收任何参数，返回 String 类型结果，等价于 () -> "Mario"，注意，return 是一个控制流语句，如果使用了 return 就必须加 {}
() -> {return "Mario";}
// 不接收任何参数，返回创建的对象
() -> new Apple(10)

// Java 编译器自动完成类型推断，可以直接将 (Apple apple1, Apple apple2) 省略为 (apple1, apple2)
Comparator<Apple> comparator = (apple1, apple2) -> apple1.getWeight().compareTo(apple2.getWeight());

// 当 Java 编译器仅需要推断一个参数的类型时，可以省略圆括号，即 (Apple apple) <=> (apple) <=> apple
Predicate<Apple> predicate = apple -> apple.getWeight() > 100;
```

综上，Lambda 的基本语法为：

```java
(parameters) -> expression
(parameters) -> {
    statements;
}
```

### 函数式接口

**只定义了一个抽象方法的接口称之为函数式接口**，如前面定义的 `ApplePredicate` 接口只定义了一个抽象方法 `test`。`@FunctionalInterface` 注解可以用来标识一个接口是函数式接口，用来规范代码，增加可读性。而 **Lambda 表达式可以直接以内联的方式为函数式接口的抽象方法提供实现，并把整个表达式作为函数式接口的实例**，当然匿名内部类也可以做到，只是比较繁琐。

除了自定义函数式接口，Java 8 API `java.util.function` 包下也提供了几个常用函数式接口

#### 常用函数式接口

- `Predicate`

```java
@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);
}
```

定义了 `test` 抽象方法，接收 `T`，返回 `boolean` 类型结果，想想我们上面定义的 `ApplePredicate` 接口，功能是不是完全相同？完全可以用现成的 `Predicate` 接口完成筛选苹果苹果的动作，而且因为其使用了泛型，所以除了筛选苹果，还可以筛选茄子、西瓜、水蜜桃等等：

```java
@FunctionalInterface
interface Predicate<T> {

    boolean test(T t);
}

private static <T> List<T> filter(List<T> list, Predicate<T> predicate) {
    List<T> result = new ArrayList<>();
    for (T t : list) {
        if (predicate.test(t)) {
            result.add(t);
        }
    }
    return result;
}

@Data
static class Eggplant {

    private String color;
    private int length;
 
    public Eggplant(String color, int length) {
        this.color = color;
        this.length = length;
    }
}
```

```java
List<Eggplant> eggplants = Arrays.asList(new Eggplant("purple", 10), new Eggplant("cyan", 15), new Eggplant("red", 13));
// 筛选茄子
List<Eggplant> longEggplants = filter(eggplants, (Eggplant eggplant) -> eggplant.getLength() > 10);
System.out.println(longEggplants);

List<String> stringList = Arrays.asList("", "a", "", "b");
// 筛选出非空字符串
Predicate<String> predicate = (String s) -> !s.isEmpty();
System.out.println(filter(stringList, predicate));

// 筛选出偶数 - Predicate
Predicate<Integer> predicate2 = (Integer i) -> i % 2 == 0;
System.out.println(predicate2.test(1000));

// 筛选出偶数 - IntPredicate
IntPredicate intPredicate = (int i) -> i % 2 == 0;
System.out.println(intPredicate.test(1000));
```

对于 `Predicate`，如果传入基本类型，Java 会完成自动装箱，存在一定的性能损耗，需要使用更多的内存；而对于 `IntPredicate `，直接传入基本类型 int，不会被自动装箱，类似的还有 `LongPredicate`、`DoublePredicate` 等函数式接口

- `Consumer`

```java
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);
}
```

该接口定义了 `accept` 抽象方法，接收 `T`，不返回结果，可以在接收元素后，对元素进行一些操作

```java
private static <T> void forEach(List<T> list, Consumer<T> consumer) {
    for (T t : list) {
        consumer.accept(t);
    }
}
```

```java
Consumer<Integer> consumer = (Integer i) -> System.out.println(i);
forEach(Arrays.asList(1, 2, 3, 4, 5), consumer);
```

同样的，为了避免基本类型被自动装箱，Java API 还提供了 `IntConsumer`、`LongConsumer` 和 `DoubleConsumer` 等函数式接口

- Function

```java
@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
}
```

`apply` 抽象方法接收 `T` 返回 `R`，常被用来将输入对象映射到输出。如输入苹果，输出重量：

```java
private static <T, R> List<R> map(List<T> list, Function<T, R> function) {
    List<R> result = new ArrayList<>(list.size());
    for (T t : list) {
        R r = function.apply(t);
        result.add(r);
    }
    return result;
}
```

```java
// 输入苹果，映射返回苹果重量
Function<Apple, Integer> function = (Apple apple) -> apple.getWeight();
System.out.println(map(inventory, function));

// 输入字符串，映射返回字符串长度
System.out.println(map(Arrays.asList("kuranado", "hello", "lambda"), (String s) -> s.length()));
```

为避免基本类型自动装箱，Java API 提供了 `IntFunction`、`LongFunction`、`DoubleFunction`、`ToIntFunction`、`ToLongFunction`、`ToDoubleFunction`、`IntToLongFunction`、`IntToDoubleFunction`、`LongToIntFunction`、`LongToDoubleFunction`、`DoubleToIntFunction`、`DoubleToLongFunction` 等

## 参考资料

- 《Java 8 实战》