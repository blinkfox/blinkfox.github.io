---
title: MVEL2.x模板指南
date: 2018-09-19 00:13:00
author: blinkfox
categories: 后端
tags:
  - Java
  - MVEL
---

## 简介

> [MVEL][1]最初作为Mike Brock创建的 Valhalla项目的表达式计算器（`expression evaluator`）。Valhalla本身是一个早期的类似 Seam 的“开箱即用”的Web 应用框架，而 Valhalla 项目现在处于休眠状态， MVEL则成为一个继续积极发展的项目。相比最初的`OGNL`、`JEXL`和`JUEL`等项目，而它具有远超它们的性能、功能和易用性 - 特别是集成方面。它不会尝试另一种JVM语言，而是着重解决嵌入式脚本的问题。关于MVEL的语法请参考[MVEL 2.x语法指南][2]

MVEL 2.0提供了一个新的，更强大的，统一的模板引擎，汇集了1.2中引入的许多模板概念。 不幸的是，1.2中的模板引擎的架构不足以用于常规维护，并且决定从头开始完全重写模板引擎。

## 一、MVEL 2.0基本模板

MVEL模板由纯文本文档中的`orb-tags`组成。 Orb标记表示引擎将在运行时计算模板的动态元素。

如果你熟悉FreeMarker，这种类型的语法将不会完全陌生。

### 1. 一个简单的模板

```java
Hello, @{person.getSex() == 'F' ? 'Ms.' : 'Mr.'} @{person.name}

This e-mail is to thank you for your interest in MVEL Templates 2.0.
```

此模板展示了可以在简单文本中嵌入表达式。当计算结果时，输出可能如下所示：

```java
Hello, Ms. Sarah Peterson

This e-mail is to thank you for your interest in MVEL Templates 2.0.
```

### 2. 转义@符号


当然，由于@符号用于表示`orb-tag`的开头，因此您可能需要对其进行转义，以防止其被编译器处理。幸运的是，只有一种情况，即当你实际上需要输出'@{'字符串在您的模板上时。

由于编译器需要@和{组合触发orb识别，你可以自由使用@符号而不转义它们。例如：

```java
Email any questions to: foo@bar.com

@{date}
@include{'disclaimer.html'}
```

但是在你需要一个@符号挨着一个orb-tag的情况下，你需要通过重复它两次来避免它：

```java
@{username}@@@{domain}
```

这是两个@转义一个符号，第三个@是标签的开始。如果你感觉这看起来太乱，你可以使用替代方法，即使用表达式标签，如下所示：

```java
@{username}@{'@'}@{domain}
```

## 二、MVEL 2.0 Orb标签

本文包含了MVEL 2.0模板引擎中所有开箱即用的orb标签。

### 1. @{} Orb表达式

@{}表达式是orb-tag的最基本形式。它包含一个对字符串求值的值表达式，并附加到输出模板中。例如：

```java
Hello, my name is @{person.name}
```

### 2. @code{} 静默代码标签

静默代码标记允许您在模板中执行MVEL表达式代码。它不返回值，并且不以任何方式影响模板的格式。

```java
@code{age = 23; name = 'John Doe'}
@{name} is @{age} years old
```
该模板将计算出：John Doe is 23 years old。

### 3. @if{}@else{} 控制流标签

@if{}和@else{}标签在MVEL模板中提供了完全的if-then-else功能。 例如：

```java
@if{foo != bar}
   Foo not a bar!
@else{bar != cat}
   Bar is not a cat!
@else{}
   Foo may be a Bar or a Cat!
@end{}
```

MVEL模板中的所有块必须用`@end{}`标签来终止，除非是`if-then-else`结构，其中`@else{}`标记表示前一个控制语句的终止。

### 4. @foreach{} Foreach迭代

foreach标签允许您在模板中迭代集合或数组。 注意：foreach的语法已经在MVEL模板2.0中改变，以使用foreach符号来标记MVEL语言本身的符号。

```java
@foreach{item : products} 
 - @{item.serialNumber}
@end{}
```

MVEL 2.0要求您指定一个迭代变量。虽然MVEL 1.2假设您没有指定别名，但由于对该默认操作有一些抱怨，因此已被删除。

### 5. 多重迭代

您可以通过逗号分隔迭代在一个foreach循环中一次性迭代多个集合：

```java
@foreach{var1 : set1, var2 : set2}
  @{var1}-@{var2}
@end{}
```

### 6. 分隔

你可以通过在`@end{}`标签中指定迭代器的文本分隔符。

```java
@foreach{item : people}@{item.name}@end{', '}
```

将返回类似这样的结果：John, Mary, Joseph。

### 7. @include{} 包含模板文件

您可以使用此标签将模板文件包含到MVEL模板中。

```java
@include{'header.mv'}

This is a test template.
```

您还可以通过在模板名称后面添加分号在include标签内执行MVEL表达式：

```java
@include {'header.mv'; title ='Foo Title'}
```

### 8. @includeNamed{} 包括一个命名模板

命名模板是已经通过TemplateRegistry预先编译并传递到运行时的模板，或者已在模板本身中声明的模板。 您只需添加：

```java
@includeNamed {'fooTemplate'}
@includeNamed {'footerTemplate'，showSomething = true}
```

你也可以在`@includeNamed{}`标签中执行MVEL代码，就像`@include{}`标签一样。

### 9. @declare{} 声明一个模板

除了包括外部文件的外部模板，并以编程方式传递它们之外，您还可以从模板中声明模板。 它允许你做这样的事情：

```java
@declare{'personTemplate'}
 Name: @{name}
 Age:  @{age}
@end{}

@includeNamed{'personTemplate'; name='John Doe'; age=22}
```

### 10. @comment{} 注释标签

注释标签允许您向模板添加不可见的注释。 例如：

```java
@comment{
  This is a comment
}
Hello: @{name}!
```

## 三、MVEL 2.0模板集成

使用MVEL模板是直接和容易的。 与常规MVEL表达式一样，它们可以解释性地执行，或者预编译并重新用于更快的评估。

### 1. org.mvel.templates.TemplateRuntime 类

`TemplateRuntime`类是模板引擎的中心。您可以通过`eval()`方法将要计算的模板传递给模板引擎。

一般来说，模板引擎遵循上下文和变量绑定的所有相同规则，使用一组重载的`eval()`方法。

下面是一个解析模板的简单例子：

```java
String template = "Hello, my name is @{name.toUpperCase()}");
Map vars = new HashMap();
vars.put("name", "Michael");

String output = (String) TemplateRuntime.eval(template, vars);
```

在执行结束时，“output”变量将包含字符串：

```java
Hello, my name is MICHAEL
```

### 2. org.mvel.templates.TemplateCompiler类

`TemplateCompiler`类允许预先编译模板。

当编译模板时，将生成一个紧凑，可重用的评估树，可以快速用于计算模板。它直接使用：

```java
String template = "1 + 1 = @{1+1}";

// 编译模板
CompiledTemplate compiled = TemplateCompiler.compileTemplate(template);

// 执行模板
String output = (String) TemplateRuntime.execute(compiled);
```

在执行结束时，“output”变量将包含字符串：

```java
1 + 1 = 2
```

[1]: https://github.com/mvel/mvel
[2]: http://blinkfox.com/mvel-2-xyu-fa-zhi-nan/