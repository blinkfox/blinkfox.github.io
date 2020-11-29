---
title: Java7新特性及使用
date: 2018-11-12 00:30:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/11/12/coffee.jpg
categories: 后端
tags:
  - Java
---

## 新特性列表

以下是Java7中的引入的部分新特性。关于Java7更详细的介绍可参考[这里](http://www.oracle.com/technetwork/java/javase/jdk7-relnotes-418459.html)。

- switch支持String
- try-with-resources
- catch多个异常
- 实例创建类型推断
- 数字字面量下划线分割
- 二进制字面量
- 增强的文件系统
- Fork/Join框架
- 其它
  - JDBC4.1规范
  - 支持动态类型语言
  - JSR341-Expression Language Specification
  - JSR203-More New I/O APIs for the Java Platform
  - 桌面客户端增强

## 一、switch支持String

`switch`现在可以接受`String`类型的参数。示例代码如下：

```java
String s = ...
switch(s) {
case "quux":
    processQuux(s);
// fall-through
case "foo":
case "bar":
    processFooOrBar(s);
    break;
case "baz":
    processBaz(s);
    // fall-through
default:
    processDefault(s);
    break;
}
```

## 二、try-with-resources

Java中某些资源是需要手动关闭的，如`InputStream`，`Writer`，`Sockets`，`Connection`等。这个新的语言特性允许try语句本身申请更多的资源，这些资源作用于try代码块，并自动关闭。

Java7之前的写法：

```java
BufferedReader br = null;
try {
    br = new BufferedReader(new FileReader(path));
    return br.readLine();
} catch (Exception e) {
    log.error("BufferedReader Exception", e);
} finally {
    if (br != null) {
        try {
            br.close();
        } catch (Exception e) {
            log.error("BufferedReader close Exception", e);
        }
    }
}
```

Java7及之后的写法：

```java
try (BufferedReader br = new BufferedReader(new FileReader(path)) {
    return br.readLine();
} catch (Exception e) {
    log.error("BufferedReader Exception", e);
}
```

## 三、catch多个异常

自Java7开始，`catch`中可以一次性捕捉多个异常做统一处理。示例如下：

Java7之前的写法：

```java
public void handle() {
    ExceptionThrower thrower = new ExceptionThrower();
    try {
        thrower.manyExceptions();
    } catch (ExceptionA a) {
        System.out.println(a.getClass());
    } catch (ExceptionB b) {
        System.out.println(b.getClass());
    } catch (ExceptionC c) {
        System.out.println(c.getClass());
    }
}
```

Java7及之后的写法：

```java
public void handle() {
    ExceptionThrower thrower = new ExceptionThrower();
    try {
        thrower.manyExceptions();
    } catch (ExceptionA | ExceptionB ab) {
        System.out.println(ab.getClass());
    } catch (ExceptionC c) {
        System.out.println(c.getClass());
    }
}
```

## 四、实例创建类型推断

从Java7开始，泛型类的实例化也不用繁琐的将泛型声明再写一遍。示例如下：

Java7之前的写法：

```java
Map<String, List<String>> map = new HashMap<String, List<String>>();
```

Java7及之后的写法：

```java
Map<String, List<String>> map = new HashMap<>();
```

## 五、数字字面量下划线分割

很长的数字可读性不好，在Java 7中可以使用下划线分隔长`int`以及`long`型整数了。如：

```java
long creditCardNumber = 1234_5678_9012_3456L;
public static final int ONE_MILLION = 1_000_000;
public static final float PI = 3.14_15F;
```

## 六、二进制字面量

现在可以使用0b前缀创建二进制字面量：

```java
int binary = 0b1001_1001;
```

使用二进制字面量这种表示方式，使用非常简短的代码就可将二进制字符转换为数据类型，如在`byte`或`short`。

```java
byte aByte = (byte) 0b001;
short aShort = (short) 0b010;
```

## 七、增强的文件系统

Java7 推出了全新的`NIO2.0 API`以此改变针对文件管理的不便，使得在`java.nio.file`包下使用`Path`、`Paths`、`Files`、`WatchService`、`FileSystem`等常用类型可以很好的简化开发人员对文件管理的编码工作。

### 1. Path接口和Paths类

`Path`接口的某些功能其实可以和`java.io`包下的`File`类等价，当然这些功能仅限于只读操作。在实际开发过程中，开发人员可以联用`Path`接口和`Paths`类，从而获取文件的一系列上下文信息。

- `int getNameCount()`: 获取当前文件节点数
- `Path getFileName()`: 获取当前文件名称
- `Path getRoot()`: 获取当前文件根目录
- `Path getParent()`: 获取当前文件上级关联目录

联用`Path`接口和`Paths`类型获取文件信息：

```java
Path path = Paths.get("G:/test/test.xml");
System.out.println("文件节点数:" + path.getNameCount());
System.out.println("文件名称:" + path.getFileName());
System.out.println("文件根目录:" + path.getRoot());
System.out.println("文件上级关联目录:" + path.getParent());
```

### 2. Files类

联用`Path`接口和`Paths`类可以很方便的访问到目标文件的上下文信息。当然这些操作全都是只读的，如果开发人员想对文件进行其它非只读操作，比如文件的创建、修改、删除等操作，则可以使用`Files`类型进行操作。

Files类型常用方法如下：

- `Path createFile()`: 在指定的目标目录创建新文件
- `void delete()`: 删除指定目标路径的文件或文件夹
- `Path copy()`: 将指定目标路径的文件拷贝到另一个文件中
- `Path move()`: 将指定目标路径的文件转移到其他路径下，并删除源文件

使用`Files`类型复制、粘贴文件示例：

```java
Files.copy(Paths.get("/test/src.xml"), Paths.get("/test/target.xml"));
```

使用`Files`类型来管理文件，相对于传统的I/O方式来说更加方便和简单。因为具体的操作实现将全部移交给`NIO2.0 API`，开发人员则无需关注。

### 3. WatchService

Java7 还为开发人员提供了一套全新的文件系统功能，那就是文件监测。在此或许有很多朋友并不知晓文件监测有何意义及目，那么请大家回想下调试成热发布功能后的Web容器。当项目迭代后并重新部署时，开发人员无需对其进行手动重启，因为Web容器一旦监测到文件发生改变后，便会自动去适应这些“变化”并重新进行内部装载。Web容器的热发布功能同样也是基于文件监测功能，所以不得不承认，文件监测功能的出现对于Java文件系统来说是具有重大意义的。

文件监测是基于事件驱动的，事件触发是作为监测的先决条件。开发人员可以使用`java.nio.file`包下的`StandardWatchEventKinds`类型提供的3种字面常量来定义监测事件类型，值得注意的是监测事件需要和`WatchService`实例一起进行注册。

`StandardWatchEventKinds`类型提供的监测事件：

- `ENTRY_CREATE`：文件或文件夹新建事件；
- `ENTRY_DELETE`：文件或文件夹删除事件；
- `ENTRY_MODIFY`：文件或文件夹粘贴事件；

使用`WatchService`类实现文件监控完整示例：

```java
public static void testWatch() {
    /* 监控目标路径 */
    Path path = Paths.get("G:/");
    try {
        /* 创建文件监控对象. */
        WatchService watchService = FileSystems.getDefault().newWatchService();

        /* 注册文件监控的所有事件类型. */
        path.register(watchService, StandardWatchEventKinds.ENTRY_CREATE, StandardWatchEventKinds.ENTRY_DELETE,
                StandardWatchEventKinds.ENTRY_MODIFY);

        /* 循环监测文件. */
        while (true) {
            WatchKey watchKey = watchService.take();

            /* 迭代触发事件的所有文件 */
            for (WatchEvent<?> event : watchKey.pollEvents()) {
                System.out.println(event.context().toString() + " 事件类型：" + event.kind());
            }

            if (!watchKey.reset()) {
                return;
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

通过上述程序示例我们可以看出，使用`WatchService`接口进行文件监控非常简单和方便。首先我们需要定义好目标监控路径，然后调用`FileSystems`类型的`newWatchService()`方法创建`WatchService`对象。接下来我们还需使用`Path`接口的`register()`方法注册`WatchService`实例及监控事件。当这些基础作业层全部准备好后，我们再编写外围实时监测循环。最后迭代`WatchKey`来获取所有触发监控事件的文件即可。

## 八、Fork/Join框架

### 1. 什么是Fork/Join框架

Java7提供的一个用于并行执行任务的框架，是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。

Fork/Join的运行流程图如下：

![Fork/Join的运行流程图](https://res.infoq.com/articles/fork-join-introduction/zh/resources/21.png)

### 2. 工作窃取算法

工作窃取（work-stealing）算法是指某个线程从其他队列里窃取任务来执行。工作窃取的运行流程图如下：

![工作窃取的运行流程图](https://res.infoq.com/articles/fork-join-introduction/zh/resources/image3.png)

工作窃取算法的优点是充分利用线程进行并行计算，并减少了线程间的竞争，其缺点是在某些情况下还是存在竞争，比如双端队列里只有一个任务时。并且消耗了更多的系统资源，比如创建多个线程和多个双端队列。

### 3. Fork/Join框架的介绍

设计一个Fork/Join框架，主要有以下两步骤：

第一步分割任务。首先我们需要有一个fork类来把大任务分割成子任务，有可能子任务还是很大，所以还需要不停的分割，直到分割出的子任务足够小。

第二步执行任务并合并结果。分割的子任务分别放在双端队列里，然后几个启动线程分别从双端队列里获取任务执行。子任务执行完的结果都统一放在一个队列里，启动一个线程从队列里拿数据，然后合并这些数据。

Fork/Join使用两个类来完成以上两件事情：

ForkJoinTask：我们要使用ForkJoin框架，必须首先创建一个ForkJoin任务。它提供在任务中执行fork()和join()操作的机制，通常情况下我们不需要直接继承ForkJoinTask类，而只需要继承它的子类，Fork/Join框架提供了以下两个子类：
RecursiveAction：用于没有返回结果的任务。
RecursiveTask ：用于有返回结果的任务。
ForkJoinPool ：ForkJoinTask需要通过ForkJoinPool来执行，任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任务。

### 4. Fork/Join框架使用示例

让我们通过一个简单的需求来使用下`Fork／Join`框架，需求是：计算`1 + 2 + 3 + 4`的结果。

使用`Fork/Join`框架首先要考虑到的是如何分割任务，如果我们希望每个子任务最多执行两个数的相加，那么我们设置分割的阈值是`2`，由于是`4`个数字相加，所以`Fork/Join`框架会把这个任务`fork`成两个子任务，子任务一负责计算`1 + 2`，子任务二负责计算`3 + 4`，然后再`join`两个子任务的结果。

因为是有结果的任务，所以必须继承`RecursiveTask`，实现代码如下：

```java
package com.blinkfox.test.other;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.Future;
import java.util.concurrent.RecursiveTask;

/**
 * CountTask.
 *
 * @author blinkfox on 2018-01-03.
 */
public class CountTask extends RecursiveTask<Integer> {

    /** 阈值. */
    public static final int THRESHOLD = 2;

    /** 计算的开始值. */
    private int start;

    /** 计算的结束值. */
    private int end;

    /**
     * 构造方法.
     *
     * @param start 计算的开始值
     * @param end 计算的结束值
     */
    public CountTask(int start, int end) {
        this.start = start;
        this.end = end;
    }

    /**
     * 执行计算的方法.
     *
     * @return int型结果
     */
    @Override
    protected Integer compute() {
        int sum = 0;

        // 如果任务足够小就计算任务.
        if ((end - start) <= THRESHOLD) {
            for (int i = start; i <= end; i++) {
                sum += i;
            }
        } else {
            // 如果任务大于阈值，就分裂成两个子任务来计算.
            int middle = (start + end) / 2;
            CountTask leftTask = new CountTask(start, middle);
            CountTask rightTask = new CountTask(middle + 1, end);

            // 等待子任务执行完，并得到结果，再合并执行结果.
            leftTask.fork();
            rightTask.fork();
            sum = leftTask.join() + rightTask.join();
        }
        return sum;
    }

    /**
     * main方法.
     *
     * @param args 数组参数
     */
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ForkJoinPool fkPool = new ForkJoinPool();
        CountTask task = new CountTask(1, 4);
        Future<Integer> result = fkPool.submit(task);
        System.out.println("result:" + result.get());
    }

}
```

## 九、其它

### 1. JDBC4.1规范

JDBC4.1主要更新了两个新特性，分别是：

#### (1). Connection，ResultSet 和 Statement 都实现了Closeable 接口

`Connection`，`ResultSet`和`Statement`都实现了`Closeable`接口，所有在`try-with-resources`语句中调用，就可以自动关闭相关资源了。

#### (2). RowSet 1.1

引入`RowSetFactory`接口和`RowSetProvider`类，可以创建JDBC driver支持的各种`Rowsets。

```java
RowSetFactory myRowSetFactory = null;
JdbcRowSet jdbcRs = null;
ResultSet rs = null;
Statement stmt = null;

try {
  myRowSetFactory = RowSetProvider.newFactory();//用缺省的RowSetFactory 实现
  jdbcRs = myRowSetFactory.createJdbcRowSet();

  //创建一个 JdbcRowSet 对象，配置数据库连接属性
  jdbcRs.setUrl("jdbc:myDriver:myAttribute");
  jdbcRs.setUsername(username);
  jdbcRs.setPassword(password);

  jdbcRs.setCommand("select ID from TEST");
  jdbcRs.execute();
}
```

`RowSetFactory`接口包括了创建不同类型的RowSet的方法：

- createCachedRowSet
- createFilteredRowSet
- createJdbcRowSet
- createJoinRowSet
- createWebRowSet

### 2. 略

---

参考文档：

- [JavaSE7 Features and Enhancements](http://www.oracle.com/technetwork/java/javase/jdk7-relnotes-418459.html)
- [Java7的新特性](https://segmentfault.com/a/1190000004417830)
- [Fork/Join框架介绍](http://www.infoq.com/cn/articles/fork-join-introduction)