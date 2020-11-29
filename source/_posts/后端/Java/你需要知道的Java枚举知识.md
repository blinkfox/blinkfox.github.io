---
title: 你需要知道的Java枚举知识
date: 2018-10-27 23:20:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/10/27/java-enum.jpg
categories: 后端
tags:
  - Java
---

## 概述

### 定义

**枚举**（`enum`全称为`enumeration`）类型是`Java 5`新增的类型，存放在`java.lang`包中，允许用常量来表示特定的数据片断，而且全部都以类型安全的形式来表示。

### 定义格式

 创建枚举类型要使用`enum`关键字，隐含了所创建的类型都是`java.lang.Enum`类的子类（`java.lang.Enum`是一个抽象类）。枚举类型符合通用模式`Class Enum<E extends Enum<E>>`，而`E`表示枚举类型的名称。枚举类型的每一个值都将映射到`protected Enum(String name, int ordinal)`构造函数中。在这里每个值的名称都被转换成一个字符串，并且序数设置表示了此设置被创建的顺序。

枚举类的定义格式如下：

```java
enum 类名 {

//枚举值

}
```

### 要点

- 需要的数据不能是任意的，而必须是一定范围内的值
- 枚举类也是一个特殊的类，构造方法默认的修饰符是`private`的
- 枚举值默认的修饰符是`public static final`，必须要位于枚举类的第一个语句
- 枚举类可以定义自己的成员变量、成员函数和带参构造方法，自定义带参构造方法时，枚举值需要传参
- 枚举类可以存在抽象的方法，但是枚举值必须要实现抽象的方法
- 可以使用`==`来比较枚举实例

### 常用方法

枚举中的一些常用方法如下：

- `int ordinal()`：返回枚举常量的序数（它在枚举声明中的位置，其中初始常量序数为零）。
- `String name()`：返回此枚举常量的名称。
- `String toString()`：返回覆盖枚举常量的`toString()`方法的值。
- `int compareTo(E o)`：比较此枚举与指定对象的顺序。
- `Class<E> getDeclaringClass()`：返回与此枚举常量的枚举类型相对应的 Class 对象。
- `static <T extends Enum<T>> T valueOf(Class<T> enumType, String name)`：返回指定名称的枚举常量指定的enumtype的方法。如：`ColorEnum color = ColorEnum.valueOf("RED");`。

## 主要应用

### 表达常量

在`Java 5`之前，定义常量的最佳方式是在`final`修饰的常量类中定义：`public static fianl...`修饰的属性，且须将构造方法设为`private`。代码示例如下：

```java
public final class ColorConst {

    public static final int RED = 1;
    public static final int GREEN = 2;
    public static final int BLUE = 3;

    private ColorConst() {}

}
```

但，**不建议在接口中定义常量**。在`《Effective Java》`一书中提到过：

> **The constant interface pattern is a poor use of interfaces**. That a class uses some constants internally is an implementation detail. Implementing a constant interface causes this implementation detail to leak into the class's exported API. It is of no consequence to the users of a class that the class implements a constant interface. In fact, it may even confuse them. Worse, it represents a commitment: if in a future release the class is modified so that it no longer needs to use the constants, it still must implement the interface to ensure binary compatibility. If a nonfinal class implements a constant interface, all of its subclasses will have their namespaces polluted by the constants in the interface.There are several constant interfaces in the java platform libraries, such as java.io.ObjectStreamConstants. These interfaces should be regarded as anomalies and should not be emulated.

大意是：**如果某个实现了常量接口的类被修改不再需要常量了，也会因为序列化兼容原因不得不保持该实现，而且非`final`类实现常量接口会导致所有子类被污染**。

现在好了，有了枚举，可以把相关的常量分组到一个枚举类型里，而且枚举提供了比常量更多的方法。

```java
public enum ColorEnum {

    RED, GREEN, BLUE

}
```

> **注意**：枚举类的名称一般以`Enum`结尾，比如`ColorEnum`等。如果你写个枚举类，取名为`Color`，那么没人能快速知道它是一个枚举类。

### 遍历

Java 中使用`values()`方法将枚举所有元素item转换成一个数组。这样就可以通过`foreach`语法来遍历枚举中的所有元素了。

```java
for (ColorEnum color: ColorEnum.values()) {
    log.info("ordinal:{}, name:{}", color.ordinal(), color.name());
}
```

输出结果；

```bash
ordinal:0, name:RED
ordinal:1, name:GREEN
ordinal:2, name:BLUE
```

### switch

在`JDK7`之前，String字符串是不支持通过`switch`语法来筛选数据，但是 Java 为枚举提供了`switch`语法的支持。使用示例如下：

```java
// 客户端传来的枚举item
ColorEnum color = ColorEnum.GREEN;

switch (color) {
    case RED: log.info("进入了 RED 的分支");
        break;
    case GREEN: log.info("进入了 GREEN 的分支");
        break;
    case BLUE: log.info("进入了 BLUE 的分支");
        break;
    default: log.info("进入了 default 的分支");
}
```

输出结果：

```bash
进入了 GREEN 的分支
```

> **注意**：`switch`后已经指定了枚举的类型，`case`后无须再使用全名`ColorEnum`。

### 自定义属性和方法

Java枚举中允许定义属性和方法，但必须在枚举实例序列的最后一个分号后再添加。Java 要求必须先定义枚举实例在前面，使用示例如下：

```java
/**
 * 关于颜色的枚举.
 * @author blinkfox on 2017/9/17.
 */
public enum ColorEnum {

    RED(1, "红色"), GREEN(2, "绿色"), BLUE(3, "蓝色");

    /** 颜色的code. */
    private int code;

    /** 颜色的名称. */
    private String name;

    /**
     * 枚举的构造方法默认且只能是private的.
     * @param code 代码值
     * @param name 名称
     */
    ColorEnum(int code, String name) {
        this.code = code;
        this.name = name;
    }

    /**
     * 根据颜色的code值获取到对应的名称.
     * @param code 颜色code
     * @return 颜色名称
     */
    public static String getNameByCode(int code) {
        for (ColorEnum color: ColorEnum.values()) {
            if (color.code == code) {
                return color.name;
            }
        }
        return null;
    }

    /**
     * 覆盖的toString方法.
     * @return 字符串
     */
    @Override
    public String toString() {
        return this.code + ":" + this.name;
    }

    /* getter方法. */

    public int getCode() {
        return code;
    }

    public String getName() {
        return name;
    }

}
```

> **注意**：枚举的构造方法默认且只能是`private`的。

### 使用枚举来表达多态

所有的枚举都继承自`java.lang.Enum`类。由于 Java 不支持多继承，所以枚举不能再继承其他类。但枚举类中可以定义抽象方法，也可以实现一个或者多个接口。由于每一个枚举值会呈现出不同的行为方式，则须要让每个枚举值分别实现方法。

```java
/**
 * 关于颜色的枚举.
 * @author blinkfox on 2017/9/17.
 */
public enum ColorEnum {

    RED(1, "红色") {
        @Override
        public void paint() {
            log.info("使用了'红色'颜料来喷漆");
        }
    },

    GREEN(2, "绿色") {
        @Override
        public void paint() {
            log.info("使用了'绿色'颜料来喷漆");
        }
    },

    BLUE(3, "蓝色") {
        @Override
        public void paint() {
            log.info("使用了'蓝色'颜料来喷漆");
        }
    };

    private static final Logger log = LoggerFactory.getLogger(ColorEnum.class);

    /** 颜色的code. */
    private int code;

    /** 颜色的名称. */
    private String name;

    /**
     * 枚举的构造方法默认且只能是private的.
     * @param code 代码值
     * @param name 名称
     */
    ColorEnum(int code, String name) {
        this.code = code;
        this.name = name;
    }

    /**
     * 使用不同的颜色来喷漆的抽象方法.
     */
    public abstract void paint();

    /**
     * 根据颜色的code值获取到对应的名称.
     * @param code 颜色code
     * @return 颜色名称
     */
    public static String getNameByCode(int code) {
        for (ColorEnum color: ColorEnum.values()) {
            if (color.code == code) {
                return color.name;
            }
        }
        return null;
    }

    /* getter方法. */

    public int getCode() {
        return code;
    }

    public String getName() {
        return name;
    }

    /**
     * 覆盖的toString方法.
     * @return 字符串
     */
    @Override
    public String toString() {
        return this.code + ":" + this.name;
    }

}
```

### 枚举集合的使用

Java 中提供了两个方便操作`enum`的集合类：`java.util.EnumSet`和`java.util.EnumMap`。`EnumSet`保证集合中的元素不重复；`EnumMap`中的`key`是`enum`类型且不能为`null`，而`value`则可以是任意类型。`EnumSet`和`EnumMap`内部以数组来实现，性能更好。

以下是`EnumMap`的使用示例：

```java
EnumMap<ColorEnum, String> colorEnumMap = new EnumMap<ColorEnum, String>(ColorEnum.class);
colorEnumMap.put(ColorEnum.RED, "这是EnumMap中的'RED'");
colorEnumMap.put(ColorEnum.GREEN, "这是EnumMap中的'GREEN'");
colorEnumMap.put(ColorEnum.BLUE, "这是EnumMap中的'BLUE'");

log.info("{}", colorEnumMap);
```

输出结果：

```bash
{1:红色=这是EnumMap中的'RED', 2:绿色=这是EnumMap中的'GREEN', 3:蓝色=这是EnumMap中的'BLUE'}
```

### 枚举单例

在`《Effective Java》`一书中强烈推荐**使用枚举来实现单例模式**，同时枚举单例代码也最为简单：

```java
public enum ColorEnumSingleton {

    INSTANCE;

    public static void doSomething(){
        // do something
    }

}
```

使用枚举单例有以下好处：

- 自由序列化
- 保证只有一个实例（即使使用反射机制也无法多次实例化一个枚举量）
- 线程安全

> **注意**：枚举单例是**饿汉**式的。

### 枚举策略

在使用 Java 的枚举时往往会结合`Switch`来进行判断以实现不同值的处理，但是我们知道多用`switch`不是一种很好的代码风格，不利用维护和适应变化，因为这不符合**开闭原则**。为此一种方法是用**策略模式**来重构原有的枚举实现。在`《Effective Java》`一书中提出了一种**枚举策略模式**很好的解决了这个问题。

具体使用方法和前面所讲的**使用枚举来表达多态**一节中的示例一样，这里就不再举例说明了。

## 总结

- 枚举类也是一个特殊的类，构造方法默认的修饰符是`private`（不管写不写）的，它们都可以定义一些属性和方法，但是不能使用`extends`关键字继承其他类，因为`enum`已经继承了`java.lang.Enum`（java是单一继承）。
- 枚举类中可以定义抽象方法，也可以实现一个或者多个接口。
- 使用枚举大大加强了程序的可读性、易用性和可维护性，并且可在此基础之上进行了扩展，使之可以像类一样去使用，更是为 Java 对离散量的表示上升了一个台阶。
- 枚举最大的缺点是：**相对于普通的常量会占用更多的内存**。所以，我还是不建议大面积的使用枚举来替代整形常量。但是如果这些常量还有关联属性或者行为等，那么强烈推荐使用枚举类型。**使用枚举类型的性能几乎是使用静态类的16倍**。
- 枚举类型对象之间的值比较，可以使用`==`直接来比较值是否相等的，不是必须使用`equals`方法。
- 推荐使用**枚举单例**来实现单例模式，可以使用**枚举策略**来简化策略模式。