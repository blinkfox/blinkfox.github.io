---
title: Java面向对象设计之工厂方法模式
date: 2018-09-14 22:50:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/09/14/factory.jpg
categories: 软件设计
tags:
  - Java
  - 设计模式
---

## 一、模式定义

工厂方法模式(`Factory Method Pattern`)又称为工厂模式，也叫虚拟构造器(Virtual Constructor)模式或者多态工厂(Polymorphic Factory)模式，它属于类创建型模式。在工厂方法模式中，工厂父类负责定义创建产品对象的公共接口，而工厂子类则负责生成具体的产品对象，这样做的目的是将产品类的实例化操作延迟到工厂子类中完成，即通过工厂子类来确定究竟应该实例化哪一个具体产品类。

## 二、模式结构

### 1. 角色组成

工厂方法模式包含如下角色：

- `Product`：抽象产品
- `ConcreteProduct`：具体产品
- `Factory`：抽象工厂
- `ConcreteFactory`：具体工厂

### 2. 结构图

![工厂方法模式结构图][1]

### 3. 时序图

![工厂方法模式时序图][2]

## 三、示例代码

首先，是抽象的产品类和具体的产品类：

```java
/**
 * 抽象产品类
 * Created by blinkfox on 16-6-29.
 */
public abstract class Product {

    /**
     * 产品类的公共方法
     */
    public void method1() {
        System.out.println("这是产品类的公共方法");
    }

    /**
     * 抽象方法
     */
    public abstract void method2();

}
```

```java
/**
 * 具体产品类1
 * Created by blinkfox on 16-6-29.
 */
public class ConcreteProduct1 extends Product {

    @Override
    public void method2() {
        System.out.println("ConcreteProduct1的method2方法");
    }

}
```

```java
/**
 * 具体产品类2
 * Created by blinkfox on 16-6-29.
 */
public class ConcreteProduct2 extends Product {

    @Override
    public void method2() {
        System.out.println("ConcreteProduct2的method2方法");
    }

}
```

然后，是抽象的工厂类和具体的工厂类：

```java
/**
 * 抽象的工厂类
 * Created by blinkfox on 16-6-29.
 */
public abstract class Factory {

    /**
     * 运用了Java中的泛型和反射技术,生成某种具体的产品
     * 其输入类型可以自行设置
     * @param c
     * @param <T>
     * @return
     */
    public abstract <T extends Product> T createProduct(Class<T> c);

}
```

```java
/**
 * 具体生产产品的工厂类
 * Created by blinkfox on 16-6-29.
 */
public class ConcreteFactory extends Factory {

    /**
     * 运用了Java中的泛型和反射技术,生成某种具体的产品
     * 其输入类型可以自行设置
     * @param c
     * @param <T>
     * @return
     */
    @Override
    public <T extends Product> T createProduct(Class<T> c) {
        Product product = null;
        try {
            product = (Product) Class.forName(c.getName()).newInstance();
        } catch (Exception e) {
            System.out.println("生产产品出错");
            e.printStackTrace();
        }
        return (T) product;
    }

}
```

最后，是客户端场景类：

```java
/**
 * 工厂方法模式客户端场景类
 * Created by blinkfox on 16-6-29.
 */
public class Client {

    public static void main(String[] args) {
        Factory factory = new ConcreteFactory();
        Product product1 = factory.createProduct(ConcreteProduct1.class);
        product1.method1();
        product1.method2();

        Product product2 = factory.createProduct(ConcreteProduct2.class);
        product2.method1();
        product2.method2();
    }

}
```

## 四、模式分析

在工厂方法模式中，核心的工厂类不再负责所有产品的创建，而是将具体创建工作交给子类去做。这个核心类仅仅负责给出具体工厂必须实现的接口，而不负责哪一个产品类被实例化这种细节，这使得工厂方法模式可以允许系统在不修改工厂角色的情况下引进新产品。

### 1. 优点

工厂方法模式的优点：

- 在工厂方法模式中，工厂方法用来创建客户所需要的产品，同时还向客户隐藏了哪种具体产品类将被实例化这一细节，用户只需要关心所需产品对应的工厂，无须关心创建细节，甚至无须知道具体产品类的类名。
- 基于工厂角色和产品角色的多态性设计是工厂方法模式的关键。它能够使工厂可以自主确定创建何种产品对象，而如何创建这个对象的细节则完全封装在具体工厂内部。工厂方法模式之所以又被称为多态工厂模式，是因为所有的具体工厂类都具有同一抽象父类。
- 使用工厂方法模式的另一个优点是在系统中加入新产品时，无须修改抽象工厂和抽象产品提供的接口，无须修改客户端，也无须修改其他的具体工厂和具体产品，而只要添加一个具体工厂和具体产品就可以了。这样，系统的可扩展性也就变得非常好，完全符合“开闭原则”。

### 2. 缺点

工厂方法模式的缺点：

- 在添加新产品时，需要编写新的具体产品类，而且还要提供与之对应的具体工厂类，系统中类的个数将成对增加，在一定程度上增加了系统的复杂度，有更多的类需要编译和运行，会给系统带来一些额外的开销。
- 由于考虑到系统的可扩展性，需要引入抽象层，在客户端代码中均使用抽象层进行定义，增加了系统的抽象性和理解难度，且在实现时可能需要用到DOM、反射等技术，增加了系统的实现难度。

### 3. 适用环境

在以下情况下可以使用工厂方法模式：

- 一个类不知道它所需要的对象的类：在工厂方法模式中，客户端不需要知道具体产品类的类名，只需要知道所对应的工厂即可，具体的产品对象由具体工厂类创建；客户端需要知道创建具体产品的工厂类。
- 一个类通过其子类来指定创建哪个对象：在工厂方法模式中，对于抽象工厂类只需要提供一个创建产品的接口，而由其子类来确定具体要创建的对象，利用面向对象的多态性和里氏代换原则，在程序运行时，子类对象将覆盖父类对象，从而使得系统更容易扩展。
- 将创建对象的任务委托给多个工厂子类中的某一个，客户端在使用时可以无须关心是哪一个工厂子类创建产品子类，需要时再动态指定，可将具体工厂类的类名存储在配置文件或数据库中。

## 五、模式扩展

工厂方法模式有很多扩展，而且与其他模式结合使用威力更大，下面介绍4种常用扩展。

### 1. 简单工厂模式

我们这样考虑一个问题：一个模块仅需要一个工厂类，没有必要把它产生出来，使用静态的方法就可以了。因此去掉工厂类中继承的抽象类，把方法改成静态即可。通用代码如下：

```java
/**
 * 简单工厂模式中的工厂类
 * Created by blinkfox on 16-6-29.
 */
public class SimpleFactory {

    /**
     * 运用了Java中的泛型和反射技术,生成某种具体的产品
     * 其输入类型可以自行设置
     * @param c
     * @param <T>
     * @return
     */
    public static  <T extends Product> T createProduct(Class<T> c) {
        Product product = null;
        try {
            product = (Product) Class.forName(c.getName()).newInstance();
        } catch (Exception e) {
            System.out.println("生产产品出错");
            e.printStackTrace();
        }
        return (T) product;
    }

}
```

```java
/**
 * 简单工厂模式客户端场景类
 * Created by blinkfox on 16-6-29.
 */
public class SimpleClient {

    public static void main(String[] args) {
        Product product1 = SimpleFactory.createProduct(ConcreteProduct1.class);
        product1.method1();
        product1.method2();

        Product product2 = SimpleFactory.createProduct(ConcreteProduct2.class);
        product2.method1();
        product2.method2();
    }

}
```

运行结果没有发生变化，但是类图简单了，调用者也比较简单，简单工厂模式是工厂方法模式的弱化，也叫做静态工厂模式。其缺点是工厂类的扩展比较困难，不符合“开闭原则”，但它仍然是一个非常实用的设计模式。

### 2. 多工厂类工厂方法模式

当我们在一个比较复杂的项目时，经常会遇到初始化一个对象很耗费精力的情况，所有的产品类都放到一个工厂方法中进行初始化会使代码结构不清晰。为了让结构清晰，我们就为每类产品定义一个创造者，然后由调用者自己去选择与哪个工厂方法关联。多工厂模式的通用代码如下：

多工厂模式的抽象工厂类：

```java
/**
 * 生成多个产品的抽象工厂类
 * Created by blinkfox on 16-7-2.
 */
public abstract class MultiFactory {

    /**
     * 生成某种产品的方法
     * @return
     */
    public abstract Product createProduct();

}
```

第一种产品的创建工厂实现：

```java
/**
 * 生成产品1的具体工厂类1
 * Created by blinkfox on 16-7-2.
 */
public class ConcreteFactory1 extends MultiFactory {

    /**
     * 生成产品1的方法
     * @return
     */
    @Override
    public Product createProduct() {
        return new ConcreteProduct1();
    }

}
```

第二种产品的创建工厂实现：

```java
/**
 * 生成产品2的具体工厂类2
 * Created by blinkfox on 16-7-2.
 */
public class ConcreteFactory2 extends MultiFactory {

    /**
     * 生成产品2的方法
     * @return
     */
    @Override
    public Product createProduct() {
        return new ConcreteProduct2();
    }

}
```

多工厂模式的客户端场景类

```java
/**
 * 多工厂方法模式客户端场景类
 * Created by blinkfox on 16-7-2.
 */
public class MultiClient {

    public static void main(String[] args) {
        Product concreteProduct1 = (new ConcreteFactory1()).createProduct();
        concreteProduct1.method1();
        concreteProduct1.method2();

        Product concreteProduct2 = (new ConcreteFactory2()).createProduct();
        concreteProduct1.method1();
        concreteProduct1.method2();
    }

}
```

### 3. 工厂方法的单例模式

单例模式的核心要求就是在内存中只有一个对象，通过工厂方法模式也可以只在内存中生成一个对象，从而实现单例的功能。

下面是单例类，其中定义了一个private的无参构造函数，目的是不允许通过new的方式创建对象，代码如下：

```java
/**
 * 工厂方法模式中的单例类
 * Created by blinkfox on 16-7-4.
 */
public class Singleton {

    /**
     * 私有化构造方法，不允许new产生一个对象
     */
    private Singleton() {}

    /**
     * 工厂方法模式中的单例模式业务方法
     */
    public void doSomething() {
        System.out.println("工厂方法模式中的单例模式方法。。。");
    }

}
```

以上单例类中不能通过正常的渠道建立一个对象，那单例的工厂类中如何建立一个单例对象呢？答案是通过反射方式创建，单例工厂类的代码如下：

```java
/**
 * 生成单例的工厂类
 * Created by blinkfox on 16-7-4.
 */
public class SingletonFactory {

    private static Singleton singleton;

    static {
        try {
            Class c = Class.forName(Singleton.class.getName());
            // 获得无参构造
            Constructor constructor = c.getDeclaredConstructor();
            // 设置无参构造是可访问的
            constructor.setAccessible(true);
            // 产生一个实例对象
            singleton = (Singleton) constructor.newInstance();
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("生成单例的工厂类方法中生成单例出错");zuihou
        }
    }

    public static Singleton getSingleton() {
        return singleton;
    }
}
```

最后是工厂方法单例模式的客户端场景类：

```java
/**
 * 工厂方法单例模式客户端场景类
 * Created by blinkfox on 16-7-4.
 */
public class SingleClient {

    public static void main(String[] args) {
        Singleton singleton = SingletonFactory.getSingleton();
        singleton.doSomething();
    }

}
```

### 4. 工厂方法的延迟初始化

何为延迟初始化？一个对象被消费完毕后，并不立即释放，工厂类保持其初始状态，等待再次使用。延迟初始化是工厂模式的一个扩展应用，其通用代码如下：

```java
/**
 * 延迟加载的工厂类
 * Created by blinkfox on 16-7-4.
 */
public class LazyFactory {

    private static final Map<String, Product> lazyMap = new HashMap<String, Product>();

    public static synchronized Product createProduct(String type) {
        Product product = null;

        // 如果map中已经有这个对象，则直接取出该对象即可，否则创建并放在缓存容器中
        if (lazyMap.containsKey(type)) {
            return lazyMap.get(type);
        }

        // 根据类型创建具体的产品对象
        if ("product1".equals(type)) {
            product = new ConcreteProduct1();
        } else {
            product = new ConcreteProduct2();
        }
        // 同时把对象放到缓存容器中
        lazyMap.put("type", product);

        return product;
    }

}
```

上面即为延迟加载的工厂类。代码比较简单，通过定义一个`map`容器来容纳所有产生的对象，如果在`map`容器中已经有的对象，则直接取出返回；如果没有，则根据需要的类型产生一个对象并放入到`map`容器中，以便下次调用。

延迟加载的工厂模式客户端场景类代码如下：

```java
/**
 * 延迟加载的工厂模式客户端场景类
 * Created by blinkfox on 16-7-4.
 */
public class LazyClient {

    public static void main(String[] args) {
        Product product1 = LazyFactory.createProduct("product1");

        Product product11 = LazyFactory.createProduct("product1");
    }

}
```

## 六、总结

- 工厂方法模式又称为工厂模式，它属于类创建型模式。在工厂方法模式中，工厂父类负责定义创建产品对象的公共接口，而工厂子类则负责生成具体的产品对象，这样做的目的是将产品类的实例化操作延迟到工厂子类中完成，即通过工厂子类来确定究竟应该实例化哪一个具体产品类。
- 工厂方法模式包含四个角色：抽象产品是定义产品的接口，是工厂方法模式所创建对象的超类型，即产品对象的共同父类或接口；具体产品实现了抽象产品接口，某种类型的具体产品由专门的具体工厂创建，它们之间往往一一对应；抽象工厂中声明了工厂方法，用于返回一个产品，它是工厂方法模式的核心，任何在模式中创建对象的工厂类都必须实现该接口；具体工厂是抽象工厂类的子类，实现了抽象工厂中定义的工厂方法，并可由客户调用，返回一个具体产品类的实例。
- 工厂方法模式是简单工厂模式的进一步抽象和推广。由于使用了面向对象的多态性，工厂方法模式保持了简单工厂模式的优点，而且克服了它的缺点。在工厂方法模式中，核心的工厂类不再负责所有产品的创建，而是将具体创建工作交给子类去做。这个核心类仅仅负责给出具体工厂必须实现的接口，而不负责产品类被实例化这种细节，这使得工厂方法模式可以允许系统在不修改工厂角色的情况下引进新产品。
- 工厂方法模式的主要优点是增加新的产品类时无须修改现有系统，并封装了产品对象的创建细节，系统具有良好的灵活性和可扩展性；其缺点在于增加新产品的同时需要增加新的工厂，导致系统类的个数成对增加，在一定程度上增加了系统的复杂性。
- 工厂方法模式适用情况包括：一个类不知道它所需要的对象的类；一个类通过其子类来指定创建哪个对象；将创建对象的任务委托给多个工厂子类中的某一个，客户端在使用时可以无须关心是哪一个工厂子类创建产品子类，需要时再动态指定。

  [1]: https://statics.sh1a.qingstor.com/2018/09/14/factory-method.jpg
  [2]: https://statics.sh1a.qingstor.com/2018/09/14/seq-factory-method.jpg
