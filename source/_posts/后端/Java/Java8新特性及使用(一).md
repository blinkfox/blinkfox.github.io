---
title: Java8新特性及使用(一)
date: 2018-11-13 22:30:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/11/13/java8-1.jpg
categories: 后端
tags:
  - Java
---

## 新特性列表

以下是Java8中的引入的部分新特性。关于Java8新特性更详细的介绍可参考[这里](http://www.oracle.com/technetwork/java/javase/8-whats-new-2157071.html)。

- 接口默认方法和静态方法
- Lambda 表达式
- 函数式接口
- 方法引用
- Stream
- Optional
- Date/Time API
- 重复注解
- 扩展注解的支持
- Base64
- JavaFX
- 其它
  - JDBC4.2规范
  - 更好的类型推测机制
  - HashMap性能提升
  - IO/NIO 的改进
  - JavaScript引擎Nashorn
  - 并发（Concurrency）
  - 类依赖分析器jdeps
  - JVM的PermGen空间被移除

## 一、接口默认方法和静态方法

Java 8用默认方法与静态方法这两个新概念来扩展接口的声明。与传统的接口又有些不一样，它允许在已有的接口中添加新方法，而同时又保持了与旧版本代码的兼容性。

### 1. 接口默认方法

默认方法与抽象方法不同之处在于抽象方法必须要求实现，但是默认方法则没有这个要求。相反，每个接口都必须提供一个所谓的默认实现，这样所有的接口实现者将会默认继承它（如果有必要的话，可以覆盖这个默认实现）。让我们看看下面的例子：

```java
private interface Defaulable {
    // Interfaces now allow default methods, the implementer may or
    // may not implement (override) them.
    default String notRequired() {
        return "Default implementation";
    }
}

private static class DefaultableImpl implements Defaulable {
}

private static class OverridableImpl implements Defaulable {
    @Override
    public String notRequired() {
        return "Overridden implementation";
    }
}
```

`Defaulable`接口用关键字`default`声明了一个默认方法`notRequired()`，`Defaulable`接口的实现者之一`DefaultableImpl`实现了这个接口，并且让默认方法保持原样。`Defaulable`接口的另一个实现者`OverridableImpl`用自己的方法覆盖了默认方法。

#### (1). 多重继承的冲突说明

由于同一个方法可以从不同的接口引入，自然而然的会有冲突的现象，规则如下：

- 一个声明在类里面的方法优先于任何默认方法
- 优先选取最具体的实现

```java
public interface A {

    default void hello() {
        System.out.println("Hello A");
    }

}
```

```java
public interface B extends A {

    default void hello() {
        System.out.println("Hello B");
    }

}
```

```java
public class C implements A, B {

    public static void main(String[] args) {
        new C().hello(); // 输出 Hello B
    }

}
```

#### (2). 优缺点

- **优点**: 可以在不破坏代码的前提下扩展原有库的功能。它通过一个很优雅的方式使得接口变得更智能，同时还避免了代码冗余，并且扩展类库。
- **缺点**: 使得**接口作为协议，类作为具体实现**的界限开始变得有点模糊。

#### (3). 接口默认方法不能重载Object类的任何方法

**接口不能提供对Object类的任何方法的默认实现**。简单地讲，每一个java类都是Object的子类，也都继承了它类中的`equals()`/`hashCode()`/`toString()`方法，那么在类的接口上包含这些默认方法是没有意义的，它们也从来不会被编译。

在JVM中，默认方法的实现是非常高效的，并且通过字节码指令为方法调用提供了支持。默认方法允许继续使用现有的Java接口，而同时能够保障正常的编译过程。这方面好的例子是大量的方法被添加到`java.util.Collection`接口中去：`stream()`，`parallelStream()`，`forEach()`，`removeIf()`等。尽管默认方法非常强大，但是在使用默认方法时我们需要小心注意一个地方：在声明一个默认方法前，请仔细思考是不是真的有必要使用默认方法。

### 2. 接口静态方法

Java 8带来的另一个有趣的特性是接口可以声明（并且可以提供实现）静态方法。在接口中定义静态方法，使用`static`关键字，例如：

```java
public interface StaticInterface {

    static void method() {
        System.out.println("这是Java8接口中的静态方法!");
    }

}
```

下面的一小段代码是上面静态方法的使用。

```java
public class Main {

    public static void main(String[] args) {
        StaticInterface.method(); // 输出 这是Java8接口中的静态方法!
    }

}
```

Java支持一个实现类可以实现多个接口，如果多个接口中存在同样的`static`方法会怎么样呢？如果有两个接口中的静态方法一模一样，并且一个实现类同时实现了这两个接口，此时并不会产生错误，因为Java8中只能通过接口类调用接口中的静态方法，所以对编译器来说是可以区分的。

二、Lambda 表达式

`Lambda`表达式（也称为闭包）是整个Java 8发行版中最受期待的在Java语言层面上的改变，Lambda允许把函数作为一个方法的参数（即：**行为参数化**，函数作为参数传递进方法中）。

一个`Lambda`可以由用逗号分隔的参数列表、`–>`符号与函数体三部分表示。

首先看看在老版本的Java中是如何排列字符串的：

```java
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");
Collections.sort(names, new Comparator<String>() {

    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }

});
```

只需要给静态方法`Collections.sort`传入一个List对象以及一个比较器来按指定顺序排列。通常做法都是创建一个匿名的比较器对象然后将其传递给sort方法。
在Java 8 中你就没必要使用这种传统的匿名对象的方式了，Java 8提供了更简洁的语法，lambda表达式：

```java
Collections.sort(names, (String a, String b) -> {
    return b.compareTo(a);
});
```

看到了吧，代码变得更段且更具有可读性，但是实际上还可以写得更短：

```java
Collections.sort(names, (String a, String b) -> b.compareTo(a));
```

对于函数体只有一行代码的，你可以去掉大括号`{}`以及`return`关键字，但是你还可以写得更短点：

```java
Collections.sort(names, (a, b) -> b.compareTo(a));
```

Java编译器可以自动推导出参数类型，所以你可以不用再写一次类型。

## 三、函数式接口

`Lambda`表达式是如何在Java的类型系统中表示的呢？每一个Lambda表达式都对应一个类型，通常是接口类型。而**函数式接口**是指仅仅只包含一个抽象方法的接口，每一个该类型的Lambda表达式都会被匹配到这个抽象方法。因为**默认方法**不算抽象方法，所以你也可以给你的函数式接口添加默认方法。

我们可以将Lambda表达式当作任意只包含一个抽象方法的接口类型，确保你的接口一定达到这个要求，你只需要给你的接口添加`@FunctionalInterface`注解，编译器如果发现你标注了这个注解的接口有多于一个抽象方法的时候会报错的。

示例如下：

```java
@FunctionalInterface
interface Converter<F, T> {
    T convert(F from);
}

Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
Integer converted = converter.convert("123");
System.out.println(converted); // 123
```

> **注**：如果`@FunctionalInterface`如果没有指定，上面的代码也是对的。

Java8 API包含了很多内建的函数式接口，在老Java中常用到的比如`Comparator`或者`Runnable`接口，这些接口都增加了`@FunctionalInterface`注解以便能用在`Lambda`上。

Java8 API同样还提供了很多全新的函数式接口来让工作更加方便，有一些接口是来自Google Guava库里的，即便你对这些很熟悉了，还是有必要看看这些是如何扩展到lambda上使用的。

### 1. Comparator (比较器接口)

`Comparator`是老Java中的经典接口， Java 8在此之上添加了多种默认方法。源代码及使用示例如下:

```java
@FunctionalInterface
public interface Comparator<T> {

    int compare(T o1, T o2);

}
```

```java
Comparator<Person> comparator = (p1, p2) -> p1.firstName.compareTo(p2.firstName);
Person p1 = new Person("John", "Doe");
Person p2 = new Person("Alice", "Wonderland");
comparator.compare(p1, p2);             // > 0
comparator.reversed().compare(p1, p2);  // < 0
```

### 2. Consumer (消费型接口)

`Consumer`接口表示执行在单个参数上的操作。源代码及使用示例如下:

```java
@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);

}
```

```java
Consumer<Person> greeter = (p) -> System.out.println("Hello, " + p.firstName);
greeter.accept(new Person("Luke", "Skywalker"));
```

#### 更多的Consumer接口

- `BiConsumer：void accept(T t, U u);`: 接受两个参数的二元函数
- `DoubleConsumer：void accept(double value);`: 接受一个double参数的一元函数
- `IntConsumer：void accept(int value);`: 接受一个int参数的一元函数
- `LongConsumer：void accept(long value);`: 接受一个long参数的一元函数
- `ObjDoubleConsumer：void accept(T t, double value);`: 接受一个泛型参数一个double参数的二元函数
- `ObjIntConsumer：void accept(T t, int value);`: 接受一个泛型参数一个int参数的二元函数
- `ObjLongConsumer：void accept(T t, long value);`: 接受一个泛型参数一个long参数的二元函数

### 3. Supplier (供应型接口)

`Supplier`接口是不需要参数并返回一个任意范型的值。其简洁的声明，会让人以为不是函数。这个抽象方法的声明，同Consumer相反，是一个只声明了返回值，不需要参数的函数。也就是说Supplier其实表达的不是从一个参数空间到结果空间的映射能力，而是表达一种生成能力，因为我们常见的场景中不止是要consume（Consumer）或者是简单的map（Function），还包括了new这个动作。而Supplier就表达了这种能力。源代码及使用示例如下:

```java
@FunctionalInterface
public interface Supplier<T> {

    T get();
}
```

```java
Supplier<Person> personSupplier = Person::new;
personSupplier.get();   // new Person
```

#### 更多Supplier接口

- `BooleanSupplier：boolean getAsBoolean();`: 返回boolean的无参函数
- `DoubleSupplier：double getAsDouble();`: 返回double的无参函数
- `IntSupplier：int getAsInt();`: 返回int的无参函数
- `LongSupplier：long getAsLong();`: 返回long的无参函数

### 4. Predicate (断言型接口)

`Predicate`接口只有一个参数，返回`boolean`类型。该接口包含多种默认方法来将`Predicate`组合成其他复杂的逻辑（比如：**与**，**或**，**非**）。`Stream`的`filter`方法就是接受`Predicate`作为入参的。这个具体在后面使用`Stream`的时候再分析深入。源代码及使用示例如下:

```java
@FunctionalInterface
public interface Predicate<T> {

    boolean test(T t);

}
```

```java
Predicate<String> predicate = (s) -> s.length() > 0;
predicate.test("foo");            // true
predicate.negate().test("foo");     // false
Predicate<Boolean> nonNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;
Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
```

#### 更多的Predicate接口

- `BiPredicate：boolean test(T t, U u);`: 接受两个参数的二元断言函数
- `DoublePredicate：boolean test(double value);`: 入参为double的断言函数
- `IntPredicate：boolean test(int value);`: 入参为int的断言函数
- `LongPredicate：boolean test(long value);`: 入参为long的断言函数

### 5. Function (功能型接口)

`Function`接口有一个参数并且返回一个结果，并附带了一些可以和其他函数组合的默认方法（`compose`, `andThen`）。源代码及使用示例如下:

```java
@FunctionalInterface
public interface Function<T, R> {

    R apply(T t);

}
```

```java
Function<String, Integer> toInteger = Integer::valueOf;
Function<String, String> backToString = toInteger.andThen(String::valueOf);
backToString.apply("123");     // "123"
```

#### 更多的Function接口

- `BiFunction ：R apply(T t, U u);`: 接受两个参数，返回一个值，代表一个二元函数；
- `DoubleFunction ：R apply(double value);`: 只处理double类型的一元函数；
- `IntFunction ：R apply(int value);`: 只处理int参数的一元函数；
- `LongFunction ：R apply(long value);`: 只处理long参数的一元函数；
- `ToDoubleFunction：double applyAsDouble(T value);`: 返回double的一元函数；
- `ToDoubleBiFunction：double applyAsDouble(T t, U u);`: 返回double的二元函数；
- `ToIntFunction：int applyAsInt(T value);`: 返回int的一元函数；
- `ToIntBiFunction：int applyAsInt(T t, U u);`: 返回int的二元函数；
- `ToLongFunction：long applyAsLong(T value);`: 返回long的一元函数；
- `ToLongBiFunction：long applyAsLong(T t, U u);`: 返回long的二元函数；
- `DoubleToIntFunction：int applyAsInt(double value);`: 接受double返回int的一元函数；
- `DoubleToLongFunction：long applyAsLong(double value);`: 接受double返回long的一元函数；
- `IntToDoubleFunction：double applyAsDouble(int value);`: 接受int返回double的一元函数；
- `IntToLongFunction：long applyAsLong(int value);`: 接受int返回long的一元函数；
- `LongToDoubleFunction：double applyAsDouble(long value);`: 接受long返回double的一元函数；
- `LongToIntFunction：int applyAsInt(long value);`: 接受long返回int的一元函数；

### 6. Operator

`Operator`其实就是`Function`，函数有时候也叫作算子。算子在Java8中接口描述更像是函数的补充，和上面的很多类型映射型函数类似。算子Operator包括：`UnaryOperator`和`BinaryOperator`。分别对应单（一）元算子和二元算子。

算子的接口声明如下：

```java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {

    static <T> UnaryOperator<T> identity() {
        return t -> t;
    }
}
```

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T,T,T> {

    public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
    }

    public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
    }
}
```

`Operator`只需声明一个泛型参数T即可。对应的使用示例如下：

```java
UnaryOperator<Integer> increment = x -> x + 1;
System.out.println("递增:" + increment.apply(2)); // 输出 递增:3

BinaryOperator<Integer> add = (x, y) -> x + y;
System.out.println("相加:" + add.apply(2, 3)); // 输出 相加:5

BinaryOperator<Integer> min = BinaryOperator.minBy((o1, o2) -> o1 - o2);
System.out.println("最小值:" + min.apply(2, 3)); // 输出 最小值:2
```

#### 更多的Operator接口

- `LongUnaryOperator：long applyAsLong(long operand);`: 对long类型做操作的一元算子
- `IntUnaryOperator：int applyAsInt(int operand);`: 对int类型做操作的一元算子
- `DoubleUnaryOperator：double applyAsDouble(double operand);`: 对double类型做操作的一元算子
- `DoubleBinaryOperator：double applyAsDouble(double left, double right);`: 对double类型做操作的二元算子
- `IntBinaryOperator：int applyAsInt(int left, int right);`: 对int类型做操作的二元算子
- `LongBinaryOperator：long applyAsLong(long left, long right);`: 对long类型做操作的二元算子

### 6. 其它函数式接口

- java.lang.Runnable
- java.util.concurrent.Callable
- java.security.PrivilegedAction
- java.io.FileFilter
- java.nio.file.PathMatcher 
- java.lang.reflect.InvocationHandler
- java.beans.PropertyChangeListener
- java.awt.event.ActionListener  
- javax.swing.event.ChangeListener

## 四、方法引用

### 1. 概述

在学习了Lambda表达式之后，我们通常使用Lambda表达式来创建匿名方法。然而，有时候我们仅仅是调用了一个已存在的方法。如下：

```java
Arrays.sort(strArray, (s1, s2) -> s1.compareToIgnoreCase(s2));
```

在Java8中，我们可以直接通过方法引用来简写Lambda表达式中已经存在的方法。

```java
Arrays.sort(strArray, String::compareToIgnoreCase);
```

这种特性就叫做**方法引用**(`Method Reference`)。

**方法引用**是用来直接访问类或者实例的已经存在的方法或者构造方法。方法引用提供了一种引用而不执行方法的方式，它需要由兼容的函数式接口构成的目标类型上下文。计算时，方法引用会创建函数式接口的一个实例。当Lambda表达式中只是执行一个方法调用时，不用Lambda表达式，直接通过方法引用的形式可读性更高一些。方法引用是一种更简洁易懂的Lambda表达式。

> **注意**: 方法引用是一个Lambda表达式，其中方法引用的操作符是双冒号`::`。

### 2. 分类

方法引用的标准形式是：`类名::方法名`。（注意：只需要写方法名，不需要写括号）

有以下四种形式的方法引用：

- 引用静态方法: ContainingClass::staticMethodName
- 引用某个对象的实例方法: containingObject::instanceMethodName
- 引用某个类型的任意对象的实例方法:ContainingType::methodName
- 引用构造方法: ClassName::new

### 3. 示例

使用示例如下：

```java
public class Person {

    String name;

    LocalDate birthday;

    public Person(String name, LocalDate birthday) {
        this.name = name;
        this.birthday = birthday;
    }

    public LocalDate getBirthday() {
        return birthday;
    }

    public static int compareByAge(Person a, Person b) {
        return a.birthday.compareTo(b.birthday);
    }

    @Override
    public String toString() {
        return this.name;
    }
}
```

```java
public class MethodReferenceTest {

    @Test
    public static void main() {
        Person[] pArr = new Person[] {
            new Person("003", LocalDate.of(2016,9,1)),
            new Person("001", LocalDate.of(2016,2,1)),
            new Person("002", LocalDate.of(2016,3,1)),
            new Person("004", LocalDate.of(2016,12,1))
        };

        // 使用匿名类
        Arrays.sort(pArr, new Comparator<Person>() {
            @Override
            public int compare(Person a, Person b) {
                return a.getBirthday().compareTo(b.getBirthday());
            }
        });

        //使用lambda表达式
        Arrays.sort(pArr, (Person a, Person b) -> {
            return a.getBirthday().compareTo(b.getBirthday());
        });

        //使用方法引用，引用的是类的静态方法
        Arrays.sort(pArr, Person::compareByAge);
    }

}
```

## 五、Stream

Java8添加的`Stream API(java.util.stream)`把真正的函数式编程风格引入到Java中。这是目前为止对Java类库最好的补充，因为`Stream API`可以极大提供Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。

流可以是无限的、有状态的，可以是顺序的，也可以是并行的。在使用流的时候，你首先需要从一些来源中获取一个流，执行一个或者多个中间操作，然后执行一个最终操作。中间操作包括`filter`、`map`、`flatMap`、`peel`、`distinct`、`sorted`、`limit`和`substream`。终止操作包括`forEach`、`toArray`、`reduce`、`collect`、`min`、`max`、`count`、`anyMatch`、`allMatch`、`noneMatch`、`findFirst`和`findAny`。 `java.util.stream.Collectors`是一个非常有用的实用类。该类实现了很多归约操作，例如将流转换成集合和聚合元素。

### 1. 一些重要方法说明

- `stream`: 返回数据流，集合作为其源
- `parallelStream`: 返回并行数据流， 集合作为其源
- `filter`: 方法用于过滤出满足条件的元素
- `map`: 方法用于映射每个元素对应的结果
- `forEach`: 方法遍历该流中的每个元素
- `limit`: 方法用于减少流的大小
- `sorted`: 方法用来对流中的元素进行排序
- `anyMatch`: 是否存在任意一个元素满足条件（返回布尔值）
- `allMatch`: 是否所有元素都满足条件（返回布尔值）
- `noneMatch`: 是否所有元素都不满足条件（返回布尔值）
- `collect`: 方法是终端操作，这是通常出现在管道传输操作结束标记流的结束

### 2. 一些使用示例

#### (1). Filter 过滤

```java
stringCollection
    .stream()
    .filter((s) -> s.startsWith("a"))
    .forEach(System.out::println);
```

### (2). Sort 排序

```java
stringCollection
    .stream()
    .sorted()
    .filter((s) -> s.startsWith("a"))
    .forEach(System.out::println);
```

### (3). Map 映射

```java
stringCollection
    .stream()
    .map(String::toUpperCase)
    .sorted((a, b) -> b.compareTo(a))
    .forEach(System.out::println);
```

### (4). Match 匹配

```java
boolean anyStartsWithA = stringCollection
        .stream()
        .anyMatch((s) -> s.startsWith("a"));
System.out.println(anyStartsWithA);      // true

boolean allStartsWithA = stringCollection
        .stream()
        .allMatch((s) -> s.startsWith("a"));
System.out.println(allStartsWithA);      // false

boolean noneStartsWithZ = stringCollection
        .stream()
        .noneMatch((s) -> s.startsWith("z"));
System.out.println(noneStartsWithZ);      // true
```

### (5). Count 计数

```java
long startsWithB = stringCollection
        .stream()
        .filter((s) -> s.startsWith("b"))
        .count();
System.out.println(startsWithB);    // 3
```

### (6). Reduce 规约

这是一个最终操作，允许通过指定的函数来将`stream`中的多个元素规约为一个元素，规越后的结果是通过`Optional`接口表示的。代码如下:

```java
Optional<String> reduced = stringCollection
        .stream()
        .sorted()
        .reduce((s1, s2) -> s1 + "#" + s2);
reduced.ifPresent(System.out::println);
```

## 六、Optional

到目前为止，臭名昭著的空指针异常是导致Java应用程序失败的最常见原因。以前，为了解决空指针异常，Google公司著名的`Guava`项目引入了`Optional`类，Guava通过使用检查空值的方式来防止代码污染，它鼓励程序员写更干净的代码。受到Google Guava的启发，`Optional`类已经成为Java 8类库的一部分。

`Optional`实际上是个容器：它可以保存类型T的值，或者仅仅保存null。`Optional`提供很多有用的方法，这样我们就不用显式进行空值检测。

我们下面用两个小例子来演示如何使用Optional类：一个允许为空值，一个不允许为空值。

```java
Optional<String> fullName = Optional.ofNullable(null);
System.out.println("Full Name is set? " + fullName.isPresent());
System.out.println("Full Name: " + fullName.orElseGet(() -> "[none]"));
System.out.println(fullName.map(s -> "Hey " + s + "!").orElse("Hey Stranger!"));
```

如果`Optional`类的实例为非空值的话，`isPresent()`返回`true`，否从返回`false`。为了防止Optional为空值，`orElseGet()`方法通过回调函数来产生一个默认值。`map()`函数对当前`Optional`的值进行转化，然后返回一个新的`Optional`实例。`orElse()`方法和`orElseGet()`方法类似，但是`orElse`接受一个默认值而不是一个回调函数。下面是这个程序的输出：

```bash
Full Name is set? false
Full Name: [none]
Hey Stranger!
```

让我们来看看另一个例子：

```java
Optional<String> firstName = Optional.of("Tom");
System.out.println("First Name is set? " + firstName.isPresent());
System.out.println("First Name: " + firstName.orElseGet(() -> "[none]"));
System.out.println(firstName.map(s -> "Hey " + s + "!").orElse("Hey Stranger!"));
System.out.println();
```

下面是程序的输出：

```bash
First Name is set? true
First Name: Tom
Hey Tom!
```

## 七、Date/Time API

Java 8 在包`java.time`下包含了一组全新的时间日期API。新的日期API和开源的`Joda-Time`库差不多，但又不完全一样，下面的例子展示了这组新API里最重要的一些部分：

### 1. Clock 时钟

`Clock`类提供了访问当前日期和时间的方法，Clock是时区敏感的，可以用来取代`System.currentTimeMillis()`来获取当前的微秒数。某一个特定的时间点也可以使用`Instant`类来表示，`Instant`类也可以用来创建老的`java.util.Date`对象。代码如下:

```java
Clock clock = Clock.systemDefaultZone();
long millis = clock.millis();
Instant instant = clock.instant();
Date legacyDate = Date.from(instant);   // legacy java.util.Date
```

### 2. Timezones 时区

在新API中时区使用`ZoneId`来表示。时区可以很方便的使用静态方法`of`来获取到。时区定义了到UTS时间的时间差，在`Instant`时间点对象到本地日期对象之间转换的时候是极其重要的。代码如下:

```java
System.out.println(ZoneId.getAvailableZoneIds());
// prints all available timezone ids
ZoneId zone1 = ZoneId.of("Europe/Berlin");
ZoneId zone2 = ZoneId.of("Brazil/East");
System.out.println(zone1.getRules());
System.out.println(zone2.getRules());
// ZoneRules[currentStandardOffset=+01:00]
// ZoneRules[currentStandardOffset=-03:00]
```

### 3. LocalTime 本地时间

`LocalTime`定义了一个没有时区信息的时间，例如 晚上10点，或者 17:30:15。下面的例子使用前面代码创建的时区创建了两个本地时间。之后比较时间并以小时和分钟为单位计算两个时间的时间差。代码如下:

```java
LocalTime now1 = LocalTime.now(zone1);
LocalTime now2 = LocalTime.now(zone2);
System.out.println(now1.isBefore(now2));  // false
long hoursBetween = ChronoUnit.HOURS.between(now1, now2);
long minutesBetween = ChronoUnit.MINUTES.between(now1, now2);
System.out.println(hoursBetween);       // -3
System.out.println(minutesBetween);     // -239
```

`LocalTime`提供了多种工厂方法来简化对象的创建，包括解析时间字符串。代码如下:

```java
LocalTime late = LocalTime.of(23, 59, 59);
System.out.println(late);       // 23:59:59
DateTimeFormatter germanFormatter = DateTimeFormatter
        .ofLocalizedTime(FormatStyle.SHORT)
        .withLocale(Locale.GERMAN);
LocalTime leetTime = LocalTime.parse("13:37", germanFormatter);
System.out.println(leetTime);   // 13:37
```

### 4. LocalDate 本地日期

`LocalDate`表示了一个确切的日期，比如`2014-03-11`。该对象值是不可变的，用起来和`LocalTime`基本一致。下面的例子展示了如何给`Date`对象加减天/月/年。另外要注意的是这些对象是不可变的，操作返回的总是一个新实例。代码如下:

```java
LocalDate today = LocalDate.now();
LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
LocalDate yesterday = tomorrow.minusDays(2);
LocalDate independenceDay = LocalDate.of(2014, Month.JULY, 4);
DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();

System.out.println(dayOfWeek);    // FRIDAY
```

从字符串解析一个LocalDate类型和解析LocalTime一样简单。代码如下:

```java
DateTimeFormatter germanFormatter = DateTimeFormatter
        .ofLocalizedDate(FormatStyle.MEDIUM)
        .withLocale(Locale.GERMAN);
LocalDate xmas = LocalDate.parse("24.12.2014", germanFormatter);
System.out.println(xmas);   // 2014-12-24
```

### 5. LocalDateTime 本地日期时间

`LocalDateTime`同时表示了时间和日期，相当于前两节内容合并到一个对象上了。`LocalDateTime`和`LocalTime`还有`LocalDate`一样，都是不可变的。`LocalDateTime`提供了一些能访问具体字段的方法。代码如下:

```java
LocalDateTime sylvester = LocalDateTime.of(2014, Month.DECEMBER, 31, 23, 59, 59);
DayOfWeek dayOfWeek = sylvester.getDayOfWeek();
System.out.println(dayOfWeek);      // WEDNESDAY
Month month = sylvester.getMonth();
System.out.println(month);          // DECEMBER
long minuteOfDay = sylvester.getLong(ChronoField.MINUTE_OF_DAY);
System.out.println(minuteOfDay);    // 1439
```

只要附加上时区信息，就可以将其转换为一个时间点`Instant`对象，`Instant`时间点对象可以很容易的转换为老式的`java.util.Date`。代码如下:

```java
Instant instant = sylvester
        .atZone(ZoneId.systemDefault())
        .toInstant();
Date legacyDate = Date.from(instant);
System.out.println(legacyDate);     // Wed Dec 31 23:59:59 CET 2014
```

格式化`LocalDateTime`和格式化时间和日期一样的，除了使用预定义好的格式外，我们也可以自己定义格式。代码如下:

```java
DateTimeFormatter formatter =
    DateTimeFormatter
        .ofPattern("MMM dd, yyyy - HH:mm");
LocalDateTime parsed = LocalDateTime.parse("Nov 03, 2014 - 07:13", formatter);
String string = formatter.format(parsed);
System.out.println(string);     // Nov 03, 2014 - 07:13
```

和`java.text.NumberFormat`不一样的是新版的`DateTimeFormatter`是不可变的，所以它是线程安全的。

关于Java8中日期API更多的使用示例可以参考[Java 8中关于日期和时间API的20个使用示例](http://blinkfox.com/java-8zhong-guan-yu-ri-qi-he-shi-jian-apide-20ge-shi-yong-shi-li/)。

## 八、重复注解

自从Java 5引入了注解机制，这一特性就变得非常流行并且广为使用。然而，使用注解的一个限制是相同的注解在同一位置只能声明一次，不能声明多次。Java 8打破了这条规则，引入了重复注解机制，这样相同的注解可以在同一地方声明多次。

重复注解机制本身必须用`@Repeatable`注解。事实上，这并不是语言层面上的改变，更多的是编译器的技巧，底层的原理保持不变。让我们看一个快速入门的例子：

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Repeatable;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

public class RepeatingAnnotations {

    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Filters {
        Filter[] value();
    }

    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    @Repeatable(Filters.class)
    public @interface Filter {
        String value();
    };

    @Filter("filter1")
    @Filter("filter2")
    public interface Filterable {
    }

    public static void main(String[] args) {
        for(Filter filter: Filterable.class.getAnnotationsByType(Filter.class)) {
            System.out.println(filter.value());
        }
    }

}
```

正如我们看到的，这里有个使用`@Repeatable(Filters.class)`注解的注解类`Filter`，`Filters`仅仅是`Filter`注解的数组，但Java编译器并不想让程序员意识到`Filters`的存在。这样，接口`Filterable`就拥有了两次`Filter`（并没有提到`Filter`）注解。

同时，反射相关的API提供了新的函数`getAnnotationsByType()`来返回重复注解的类型（请注意`Filterable.class.getAnnotation(Filters.class`)`经编译器处理后将会返回Filters的实例）。