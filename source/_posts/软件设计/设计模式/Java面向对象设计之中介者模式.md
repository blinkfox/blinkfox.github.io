---
title: Java面向对象设计之中介者模式
date: 2018-10-17 23:40:00
author: blinkfox
categories: 软件设计
tags:
  - Java
  - 设计模式
---

## 一、模式动机

在用户与用户直接聊天的设计方案中，用户对象之间存在很强的关联性，将导致系统出现如下问题：

- 系统结构复杂：对象之间存在大量的相互关联和调用，若有一个对象发生变化，则需要跟踪和该对象关联的其他所有对象，并进行适当处理。
- 对象可重用性差：由于一个对象和其他对象具有很强的关联，若没有其他对象的支持，一个对象很难被另一个系统或模块重用，这些对象表现出来更像一个不可分割的整体，职责较为混乱。
- 系统扩展性低：增加一个新的对象需要在原有相关对象上增加引用，增加新的引用关系也需要调整原有对象，系统耦合度很高，对象操作很不灵活，扩展性差。
- 在面向对象的软件设计与开发过程中，根据“单一职责原则”，我们应该尽量将对象细化，使其只负责或呈现单一的职责。
- 对于一个模块，可能由很多对象构成，而且这些对象之间可能存在相互的引用，为了减少对象两两之间复杂的引用关系，使之成为一个松耦合的系统，我们需要使用中介者模式，这就是中介者模式的模式动机。

## 二、模式定义

> **中介者模式(`Mediator Pattern`)**：用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。中介者模式又称为**调停者模式**，它是一种对象行为型模式。

## 三、模式结构

### 1. 角色组成

中介者模式包含如下角色：

- `Mediator`: 抽象中介者
- `ConcreteMediator`: 具体中介者
- `Colleague`: 抽象同事类
- `ConcreteColleague`: 具体同事类

### 2. 结构图

![命令模式结构图][1]

[1]: https://statics.sh1a.qingstor.com/2018/10/17/mediator.jpg

## 四、示例代码

首先，是抽象的`Mediator`类和具体的`ConcreteMediator`类：

```java
/**
 * 通用抽象中介者类.
 *
 * Created by blinkfox on 16/8/21.
 */
public abstract class Mediator {

    /** 定义同事类1. */
    protected ConcreteColleague1 colleague1;

    /** 定义同事类2. */
    protected ConcreteColleague2 colleague2;

    /* getter 和 setter 方法 */
    public ConcreteColleague1 getColleague1() {
        return colleague1;
    }

    public void setColleague1(ConcreteColleague1 colleague1) {
        this.colleague1 = colleague1;
    }

    public ConcreteColleague2 getColleague2() {
        return colleague2;
    }

    public void setColleague2(ConcreteColleague2 colleague2) {
        this.colleague2 = colleague2;
    }

    /**
     * 中介者模式的抽象业务逻辑1.
     */
    public abstract void doSomething1();

    /**
     * 中介者模式的抽象业务逻辑2.
     */
    public abstract void doSomething2();

}
```

```java
/**
 * 具体的通用中介者类.
 *
 * Created by blinkfox on 16/8/21.
 */
public class ConcreteMediator extends Mediator {

    /**
     * 中介者模式的具体业务逻辑1.
     */
    @Override
    public void doSomething1() {
        super.colleague1.selfMethod1();
        super.colleague2.selfMethod2();
    }

    /**
     * 中介者模式的具体业务逻辑2.
     */
    @Override
    public void doSomething2() {
        super.colleague1.selfMethod1();
        super.colleague2.selfMethod2();
    }

}
```

其实，是抽象的`Colleague`类和具体的`ConcreteColleague`类：

```java
/**
 * 抽象的同事类.
 *
 * Created by blinkfox on 16/8/21.
 */
public abstract class Colleague {

    /** 中介者. */
    protected Mediator mediator;

    public Colleague(Mediator mediator) {
        this.mediator = mediator;
    }

}
```

```java
/**
 * 具体的同事类1.
 *
 * Created by blinkfox on 16/8/21.
 */
public class ConcreteColleague1 extends Colleague {

    public ConcreteColleague1(Mediator mediator) {
        super(mediator);
    }

    /**
     * 自有方法.
     */
    public void selfMethod1() {
        System.out.println("------ConcreteColleague1-处理自己的业务逻辑1--------");
    }

    /**
     * 依赖方法.
     */
    public void depMethod1() {
        System.out.println("------ConcreteColleague1-委托给中介者的业务逻辑1--------");
        super.mediator.doSomething1();
    }

}
```

```java
/**
 * 具体的同事类2.
 *
 * Created by blinkfox on 16/8/21.
 */
public class ConcreteColleague2 extends Colleague {

    public ConcreteColleague2(Mediator mediator) {
        super(mediator);
    }

    /**
     * 自有方法2.
     */
    public void selfMethod2() {
        System.out.println("------ConcreteColleague2-处理自己的业务逻辑2--------");
    }

    /**
     * 依赖方法2.
     */
    public void depMethod2() {
        System.out.println("------ConcreteColleague2-委托给中介者的业务逻辑2--------");
        super.mediator.doSomething2();
    }

}
```

以下是中介者模式的客户端场景类：

```java
/**
 * 中介者模式的场景类
 * Created by blinkfox on 16/8/21.
 */
public class MediatorClient {

    public static void main(String[] args) {
        Mediator mediator = new ConcreteMediator();

        ConcreteColleague1 colleague1 = new ConcreteColleague1(mediator);
        ConcreteColleague2 colleague2 = new ConcreteColleague2(mediator);
        mediator.setColleague1(colleague1);
        mediator.setColleague2(colleague2);

        colleague1.depMethod1();
        colleague2.depMethod2();
        mediator.doSomething1();
        mediator.doSomething2();
    }

}
```

## 五、模式分析

中介者模式可以使对象之间的关系数量急剧减少。

中介者承担两方面的职责：

- **中转作用（结构性）**：通过中介者提供的中转作用，各个同事对象就不再需要显式引用其他同事，当需要和其他同事进行通信时，通过中介者即可。该中转作用属于中介者在结构上的支持。
- **协调作用（行为性）**：中介者可以更进一步的对同事之间的关系进行封装，同事可以一致地和中介者进行交互，而不需要指明中介者需要具体怎么做，中介者根据封装在自身内部的协调逻辑，对同事的请求进行进一步处理，将同事成员之间的关系行为进行分离和封装。该协调作用属于中介者在行为上的支持。

### 1. 优点

中介者模式的优点：

- 简化了对象之间的交互。
- 将各同事解耦。
- 减少子类生成。
- 可以简化各同事类的设计和实现。

### 2. 缺点

中介者模式的缺点：

- 在具体中介者类中包含了同事之间的交互细节，**可能会导致具体中介者类非常复杂，使得系统难以维护**。

### 3. 适用环境

在以下情况下可以使用中介者模式：

- 系统中对象之间存在复杂的引用关系，产生的相互依赖关系结构混乱且难以理解。
- 一个对象由于引用了其他很多对象并且直接和这些对象通信，导致难以复用该对象。
- 想通过一个中间类来封装多个类中的行为，而又不想生成太多的子类。可以通过引入中介者类来实现，在中介者中定义对象。
- 交互的公共行为，如果需要改变行为则可以增加新的中介者类。

## 六、模式总结

- 中介者模式用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。中介者模式又称为调停者模式，它是一种对象行为型模式。
- 中介者模式包含四个角色：抽象中介者用于定义一个接口，该接口用于与各同事对象之间的通信；具体中介者是抽象中介者的子类，通过协调各个同事对象来实现协作行为，了解并维护它的各个同事对象的引用；抽象同事类定义各同事的公有方法；具体同事类是抽象同事类的子类，每一个同事对象都引用一个中介者对象；每一个同事对象在需要和其他同事对象通信时，先与中介者通信，通过中介者来间接完成与其他同事类的通信；在具体同事类中实现了在抽象同事类中定义的方法。
- 通过引入中介者对象，可以将系统的网状结构变成以中介者为中心的星形结构，中介者承担了中转作用和协调作用。中介者类是中介者模式的核心，它对整个系统进行控制和协调，简化了对象之间的交互，还可以对对象间的交互进行进一步的控制。
- 中介者模式的主要优点在于简化了对象之间的交互，将各同事解耦，还可以减少子类生成，对于复杂的对象之间的交互，通过引入中介者，可以简化各同事类的设计和实现；中介者模式主要缺点在于具体中介者类中包含了同事之间的交互细节，可能会导致具体中介者类非常复杂，使得系统难以维护。
- 中介者模式适用情况包括：系统中对象之间存在复杂的引用关系，产生的相互依赖关系结构混乱且难以理解；一个对象由于引用了其他很多对象并且直接和这些对象通信，导致难以复用该对象；想通过一个中间类来封装多个类中的行为，而又不想生成太多的子类。