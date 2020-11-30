---
title: Java面向对象设计之责任链模式
date: 2018-11-4 22:00:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/11/04/chain.jpg
categories: 软件设计
tags:
  - Java
  - 设计模式
---

## 模式动机

很多情况下，在一个软件系统中可以处理某个请求的对象不止一个。例如审批工作流等，他们可以构成一条处理采购单的链式结构，采购单(可以看作是要处理的信息)沿着这条链进行传递，这条链就称为责任链。责任链可以是一条直线、一个环或者一个树形结构，最常见的职责链是直线型，即沿着一条单向的链来传递请求。链上的每一个对象都是请求处理者，责任链模式可以将请求的处理者组织成一条链，并让请求沿着链传递，由链上的处理者对请求进行相应的处理。在此过程中，客户端实际上无须关心请求的处理细节以及请求的传递，只需将请求发送到链上即可，从而实现请求发送者和请求处理者解耦。

## 模式定义

> **定义**：责任链模式(`Chain of Responsibility Pattern`)是使多个对象都有机会处理请求，从而避免请求的发送者与请求处理者耦合在一起。将这些对象连接成一条链，并且沿着这条链传递请求，直到有对象处理它为止。它是一种对象行为型模式。

**实质**：责任链上的处理者负责处理请求，客户只需要将请求发送到职责链上即可，无须关心请求的处理细节和请求的传递，从而实现请求发送者与请求处理者的解耦。

## 模式结构

### 参与角色

- `Handler`（抽象处理者）：处理请求的接口，一般设计为具有抽象请求处理方法的抽象类，以便于不同的具体处理者进行继承，从而实现具体的请求处理方法。此外，由于每一个请求处理者的下家还是一个处理者，因此抽象处理者本身还包含了一个本身的引用(`nextHandler`)作为其对下家的引用，以便将处理者链成一条链；
- `ConcreteHandler`（具体处理者）：抽象处理者的子类，可以处理用户请求，其实现了抽象处理者中定义的请求处理方法。在具体处理请求时需要进行判断，若其具有相应的处理权限，那么就处理它；否则，其将请求转发给后继者，以便让后面的处理者进行处理。

在责任链模式里，由每一个请求处理者对象对其下家的引用而连接起来形成一条请求处理链。请求将在这条链上一直传递，直到链上的某一个请求处理者能够处理此请求。**发出这个请求的客户端并不知道链上的哪一个请求处理者将处理这个请求，这使得系统可以在不影响客户端的情况下动态地重新组织链和分配责任**。

### UML类图

![责任链模式UML类图](https://statics.sh1a.qingstor.com/2018/11/04/chain-of-responsibility.png)

## 代码示例

首先，责任链模式的核心在于对请求处理者的抽象。在实现过程中，抽象处理者一般会被设定为**抽象类**，其典型实现代码如下所示：

```java
/**
 * 责任连模式的抽象处理者角色.
 *
 * Created by blinkfox on 16/7/11.
 */
public abstract class Handler {

    /** 后继处理者角色. */
    protected Handler nextHandler;

    /**
     * 处理请求的抽象方法.
     *
     * @param condition 条件
     */
    public abstract void handle(String condition);

    /**
     * nextHandler的Setter方法.
     *
     * @param nextHandler 后继处理器
     */
    public void setNextHandler(Handler nextHandler) {
        this.nextHandler = nextHandler;
    }

}
```

其次，是若干个具体的处理角色类。

```java
/**
 * 具体处理角色1.
 *
 * Created by blinkfox on 16/7/11.
 */
public class ConcreteHandler1 extends Handler {

    /**
     * 具体处理角色1的处理方法.
     *
     * @param condition 条件
     */
    @Override
    public void handle(String condition) {
        // 如果是自己的责任，就自己处理，负责传给下家处理
        if ("ConcreteHandler1".equals(condition)) {
            System.out.println( "具体处理角色1的处理方法handled1...");
        } else {
            System.out.println( "具体处理角色1 通过...");
            nextHandler.handle(condition);
        }
    }

}
```

```java
/**
 * 具体处理角色2.
 *
 * Created by blinkfox on 16/7/11.
 */
public class ConcreteHandler2 extends Handler {

    /**
     * 具体处理角色2的处理方法.
     *
     * @param condition 条件
     */
    @Override
    public void handle(String condition) {
        // 如果是自己的责任，就自己处理，负责传给下家处理
        if ("ConcreteHandler2".equals(condition)) {
            System.out.println( "具体处理角色2的处理方法handled1...");
        } else {
            System.out.println( "具体处理角色2 通过...");
            nextHandler.handle(condition);
        }
    }

}
```

```java
/**
 * 具体处理角色n.
 *
 * Created by blinkfox on 16/7/11.
 */
public class ConcreteHandlerN extends Handler {

    /**
     * 这里假设n是链的最后一个节点必须处理掉.
     * 在实际情况下，可能出现环，或者是树形，这里并不一定是最后一个节点.
     *
     * @param condition 参数条件
     */
    @Override
    public void handle(String condition) {
        System.out.println( "具体处理角色n的处理方法 结束...");
    }

}
```

最后，是客户端场景类，代码调用示例如下：

```java
/**
 * 责任连模式的客户端场景类.
 *
 * Created by blinkfox on 16/7/11.
 */
public class ChainClient {

    /**
     * 主入口方法.
     *
     * @param args 数组参数
     */
    public static void main(String[] args) {
        Handler handler1 = new ConcreteHandler1();
        Handler handler2 = new ConcreteHandler2();
        Handler handlern = new ConcreteHandlerN();

        handler1.setNextHandler(handler2);
        handler2.setNextHandler(handlern);

        //假设这个请求是ConcreteHandler2的责任
        handler1.handle("ConcreteHandler2");
    }

}
```

> **注**：责任链模式并不创建职责链，职责链的创建工作必须由系统的其他部分来完成，一般由使用该责任链的客户端创建。职责链模式降低了请求的发送者和请求处理者之间的耦合，从而使得多个请求处理者都有机会处理这个请求。

## 模式分析

### 使用场景

在实际软件开发中，如果遇到有多个对象可以处理同一请求时可以考虑使用职责链模式，最常见的例子包括在 Java Web 应用开发中创建一个过滤器（Filter）链来对请求数据进行过滤（中文字符乱码的处理）、在工作流系统中实现公文的分级审批、在Struts应用中添加不同的拦截器(常用的有类型转化、异常处理，数据校验…)以增强Struts2的功能等。

### 纯与不纯的责任链模式

- **纯的责任链模式**要求一个具体的处理者对象只能在两个行为中选择一个：一是承担责任，而是把责任推给下家。不允许出现某一个具体处理者对象在承担了一部分责任后又把责任向下传的情况；
- 在纯责任链模式里面，一个请求必须被某一个处理者对象所接收；
- 在不纯的责任链模式里面，一个请求可以最终不被任何接收端对象所接收。

纯的责任链模式的实际例子很难找到，一般看到的例子均是不纯的责任链模式的实现。

### 优点

- 降低耦合度，使请求的发送者和接收者解耦，便于灵活的、可插拔的定义请求处理过程；
- 简化、封装了请求的处理过程，并且这个过程对客户端而言是透明的，以便于动态地重新组织链以及分配责任，增强请求处理的灵活性；
- 可以从职责链任何一个节点开始，也可以随时改变内部的请求处理规则，每个请求处理者都可以去动态地指定他的继任者；
- 职责链可简化对象间的相互连接。它们仅需保持一个指向其后继者的引用，而不需保持它所有的候选接受者的引用；
- 增加新的请求处理类很方便。

### 缺点

- 不能保证请求一定被接收。既然一个请求没有明确的接收者，那么就不能保证它一定会被处理；
- 该请求可能一直到链的末端都得不到处理。一个请求也可能因该链没有被正确配置而得不到处理；
- 系统性能将受到一定影响，而且在进行代码调试时不太方便；可能会造成循环调用。

## 总结

- 在职责链模式里，很多对象由每一个对象对其下家的引用而连接起来形成一条链。请求在这个链上传递，直到链上的某一个对象决定处理此请求。发出这个请求的客户端并不知道链上的哪一个对象最终处理这个请求，这使得系统可以在不影响客户端的情况下动态地重新组织链和分配责任。
- 职责链模式的主要优点在于可以降低系统的耦合度，简化对象的相互连接，同时增强给对象指派职责的灵活性，增加新的请求处理类也很方便；其主要缺点在于不能保证请求一定被接收，且对于比较长的职责链，请求的处理可能涉及到多个处理对象，系统性能将受到一定影响，而且在进行代码调试时不太方便。