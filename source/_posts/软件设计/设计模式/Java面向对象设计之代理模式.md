---
title: Java面向对象设计之代理模式
date: 2018-09-20 21:50:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/09/20/proxy.jpg
categories: 软件设计
tags:
  - Java
  - 设计模式
---

## 一、模式动机

在某些情况下，一个客户不想或者不能直接引用一个对象，此时可以通过一个称之为“代理”的第三者来实现间接引用。代理对象可以在客户端和目标对象之间起到中介的作用，并且可以通过代理对象去掉客户不能看到 的内容和服务或者添加客户需要的额外服务。

通过引入一个新的对象来实现对真实对象的操作或者将新的对象作为真实对象的一个替身，这种实现机制即为代理模式，通过引入代理对象来间接访问一个对象，这就是代理模式的模式动机。

## 二、模式定义

> **代理模式(`Proxy Pattern`)**：给某一个对象提供一个代理，并由代理对象控制对原对象的引用。代理模式的英文叫做`Proxy`或`Surrogate`，它是一种对象结构型模式。

## 三、模式结构

### 1. 角色组成

代理模式包含如下角色：

- `Subject`: 抽象主题角色
- `RealSubject`: 真实主题角色
- `Proxy`: 代理主题角色

### 2. 结构图

![代理模式结构图][1]

## 四、示例代码

首先，是抽象的主题接口和真实主题类：

```java
package com.blinkfox.patterns.proxy;

/**
 * 抽象主题类
 * Created by blinkfox on 2017/1/1.
 */
public interface ISubject {

    /**
     * 定义一个方法
     */
    public void request();

}
```

```java
package com.blinkfox.patterns.proxy;

/**
 * 真实主题类
 * Created by blinkfox on 2017/1/1.
 */
public class RealSubject implements ISubject {

    /**
     * 实现方法
     */
    @Override
    public void request() {
        System.out.println("真实主题类请求方法...");
    }

}
```

然后，是代理类：

```java
package com.blinkfox.patterns.proxy;

/**
 * 代理类
 * Created by blinkfox on 2017/1/1.
 */
public class Proxy implements ISubject {

    private ISubject subject;

    public Proxy(ISubject subject) {
        this.subject = subject;
    }

    @Override
    public void request() {
        this.before();
        this.subject.request();
        this.after();
    }

    /**
     * 预处理
     */
    private void before() {
        System.out.println("执行前(before)的处理...");
    }

    /**
     * 善后处理
     */
    private void after() {
        System.out.println("执行后(after)的处理...");
    }

}
```

最后，是客户端场景测试类：

```java
package com.blinkfox.patterns.proxy;

/**
 * 代理模式客户端场景类
 * Created by blinkfox on 2017/1/1.
 */
public class ProxyClient {

    public static void main(String[] args) {
        ISubject subject = new RealSubject();
        Proxy proxy = new Proxy(subject);
        proxy.request();
    }

}
```

## 五、模式分析

### 1. 优点

代理模式的优点：

- 代理模式能够协调调用者和被调用者，在一定程度上降低了系 统的耦合度。
- 远程代理使得客户端可以访问在远程机器上的对象，远程机器可能具有更好的计算性能与处理速度，可以快速响应并处理客户端请求。
- 虚拟代理通过使用一个小对象来代表一个大对象，可以减少系统资源的消耗，对系统进行优化并提高运行速度。
- 保护代理可以控制对真实对象的使用权限。

### 2. 缺点

代理模式的缺点：

- 由于在客户端和真实主题之间增加了代理对象，因此有些类型的代理模式可能会造成请求的处理速度变慢。
- 实现代理模式需要额外的工作，有些代理模式的实现非常复杂。

### 3. 适用环境

根据代理模式的使用目的，常见的代理模式有以下几种类型：

- **远程(Remote)代理**：为一个位于不同的地址空间的对象提供一个本地的代理对象，这个不同的地址空间可以是在同一台主机中，也可是在另一台主机中，远程代理又叫做大使(Ambassador)。
- **虚拟(Virtual)代理**：如果需要创建一个资源消耗较大的对象，先创建一个消耗相对较小的对象来表示，真实对象只在需要时才会被真正创建。
- **Copy-on-Write代理**：它是虚拟代理的一种，把复制（克隆）操作延迟 到只有在客户端真正需要时才执行。一般来说，对象的深克隆是一个开销较大的操作，Copy-on-Write代理可以让这个操作延迟，只有对象被用到的时候才被克隆。
- **保护(Protect or Access)代理**：控制对一个对象的访问，可以给不同的用户提供不同级别的使用权限。
- **缓冲(Cache)代理**：为某一个目标操作的结果提供临时的存储空间，以便多个客户端可以共享这些结果。
- **防火墙(Firewall)代理**：保护目标不让恶意用户接近。
- **同步化(Synchronization)代理**：使几个用户能够同时使用一个对象而没有冲突。
- **智能引用(Smart Reference)代理**：当一个对象被引用时，提供一些额外的操作，如将此对象被调用的次数记录下来等。

## 模式总结

- 在代理模式中，要求给某一个对象提供一个代理，并由代理对象控制对原对象的引用。代理模式的英文叫做Proxy或Surrogate，它是一种对象结构型模式。
- 代理模式包含三个角色：抽象主题角色声明了真实主题和代理主题的共同接口；代理主题角色内部包含对真实主题的引用，从而可以在任何时候操作真实主题对象；真实主题角色定义了代理角色所代表的真实对象，在真实主题角色中实现了真实的业务操作，客户端可以通过代理主题角色间接调用真实主题角色中定义的方法。
- 代理模式的优点在于能够协调调用者和被调用者，在一定程度上降低了系统的耦合度；其缺点在于由于在客户端和真实主题之间增加了代理对象，因此有些类型的代理模式可能会造成请求的处理速度变慢，并且实现代理模式需要额外的工作，有些代理模式的实现非常复杂。远程代理为一个位于不同的地址空间的对象提供一个本地的代表对象，它使得客户端可以访问在远程机器上的对象，远程机器可能具有更好的计算性能与处理速度，可以快速响应并处理客户端请求。
- 如果需要创建一个资源消耗较大的对象，先创建一个消耗相对较小的对象来表示，真实对象只在需要时才会被真正创建，这个小对象称为虚拟代理。虚拟代理通过使用一个小对象来代表一个大对象，可以减少系统资源的消耗，对系统进行优化并提高运行速度。
- 保护代理可以控制对一个对象的访问，可以给不同的用户提供不同级别的使用权限。

  [1]: https://statics.sh1a.qingstor.com/2018/09/20/proxy-structure.jpg
