---
title: ArrayList源码解读(一)
date: 2018-06-15 17:51:00
comments: true
categories: [Java]
tags: [Java,ArrayList]
---

Java 集合中的 ArrayList 应该是开发过程中最常用到的类了，记得第一次去上海面试时，还被面试官问到 ArrayList 的扩容规则，可见深入了解其实现原理是很有必要的。

本文使用 JDK 1.8。

<!-- more -->

首先需要知道的一点是之所以能够向 ArrayList 中添加任意个数的元素，而不需要指定 ArrayList 的大小，是因为 ArrayList 基于数组实现了动态扩容。其大致原理就是 ArrayList 会定义一个用于存储数据的数组，当添加的元素超过该数组的容量时，ArrayList 会重新创建一个更大的数组，将原数组中的内容复制到新数组中后，再向新数组中添加元素。为了能像该数组中添加各种类型的数据，要求该数组为 Object 数组。另外 ArrayList 没有实现同步，所以执行效率较高，如需同步需要使用 `synchronized` 关键字。

## 源码分析

### 变量

下面来分析 ArrayList 源码证明上面的说法：

ArrayList 类定义了一个名为 `DEFAULT_CAPACITY` 的静态 final 变量，该变量表示 ArrayList 的默认初始化容量为 10：

```
private static final int DEFAULT_CAPACITY = 10;
```

两个不包含任何元素的 Object 数组：

```
private static final Object[] EMPTY_ELEMENTDATA = {};

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
```

**真正存储 ArrayList 中元素的 Object 数组**，该数组使用 transient 关键字修饰，所以不会被序列化，这是出于安全考虑，因为 ArrayList 中存储的元素可能会包含一些敏感信息（如密码），序列化之后会在网络中传输，非常危险（事实上，Oracle 官方最近也发声说 Java 序列化是 Java 中最糟糕的存在，将在以后的 JDK 版本中逐渐废除序列化这一特性）。此外为了简化嵌套类访问该 Object 数组，所以没有使用 private 修饰符：

```
transient Object[] elementData; // non-private to simplify nested class access
```

modCount 变量继承自 AbstractList 类，用于记录 **ArrayList 结构性变化的次数**，在 add，remove，addAll，removeAll，trimToSize，clear 等会引起 ArrayList 结构性变化的方法中都会使该变量值加 1，在 ArrayList 的迭代器内部类中会将该变量的值 赋给 `expectedModCount` 变量，这样迭代时如果 modCount 和 expectModCount 如果不相等就可以抛出 `ConcurrentModificationException` 异常：

```
protected transient int modCount = 0;
```

size 为 ArrayList 中**实际保存元素个数**：

```
private int size;
```

### size

返回 ArrayList 中保存的元素个数：

```
public int size() {
    return size;
}
```

### 构造方法

无参构造方法，会把不包含任何元素的 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 数组赋给 elementData：

```
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

指定 ArrayList 初始化容量的构造方法：

```
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
    }
}
```

从该构造方法得知，当传入的容量 initialCapacity 大于 0 时，会创建一个 initialCapacity 大小的 Object 数组并赋给 elemenData。若 initialCapacity 等于 0，则将不包含任何元素的 EMPTY_ELEMENTDATA 空数组赋给 elementData。

可传入集合的构造方法：

```
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

该构造方法将传入的集合转换为数组赋给 elementData，若 elemenData 类型不是 Object 数组，则调用 Arrays.copyOf() 方法将 elementData 转换为 Object 数组。

### trimToSize

trimToSize() 方法将 ArrayList 的容量调整为其实际拥有的元素个数，比如 ArrayList 当前容量为 10，但却只拥有 2 个元素，此时就可以调用 trimeToSize() 方法将 ArrayList 的容量调整为 2，从而减少空间浪费。因为 size 表示 ArrayList 实际存储元素个数，elementData.length 表示 ArrayList 的容量，所以具体实现原理如下：

```
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0) ? EMPTY_ELEMENTDATA : Arrays.copyOf(elementData, size);
    }
}
```

### ensureCapacity

该方法用于确保 ArrayList 至少拥有 minCapacity 的容量：

```
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != EMPTY_ELEMENTDATA)
        // any size if real element table
        // larger than default for empty table. It's already supposed to be
        // at default size.
        : DEFAULT_CAPACITY;
    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}
```

### isEmpty

当 ArrayList 中实际元素个数为 0 时，isEmpty() 方法返回 true：

```
public boolean isEmpty() {
    return size == 0;
}
```

### contains

contains() 方法用于判断 ArrayList 中是否包含某元素，该方法调用 indexOf() 方法查找元素。indexOf 方法通过 for 循环遍历查找元素需要分两种情况：元素为 `null` 时通过 `==` 判断元素是否等于 `null`，元素非空时通过 `equals()` 方法进行比较，若查找到对应元素返回其索引，否则返回 -1，containes() 方法根据 indexOf() 返回值的正负判断是否包含要查找的元素：

```
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}

public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++) if (elementData[i] == null) return i;
    } else {
        for (int i = 0; i < size; i++) if (o.equals(elementData[i])) return i;
    }
    return - 1;
}
```

同理，ArrayList 的 lastIndexOf() 方法查找元素和 indexOf() 类似，区别在于 for 循环使用倒序遍历：

```
public int lastIndexOf(Object o) {
    if (o == null) {
        for (int i = size - 1; i >= 0; i--) if (elementData[i] == null) return i;
    } else {
        for (int i = size - 1; i >= 0; i--) if (o.equals(elementData[i])) return i;
    }
    return - 1;
}
```

### clone

ArrayList 的 clone() 方法实现的是`浅拷贝`，所谓`浅拷贝`即只拷贝对象的引用，不需要再申请新的堆内存放对象；而`深拷贝`会申请堆内存，然后拷贝出一份新的对象。

```
public Object clone() {
    try {
        ArrayList <?> v = (ArrayList <?>) super.clone();
        v.elementData = Arrays.copyOf(elementData, size);
        v.modCount = 0;
        return v;
    } catch(CloneNotSupportedException e) {
        // this shouldn't happen, since we are Cloneable
        throw new InternalError(e);
    }
}
```

下面以一个小例子来帮助理解什么是 `浅拷贝`：

```
public class Test {

    public static void main(String[] args) {
        Student student = new Student("jing", 14);
        Student student2 = new Student("wang", 16);
        ArrayList <Student> list = new ArrayList <>();
        list.add(student);
        list.add(student2);
    }

}

class Student {

    private String name;
    private int age;

    Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

}
```

上面的代码在内存中的情景如下图：

![mark](http://imgblog.kuranado.com/blog/180613/5cfmJLGLGI.png)

调用 ArrayList 的 clone 方法实现浅拷贝：

```
ArrayList<Student> listClone = (ArrayList<Student>) list.clone();
System.out.println("list:" + list);
System.out.println("listClone" + listClone);
```

此时内存中情景：

![mark](http://imgblog.kuranado.com/blog/180613/Dl6IC5d0ah.png)

同时控制台打印出的结果如下：

```
list:[com.kuranado.Student@74a14482, com.kuranado.Student@1540e19d]
listClone[com.kuranado.Student@74a14482, com.kuranado.Student@1540e19d]
```

这表明 ArrayList 的 clone 方法并没有创建新的对象，只是把原有对象的引用交给 listClone 罢了。

如果调用 remove 方法删除元素，则只需要删除对应的引用，当对象没有任何引用指向它时，才交由垃圾回收器回收：

```
listClone.remove(student2);
```

![mark](http://imgblog.kuranado.com/blog/180613/kbIJFeJja1.png)

### toArray

toArray() 方法有两个，一个返回 Object 数组，一个返回泛型数组：

```
public Object[] toArray() {
    // 将 ArrayList 中的元素拷贝到新数组中返回
    return Arrays.copyOf(elementData, size);
}

@SuppressWarnings("unchecked") 
public <T> T[] toArray(T[] a) {
    // 传入的泛型数组长度小于 ArrayList 中元素实际个数
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        // 将 ArrayList 中的元素拷贝到新数组中返回
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    // 传入的泛型数组长度大于等于 ArrayList 中元素实际个数，将 ArrayList 中的元素拷贝到传入的泛型数组中
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size) 
        a[size] = null;
    return a;
}
```

### get

get() 方法根据索引查找元素，首先调用 rangeCheck() 方法检查索引值是否越界，越界则抛出 `IndexOutOfBoundsException` 异常，然后再调用 elementData() 方法根据索引返回 elementData 数组中对应元素：

```
public E get(int index) {
    // 检查索引是否越界
    rangeCheck(index);
    
    return elementData(index);
}

@SuppressWarnings("unchecked")
E elementData(int index) {
    // 类型转换，返回对应类型元素
    return (E) elementData[index];
}
    
private void rangeCheck(int index) {
    // 索引值大于等于 ArrayList 中实际元素个数
    if (index >= size) 
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

### set

 set() 方法首先检查索引参数是否越界，越界则抛出 `IndexOutOfBoundsException` 异常，设置新值后返回修改前的元素值：
```
public E set(int index, E element) {
    rangeCheck(index);

    // 取出修改前的元素值
    E oldValue = elementData(index);
    // 设置新值
    elementData[index] = element;
    return oldValue;
}
```

### add

向 ArrayList 中添加元素，牵扯到 ArrayList 扩容的核心实现，是非常重要的一部分，自己编写类似的工具类时，也可以借鉴其相关思想。而且 ArrayList 扩容原理也是面试常问问题：

```
public boolean add(E e) {
    // 要向 ArrayList 中添加一个元素，首先需要确保 ArrayList 的 elementData 数组至少拥有 size + 1 的容量
    ensureCapacityInternal(size + 1); // Increments modCount!!
    // 添加元素，然后 size + 1
    elementData[size++] = e;
    return true;
}

/**
 * ensureCapacityInternal() 方法用于确保 ArrayList 至少拥有 minCapacity 的容量：
 */
private void ensureCapacityInternal(int minCapacity) {
    // 如果 elementData 为空数组
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        // 在默认容量（10）和 minCapacity 中取最大值
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    // 如果 minCapacity 大于 ArrayList 当前容量，则说明需要将 ArrayList 需要扩容至至少 minCapacity 的容量
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

重点在于 grow() 方法，该方法基本体现了 ArrayList 的扩容原理：首先 oldCapacity 为 ArrayList 的原容量，newCapacity 为 ArrayList 扩容后的新容量大小，并且`newCapacity = oldCapacity + (oldCapacity >> 1);` 说明了新容量是旧容量的 `1.5` 倍。看到这里面试官问你 ArrayList 扩容后的容量是扩容前的多少倍时，如果回答 `1.5` 倍，其实并不是一个太满意的答案。因为当新容量小于 minCapacity 时，会将 minCapacity 赋给 newCapacity，而如果新容量大于 `Integer.MAX_VALUE - 8` 时，则新容量的值等于 `Integer.MAX_VALUE`，所以 ArrayList 扩容后的新容量并不一定是准确的 1.5 倍！

add(int index, E element) 方法在指定索引处插入元素，其扩容实现和 add 方法相同。另外因为 ArrayList 使用数组保存元素，所以其是一个线性表，线性表中插入元素需要将元素依次后移：

```
public void add(int index, E element) {
    // 检查索引是否越界，因为可能会将元素添加到 ArrayList 最后的位置，所以 index == size 并不算作是越界
    rangeCheckForAdd(index);

    ensureCapacityInternal(size + 1); // Increments modCount!!
    // index 处及其后面的元素全部向后移动一个位置
    System.arraycopy(elementData, index, elementData, index + 1, size - index);
    elementData[index] = element;
    size++;
}

private void rangeCheckForAdd(int index) {
    if (index > size || index < 0) 
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

为了便于理解，此处画图分析，假设 ArrayList 中当前有 6 个元素，即 size = 6，执行 add(1, "g"); 后首先会检查索引是否越界，然后调用 ensureCapacityInternal(6 + 1); 确保 ArrayList 的容量至少为 7，从 index = 1 处开始后面的 size - index = 5 个元素依次向后移动一个位置，最后将元素 g 插入到 index = 1 处：

![mark](http://imgblog.kuranado.com/blog/180614/bedjk6jE6b.png)

### addAll

addAll(Collection<? extends E> c) 方法直接将 Collection 中的元素添加到 ArrayList 末尾，该方法使用泛型`上限通配符`约定 Collection 中的元素类型：

```
public boolean addAll(Collection <? extends E> c) {
    Object[] a = c.toArray();
    // Collection 集合中元素个数
    int numNew = a.length;
    // 确保 ArrayList 至少拥有 size + a.length 的容量
    ensureCapacityInternal(size + numNew); // Increments modCount
    // 将 Collection 中的元素添加到 ArrayList 的末尾
    System.arraycopy(a, 0, elementData, size, numNew);
    // 更新 size 值
    size += numNew;
    return numNew != 0;
}
```

addAll(int index, Collection <? extends E> c) 方法在指定索引处插入集合元素，元素需要依次向后移动，原理和 add(int index, E element) 相同，不再赘述：

```
public boolean addAll(int index, Collection <? extends E> c) {
    // 检查索引是否越界
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    // Collection 集合中元素个数
    int numNew = a.length;
    ensureCapacityInternal(size + numNew); // Increments modCount
    // 需要向后移动的元素个数
    int numMoved = size - index;
    // 如果 Collection 并不是要插入到 ArrayList 末尾
    if (numMoved > 0) 
        // numMoved 个元素向后移动 numNew 个位置
        System.arraycopy(elementData, index, elementData, index + numNew, numMoved);

    // Collection 集合中的元素拷贝到 elementData 数组中
    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```

### clear

clear() 方法使用 for 循环遍历将 elementData 数组中的所有元素都置为 null，并且设置 size = 0 表示 ArrayList 中没有任何元素；clear() 方法中并没有调整 ArrayList 的容量，所以开发者在使用 clear() 方法之后可以调用 trimToSize() 方法释放一部分空间：

```
public void clear() {
    modCount++;

    // clear to let GC do its work
    for (int i = 0; i < size; i++) elementData[i] = null;

    size = 0;
}
```

### remove

根据索引移除元素：

```
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    // 需要移动的元素数
    int numMoved = size - index - 1;
    // 要移除的元素不是 ArrayList 的最后一个元素
    if (numMoved > 0) System.arraycopy(elementData, index + 1, elementData, index, numMoved);
    elementData[--size] = null; // clear to let GC do its work
    return oldValue;
}
```

同样假设 ArrayList 中当前有 6 个元素，即 size = 6，执行 remove(1); 后首先检查索引是否越界，numMoved = size - index - 1 = 6 - 1 -1 = 4，numMoved 大于 0 说明要移除的元素不是最后一个元素，所以需要将 index = 1 后面的 4 个元素依次向前移动一个位置：

![mark](http://imgblog.kuranado.com/blog/180614/E5gkfg0m1b.png)

remove(Object o) 查找对应的元素然后删除该元素，根据要查找元素是否为 null，分两种情况进行 for 循环遍历，一旦查找到元素，则将元素的索引传给 fastRemove() 方法，而 fasetRemove() 方法移除元素的原理和 remove(int index) 方法完全相同：

```
public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++) 
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++) if (o.equals(elementData[index])) {
            fastRemove(index);
            return true;
        }
    }
    return false;
}

private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0) 
        System.arraycopy(elementData, index + 1, elementData, index, numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```

### removeRange

注意该方法使用 `protected` 修饰，只有同类、同包、子类才能够访问到该方法，也就是说我们如果需要使用 removeRange 方法就必须自定义类继承 ArrayList 才能够使用该方法，那为什么 JDK 不直接把该方法暴露出来给程序员调用呢？原因是为了减少方法冗余，因为和 removeRange 同样的功能完全可以通过 `list.subList(fromIndex, toIndex).clear()` 实现。其中 subList() 方法返回 ArrayList 的内部类 SubList，SubList 类继承了 AbstractList 类的 clear() 方法，该 clear() 方法调用 SubList 内部类中的 removeRange() 方法，而该 removeRange() 方法调用了 ArrayList 外部类的 removeRange() 方法。关于方法冗余 《Effective Java》 第 40 节中也有提到，不过我更推荐大家去看下这个问题下的回答：[Why is Java's AbstractList's removeRange() method protected?
](https://stackoverflow.com/questions/2289183/why-is-javas-abstractlists-removerange-method-protected)

```
// Java 中的区间都是左闭右开型，即移除 [formIndex, toIndex) 区间内的元素
protected void removeRange(int fromIndex, int toIndex) {
    modCount++;
    // 要向前移动的元素数
    int numMoved = size - toIndex;
    // 因为是向前移动元素，且不需要保留旧元素，所以直接移动覆盖旧元素即可
    System.arraycopy(elementData, toIndex, elementData, fromIndex, numMoved);

    // clear to let GC do its work
    int newSize = size - (toIndex - fromIndex);
    for (int i = newSize; i < size; i++) {
        elementData[i] = null;
    }
    size = newSize;
}
```

假设 ArrayList 中当前有 6 个元素，即 size = 6，执行 removeRange(1, 3)，要移动的元素数 numMoved = size - toIndex = 6 - 3 = 3， newSize = size - (toIndex - fromIndex) = 6 - (3 - 1) = 4：

![mark](http://imgblog.kuranado.com/blog/180615/Gjj85CL300.png)

### batchRemove

removeAll 删除 ArrayList 中 ArrayList 和 Collection 交集部分的元素，保留 ArrayList 中其它元素。相反，retainAll 保留 ArrayList 中 ArrayList 和 Collection 交集部分的元素，删除 ArrayList 中其它元素。重点在于 batchRemove 方法的实现，而该方法 try 代码块中实际上是两层 for 循环，所以效率较低：

```
public boolean removeAll(Collection <?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, false);
}

public boolean retainAll(Collection <?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, true);
}

private boolean batchRemove(Collection <?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    int r = 0,
    w = 0;
    boolean modified = false;
    try {
        for (; r < size; r++) if (c.contains(elementData[r]) == complement) elementData[w++] = elementData[r];
    } finally {
        // Preserve behavioral compatibility with AbstractCollection,
        // even if c.contains() throws.
        // r != size 说明上面的 for 循环出现了异常
        if (r != size) {
            System.arraycopy(elementData, r, elementData, w, size - r);
            w += size - r;
        }
        // for 循环正常结束
        if (w != size) {
            // clear to let GC do its work
            for (int i = w; i < size; i++) elementData[i] = null;
            modCount += size - w;
            size = w;
            modified = true;
        }
    }
    return modified;
}
```

以 removeAll 方法为例：

```
ArrayList <String> list = new ArrayList <>(Arrays.asList("a", "b", "c", "d", "e", "f"));
Collection <String> collection = new HashSet <>(Arrays.asList("b", "g", "e"));
list.removeAll(collection);
System.out.println(list);
```

代码执行过程如下：

![mark](http://imgblog.kuranado.com/blog/180615/FccL2heK5g.png)

到此 ArrayList 源码已经分析一半了，写的有点乱，而且文章冗长，本想着只抽重要方法分析一下就得了，可当认真读源码时才发现大多数方法都是一环扣一环，每个方法都有其设计精髓之处，以我的水平很难充分挖掘其设计思想，所以写的不好还请大家见谅！

另外一半代码下篇文章继续！

## 总结

- size 变量表示 ArrayList 中实际保存元素个数
- elementData 数组是实际保存 ArrayList 元素的 Object 数组
- grow() 方法是 ArrayList 扩容的核心实现
- modCount 和 expectedModCount 值不相等时抛出 `ConcurrentModificationException` 异常