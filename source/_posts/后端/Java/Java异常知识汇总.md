---
title: Java异常知识汇总
date: 2018-10-28 00:30:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/10/28/exception.jpg
categories: 后端
tags:
  - Java
---

## 前言

### 为什么要使用异常

在我们的程序中，任何时候任何地方因为任何原因都有可能会出现异常，在没有异常机制的时候我们是这样处理的：通过函数的返回值来判断是否发生了异常（这个返回值通常是已经约定好了的），调用该函数的程序负责检查并且分析返回值。虽然可以解决异常问题，但是这样做存在几个缺陷：

- 容易混淆。如果约定返回值为 -1 时表示出现异常，那么当程序最后的计算结果真的为 -1 呢？
- 代码可读性差。将异常处理代码和程序代码混淆在一起将会降低代码的可读性。
- 由调用函数来分析异常，这要求程序员对库函数有很深的了解。

> 在面向对象编程中提供的异常处理机制是提供代码健壮的强有力的方式。使用异常机制它能够降低错误处理代码的复杂度，如果不使用异常，那么就必须检查特定的错误，并在程序中的许多地方去处理它，而如果使用异常，那就不必在方法调用处进行检查，因为异常机制将保证能够捕获这个错误，并且，只需在一个地方处理错误，即所谓的异常处理程序中。这种方式不仅节约代码，而且把“概述在正常执行过程中做什么事”的代码和“出了问题怎么办”的代码相分离。总之，与以前的错误处理方法相比，异常机制使代码的阅读、编写和调试工作更加井井有条。（摘自《Think in java 》）。

### 基本定义

> 异常情形是指阻止当前方法或者作用域继续执行的问题。——《Think in java》

总的来说异常处理机制就是当程序发生异常时，它强制终止程序运行，记录异常信息并将这些信息反馈给我们，由我们来确定是否处理异常。

## 异常体系

在Java中，所有的事件都能由类描述，Java中的异常就是由`java.lang`包下的异常类来描述的。Java定义了一个异常类的层次结构，其以`Throwable`（万物即可抛）开始，派生出了`Error`和`Exception`，而`Exception`又派生出了`CheckedException`和`RuntimeException`。如下图所示：

![Java异常体系](https://statics.sh1a.qingstor.com/2018/10/28/java-exception.png)

### Throwable

Throwable（可抛出）是异常类的最终父类，它有两个子类，`Error`与`Exception`。

Throwable 中常用方法有：

- `synchronized Throwable getCause()`：此方法返回异常产生的原因，如果不知道原因的话返回`null`。
- `String getMessage()`：方法返回`Throwable`的`String`型信息，当异常通过构造器创建后可用。
- `String getLocalizedMessage()`：此方法通过被重写来得到用本地语言表示的异常信息返回给调用程序。`Throwable`类通常只是用`getMessage()`方法来实现返回异常信息。
- `void printStackTrace()`：该方法打印栈轨迹信息到标准错误流。该方法能接受`PrintStream`和`PrintWriter`作为参数实现重载，这样就能实现打印栈轨迹到文件或流中。
- `String toString()`：方法返回`String`格式的`Throwable`信息，此信息包括`Throwable`的名字和本地化信息。

### Error

Error（错误）：表示程序无法处理的错误，一般与程序员的执行操作无关。理论上这些错误是不允许发生的，如果发生，也不应该试图通过程序去处理，所以 Error 不是`try-catch`的处理对象，而 JVM 一般的处理方式是终止发生错误的线程。Error 类常见子类有`VirtualMachineError`、`StackOverFlowError`、`OutOfMemoryError`等。

在Java运行时内存中，除程序计数器外的虚拟机栈、堆、方法区在请求的内存无法被满足时都会抛出`OutOfMemoryError`；而如果线程请求的栈深度超出虚拟机允许的深度时，就会抛出`StackOverFlowError`。

### Exception

Exception（异常）：出现原因取决于程序，所以程序也理应通过`try-catch`处理。Exception 异常分为两类：`CheckedException`和`RuntimeException`，即**检查异常**与**运行时异常**。

- 检查异常：编译器要求必须处理，否则不能通过编译，使用`try-catch`捕获或者`throws`抛出。常见的检查异常有`IOException`及其子类、`EOFExcption`(文件已结束异常)、`FileNotFoundException`（文件未找到异常）。
- 运行时异常（也叫非检查异常）：编译期不会检查，所以在程序中可不处理，但如果发生，会在运行时抛出。

## 异常处理

### 处理机制

在 Java 应用程序中，异常处理机制为：**抛出异常**、**捕捉异常**。

- **抛出异常**：当一个方法出现错误引发异常时，方法创建异常对象并交付运行时系统，异常对象中包含了异常类型和异常出现时的程序状态等异常信息。运行时系统负责寻找处置异常的代码并执行。
- **捕获异常**：在方法抛出异常之后，运行时系统将转为寻找合适的异常处理器。潜在的异常处理器是异常发生时依次存留在调用栈中的方法的集合。当异常处理器所能处理的异常类型与方法抛出的异常类型相符时，即为合适 的异常处理器。运行时系统从发生异常的方法开始，依次回查调用栈中的方法，直至找到含有合适异常处理器的方法并执行。当运行时系统遍历调用栈而未找到合适的异常处理器，则运行时系统终止。同时，意味着Java程序的终止。

对于运行时异常、错误或可查异常，Java技术所要求的异常处理方式有所不同。

- 对于方法运行中可能出现的Error，当运行方法不欲捕捉时，Java允许该方法不做任何抛出声明。因为，大多数 Error 异常属于永远不能被允许发生的状况，也属于合理的应用程序不该捕捉的异常。
- 对于所有的检查异常，Java规定：**一个方法必须捕捉，或者声明抛出方法之外。也就是说，当一个方法选择不捕捉检查异常时，它必须声明将抛出异常**。
- 对于所有运行时异常，Java规定：**运行时异常将由Java运行时系统自动抛出，允许应用程序忽略运行时异常**。

能够捕捉异常的方法，需要提供相符类型的异常处理器。所捕捉的异常，可能是由于自身语句所引发并抛出的异常，也可能是由某个调用的方法或者Java运行时 系统等抛出的异常。也就是说，**一个方法所能捕捉的异常，一定是Java代码在某处所抛出的异常。简单地说，异常总是先被抛出，后被捕捉的**。

任何Java代码都可以通过 Java 的`throw`语句抛出异常。

从方法中抛出的任何异常都必须使用`throws`子句。

捕捉异常通过`try-catch`语句或者`try-catch-finally`语句实现。

> 总体来说，Java规定：对于检查异常必须捕捉、或者声明抛出。允许忽略非检查的`RuntimeException`和`Error`。

### 处理方式

`try-catch`语句还可以包括第三部分，就是`finally`子句。它表示无论是否出现异常，都应当执行的内容。`try-catch-finally`语句的一般语法形式为：

```java
try {
    // 可能会发生异常的程序代码
} catch (Exception1 e1) {
    // 捕获并处理try抛出的异常类型Type1
} catch (Exception2 e2) {
    // 捕获并处理try抛出的异常类型Type2
} finally {
    // 无论是否发生异常，都将执行的语句块
}
```

Java7及之后的版本可这样使用：

```java
try (MyResource mr = new MyResource()) {
    System.out.println("MyResource created in try-with-resources");
} catch (Exception1 | Exception2 e) {
    // 捕获并统一处理 try 抛出的多种异常类型，不需要finally块
}
```

- `try`块：用于捕获异常。其后可接零个或多个`catch`块，如果没有`catch`块，则必须跟一个`finally`块。
- `catch`块：用于处理`try`捕获到的异常。
- `finally`块：无论是否捕获或处理异常，`finally`块里的语句都会被执行。当在`try`块或`catch`块中遇到`return`语句时，`finally`语句块将在方法返回之前被执行。在以下 4 种特殊情况下，`finally`块不会被执行：
  - 在`finally`语句块中发生了异常
  - 在前面的代码中用了`System.exit()`退出程序
  - 程序所在的线程死亡
  - 关闭`CPU`

### 异常处理语句的语法规则

- 必须在`try`之后添加`catch`或`finally`块。`try`块后可同时接`catch`和`finally`块，但至少有一个块。
- 必须遵循块顺序：若代码同时使用`catch`和`finally`块，则必须将`catch`块放在`try`块之后。
- `catch`块与相应的异常类的类型相关。
- 一个`try`块可能有多个`catch`块。若如此，则执行第一个匹配块。即Java虚拟机会把实际抛出的异常对象依次和各个catch代码块声明的异常类型匹配，如果异常对象为某个异常类型或其子类的实例，就执行这个`catch`代码块，不会再执行其他的`catch`代码块。
- 可嵌套`try-catch-finally`结构。
- 在`try-catch-finally`结构中，可重新抛出异常。
- 除了下列情况，总将执行`finally`做为结束：
  - JVM 过早终止（调用 System.exit(int)）；
  - 在`finally`块中抛出一个未处理的异常；
  - 计算机断电、失火、或遭遇病毒攻击。

## 异常抛出

任何Java代码都可以抛出异常，如：自己编写的代码、来自Java开发环境包中代码，或者Java运行时系统。无论是谁，都可以通过Java的`throw`语句抛出异常。从方法中抛出的任何异常都必须使用`throws`子句。

### throws抛出异常

如果一个方法可能会出现异常，但没有能力处理这种异常，可以在方法声明处用`throws`子句来声明抛出异常。`throws`语句用在方法定义时声明该方法要抛出的异常类型，如果抛出的是`Exception`异常类型，则该方法被声明为抛出所有的异常。多个异常可使用逗号分割。`throws`语句的语法格式为：

```java
methodname throws Exception1, Exception2, ... , ExceptionN {

}
```

方法名后的`throws Exception1, Exception2, ... , ExceptionN`为声明要抛出的异常列表。当方法抛出异常列表的异常时，方法将不对这些类型及其子类类型的异常作处理，而抛向调用该方法的方法，由他去处理。使用`throws`关键字将异常抛给调用者后，如果调用者不想处理该异常，可以继续向上抛出，但最终要有能够处理该异常的调用者。

throws抛出异常的规则：

- 如果是非检查异常（`unchecked exception`），即`Error`、`RuntimeException`或它们的子类，那么可以不使用`throws`关键字来声明要抛出的异常，编译仍能顺利通过，但在运行时会被系统抛出。
- 必须声明方法可抛出的任何检查异常（`checked exception`）。即如果一个方法可能出现受检查异常，要么用`try-catch`语句捕获，要么用`throws`子句声明将它抛出，否则会导致编译错误。
- 仅当抛出了异常，该方法的调用者才必须处理或者重新抛出该异常。当方法的调用者无力处理该异常的时候，应该继续抛出，而不是囫囵吞枣。
- 调用方法必须遵循任何检查异常的处理和声明规则。若覆盖一个方法，则不能声明与覆盖方法不同的异常。声明的任何异常必须是被覆盖方法所声明异常的同类或子类。

### 使用throw抛出异常

`throw`总是出现在函数体中，用来抛出一个`Throwable`类型的异常。程序会在`throw`语句后立即终止，它后面的语句执行不到，然后在包含它的所有`try`块中（可能在上层调用函数中）从里向外寻找含有与其匹配的`catch`子句的`try`块。异常是异常类的实例对象，我们可以创建异常类的实例对象通过`throw`语句抛出。该语句的语法格式为：

```java
throw new ExceptionName();
```

> **注**：如果抛出了检查异常，则还应该在方法头部声明方法可能抛出的异常类型。该方法的调用者也必须检查处理抛出的异常。

### 异常链

在设计模式中有一个设计模式叫做**责任链模式**，该模式是将多个对象链接成一条链，客户端的请求沿着这条链传递直到被接收、处理。同样Java异常机制也提供了这样一条链：**异常链**。

我们知道每遇到一个异常信息，我们都需要进行`try-catch-finally`,一个还好，如果出现多个异常呢？分类处理肯定会比较麻烦，那就一个`Exception`解决所有的异常吧。这样确实是可以，但是这样处理势必会导致后面的维护难度增加。最好的办法就是将这些异常信息封装，然后捕获我们的封装类即可。

我们有两种方式处理异常，一是`throws`抛出交给上级处理，二是`try-catch`做具体处理。但是这个与上面有什么关联呢？`try-catch`的`catch`块我们可以不需要做任何处理，仅仅只用`throw`这个关键字将我们封装异常信息主动抛出来。然后在通过关键字`throws`继续抛出该方法异常。它的上层也可以做这样的处理，以此类推就会产生一条由异常构成的异常链。

**通过使用异常链，我们可以提高代码的可理解性、系统的可维护性和友好性**。

同理，我们有时候在捕获一个异常后抛出另一个异常信息，并且希望将原始的异常信息也保持起来，这个时候也需要使用异常链。

在异常链的使用中，`throw`抛出的是一个新的异常信息，这样势必会导致原有的异常信息丢失，如何保持？在`Throwable`及其子类中的构造器中都可以接受一个`cause`参数，该参数保存了原有的异常信息，通过`getCause()`就可以获取该原始异常信息。使用方式如下：

```java
public class Test {

    public void f() throws MyException{
         try {
             FileReader reader = new FileReader("test.txt");
             Scanner in = new Scanner(reader);
             System.out.println(in.next());
        } catch (FileNotFoundException e) {
            //e 保存异常信息
            throw new MyException("文件没有找到--01", e);
        }
    }

    public void g() throws MyException{
        try {
            f();
        } catch (MyException e) {
            //e 保存异常信息
            throw new MyException("文件没有找到--02", e);
        }
    }

    public static void main(String[] args) {
        Test t = new Test();
        try {
            t.g();
        } catch (MyException e) {
            e.printStackTrace();
        }
    }
}
```

如果在程序中,去掉`e`，也就是：`throw new MyException(“文件没有找到–02″);`那么异常信息就保存不了。

## 自定义异常

Java确实给我们提供了非常多的异常，但是异常体系是不可能预见所有的希望加以报告的错误。所以，Java允许我们自定义异常来表现程序中可能会遇到的特定问题，总之就是一句话：我们不必拘泥于Java中已有的异常类型。

Java自定义异常的使用要经历如下四个步骤：

- 定义一个类继承`Throwable`或其子类。
- 添加构造方法(当然也可以不用添加，使用默认构造方法)。
- 在某个方法类抛出该异常。
- 捕捉该异常。

示例如下：

```java
/**
 *自定义异常 继承Exception类.
 */
public class MyException extends Exception {

    public MyException(){

    }

    public MyException(String message){
        super(message);
    }

}

/**
 * 测试抛出和捕捉异常的类.
 */
public class Test {

    public void display(int i) throws MyException{
        if (i == 0) {
            throw new MyException("该值不能为0.......");
        } else{
            System.out.println( i / 2);
        }
    }

    public static void main(String[] args) {
        Test test = new Test();
        try {
            test.display(0);
            System.out.println("---------------------");
        } catch (MyException e) {
            e.printStackTrace();
        }
    }
}
```

## 最佳实践

- 尽可能的减小`try`块。
- 不要在构造函数中抛出异常。
- 如果使用Java7及以后的版本，一个catch子句中可以捕获多个异常。
- 充分使用`finally`块，保证所有资源都被正确释放；如果使用Java7及以后的版本，那么更推荐使用`try-with-resource`语法。
- `catch`语句应当尽量指定具体的异常类型，而不应该指定涵盖范围太广的`Exception`类。 不要一个`Exception`试图处理所有可能出现的异常。
- 不要忽略异常。既然捕获了异常，就要对它进行适当的处理。不要捕获异常之后又把它丢弃，不予理睬。
- 在异常处理模块中提供适量的错误原因信息，组织错误信息使其易于理解和阅读。
- 减轻`finally`的任务，finally块仅仅用来释放资源是最合适的。不要在`finally`中使用`return`、抛出异常等。
- 为了给调用者提供尽可能多的信息，从而可以更好地避免/处理异常。对异常进行Javadoc文档说明，并且描述抛出异常的场景。
- 不要捕获`Throwable`。`Throwable`是所有异常和错误的父类。如果`catch`了`throwable`，那么不仅仅会捕获所有`Exception`，还会捕获`Error`。而`Error`是表明无法恢复的JVM错误。因此除非绝对肯定能够处理或者被要求处理`Error`，不要捕获`Throwable`。
- 包装异常时要包含原始的异常。包装异常时，一定要把原始的异常设置为`cause`(`Exception`有构造方法可以传入`cause`)。否则，丢失了原始的异常信息会让错误的分析变得困难。

## 常见异常及解释

以下是常见[Java异常](http://rymden.nu/exceptions.html)的**非技术角度**的理解。阅读有风险，理解需谨慎。

### java.lang

- `ArithmeticException`：你正在试图使用电脑解决一个自己解决不了的数学问题，请重新阅读你的算术表达式并再次尝试。
- `ArrayIndexOutOfBoundsException`：请查看[IndexOutOfBoundsException](http://rymden.nu/exceptions.html#IndexOutOfBoundsException)。不同之处在于这个异常越界的元素不止一个。
- `ArrayStoreException`：你已用光了所有数组，需要从数组商店中购买更多的数组。
- `ClassCastException`：你需要呆在自己出生的种姓或阶级。Java 不会允许达利特人表现得像刹帝利或者高贵种族的人假装成为工人阶级。为了保持向前兼容，Java 1.0中把Caste误写为Cast保留到了现在。
- `ClassNotFoundException`：你似乎创造了自己的类。这也是目前 Java 还未实现的种姓制度，但是 Java 明显使用了巴厘岛的种姓制度。也就是说，如果你是一个武士，也就相当于印度种姓制度中的第三层——吠舍。
- `CloneNotSupportedException`：你是一名克隆人。找到你的原型，告诉他你想做什么，然后自杀。
- `IllegalAccessException`：你是一个正在运行 Java 程序入室盗窃的小偷，请结束对电脑的盗窃行为，离开房子，然后再试一次。
- `IllegalArgumentException`：你试图反对之前的异常。
- `IllegalMonitorStateException`：请打开你的电脑屏幕背面。
- `IllegalStateException`：你来自一个尚未被联合国承认的国家，也许是库尔德斯坦或者巴勒斯坦。拿到真正的国籍后重新编译你的 Java 代码，然后再试一次。
- `IllegalThreadStateException`：你电脑的一颗螺丝上到了错误的螺纹孔里，请联系你的硬盘供应商。
- `IndexOutOfBoundsException`：你把食指放在了无法接收的地方，重新放置，再试一次。
- `InstantiationException`：不是每件事都会立即发生，请更耐心一点。
- `InterruptedException`：告诉你的同事、室友等，当你工作的时候，请勿打扰。
- `NegativeArraySizeException`：你创建了一个负数长度的数组。这会丢失信息，长期发展将会毁灭宇宙。不过放宽心，Java 发现了你正在做的事，不要再这么干了。
- `NoSuchFieldException`：你正试图去一个不存在的区域游览。如果你试图去参观一个事实上不存在，其实已经是最高机密的飞机场时，也会得到这个异常。我可以给你示例，然后不得不杀了你。
- `NoSuchMethodException`：不要使用那个方法！拜托了，就像我们一直做的那样去解决事情吧。
- `NullPointerException`：你没有狗。请你先找一只狗，比如一只布烈塔尼獵犬，然后再试一次。
- `NumberFormatException`：你正在使用过时的测量单位，比如英寸或者品脱。请转换成国际基本单位。有一个已知的 bug 会导致 Java 抛出这个异常，那就是你太矮了或者太高了。
- `RuntimeException`：你不能跑得足够快，可能因为你太胖了。关掉你的电脑，出门锻炼吧。
- `SecurityException`：你已被认为是国家安全的一个威胁。请你呆在原地别动，然后等着警察来并带你走。
- `StringIndexOutOfBoundsException`：你的内裤和这个地方格格不入。换掉它们，再试一次。另外如果你根本不穿任何内裤，也会得到这个异常。
- `UnsupportedOperationException`：因为一些原因，你正试图做一个在道德上不被 Java 支持的手术。包括不必要的截肢，例如割包皮。请停止滥用你的身体，不要移除你的孩子，该死的！

### java.util

- `ConcurrentModificationException`：有人修改了你的 Java 代码。你应该更改密码。
- `EmptyStackException`：为了让 Java 工作，你必须在桌子上放一叠 Java 书籍。当然，如果书很厚的话，一本就够了。
- `MissingResourceException`：你太穷了，不配使用 Java。换一个更便宜的语言吧（比如 Whitespace、Shakesperre、Cow、Spaghetti 或者 C#）。
- `NoSuchElementException`：这里只存在四种元素（地球、水、空气、火）。《第五元素》只是部电影而已。
- `TooManyListenersException`：你被太多秘密机构窃听了，SecurityException 马上就到。

### java.awt

- `AWTException`：你正在使用AWT，也就是说你的图形界面会很丑。这个异常只是一个警告可以被忽略。
- `FontFormatException`：你的布局很丑陋，或者你选择了一个糟糕的字体，或者太多的字体。请咨询一名专业的设计师。
- `HeadlessException`：Java 认为身为一名程序员，你实在是太蠢了。
- `IllegalComponentStateException`：你的一个硬件（例如硬盘、CPU、内存）坏掉了。请联系你的硬件供应商。

### java.awt.color

- `CMMException`：你的 CMM 坏掉了，真是见鬼了。我经常烧毁自己的房子，然后去一个新的城市重新开始。
- `ProfileDataException`：你的个人档案包含可疑信息。如果你不是一名共产主义者、恐怖分子或者无神论者，请联系 CIA 修正错误。

### java.awt.datatransfer

- `MimeTypeParseException`：你的哑剧（Mime）糟透了，没人能够理解你到底想表达什么。尝试一些更简单的事情吧，比如迎风散步，或者被困在一个看不见的盒子里。
- `UnsupportedFlavorException`：你正试图使用一种 Java 不知道的香料。大部分人似乎只知道使用香草和樱桃。

### java.beans

- `IntrospectionException`：你太内向了，你应该变得外向一些。 别再当一个呆子，出门去见见人吧！
- `PropertyVetoException`：你的一部分财产被冻结了。这条信息应该已经告诉你谁干的和原因。如果没看见，你可能也不该询问。

### java.io

- `CharConversionException`：你一直试图焚烧一些不燃物。也可能是因为你试着把自己变成一条鱼，但这不可能发生。
- `EOFException`：你得到这条异常是因为你不知道EOF是什么意思。但是，我并不打算告诉你，因为你是一个不学无术的人。
- `FileNotFoundException`：一名木匠应该总是知道他的工具放在哪里。
- `InterruptedIOException`：你不顾之前的 IOException，一直在使用 IO，然后你的活动就被中断了。
- `InvalidClassException`：查看 ClassNotFoundException。
- `InvalidObjectException`：反对无效，就像他们在法庭上说的一样。
- `IOException`：IO 代表输入、输出，并且不得不做收发数据的事。IO 是一个安全问题，不应使用。
- `NotActiveException`：这个异常意味着两件事。要么是未激活，需要激活；要么是已激活，需要停止。到开始工作为止，激活与未激活都是随机的。
- `NotSerializableException`：你正试图把一部电影改成电视剧。
- `ObjectStreamException`：你提出了一连串的反对（Object）意见。提出新的意见前，请限制自己一下，等待法官作出判决。查看 InvalidObjectException。
- `OptionalDataException`：你似乎认为一些可选数据是必须的。不要让事情变得复杂。
- `StreamCorruptedException`：你的数据流被损坏了，这意味着它已经被截包，并在黑市上贩卖。
- `SyncFailedException`：你试图与其他人同步你的失败，然后被证明比他人更加失败。去找一些跟你同等水平的人吧。
- `UnsupportedEncodingException`：如果你想在网上发送自己的代码，必须与美国国家安全局核对你的加密密匙。如果不这么做，将把你视为恐怖分子，并以适当方式处理。如果你得到这个异常，能跑多快跑多快。
- `UTFDataFormatException`：UTF 代表通用传输格式，是一种无论你使用哪种格式都会用到的数据传输方式。你试图通过 UTF 传输错误格式的数据。
- `WriteAbortedException`：你需要在程序中的某处写上“aborted”。这通常没什么意义，但你就得这样做。

### java.net

- `BindException`：Java编程和束缚不能混为一谈。
- `ConnectException`：你正试图与一个不能连接的事物建立连接。试着连接其他事物吧。也许可以通过一个特殊的连接对象实现你想要的连接。
- `MalformedURLException`：你正在制作一个形状错误的壶（例如一个“L”状），或者你有拼写错误的单词“urn”（例如“url”）。
- `NoRouteToHostException`：没有通往主机的“道路”，请联系公路管理员。
- `PortUnreachableException`：港口必须正确地放置在水边。如果在内陆，它们将会无法接触。
- `ProtocolException`：这是一个严重违反规定的结果（例如在你主机上的“puk韓g”）。解决方法很简单：不要那样做！
- `SocketException`：你把电脑连接到了错误的电源插座。大部分情况下你不得不寻找其它插座，但一些电脑背部有一个开关，可以设置电源插座类型。
- `SocketTimeoutException`：你的电脑连接了一个带计时器的电源插座，并且时间已经走完。只有烙铁和相似的东西才会使用这种插座。
- `UnknownHostException`：你的父母没有教过你不要和陌生人说话么？
- `UnknownServiceException`：你正试图进入接近一个未知服务。众所周知，未知服务或许是特工组织。
- `URISyntaxException`：“You are I”是一个语法错误的句子。将其改为“You are me”，别管那到底啥意思。

### java.rmi

- `AccessException`：你正在使用“Microsoft Access”。请不要这样做。
- `AlreadyBoundException`：不管在 java.net.BindException 的描述中是什么状况，RMI 都提供捆绑服务。然而，你不能绑一个已经被捆绑的人。
- `ConnectException`：你正试图与一个不能连接的事物建立连接。试着连接其他事物吧。也许可以通过一个特殊的连接对象实现你想要的连接。
- `ConnectIOException`：你正试图通过 IO 与另一个不能被连接的事物建立连接。尝试连接其他事物吧。或许你可以通过一个特殊的连接对象实现想要的连接。
- `MarshalException`：你的“marshal”出问题了。你应做的事取决于我们正在讨论的是哪种“marshal”。他可以是陆军元帅、警察、消防队员或者只不过是一名普通的司仪。注意这个异常与马绍尔群岛共和国没有任何关系，也称为 RMI。
- `NoSuchObjectException`：你正试图使用一个不存在的对象。以爱因斯坦之名，创造它或者不要使用它！
- `NotBoundException`：如果你正在使用奴隶，请确认至少有一个人被绑住了。
- `RemoteException`：这是一条远程抛出的特殊异常。如果其他人的应用变得不稳定，以致于不能产生一条异常，相反地，你可能会得到这条异常。请找到源头并提醒那位程序员这个错误。
- `RMISecurityException`：马绍尔群岛共和国变得不稳定了。如果你住在这儿，你最好离开，直到安全得到保障为止都别回来。如果你住在其他地方，可以无视这个异常。
- `ServerException`：第二发球（或者双发失误同样适用）。
- `ServerRuntimeException`：只要是网球比赛都很长。当你花太长时间发球时，就会得到这条异常。
- `StubNotFoundException`：当你去看电影的时候，你应该一直保留自己的票根。如果不这么做，并且离开了电影院，你就不能重新进去，不得不去买张新票。所以保留你的票根！
- `UnexpectedException`：这个异常对你来说应该会成为一个大惊喜。如果发生了，所有事都变成它应该的样子。
- `UnknownHostException`：你父母没有教过你不要和陌生人说话吗？
- `UnmarshalException`：.你没有完成一名法律工作人员的职责（例如你曾经的法官工作）。注意这个正确的术语是“曾经”（used to）。你已经被解雇（fire）了（如果你是一名消防队员（firefighter），这可真是讽刺啊）。

### java.security

- `AccessControlException`：你失去了对 Microsoft Access 的控制。如果你无法重获控制或者通过其他方式停止程序，你应该尽快切断电脑电源。
- `DigestException`：你应该注意自己的食物，消化不良也能变成严重的问题。
- `GeneralSecurityException`：在某些地方做一些事情并不安全。如果你有足够的权力，你应该随机入侵一个国家（最好在中东地区）。如果你没有那种权力，至少应该有一把枪。
- `InvalidAlgorithmParameterException`：你向一位残疾人用他不能理解的方式解释你的算法。简单一点！
- `InvalidKeyException`：这个异常有两种不同的原因：1、你正在使用错误的钥匙。我的建议是在你的钥匙上画不同颜色的小点来帮助你记住哪一把对应哪一个锁。2、 你不能锁住残疾人却不给他们钥匙，如果他们足够聪明发现如何使用钥匙，他们就有自由移动的权- 利。
- `InvalidParameterException`：你使用了蔑视的术语去描述一名残疾人。
- `KeyException`：不要尝试不用钥匙就能开锁。
- `KeyManagementException`：你遗失了自己的钥匙。很可能忘在办公室（如果你正试图进入你家）或者忘在家里（如果你正试图进入办公室）。
- `KeyStoreException`：延续之前 KeyManagementException 的解释就是你的钱包有个洞。
- `NoSuchAlgorithmException`：你试图用以前未知的方法解决问题。停止创新吧，用老算法重写一遍。你也可以为自己的想法申请专利，然后等待未来 Java 发布新版本的时候纳入其中。
- `NoSuchProviderException`：如果你是一名单亲妈妈，你没法成为家庭主妇。首先，你得为家庭找到一名供养者。
- `PrivilegedActionException`：你试图采取一个行动，但是没有得到权限。比如，只有名人才可以做到地从谋杀中逃脱，只有天主教神父和耶和华的高级见证人才能做地猥亵儿童，只有在私人企业担任管理职位的人才能被允许地偷钱。
- `ProviderException`：你是一名妇女并试图供养一个家庭。显而易见，你的丈夫不能成为一名“家庭主妇”，所以你得让他供养个家庭。想象一下，Java固执且不肯改变，事情就是这样工作的，解决它。
- `SignatureException`：要么你是伪造的其他人的签名，要么是无法接受你的签名。一个签名不能太丑陋、太易读或太大。
- `UnrecoverableKeyException`：该死。你把你的钥匙扔进了下水沟。我唯一能安慰你的就是其他人也无法恢复钥匙，所以倒不是必须换掉你的锁。

### java.text

- `ParseException`：你做的没有任何意义，冷静下来，再试一次。

---

参考文档：

- [java提高篇之异常（上）](http://www.importnew.com/20629.html)
- [java提高篇之异常（下）](http://www.importnew.com/20645.html)
- [深入理解java异常处理机制](http://www.importnew.com/14688.html)
- [Java 中 9 个处理 Exception 的最佳实践](http://www.importnew.com/26775.html)
- [Java常见异常及解释](http://www.importnew.com/16725.html)