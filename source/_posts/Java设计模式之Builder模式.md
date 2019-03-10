---
title: Java 设计模式之 Builder 模式
date: 2018-12-22 14:12:00
comments: true
categories: [Java]
tags: [Java, 设计模式, Builder 模式]
---

前段时间，工作中给一个对象设置属性，公司的代码风格都是创建这个对象，然后一个个 set 进去。仅仅是这样一个功能，却因为对象的属性太多，导致需要写很多行代码才完成这么一个简单的工作。所以我想起了 Builder 模式，通过链式调用可以使代码更简洁

<!-- more -->

Builder 模式又称为建造者、生成器模式。日常开发中，很多类名包含 `Builder` 的都是生成器模式，例如 StringBuilder 的 append 方法，

## 生成器模式的本质

- 分离了对象子组件的单独构造和装配，从而可以构造出复杂的对象。建造者模式适用于某个对象的构建过程较为复杂的情况
- 由于实现了构建和装配的解耦，不同的构建器，相同的装配，也可以做出不同的对象；相同的构建器，不同的装配顺序也可以做出不同的对象。构建和装配的解耦，实现了更好的复用

## 生成器模式的构成

生成器模式的构成分为两个部分：

1. Builder 接口：定义如何**构建**各个部件，了解每个部件的构建实现细节
2. Director 接口：定义如何**装配**各个部件，负责产品的整体构建算法

## 举个栗子

下面以生产电脑为例，讲解 Builder 模式：

### Computer 类

```
package com.kuranado.builder;

import lombok.Getter;
import lombok.Setter;

/**
 * 电脑类
 * @Author: Xinling Jing
 * @Date: 2018-12-21 15:01
 */
@Setter
@Getter
public class Computer {

    private CPU cpu;
    private Graphics graphics;
    private HardDisk hardDisk;

    public Computer(CPU cpu, Graphics graphics, HardDisk hardDisk) {
        this.cpu = cpu;
        this.graphics = graphics;
        this.hardDisk = hardDisk;
    }

    public void start() {
        System.out.println("开机中...");
    }
}

class Hardware {

    /**
     * 品牌
     */
    private String brand;
    /**
     * 型号
     */
    private String model;

    public Hardware(String brand, String model) {
        this.brand = brand;
        this.model = model;
    }
}

/**
 * CPU
 */
class CPU extends Hardware {

    /**
     * Cache
     */
    private String cache;

    public CPU(String brand, String model, String cache) {
        super(brand, model);
        this.cache = cache;
    }
}

/**
 * 显卡
 */
class Graphics extends Hardware {

    /**
     * 显存
     */
    private String videoMemory;

    public Graphics(String brand, String model, String videoMemory) {
        super(brand, model);
        this.videoMemory = videoMemory;
    }
}

/**
 * 硬盘
 */
class HardDisk extends Hardware {

    /**
     * 硬盘容量
     */
    private String capacity;

    public HardDisk(String brand, String model, String capacity) {
        super(brand, model);
        this.capacity = capacity;
    }
}
```

### ComputerBuilder 接口

```
package com.kuranado.builder;

/**
 * @Author: Xinling Jing
 * @Date: 2018-12-22 10:17
 */
public interface ComputerBuilder {

    CPU buildCPU();
    Graphics buildGraphics();
    HardDisk buildHardDisk();

}
```

### MacComputerBuilder 实现类

具体的 Builder 知道每个部件的实现细节。可以定义很多种 Builder，每种 Builder 在部件细节实现上都不尽相同，从而构建出不同的对象

```
package com.kuranado.builder;

/**
 * @Author: Xinling Jing
 * @Date: 2018-12-22 10:19
 */
public class MacComputerBuilder implements ComputerBuilder {

    @Override
    public CPU buildCPU() {
        System.out.println("构建 Mac CPU");
        return new CPU("Intel", "Xeon W", "19MB");
    }

    @Override
    public Graphics buildGraphics() {
        System.out.println("构建 Mac 显卡");
        return new Graphics("AMD", "Radeon Pro Vega 56", "8GB");
    }

    @Override
    public HardDisk buildHardDisk() {
        System.out.println("构建 Mac 硬盘");
        return new HardDisk("三星", "SM128C", "2TB");
    }
}
```

### ComputerDirector 接口

```
package com.kuranado.builder;

/**
 * @Author: Xinling Jing
 * @Date: 2018-12-22 10:39
 */
public interface ComputerDirector {

    Computer directorComputer();

}
```

### MacComputerDirector 实现类

具体的 Director 负责将部件装配起来，Director 知道整个产品的装配细节。可以定义很多 Director，从而装配出不同的对象

```
package com.kuranado.builder;

import lombok.Getter;
import lombok.Setter;

/**
 * @Author: Xinling Jing
 * @Date: 2018-12-22 10:40
 */
@Setter
@Getter
public class MacComputerDirector implements ComputerDirector {

    private ComputerBuilder computerBuilder;

    public MacComputerDirector(ComputerBuilder computerBuilder) {
        this.computerBuilder = computerBuilder;
    }

    @Override
    public Computer directorComputer() {

        // 调用 Builder 构建每个部件
        CPU cpu = computerBuilder.buildCPU();
        Graphics graphics = computerBuilder.buildGraphics();
        HardDisk hardDisk = computerBuilder.buildHardDisk();

        // 装配电脑
        return new Computer(cpu, graphics, hardDisk);
    }
}
```

实际应用中 Director 需要进行复杂的运算，然后根据需要，调用 Builder 中的方法生成需要的部件对象并按照某种算法装配这些部件。实际开发中可能会有如下几种情况(摘自《研磨设计模式》)：

> - 在运行指导者的时候，会按照整体构建算法的步骤进行运算，可能先运行前几步运算，到了某一步骤，需要具体创建某个部件对象了，然后就调用 Builder 中创建相应部件调度方法来创建具体的部件。同时把前面运算得到的数据传递给 Builder，因为在 Builder 内部实现创建和组装部件的时候，可能会需要这些数据
> - Builder 创建完具体的部件对象后，会把创建好的部件对象返回给装配者，装配者继续后续的算法运算，可能会用到已经创建好的对象
> - 如此反复下去，直到整个构建算法完成，整个产品也就创建好了

### 客户端调用

```
package com.kuranado.builder;

/**
 * @Author: Xinling Jing
 * @Date: 2018-12-17 21:12
 */
public class Client {

    public static void main(String[] args) {
        ComputerDirector director = new MacComputerDirector(new MacComputerBuilder());
        Computer computer = director.directorComputer();
        
        computer.start();
    }
}
```

程序运行结果：

```
构建 Mac CPU
构建 Mac 显卡
构建 Mac 硬盘
开机中...
```

## 生成器模式的优点

- 松散耦合：生成器模式可以用同一个构建算法构建出表现上完全不同的产品，实现产品构建和产品表现上的分离。
- 可以很容易的改变产品的内部表示：Builder 对象提供接口给 Director 使用，所以具体不见的创建和装配方式被 Builder 接口隐藏了，Director 并不知道具体的实现细节。
- 复用性好：因为实现了构建算法和具体产品实现的分离，所以构建算法和具体的产品装配两者都可以复用。同一个构建算法可以应用到不用的具体产品实现中，同一个具体产品实现，也可以配合不同的构建算法。

这就是生成器模式，是不是很简单呢！

## 思考

现在在考虑一个问题，在 directorComputer 方法中我们组装电脑使用了这样一行代码：

```
return new Computer(cpu, graphics, hardDisk);
```

想想这里有没有可以优化的地方呢？

想不到？

那倘若 Computer 类不止 CPU、Gragphics、HardDisk 这三个属性呢，比如 Computer 类还有 NetworkCard、SoundCard、MainBoard、Power、CDRom、Memory 等属性，那这时我们是不是要写这样一行代码来创建 Computer 对象：

```
return new Computer(cpu, graphics, hardDisk, networkCard, soundCard, mainBoard, power, cdRom, memory);
```

遇到这种参数较多的情况，很多人还会重叠构造器：

```
public Computer(CPU cpu) {
    this(cpu, null);
}

public Computer(CPU cpu, Graphics graphics) {
    this(cpu, graphics, null);
}

public Computer(CPU cpu, Graphics graphics, HardDisk hardDisk) {
    this(cpu, graphics, hardDisk, null)
}

...

public Computer(cpu, graphics, hardDisk, networkCard, soundCard, mainBoard, power, cdRom, memory) {
    this.cpu = cpu;
    this.graphics = graphics;
    this.hardDisk = hardDisk;
    ...
}
```

在参数很多的情况下，重叠构造器有很多缺点：

1. 层层嵌套，代码不灵活
2. 不优雅
3. 如果增加参数那就是噩梦

这时候就有人说了：可以提供空参构造方法，然后一个个调 setter 方法设置属性就不用重叠构造器了！

```
Computer computer = new Computer();
computer.setCPU(cpu);
computer.setGragphics(gragphics);
computer.setHardDisk(hardDisk);
computer.setNetworkCard(networkCard);
computer.setSoundCard(soundCard);
computer.setMainBoard(mainBoard);
computer.setPower(power);
computer.setCDRom(cdRom);
computer.setMemory(memory);
```

事实上，我们项目组的同事都是通过这种 set 的形式设置对象的，然而这真的是一个好的代码风格吗？《阿里 Java 开发手册》中推荐方法代码不超过 80 行，照这样创建对象很难符合规范。在工作中发现了这一点，想到了 Android 的 Api 中存在大量的链式调用，我们也可以将自己的代码修改成链式调用：

```
package com.kuranado.builder;

import lombok.Getter;
import lombok.Setter;

/**
 * 电脑类
 * @Author: Xinling Jing
 * @Date: 2018-12-21 15:01
 */
@Setter
@Getter
public class Computer {

    private CPU cpu;
    private Graphics graphics;
    private HardDisk hardDisk;

    public Computer(Builder builder) {
        this.cpu = builder.cpu;
        this.graphics = builder.graphics;
        this.hardDisk = builder.hardDisk;
    }

    public void start() {
        System.out.println("开机中...");
    }

    public static class Builder {

        private CPU cpu;
        private Graphics graphics;
        private HardDisk hardDisk;

        public Builder setCpu(CPU cpu) {
            this.cpu = cpu;
            return this;
        }

        public Builder setGraphics(Graphics graphics) {
            this.graphics = graphics;
            return this;
        }

        public Builder setHardDisk(HardDisk hardDisk) {
            this.hardDisk = hardDisk;
            return this;
        }

        Computer build() {
            return new Computer(this);
        }
    }
}
```

内部类中的每个 set 方法都返回内部类自己，对于 set 方法可以使用 IDEA 自动生成，但需要注意需要选择 Builder 的模板：

![](http://imgblog.kuranado.com/1545458439.png)

这样我们就可以这样创建 Computer 对象：

```
new Computer.Builder().setCpu(cpu).setGraphics(graphics).setHardDisk(hardDisk).build();
```

这样即便在属性很多时代码看起来也很简洁，当然这种链式调用也存在一定缺点，比如：

- 每次创建外部类对象，都需要再创建一个内部类，所以需要消耗更多的内存
- 每个属性要同时在内部类和外部类中定义

所以大家具体根据使用场景而定

## 参考资料

- 《Effective Java》
- 《研磨设计模式》
- [学长博客](https://mrdear.cn/2018/03/06/experience/design_patterns--builder-model/)
- [【GOF23设计模式】 建造者模式详解](https://www.youtube.com/watch?v=03ly8k1FkVc)