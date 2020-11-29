---
title: Java8新特性及使用(二)
date: 2018-11-14 22:00:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/11/13/java8-2.jpg
categories: 后端
tags:
  - Java
---

## 扩展注解的支持

Java 8扩展了注解的上下文。**现在几乎可以为任何东西添加注解：局部变量、泛型类、父类与接口的实现，就连方法的异常也能添加注解**。下面演示几个例子：

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.util.ArrayList;
import java.util.Collection;

public class Annotations {

    @Retention(RetentionPolicy.RUNTIME)
    @Target({ ElementType.TYPE_USE, ElementType.TYPE_PARAMETER })
    public @interface NonEmpty {
    }

    public static class Holder<@NonEmpty T> extends @NonEmpty Object {
        public void method() throws @NonEmpty Exception {
        }
    }

    @SuppressWarnings("unused")
    public static void main(String[] args) {
        final Holder<String> holder = new @NonEmpty Holder<String>();
        @NonEmpty Collection<@NonEmpty String> strings = new ArrayList<>();
    }

}
```

## Base64

在Java 8中，Base64编码已经成为Java类库的标准。它的使用十分简单，下面让我们看一个例子：

```java
import java.nio.charset.StandardCharsets;
import java.util.Base64;

public class Base64s {

    public static void main(String[] args) {
        final String text = "Base64 finally in Java 8!";

        final String encoded = Base64.getEncoder().encodeToString(text.getBytes(StandardCharsets.UTF_8));
        System.out.println(encoded);

        final String decoded = new String(Base64.getDecoder().decode(encoded), StandardCharsets.UTF_8);
        System.out.println(decoded);
    }

}
```

程序在控制台上输出了编码后的字符与解码后的字符：

```bash
QmFzZTY0IGZpbmFsbHkgaW4gSmF2YSA4IQ==
Base64 finally in Java 8!
```

Base64类同时还提供了对URL、MIME友好的编码器与解码器（`Base64.getUrlEncoder() / Base64.getUrlDecoder()`, `Base64.getMimeEncoder() / Base64.getMimeDecoder()`）。

## JavaFX

`JavaFX`是一个强大的图形和多媒体处理工具包集合，它允许开发者来设计、创建、测试、调试和部署富客户端程序，并且和Java一样跨平台。从Java8开始，JavaFx已经内置到了JDK中。关于JavaFx更详细的文档可参考[JavaFX中文文档](http://www.javafxchina.net/blog/docs/)。

## 其它

### 1. JDBC4.2规范

JDBC4.2主要有以下几点改动：

- 增加了对`REF Cursor`的支持
- 修改返回值大小范围（update count）
- 增加了`java.sql.DriverAction`接口
- 增加了`java.sql.SQLType`接口
- 增加了`java.sql.JDBCtype`枚举
- 对`java.time`包时间类型的支持

### 2. 更好的类型推测机制

Java 8在类型推测方面有了很大的提高。在很多情况下，编译器可以推测出确定的参数类型，这样就能使代码更整洁。让我们看一个例子：

```java
public class Value<T> {

    public static<T> T defaultValue() {
        return null;
    }

    public T getOrDefault(T value, T defaultValue) {
        return (value != null) ? value : defaultValue;
    }

}
```

这里是`Value<String>`类型的用法。

```java
public class TypeInference {

    public static void main(String[] args) {
        final Value<String> value = new Value<>();
        value.getOrDefault("22", Value.defaultValue());
    }

}
```

`Value.defaultValue()`的参数类型可以被推测出，所以就不必明确给出。在Java 7中，相同的例子将不会通过编译，正确的书写方式是`Value.<String>defaultValue()`。

### 3. HashMap性能提升

Java8中，HashMap内部实现又引入了红黑树，使得HashMap的总体性能相较于Java7有比较明显的提升。以下是对Hash均匀和不均匀的情况下的性能对比

#### (1). Hash较均匀的情况

![Hash较均匀时的性能对比](https://images2017.cnblogs.com/blog/647994/201801/647994-20180105204924753-361068557.png)

#### (2). Hash极不均匀的情况

![Hash极不均匀时的性能对比](https://images2017.cnblogs.com/blog/647994/201801/647994-20180105205031643-1765887276.png)

### 4. IO/NIO 的改进

Java8 对`IO/NIO`也做了一些改进。主要包括：改进了`java.nio.charset.Charset`的实现，使编码和解码的效率得以提升，也精简了`jre/lib/charsets.jar`包；优化了`String(byte[], *)`构造方法和`String.getBytes()`方法的性能；还增加了一些新的`IO/NIO`方法，使用这些方法可以从文件或者输入流中获取流（`java.util.stream.Stream`），通过对流的操作，可以简化文本行处理、目录遍历和文件查找。

新增的 API 如下：

- `BufferedReader.line()`: 返回文本行的流`Stream<String>`
- `File.lines(Path, Charset)`: 返回文本行的流`Stream<String>`
- `File.list(Path)`: 遍历当前目录下的文件和目录
- `File.walk(Path, int, FileVisitOption)`: 遍历某一个目录下的所有文件和指定深度的子目录
- `File.find(Path, int, BiPredicate, FileVisitOption...)`: 查找相应的文件

下面就是用流式操作列出当前目录下的所有文件和目录：

```java
Files.list(new File(".").toPath()).forEach(System.out::println);
```

### 5. JavaScript引擎Nashorn

Java 8提供了一个新的`Nashorn javascript`引擎，它允许我们在JVM上运行特定的javascript应用。Nashorn javascript引擎只是`javax.script.ScriptEngine`另一个实现，而且规则也一样，允许Java和JavaScript互相操作。这里有个小例子：

```java
ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName("JavaScript");

System.out.println(engine.getClass().getName());
System.out.println("Result:" + engine.eval("function f(){return 1;}; f() + 1;"));
```

输出如下：

```bash
jdk.nashorn.api.scripting.NashornScriptEngine
Result: 2
```

### 6. 并发（Concurrency）

在新增`Stream`机制与`Lambda`的基础之上，在`java.util.concurrent.ConcurrentHashMap`中加入了一些新方法来支持聚集操作。同时也在`java.util.concurrent.ForkJoinPool`类中加入了一些新方法来支持共有资源池（common pool）（请查看我们关于Java 并发的免费课程）。

新增的`java.util.concurrent.locks.StampedLock`类提供一直基于容量的锁，这种锁有三个模型来控制读写操作（它被认为是不太有名的`java.util.concurrent.locks.ReadWriteLock`类的替代者）。

在`java.util.concurrent.atomic`包中还增加了下面这些类：

- DoubleAccumulator
- DoubleAdder
- LongAccumulator
- LongAdder

### 7. 类依赖分析器jdeps

`Jdeps`是一个功能强大的命令行工具，它可以帮我们显示出包层级或者类层级java类文件的依赖关系。它接受class文件、目录、jar文件作为输入，默认情况下，`jdeps`会输出到控制台。

作为例子，让我们看看现在很流行的Spring框架的库的依赖关系报告。为了让报告短一些，我们只分析一个jar: `org.springframework.core-3.0.5.RELEASE.jar`.

`jdeps org.springframework.core-3.0.5.RELEASE.jar`这个命令输出内容很多，我们只看其中的一部分，这些依赖关系根绝包来分组，如果依赖关系在classpath里找不到，就会显示not found.

```bash
C:\Program Files\Java\jdk1.8.0\jre\lib\rt.jar
   org.springframework.core (org.springframework.core-3.0.5.RELEASE.jar)
      -> java.io
      -> java.lang
      -> java.lang.annotation
      -> java.lang.ref
      -> java.lang.reflect
      -> java.util
      -> java.util.concurrent
      -> org.apache.commons.logging                         not found
      -> org.springframework.asm                            not found
      -> org.springframework.asm.commons                    not found
   org.springframework.core.annotation (org.springframework.core-3.0.5.RELEASE.jar)
      -> java.lang
      -> java.lang.annotation
      -> java.lang.reflect
      -> java.util
```

### 8. JVM的PermGen空间被移除

`PermGen`空间被移除了，取而代之的是`Metaspace（JEP 122）`。JVM选项`-XX:PermSize`与`-XX:MaxPermSize`分别被`-XX:MetaSpaceSize`与`-XX:MaxMetaspaceSize`所代替。

---

参考文档：

- [What's New in JDK 8](http://www.oracle.com/technetwork/java/javase/8-whats-new-2157071.html)
- [Java 8新特性终极指南](http://www.importnew.com/11908.html)