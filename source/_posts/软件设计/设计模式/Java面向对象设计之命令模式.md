---
title: Java面向对象设计之命令模式
date: 2018-10-16 22:40:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/10/16/command.jpg
categories: 软件设计
tags:
  - Java
  - 设计模式
---

## 一、模式动机

在软件设计中，我们经常需要向某些对象发送请求，但是并不知道请求的接收者是谁，也不知道被请求的操作是哪个，我们只需在程序运行时指定具体的请求接收者即可，此时，可以使用命令模式来进行设计，使得请求发送者与请求接收者消除彼此之间的耦合，让对象之间的调用关系更加灵活。

命令模式可以对发送者和接收者完全解耦，发送者与接收者之间没有直接引用关系，发送请求的对象只需要知道如何发送请求，而不必知道如何完成请求。这就是命令模式的模式动机。

## 二、模式定义

> **命令模式(`Command Pattern`)**：将一个请求封装为一个对象，从而使我们可用不同的请求对客户进行参数化；对请求排队或者记录请求日志，以及支持可撤销的操作。命令模式是一种对象行为型模式，其别名为动作(`Action`)模式或事务(`Transaction`)模式。

## 三、模式结构

### 1. 角色组成

命令模式包含如下角色：

- `Command`: 抽象命令类
- `ConcreteCommand`: 具体命令类
- `Invoker`: 调用者
- `Receiver`: 接收者
- `Client`: 客户类

### 2. 结构图

![命令模式结构图][1]

[1]: https://statics.sh1a.qingstor.com/2018/10/16/command-structure.jpg

## 四、示例代码

首先，是抽象的`Receiver`类和具体的`Receiver`类：

```java
/**
 * 通用的抽象 Receiver 接收者.
 *
 * Created by blinkfox on 16/8/17.
 */
public abstract class Receiver {

    /**
     * 定义每个接收者都必须完成的业务.
     */
    public abstract void doSomething();

}
```

```java
/**
 * 具体的 Receiver 类1.
 *
 * Created by blinkfox on 16/8/17.
 */
public class ConcreteReceiver1 extends Receiver {

    @Override
    public void doSomething() {
        System.out.println("ConcreteReceiver1 处理的业务逻辑...");
    }

}
```

```java
/**
 * 具体的 Receiver 类2.
 *
 * Created by blinkfox on 16/8/17.
 */
public class ConcreteReceiver2 extends Receiver {

    @Override
    public void doSomething() {
        System.out.println("ConcreteReceiver2 处理的业务逻辑...");
    }

}
```

其实，是抽象的`Command`类和具体的`Command`类：

```java
/**
 * 抽象的 Command 类.
 *
 * Created by blinkfox on 16/8/17.
 */
public abstract class Command {

    /**
     * 命令的抽象执行命令的方法.
     */
    public abstract void execute();

}
```

```java
/**
 * 具体的 Command 命令类1.
 *
 * Created by blinkfox on 16/8/17.
 */
public class ConcreteCommand1 extends Command {

    /** 对哪个receiver类进行处理. */
    private Receiver receiver;

    public ConcreteCommand1(Receiver receiver) {
        this.receiver = receiver;
    }

    /**
     * 必须实现的一个命令.
     */
    @Override
    public void execute() {
        this.receiver.doSomething();
    }

}
```

```java
/**
 * 具体的 Command 命令类2.
 *
 * Created by blinkfox on 16/8/17.
 */
public class ConcreteCommand2 extends Command {

    /** 对哪个receiver类进行处理. */
    private Receiver receiver;

    public ConcreteCommand2(Receiver receiver) {
        this.receiver = receiver;
    }

    /**
     * 必须实现的命令.
     */
    @Override
    public void execute() {
        this.receiver.doSomething();
    }

}
```

最后，调用者`Invoker`类：

```java
/**
 * 调用者 Invoker 类.
 *
 * Created by blinkfox on 16/8/17.
 */
public class Invoker {

    private Command command;

    public void setCommand(Command command) {
        this.command = command;
    }

    /**
     * 执行命令.
     */
    public void action() {
        this.command.execute();
    }
}
```

以下是命令模式的客户端场景类：

```java
/**
 * 命令模式的场景类.
 *
 * Created by blinkfox on 16/8/17.
 */
public class CommandClient {

    public static void main(String[] args) {
        Invoker invoker = new Invoker();
        Receiver receiver = new ConcreteReceiver1();
        Command command = new ConcreteCommand1(receiver);

        // 把命令交给调用者执行
        invoker.setCommand(command);
        invoker.action();
    }

}
```

## 五、模式分析

**命令模式的本质是对命令进行封装，将发出命令的责任和执行命令的责任分割开**。

- 每一个命令都是一个操作：请求的一方发出请求，要求执行一个操作；接收的一方收到请求，并执行操作。
- 命令模式允许请求的一方和接收的一方独立开来，使得请求的一方不必知道接收请求的一方的接口，更不必知道请求是怎么被接收，以及操作是否被执行、何时被执行，以及是怎么被执行的。
- 命令模式使请求本身成为一个对象，这个对象和其他对象一样可以被存储和传递。
- 命令模式的关键在于引入了抽象命令接口，且发送者针对抽象命令接口编程，只有实现了抽象命令接口的具体命令才能与接收者相关联。

### 1. 优点

命令模式的优点：

- 降低系统的耦合度。
- 新的命令可以很容易地加入到系统中。
- 可以比较容易地设计一个命令队列和宏命令（组合命令）。
- 可以方便地实现对请求的`Undo`和`Redo`。

### 2. 缺点

命令模式的缺点：

- 使用命令模式可能会导致某些系统有过多的具体命令类。因为针对每一个命令都需要设计一个具体命令类，因此某些系统可能需要大量具体命令类，这将影响命令模式的使用。

### 3. 适用环境

在以下情况下可以使用命令模式：

- 系统需要将请求调用者和请求接收者解耦，使得调用者和接收者不直接交互。
- 系统需要在不同的时间指定请求、将请求排队和执行请求。
- 系统需要支持命令的撤销(Undo)操作和恢复(Redo)操作。
- 系统需要将一组操作组合在一起，即支持宏命令

## 六、模式总结

- 在命令模式中，将一个请求封装为一个对象，从而使我们可用不同的请求对客户进行参数化；对请求排队或者记录请求日志，以及支持可撤销的操作。命令模式是一种对象行为型模式，其别名为动作模式或事务模式。
- 命令模式包含四个角色：抽象命令类中声明了用于执行请求的`execute()`等方法，通过这些方法可以调用请求接收者的相关操作；具体命令类是抽象命令类的子类，实现了在抽象命令类中声明的方法，它对应具体的接收者对象，将接收者对象的动作绑定其中；调用者即请求的发送者，又称为请求者，它通过命令对象来执行请求；接收者执行与请求相关的操作，它具体实现对请求的业务处理。
- 命令模式的本质是对命令进行封装，将发出命令的责任和执行命令的责任分割开。命令模式使请求本身成为一个对象，这个对象和其他对象一样可以被存储和传递。
- 命令模式的主要优点在于降低系统的耦合度，增加新的命令很方便，而且可以比较容易地设计一个命令队列和宏命令，并方便地实现对请求的撤销和恢复；其主要缺点在于可能会导致某些系统有过多的具体命令类。
- 命令模式适用情况包括：需要将请求调用者和请求接收者解耦，使得调用者和接收者不直接交互；需要在不同的时间指定请求、将请求排队和执行请求；需要支持命令的撤销操作和恢复操作，需要将一组操作组合在一起，即支持宏命令。