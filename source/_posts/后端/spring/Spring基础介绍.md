---
title: Spring基础介绍
date: 2018-09-17 22:32:00
author: blinkfox
categories: 后端
tags:
  - Spring
  - Java
---

## 一、Spring 概述

### （一）Spring 的简史

[Spring][1] 的历史网上有很多的介绍，下面是 Spring 发展历程的一个简介。

#### 1. 第一阶段：xml 配置

在 Spring 1.x 时代，使用 Spring 开发满眼都是 xml 配置的 Bean，随着项目的扩大，我们需要把 xml 配置文件放到不同的配置文件里，那时候需要频繁地在开发的类和配置文件之间切换。

#### 2. 第二阶段：注解配置

在 Spring 2.x 时代，随着 JDK 1.5 带来的注解支持，Spring 提供了声明 Bean 的注解（如：@Component、@Service），大大减少了配置量。这时 Spring 圈子里存在着一种争论：注解配置和 xml 配置究竟哪个更好？我们最终的选择是应用的基本配置（如：数据库配置）用 xml，业务配置用注解。

#### 3. 第三阶段：Java 配置

从 Spring 3.x 到现在，Spring 提供了 Java 配置的能力，使用 Java 配置可以让你更理解你配置的 Bean。我们目前刚好处于这个时代，Spring 4.x 和 Spring Boot 都推荐使用 Java 配置。

### （二）Spring 概述

Spring 框架是一个轻量级的企业级开发的一站式解决方案。所谓解决方案就是可以基于 Spring 解决 JavaEE 开发的所有问题。Spring 框架主要提供了`IoC`容器、AOP、数据访问、Web 开发、消息、测试等相关技术的支持。

Spring 使用简单的 POJO（`Plain Old Java Object`，即无任何限制的普通Java对象）来进行企业级开发。每一个被 Spring 管理的 Java 对象都被称之为 Bean；而 Spring 提供了一个 IoC 容器用来初始化对象，解决对象间的依赖管理和对象的使用。

#### 1. Spring 的模块

Spring 是模块化的，这意味着你可以只使用你需要的Spring的模块。如下图所示：

![Spring 模块][2]

图中的每个最小单元，Spring 都至少有一个对应的 jar 包。

##### （1）核心容器（Core Contariner）

- Spring-Core：核心工具类，Spring 其他模块大量使用 Spring-Core
- Spring-Beans：Spring 定义 Bean 的支持
- Spring-Context：运行时 Spring 容器
- Spring-Context-Support：容器对第三方包的集成支持
- Spring-Expression：使用表达式语言在运行时查询和操作对象

##### （2）AOP

- Spring-AOP：基于代理的 AOP 支持
- Spring-Aspects：基于 AspectJ 的 AOP 支持

##### （3）消息（Messaging）

- Spring-Messaging：对消息架构和协议的支持

##### （4）Web

- Spring-Web：提供基础的 Web 集成的功能，在 Web 项目中提供 Spring 的容器
- Spring-Webmvc：提供基于 Servlet 的 Spring MVC
- Spring-WebSocket：提供 WebSocket 功能
- Spring-Webmvc-Portlet：提供 Portlet 环境功能

##### （5）数据访问/集成（Data Access/Integration）

- Spring-JDBC：提供以 JDBC 访问数据库的支持
- Spring-TX：提供编程式和声明式的事务支持
- Spring-ORM：提供对对象/关系映射技术的支持
- Spring-OXM：提供对对象/xml 映射技术的支持
- Spring-JMS：提供对 JMS 的支持

#### 1. Spring 的生态

Spring 发展到现在已经不仅仅是 Spring 框架本身的内容，Spring 目前提供了大量的基于 Spring 的项目，可以用来更深入地降低我们的开发难度，提高开发效率。
目前 Spring 的生态里主要有以下项目，我们可以根据自己项目的需要来选择使用相应的项目。

- Spring Boot：使用默认开发配置来实现快速开发
- Spring XD：用来简化大数据应用开发
- Spring Cloud：为分布式系统开发提供工具集
- Spring Data：对主流关系型和 NoSQL 数据库的支持
- Spring Integration：通过消息机制对企业集成模式（EIP）的支持
- Spring Batch：简化及优化大量数据的批处理操作
- Spring Security：通过认证和授权保护应用
- Spring HATEOAS：基于 HATEOAS 原则简化 REST 服务开发
- Spring Social：与社交网络 API（如：Facebook、新浪微博等）的集成
- Spring AMQP：对基于 AMQP 的消息的支持
- Spring Mobile：提供对手机设备检测的功能，给不同的设备返回不同的页面的功能
- Spring for Android：主要提供在 Android 上消费 RESTful API 的功能
- Spring Web Flow：基于 SpringMVC 提供基于向导流程式的 Web 应用开发
- Spring Web Services：提供了基于协议有限的 SOAP/Web 服务
- Spring LDAP：简化使用 LDAP 开发
- Spring Session：提供一个 API 及实现来管理用户会话信息

## 二、Spring 项目快速搭建

这里我们使用目前 Java 主流的项目构建工具[Maven][3]来搭建项目。

### （一）Maven 介绍

Apache Maven 是一个基于项目对象模型（Project Object Model，POM）的软件项目管理工具。Maven 可用来管理项目的依赖、编译、打包、文档等信息。使用 Maven 来管理项目时，项目依赖的 jar 包将不再包含在项目内，而是集中放置在用户目录下的 .m2 文件夹下。关于 Maven 的详细安装介绍可参考[这里][4]。

### （二）创建项目

在创建项目之前，须确保你的计算机上已经安装好有 Java 和 Maven 环境。然后，打开终端通过以下简单的命令就可以在你的当前目录下创建一个 Jave web 的项目结构：

```bash
mvn archetype:generate -DgroupId=com.blinkfox -DartifactId=springdemo -DpackageName=com.blinkfox.springdemo -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false
```

其中`-DgroupId=com.blinkfox`是组织名，`-DartifactId=springdemo`是该组织下的项目名称，`-DarchetypeArtifactId=maven-archetype-webapp`代表创建一个简单的 webapp 项目。

创建项目的时候，Maven会自动下载一些需要用到的 jar 包和 Maven 插件。如果顺利创建成功的话，就会在你的当前目录下看到名为 springdemo 的项目，其中包含`src`的文件夹和`pom.xml`文件。且在你的终端会看到如下输出：

![Maven创建项目成功][5]

### （三）添加 Spring 依赖

接下来需要通过修改 pom.xml 来添加 Spring 的依赖，添加编译插件，且将编译级别设置为1.7，pom.xml文件的修改如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.blinkfox</groupId>
    <artifactId>springdemo</artifactId>
    <packaging>war</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>springdemo Maven Webapp</name>
    <url>http://maven.apache.org</url>

    <properties>
        <java.version>1.7</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.3.3.RELEASE</version>
        </dependency>
    </dependencies>
    <build>
        <finalName>springdemo</finalName>
        <!-- 指定maven的默认操作为 -->
        <defaultGoal>compile</defaultGoal>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## 三、Spring 基础配置

Spring 框架本身有四大原则：

- 使用 POJO 进行轻量级和最小侵入式开发
- 通过依赖注入和面向接口编程来实现松耦合
- 通过 AOP 和默认习惯进行声明式编程
- 使用 AOP 和模板(template)减少模式化代码

Spring 的所有功能设计和实现都是基于此四大原则。

### （一）依赖注入

#### 1. 重点说明

我们经常说的控制反转（Inversion of Control，IoC）和依赖注入（dependency injection，DI）在 Spring 环境下是等同的概念，控制反转是通过依赖注入实现的。所谓依赖注入指的是容器负责创建对象和维护对象间的依赖关系，而不是通过对象本身负责自己的创建和解决自己的依赖。

依赖注入的主要目的是为了解耦，体现了一种“组合”的理念。如果你希望你的类具备某项功能的时候，是继承自一个具有此功能的父类好呢？还是组合另外一个具有这个功能的类好呢？答案是不言而喻的，继承一个父类，之类将与父类耦合，组合另外一个类则使耦合度大大降低。

Spring IoC 容器（ApplicationContext）负责创建 Bean，并通过容器将功能类 Bean 注入到你需要的 Bean 中。Spring 提供使用 xml、注解、Java 配置、groovy 配置实现 Bean 的创建和注入。

无论是 xml 配置、注解配置还是 Java 配置，都被称为配置元数据，所谓元数据即描述数据的数据。元数据本身不具备任何可执行的能力，只能通过外界代码来对这些元数据行解析后进行一些有意义操作。Spring 容器解析这些配置元数据进行 Bean 初始化、配置和管理依赖。

声明 Bean 的注解：

- `@Component`: 组件，没有明确角色
- `@Controller`: 在展现层（MVC -> Spring MVC）使用
- `@Service`: 在业务逻辑层（service层）使用
- `@Repository`: 在数据访问层（dao层）使用

注入 Bean 的注解，一般情况下通用：

- `@Autowired`: Spring 提供的注解
- `@Inject`: JSR-330 提供的注解
- `@Resource`: JSR-250 提供的注解

`@Autowired`、`@Inject`、`@Resource`可注解在 set 方法上或者属性上，推荐注解在属性上，优点是代码更少、层次更清晰。

#### 2. 代码示例

（1）编写功能类的 Bean。

```java
package com.blinkfox.service.impl;

import org.springframework.stereotype.Service;

/**
 * Created by blinkfox on 2016/10/27.
 */
@Service
public class FunctionService {

    public String sayHello(String word) {
        return "Hello " + word + "!";
    }

}
```

> **代码解释**：
> 1. 使用 @Service 注解声明当前 FunctionService 类是 Spring 管理的一个 Bean。其中，使用 @Component、@Service、@Repository、@Controller 是等效的，可根据需要选用。

（2）使用功能类的 Bean。

```java
package com.blinkfox.service.impl;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * Created by blinkfox on 2016/10/27.
 */
@Service
public class UseFunctionService {

    @Autowired
    private FunctionService functionService;


    public String sayHello(String word) {
        return functionService.sayHello(word);
    }

}
```

> **代码解释**：
> 1. 使用 @Service 注解声明当前 UseFunctionService 类是 Spring 管理的一个 Bean。
> 2. 使用 @Autowired 将 FunctionService 的实体 Bean 注入到 UseFunctionService 中，让 UseFunctionService 具备 FunctionService 的功能，此处使用 JSR-330 的 @Inject 注解或者 JSR-250 的 @Resource 注解是等效的。

（3）配置类。

```java
package com.blinkfox.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
 * Created by blinkfox on 2016/10/27.
 */
@Configuration
@ComponentScan("com.blinkfox.service.impl")
public class DiConfig {
}
```

> **代码解释**：
> 1. 使用 @Configuration 注解声明当前类是一个配置类。
> 2. 使用 @ComponentScan 将 自动扫描包名下所有使用的 @Component、@Service、@Repository、@Controller 类，并注册为 Bean。

（4）运行。

```java
package com.blinkfox.maintest;

import com.blinkfox.config.DiConfig;
import com.blinkfox.service.impl.UseFunctionService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by blinkfox on 2016/10/27.
 */
public class FunctionMain {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(DiConfig.class);
        UseFunctionService useFunctionService = context.getBean(UseFunctionService.class);
        System.out.println(useFunctionService.sayHello("Spring"));
        context.close();
    }

}
```

> **代码解释**：
> 1. 使用 AnnotationConfigApplicationContext 作为 Spring 容器，接收使用一个配置类作为参数。
> 2. 获得声明配置的 UseFunctionService 的 Bean。

### （二）Java 配置

#### 1. 重点说明

Java 配置是 Spring4.x 推荐的配置方式，可以完全替代 xml 配置；Java 配置也是 Spring Boot 推荐的配置方式。

Java 配置是通过 @Configuration 和 @Bean 来实现的。

- @Configuration 声明当前类是一个配置类，相当于一个Spring配置的 xml 文件。
- @Bean 注解在方法上，声明当前方法的返回值是一个 Bean。

何时使用 Java 配置或者注解配置呢？我们主要的原则是：全局配置使用 Java 配置（如数据库相关配置、MVC相关配置），业务 Bean 的配置使用注解配置（@Service、@Component、@Repository、@Controller）。

#### 2. Java配置代码示例

（1）编写功能类的 Bean

```java
package com.blinkfox.service.impl;

/**
 * Created by blinkfox on 2016/10/27.
 */
// 1
public class JavaConfigService {

    public String sayHello(String word) {
        return "Hello " + word + "!";
    }

}
```

> **代码解释**：
> 1. 此处没有使用 @Service 声明 Bean。

（2）使用功能类的 Bean

```java
package com.blinkfox.service.impl;

/**
 * Created by blinkfox on 2016/10/27.
 */
// 1
public class UseJavaConfigService {
    // 2
    private JavaConfigService javaConfigService;

    public void setJavaConfigService(JavaConfigService javaConfigService) {
        this.javaConfigService = javaConfigService;
    }

    public String sayHello(String word) {
        return javaConfigService.sayHello(word);
    }

}
```

> **代码解释**：
> 1. 此处没有使用 @Service 声明 Bean。
> 2. 此处没有使用 @Autowired 注解注入 Bean。

（3）Java 配置类

```java
package com.blinkfox.config;

import com.blinkfox.service.impl.JavaConfigService;
import com.blinkfox.service.impl.UseJavaConfigService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Created by blinkfox on 2016/10/27.
 */
@Configuration // 1
public class JavaConfig {

    @Bean  // 2
    public JavaConfigService javaConfigService() {
        return new JavaConfigService();
    }

    @Bean
    public UseJavaConfigService useJavaConfigService() {
        UseJavaConfigService useJavaConfigService = new UseJavaConfigService();
        useJavaConfigService.setJavaConfigService(javaConfigService()); // 3
        return useJavaConfigService;
    }

}
```

> **代码解释**：
> 1. 使用 @Configuration 注解表明当前类是一个配置类，这意味着这个类型里可能有0个或者多个 @Bean 注解，此处没有使用包扫描，是因为所有的 Bean 都在此类中定义了。
> 2. 使用 @Bean 注解声明当前方法 JavaConfigService 的返回值是一个 Bean，Bean的名称是方法名。
> 3. 注入 FunctionService 的 Bean 时候直接调用 javaConfigService()。

（4）运行

```java
package com.blinkfox.maintest;

import com.blinkfox.config.JavaConfig;
import com.blinkfox.service.impl.UseJavaConfigService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by blinkfox on 2016/10/27.
 */
public class JavaConfigMain {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(JavaConfig.class);
        UseJavaConfigService useJavaConfigService = context.getBean(UseJavaConfigService.class);
        System.out.println(useJavaConfigService.sayHello("Spring Java Config"));
        context.close();
    }

}
```

### （三）AOP

#### 1. 重点说明

AOP：面向切面编程，是面向对象编程（OOP）的补充。

Spring 的 AOP 的存在目的是为了解耦。AOP 可以让一组类共享相同的行为。在 OOP 中只能通过继承和实现接口来共享相同的行为，从而使代码的耦合度增强，且类继承只能为单继承，阻碍更多行为添加到一组类上，AOP 弥补了 OOP 的不足。

Spring 支持 AspectJ 的注解式切面编程。

- 使用 @AspectJ 声明是一个切面。
- 使用 @After、@Before、Around 定义通知（advice）类型，可直接将拦截规则（切点）作为参数。
- 其中 @After、@Before、Around 参数的拦截规则为切点（PointCut），为了使切点复用，可使用 @PointCut 专门定义拦截规则，然后在 @After、@Before、Around 的参数中调用。
- 其中符合条件的每一个拦截处为连接点（JoinPoint）。

Spring本身在事务处理（@Transcational）和数据缓存（@Cacheable）等都使用注解拦截。下面示例将演示基于注解和方法规则的拦截方式，演示一种模拟记录操作的日志系统的实现。

#### 2. 注解拦截代码示例

（1）添加 Spring aop 支持及 AspectJ 依赖。

```java
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-aop</artifactId>
	<version>4.3.3.RELEASE</version>
</dependency>
<dependency>
	<groupId>org.aspectj</groupId>
	<artifactId>aspectjrt</artifactId>
	<version>1.8.9</version>
</dependency>
<dependency>
	<groupId>org.aspectj</groupId>
	<artifactId>aspectjweaver</artifactId>
	<version>1.8.9</version>
</dependency>
```
（2）编写拦截规则的注解。

```java
package com.blinkfox.annotation;

import java.lang.annotation.*;

/**
 * Created by blinkfox on 2016/10/29.
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface LogAction {

    String name() default "这是默认的操作名称";

}
```

> **代码解释**：
> 注解本身是没有功能的，就和 xml 一样。注解和 xml 都是一种元数据，元数据即解释数据的数据，这就是所谓的配置。注解的功能来自用这个注解的地方。

（3）编写使用注解的被拦截类。

```java
package com.blinkfox.service.impl;

import com.blinkfox.annotation.LogAction;
import org.springframework.stereotype.Service;

/**
 * Created by blinkfox on 2016/10/29.
 */
@Service
public class DemoAnnotationService {

    @LogAction(name = "注解式拦截的 add 操作")
    public void add() {

    }

}
```

（4）编写使用方法规则被拦截规类。

```java
package com.blinkfox.service.impl;

import org.springframework.stereotype.Service;

/**
 * Created by blinkfox on 2016/10/29.
 */
@Service
public class DemoMethodService {

    public void add() {

    }

}
```

（5）编写切面。

```java
package com.blinkfox.aop;

import com.blinkfox.annotation.LogAction;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;

import java.lang.reflect.Method;

/**
 * Created by blinkfox on 2016/10/29.
 */
@Aspect // 1
@Component // 2
public class LogAspect {

    @Pointcut("@annotation(com.blinkfox.annotation.LogAction)") // 3
    public void annotationPointCut() {

    }

    @After("annotationPointCut()") // 4
    public void after(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        LogAction logAction = method.getAnnotation(LogAction.class);
        System.out.println("---注解式拦截:" + logAction.name()); // 5
    }

    @After("execution(* com.blinkfox.service.impl.DemoMethodService.*(..))") // 6
    public void before(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        System.out.println("---方法规则式拦截:" + method.getName());
    }

}
```

> **代码解释**：
> 1. 通过 @Aspect 注解声明一个切面。
> 2. 通过 @Component 让此切面成为 Spring 容器管理的Bean。
> 3. 通过 @PointCut 注解声明切点。
> 4. 通过 @After 注解声明一个通知类型，并使用 @PointCut定义的切点。
> 5. 通过可获得注解上的属性，然后做日志记录相关的操作，下面相同。
> 6. 通过 @Before 注解声明一个通知类型，此通知直接使用拦截规则作为参数。

（6）配置类。

```java
package com.blinkfox.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

/**
 * Created by blinkfox on 2016/10/29.
 */
@Configuration
@ComponentScan("com.blinkfox")
@EnableAspectJAutoProxy
public class AopConfig {

}
```

> **代码解释**：
> 1. 使用 @EnableAspectJAutoProxy 注解开启 Spring 对 AspectJ的支持。

（6）运行。

```java
package com.blinkfox.maintest;

import com.blinkfox.config.AopConfig;
import com.blinkfox.service.impl.DemoAnnotationService;
import com.blinkfox.service.impl.DemoMethodService;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Created by blinkfox on 2016/10/29.
 */
public class AopMain {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext(AopConfig.class);
        DemoAnnotationService demoAnnotationService = context.getBean(DemoAnnotationService.class);
        DemoMethodService demoMethodService = context.getBean(DemoMethodService.class);
        demoAnnotationService.add();
        demoMethodService.add();
        context.close();
    }

}
```

  [1]: https://spring.io/
  [2]: https://statics.sh1a.qingstor.com/2019/03/02/spring-moudle.png
  [3]: http://maven.apache.org/
  [5]: https://statics.sh1a.qingstor.com/2019/03/02/maven-build-project.png
