---
title: Google Java编程风格指南
date: 2018-10-05 16:38:00
author: blinkfox
categories: 编程之道
tags:
  - Java
  - Google
---

## 1 前言

这份文档是`Google Java`编程风格规范的完整定义。当且仅当一个`Java`源文件符合此文档中的规则，我们才认为它符合`Google`的`Java`编程风格。

与其它的编程风格指南一样，这里所讨论的不仅仅是编码格式美不美观的问题，同时也讨论一些约定及编码标准。然而，这份文档主要侧重于我们所普遍遵循的规则，对于那些不是明确强制要求的，我们尽量避免提供意见。

### 1.1 术语说明

本文档中除非特殊说明，否则：

- 术语`class`可表示一个普通类、枚举类、接口或者注解。
- 术语`comment`只用来指代实现的注释(`implementation comments`)，我们不使用文档注释(`documentation comments`)一词，而是用`Javadoc`。

其他术语说明，将在文档中需要说明的地方单独说明。

### 1.2 指南说明

本文档中的示例代码并不作为规范。也就是说，虽然示例代码是遵循Google编程风格，但并不意味着这是展现这些代码的唯一方式。示例中的格式选择不应该被强制定为规则。

## 2 源文件基础

### 2.1 文件名

源文件以其最顶层的类名（其中只有一个）来命名，大小写敏感，文件扩展名为`.java`。

### 2.2 文件编码：UTF-8

源文件编码格式使用`UTF-8`。

### 2.3 特殊字符

#### 2.3.1 空格字符

除了换行符外，`ASCII`水平空白字符（0x20）是源码文件中唯一支持的空格字符。这意味着：

- 所有其他空白字符将被转义。
- `Tab`字符不被用作缩进控制。

#### 2.3.2 特殊转义字符串

任何需要转义字符串表示的字符（例如：`\b`, `\t`, `\n`, `\f`, `\r`, `\'`, `\\`等），采用这种转义字符串的方式表示，而不采用对应字符的八进制数（例如 `\012`）或`Unicode`码（例如：`\u000a`）表示。

#### 2.3.3 非ASCII字符

对于其余非`ASCII`字符，直接使用`Unicode`字符（例如 `∞`），或者使用对应的`Unicode`码（例如：`\u221e`）转义，都是允许的。**唯一需要考虑的是，何种方式更能使代码容易阅读和理解**。

> **注意**：在使用`Unicode`码转义，或者甚至是有时直接使用`Unicode`字符的时候，建议多添加一些注释说明，将对别人读懂代码很有帮助。

例子：

| 示例  | 结论  |
| ------------ | ------------ |
| String unitAbbrev = "μs";	 | 赞：即使没有注释也非常清晰。  |
| String unitAbbrev = "\u03bcs"; // "μs"  | 允许，但没有理由要这样做。  |
| String unitAbbrev = "\u03bcs"; // Greek letter mu, "s"  | 允许，但这样做显得笨拙还容易出错。  |
| String unitAbbrev = "\u03bcs";  | 很糟：读者根本看不出这是什么。  |
| return '\ufeff' + content; // byte order mark  | 很好：对于非打印字符，使用转义，并在必要时写上注释。  |

> **注意**：永远不要由于害怕某些程序可能无法正确处理非`ASCII`字符而让你的代码可读性变差。当程序无法正确处理非`ASCII`字符时，它自然无法正确运行，你就会去`fix`这些问题的了。(言下之意就是大胆去用非`ASCII`字符，如果真的有需要的话)

## 3 源文件结构

源文件按照先后顺序，由以下几部分组成：

- 许可证(`License`)或版权信息(`copyright`)（如果需要）
- `package`语句
- `import`语句
- `class`类声明（每个源码文件只能有唯一一个顶级`class`）。

> **注意**：以上每个部分之间应该只有**一个空行**作为间隔。

### 3.1 许可证或版权信息

如果一个文件包含许可证或版权信息，那么它应当被放在文件最前面。

### 3.2 package语句

`package`语句不换行，单行长度限制(4.4节)不适用于package语句。(即package语句写在一行里)

### 3.3 import语句

#### 3.3.1 import不使用通配符

`import`语句中不应该使用通配符，不管是否是静态导入。

#### 3.3.2 import不换行

`import`语句不换行，列限制(4.4节)并不适用于`import`语句。(每个`import`语句独立成行)

#### 3.3.3 顺序和间距

`import`语句可分为以下几组，按照顺序，每组由**一个空行**分隔：

- 所有的静态导入(static import)归为一组
- `com.google`包的`import`归为一组
- 使用的第三方包的导入，每个顶级按字典顺序归为一组。例如：`android`, `com`, `junit`, `org`, `sun`
- `java`包归为一组
- `javax`包归为一组

> **注意**：同一组内的`import`语句之间不应用空行隔开，同一组中的`import`语句按字典序排列。

### 3.4 类声明

#### 3.4.1 只声明唯一一个顶级class

每个源文件中只能有一个顶级class。

#### 3.4.2 类成员顺序

类成员的顺序对代码的易读性有很大影响，但是没有一个统一正确的标准。不同的类可能有不同的排序方式。

最重要的一点，**每个类应该以某种逻辑去排序它的成员，维护者应该要能解释这种排序逻辑**。比如，新的方法不能总是习惯性地添加到类的结尾，因为这样就是按时间顺序而非某种逻辑来排序的。

##### 3.4.2.1 重载：永不分离

当一个类有多个构造函数，或是多个同名方法，这些方法应该按顺序出现在一起，中间不要放进其它方法。

## 4 格式

> **术语说明**：块状结构(`block-­like construct`)指的是一个类，方法或构造函数的主体。需要注意的是，数组初始化中的初始值可被选择性地视为块状结构(4.8.3.1节)。

### 4.1 大括号

#### 4.1.1 使用大括号(即使是可选的)

大括号一般用在`if`, `else`, `for`, `do`, `while`等语句，即使只有一条语句(或是空)，也应该把大括号写上。

#### 4.1.2 非空语句块采用`K&R`风格

对于非空语句块，大括号遵循`Kernighan`和`Ritchie`风格 ([Egyptian brackets](https://blog.codinghorror.com/new-programming-jargon/)):

- 左大括号前不换行
- 左大括号后换行
- 右大括号前换行
- 如果右大括号结束是一个`语句块`或者`方法体`、`构造函数体`或者`有命名的类体`，则需要换行。当右括号后面接`else`或者`逗号`时，不应该换行。

示例：

```java
return () -> {
    while (condition()) {
        method();
    }
};

return new MyClass() {
    @Override public void method() {
        if (condition()) {
            try {
                something();
            } catch (ProblemException e) {
                recover();
            }
        } else if (otherCondition()) {
            somethingElse();
        } else {
            lastThing();
        }
    }
};
```

一些例外的情况，将在`4.8.1`节讲`枚举类型`的时候讲到。

#### 4.1.3 空语句块：使代码更简洁

一个空的语句块，可以在左大括号之后直接接右大括号，中间不需要空格或换行。但是当一个由几个语句块联合组成的语句块时，则需要换行。（例如：`if/else` 或者`try/catch/finally`）.

示例：

```java
// 这是可接受的
void doNothing() {}

// 这同样是可接受的
void doNothingElse() {
}
```

```java
// 这是不可接受的：多块语句中没有简洁的空语句块
try {
    doSomething();
} catch (Exception e) {}
```

### 4.2块缩进：2个空格

每当一个新的语句块产生，缩进就增加两个空格。当这个语句块结束时，缩进恢复到上一层级的缩进格数。缩进要求对整个语句块中的代码和注释都适用。（例子可参考之前4.1.2节中的例子）。

> **注意**：根据实际的编程经验，`2`个空格缩进的代码在当前大屏的计算机上会显得十分拥挤，反而使得代码`臃肿`不够美观。所以，我这里建议使用`4`个空格来缩进，会使得更加美观，而且能侧面督促开发人员减少代码的嵌套层数。

### 4.3 一行一个语句

每条语句结束都需要换行。

### 4.4 列长度限制：100

Java代码的列长度限制为`100个`字符。 除了如下所述，任何超过此限制的行都必须跳行。这在4.5节会有详细解释。

例外：

- 不可能满足行长度限制的行(例如，Javadoc中的一个长URL，或是一个长的JSNI方法参考)
- `package`和`import`语句(见3.2节和3.3节)
- 注释中那些可能被剪切并粘贴到shell中的命令行

> **注意**：当前的计算机屏幕都已经比很宽了，而且变量及方法命名都较长，`100`个字符的长度反而会出现很多不必要的跳行，已经不适应当今的情况了，根据实际编程经验，我这里建议使用`120`个字符的宽度更为合适。

### 4.5 换行

**术语说明**：一般情况下，一行长代码为了避免超出列限制(`100`个字符)而被分为多行，我们称之为断行(`line­-wrapping`)。

我们并没有全面，确定性的准则来决定在每一种情况下如何断行。很多时候，对于同一段代码会有好几种有效的换断行方式。

> **注意**: 提取`方法`或`局部变量`可以解决问题，而不不需要进行断行。

#### 4.5.1 在何处断行

断行的主要原则是：**选择在更高级的语法逻辑处断行**。其他一些原则如下：

- 当一个非赋值运算的语句断行时，在运算符号之前断行。（这与Google的C++规范和JavaScrip规范等其他规范不同）。
- 如果要在非赋值运算符处断行，那么在该符号前断开(比如`+`操作符，它将位于下一行)。以下的`类运算符`也可作为参考：
  - 点操作符`.`
  - 类型界限中的`&`、`||`等（例如：`<T extends Foo & Bar>`)
- 当要在一个赋值运算语句处断行时，一般在赋值符号之后断行。但是也可以在之前断行。(例如：`=`，它与前面的内容留在同一行)。
  - 这条规则也适用于`foreach`语句中的冒号。
- 方法名或构造函数名与左括号留在同一行。
- 逗号(`,`)与其前面的内容留在同一行。也就是在逗号之后断行。
- `Lambda`表达式在箭头符号(`->`)后断行。

示例：

```java
MyLambda<String, Long, Object> lambda =
    (String label, Long value, Object obj) -> {
        ...
    };

Predicate<String> predicate = str ->
    longExpressionInvolving(str);
```

> **注意**：换行的主要目标是使代码更清晰易读。

#### 4.5.2 断行的缩进：至少+4个空格

自动换行时，第一行后的每一行至少比第一行多缩进`4`个空格(注意：制表符不用于缩进。见2.3.1节)。

当存在连续自动换行时，缩进可能会多缩进不只`4`个空格(语法元素存在多级时)。一般而言，两个连续行使用相同的缩进当且仅当它们开始于同级语法元素。

第4.6.3水平对齐一节中指出，不鼓励使用可变数目的空格来对齐前面行的符号。

### 4.6 空白

#### 4.6.1 垂直空白

以下情况需要使用单行空行：

- 类成员之间需要单个空行隔开：例如：`字段`，`构造函数`，`方法`，`嵌套类`，`静态初始化块`，`实例初始化块`。但也有以下两种例外情况：
  - 两个连续字段之间的空行是可选的，根据需要使用空行来创建字段间的逻辑分组。
  - 枚举常量之间的的空行也是可选的，根据需要使用空行来创建枚举常量间的逻辑分组。
- 在方法体内，根据代码的逻辑分组的需要，设置空白行作为间隔。
- 类的第一个成员之前或最后一个成员之后，使用空行(可选)。
- 本文档所介绍的其他章节的空行要求(比如3.3节：`import`语句)。

#### 4.6.2 水平空白

除了语法、其他规则、词语分隔、注释和javadoc外，水平的ASCII空格只在以下情况出现：

- 所有保留的关键字与紧接它之后的位于同一行的左大括号之间需要用空格隔开。(例如：`if`, `for` `catch`等)
- 所有保留的关键字与在它之前的右大括号之间需要空格隔开。（例如：`else`、`catch`）
- 在左大括号之前都需要空格隔开。只有两种例外：
  - `@SomeAnnotation({a, b})`
  - `String[][] x = {{"foo"}};`
- 所有的二元运算符和三元运算符的两边，都需要空格隔开。(例如：`a + b`、`b = a < 0 ? 0 : a`)
- 逗号(`,`)、冒号(`:`)、分号(`;`)和右小括号(`)`)、Lambda箭头符号(`->`)之后，需要空格隔开。
- `//`双斜线开始一行注释时，双斜线两边都应该用空格隔开。并且可使用多个空格。（可选，例如：`a = 0; // 赋值为0`）
- 变量声明时，变量类型和变量名之间需要用空格隔开。（例如：`List<String> list`）
- 初始化一个数组时，花括号之间可以用空格隔开，也可以不使用。（可选，例如：`new int[] {5, 6}`和`new int[] { 5, 6 }`）

> **注意**：这个规则并不要求或禁止一行的开关或结尾需要额外的空格，只对内部空格做要求。

#### 4.6.3 水平对齐：不做要求

> **术语说明**：水平对齐，是指通过添加多个空格，使本行的某一符号与上一行的某一符号上下对齐。

这种对齐是被允许的，但是不会做强制要求。

以下是没有水平对齐和水平对齐的例子：

```java
private int x;   // 这种挺好
private Color color;   // 同上

private int   x;      // 允许，但是未来会继续编辑
private Color color;  // 可能会使它对不齐
```

> **注意**：水平对齐能够增加代码的可读性，但是增加了未来维护代码的难度。考虑到维护时只需要改变一行代码，之前的对齐可以不需要改动。为了对齐，你更有可能改了一行代码，同时需要更改附近的好几行代码，而这几行代码的改动，可能又会引起一些为了保持对齐的代码改动。那原本这行改动，我们称之为**爆炸半径**。这种改动，在最坏的情况下可能会导致大量的无意义的工作，即使在最好的情况下，也会影响版本历史信息，减慢代码`review`的速度，引起更多`merge`代码冲突的情况。

### 4.7 分组小括号：推荐使用

除非作者和`reviewer`都认为去掉小括号也不会使代码被误解，或是去掉小括号能让代码更易于阅读，否则我们不应该去掉小括号。我们没有理由假设读者能记住整个Java运算符优先级表。

### 4.8 特殊结构

#### 4.8.1 枚举类型

枚举常量间用逗号隔开，换行是可选的。而且还允许附加的空行（通常只有一个）。以下就是一种可能性的示例：

```java
private enum Answer {
    YES {
        @Override public String toString() {
            return "yes";
        }
    },

    NO,
    MAYBE
}
```

没有方法和Javadoc的枚举类可写成数组初始化的格式：

```java
private enum Suit { CLUBS, HEARTS, SPADES, DIAMONDS }
```

由于枚举类也是一个类，因此所有适用于其它类的格式规则也适用于枚举类。

#### 4.8.2 变量声明

##### 4.8.2.1 每次声明一个变量

不要使用组合声明。例如：`int a, b;`是不允许的。

##### 4.8.2.2 需要时才声明，尽快进行初始化

不要在一个代码块的开头把局部变量一次性都声明了(这是c语言的做法)，而是在第一次需要使用它时才声明。局部变量在声明时最好就进行初始化，或者声明后尽快进行初始化。

#### 4.8.3 数组

##### 4.8.3.1 数组初始化：可写成块状结构

数组初始化可以写成块状结构，例如以下格式的写法都是允许的：

```java
new int[] {           new int[] {
  0, 1, 2, 3            0,
}                       1,
                        2,
new int[] {             3,
  0, 1,               }
  2, 3
}                     new int[]
                          {0, 1, 2, 3}
```

##### 4.8.3.2 非C风格的数组声明

中括号是类型的一部分：`String[] args`， 而非`String args[]`。

#### 4.8.4 switch语句

**术语说明**：`switch`块的大括号内是一个或多个语句组。每个语句组包含一个或多个`switch`标签(`case FOO: `或`default:`)，后面跟着一条或多条语句。

##### 4.8.4.1 缩进

和其他语句块一样，`switch`大括号之后缩进两个字符。每个`switch`标签之后，后面紧接的非标签的新行，按照大括号相同的处理方式缩进两个字符。在标签结束后，恢复到之前的缩进，类似大括号结束。

##### 4.8.4.2 继续向下执行的注释

在一个`switch`块内，每个语句组要么通过`break`、`continue`、`return`或`抛出异常`来终止，要么通过一条注释来说明程序将继续执行到下一个语句组，任何能表达这个意思的注释都是可以的(典型的是用`// fall through`)。这个特殊的注释并不需要在最后一个语句组(一般是`default`)中出现。例如：

```java
switch (input) {
    case 1:
    case 2:
        prepareOneOrTwo();
        // fall through
    case 3:
        handleOneTwoOrThree();
        break;
    default:
        handleLargeNumber(input);
}
```

> **注意**：在`case 1`之后不需要该注释，仅在语句组的末尾。

##### 4.8.4.3 default标签需要显式声明

每个`switch`语句中，都需要显式声明`default`标签。即使没有任何代码也需要显示声明。

> **注意**：枚举类型的`switch`语句可以省略`default`语句组，如果它包含覆盖该类型的所有可能值的显式情况。这使得IDE或其他静态分析工具能够在丢失任何情况时发出警告。

#### 4.8.5 注解

注解应用到类、方法或者构造方法时，应紧接`Javadoc`之后。每一行只有一个注解。注解所在行不受列长度限制，也不需要增加缩进。例如：

```java
@Override
@Nullable
public String getNameIfPresent() { ... }
```

**例外**：如果注解只有一个，并且不带参数。则它可以和类或方法名放在同一行。例如：

```java
@Override public int hashCode() { ... }
```

注解应用到成员变量时，也是紧接`Javadoc`之后。不同的是，多个注解可以放在同一行。例如：

```java
@Partial @Mock DataLoader loader;
```

对于参数或者局部变量使用注解的情况，没有特定的规范。

#### 4.8.6 注释

##### 4.8.6.1 块注释风格

注释的缩进与它所注释的代码缩进相同。可以采用`/* */`进行注释，也可以用`//`进行注释。当使用`/* */`进行多行注释时，每一行都应该以`*`开始，并且`*`应该上下对齐。

例如：

```java
/*
 * This is
 * okay.
 */

// And so
// is this.

/* Or you can
 * even do this. */
```

> **注意**：多行注释时，如果你希望集成开发环境能自动对齐注释，你应该使用`/* */`，`//`一般不会自动对齐。

#### 4.8.7 修饰符

类和成员变量的修饰符，按`Java Lauguage Specification`中介绍的先后顺序排序。具体是：

```java
public protected private abstract default static final transient volatile synchronized native strictfp
```

#### 4.8.8 数字字面量

长整型的数字字面量使用大写的`L`作为后缀，不得使用小写（以免与数字1混淆）。例如：使用`3000000000L`，而不是`3000000000l`。

## 5 命名约定

### 5.1 对所有标识符都通用的规则

标识符只能使用`ASCII`字母和数字，因此每个有效的标识符名称都能匹配正则表达式`\w+`。

在Google其它编程语言风格中使用的特殊前缀或后缀，如`name_`, `mName`, `s_name`和`kName`，在Java编程风格中都不再使用。

### 5.2 标识符类型的规则

#### 5.2.1 包名

包名全部小写，连续的单词只是简单地连接起来，不使用下划线。例如：使用`com.example.deepspace`，而不是`com.example.deepSpace`或者`com.example.deep_space`。

#### 5.2.2 类名

类名都以`UpperCamelCase`风格编写。

类名通常是名词或名词短语。例如：`Character`或者`ImmutableList`。接口名称也可以是名词或名词短语（例如：`List`），但有时可能是形容词或形容词短语（例如：`Readable`）。现在还没有特定的规则或行之有效的约定来命名注解类型。

测试类的命名以它要测试的类的名称开始，以`Test`结束。例如：`HashTest`或`HashIntegrationTest`。

#### 5.2.3 方法名

方法名都以`lowerCamelCase`风格编写。

方法名通常是动词或动词短语。例如：`sendMessage`或者`stop`。

下划线可能出现在JUnit测试方法名称中用以分隔名称的逻辑组件。一个典型的模式是：`test<MethodUnderTest>_<state>`，例如：`testPop_emptyStack`。 并不存在唯一正确的方式来命名测试方法。

#### 5.2.4 常量名

常量名命名模式为`CONSTANT_CASE`，全部字母大写，用下划线分隔单词。那到底什么算是一个常量呢？

每个常量都是一个静态`final`字段，其内容是不可变的，且没有可检测的副作用。这包括原始类型、字符串、不可变类型和不可变类型的不可变集合。如果任何一个实例的观测状态是可变的，则它肯定不会是一个常量。只是永远不打算改变对象也是不够的。例如：

```java
// 常量
static final int NUMBER = 5;
static final ImmutableList<String> NAMES = ImmutableList.of("Ed", "Ann");
static final ImmutableMap<String, Integer> AGES = ImmutableMap.of("Ed", 35, "Ann", 32);
static final Joiner COMMA_JOINER = Joiner.on(','); // 因为Joiner是不可变的
static final SomeMutableType[] EMPTY_ARRAY = {};
enum SomeEnum { ENUM_CONSTANT }

// 非常量
static String nonFinal = "non-final";
final String nonStatic = "non-static";
static final Set<String> mutableCollection = new HashSet<String>();
static final ImmutableSet<SomeMutableType> mutableElements = ImmutableSet.of(mutable);
static final ImmutableMap<String, SomeMutableType> mutableValues =
    ImmutableMap.of("Ed", mutableInstance, "Ann", mutableInstance2);
static final Logger logger = Logger.getLogger(MyClass.getName());
static final String[] nonEmptyArray = {"these", "can", "change"};
```

这些常量的名字通常是名词或名词短语。

#### 5.2.5 非常量字段名

非常量字段名以`lowerCamelCase`风格编写。

这些名字通常是名词或名词短语。例如：`computedValues`或者`index`。

#### 5.2.6 参数名

参数名以`lowerCamelCase`风格编写。

参数应该避免用单个字符命名。

#### 5.2.7 局部变量名

局部变量名以`lowerCamelCase`风格编写。

即使局部变量是`final`和`不可改变`的，也不应该把它示为常量，当然也就不能用常量的规则去命名它。

#### 5.2.8 类型变量名

类型变量可用以下两种风格之一进行命名：

- 单个的大写字母，后面可以视具体情况跟一个数字(如：`E`, `T`, `X`, `T2`)。
- 以类命名方式(5.2.2节)，后面加个大写的T(如：`RequestT`, `FooBarT`)。

### 5.3 驼峰式命名法(CamelCase)

**驼峰式命名法**分大驼峰式命名法(`UpperCamelCase`)和小驼峰式命名法(`lowerCamelCase`)。有时，我们有不只一种合理的方式将一个英语词组转换成驼峰形式，如缩略语或不寻常的结构(例如：`IPv6`或`iOS`)。Google指定了以下的转换方案。

名字从散文形式(prose form)开始:

- 把短语转换为纯`ASCII`码，并且移除任何单引号。例如：`Müller’s algorithm`将变成`Muellers algorithm`。
- 把这个结果切分成单词，在空格或其它标点符号(通常是连字符)处分割开。
  - **推荐**：如果某个单词已经有了常用的驼峰表示形式，按它的组成将它分割开(如`AdWords`将分割成`ad words`)。 
  - 需要注意的是iOS并不是一个真正的驼峰表示形式，因此该推荐对它并不适用。
- 现在将所有字母都小写(包括缩写)，然后将单词的第一个字母大写：
  - 每个单词的第一个字母都大写，来得到大驼峰式命名。
  - 除了第一个单词，每个单词的第一个字母都大写，来得到小驼峰式命名。
- 最后将所有的单词连接起来得到一个标识符。

示例：

| 散文形式  | 正确  | 不正确  |
| ------------ | ------------ | ------------ |
| "XML HTTP request"  | XmlHttpRequest  | XMLHTTPRequest  |
| "new customer ID"  | newCustomerId  | newCustomerID  |
| "inner stopwatch"  | innerStopwatch  | innerStopWatch  |
| "supports IPv6 on iOS?"  | supportsIpv6OnIos  | supportsIPv6OnIOS  |
| "YouTube importer"  | YouTubeImporter YoutubeImporter^  | 无   |

加`^`号处表示可以，但不推荐。

> **注意**：在英语中，某些带有连字符的单词形式不唯一。例如：`nonempty`和`non-empty`都是正确的，因此方法名`checkNonempty`和`checkNonEmpty`也都是正确的。

## 6 编程实践

### 6.1 `@Override`：总是使用

只要是合法的方法，就把`@Override`注解加上。这包括覆盖超类方法的类方法，实现接口方法的类方法。

**例外**：当父方法为`@Deprecated`时，可以省略`@Override`。

### 6.2 捕获的异常：不能忽视

除了下面的例子，对捕获的异常不做任何响应是极少的。(典型的响应方式是打印日志，或者如果它被认为是不可能的，则把它当作一个`AssertionError`重新抛出。)

如果它确实是不需要在`catch`块中做任何响应，需要做注释加以说明(如下面的例子)。

```java
try {
    int i = Integer.parseInt(response);
    return handleNumericResponse(i);
} catch (NumberFormatException ok) {
    // 它不是一个数字，不过没关系，继续
}
return handleTextResponse(response);
```

例外：在测试中，如果一个捕获的异常被命名为`expected`，则它可以被不加注释地忽略。下面是一种非常常见的情形，用以确保所测试的方法会抛出一个期望中的异常， 因此在这里就没有必要加注释。

```java
try {
    emptyStack.pop();
    fail();
} catch (NoSuchElementException expected) {
}
```

6.3 静态成员：使用类来调用

使用类名调用静态的类成员，而不是具体某个对象或表达式。

```java
Foo aFoo = ...;
Foo.aStaticMethod(); // 好
aFoo.aStaticMethod(); // 糟
somethingThatYieldsAFoo().aStaticMethod(); // 很糟
```

6.4 `Finalizers`: 禁用

极少会去重载`Object.finalize`。

> **注意**：不要使用`finalize`。如果你非要使用它，请先仔细阅读和理解`Effective Java第7条款`：“Avoid Finalizers”，然后不要使用它。

## 7 Javadoc

### 7.1 格式

#### 7.1.1 一般形式

`Javadoc`块的基本格式如下所示：

```java
/**
 * Multiple lines of Javadoc text are written here,
 * wrapped normally...
 */
public int method(String p1) { ... }
```

或者是以下单行形式：

```java
/** An especially short bit of Javadoc. */
```

基本格式总是可以接受的。当整个`Javadoc`块能容纳于一行时(且没有标记`@XXX`)，就可以使用单行形式。

#### 7.1.2 段落

空行(只包含最左侧星号的行)会出现在段落之间和`Javadoc`标记(`@XXX`)之前(如果有的话)。 除了第一个段落，每个段落第一个单词前都有标签`<p>`，并且它和第一个单词间没有空格。

#### 7.1.3 Javadoc标记

标准的`Javadoc`标记按以下顺序出现：`@param`, `@return`, `@throws`, `@deprecated`, 前面这4种标记如果出现，描述都不能为空。 当描述无法在一行中容纳，连续行需要至少再缩进`4`个空格(**注**：如果你的缩进统一采用采用`4`个空格，那么这里就应该是`8`个空格)。

#### 7.2 摘要片段

每个类或成员的`Javadoc`以一个简短的摘要片段开始。这个片段是非常重要的，在某些情况下，它是唯一出现的文本，比如在类和方法索引中。

这只是一个小片段，可以是一个名词短语或动词短语，但不是一个完整的句子。它不会以`A {@code Foo} is a...`或者`This method returns...`开头, 它也不会是一个完整的祈使句，如`Save the record.`。然而，由于开头大写及被加了标点，它看起来就像是个完整的句子。

> **注意**：一个常见的错误是把简单的Javadoc写成`/** @return the customer ID */`，这是不正确的。它应该写成`/** Returns the customer ID. */`。

### 7.3 在哪里使用Javadoc

至少在每个`public`类及它的每个`public`和`protected`成员处使用`Javadoc`，以下是一些例外：

#### 7.3.1 例外：不言自明的方法

对于简单明显的方法如`getFoo`，`Javadoc`是可选的(可以不写)。这种情况下除了写`Returns the foo`，确实也没有什么值得写了。

单元测试类中的测试方法可能是不言自明的最常见例子了，我们通常可以从这些方法的描述性命名中知道它是干什么的，因此不需要额外的文档说明。

> **注意**：如果有一些相关信息是需要读者了解的，那么以上的例外不应作为忽视这些信息的理由。例如，对于方法名`getCanonicalName`，就不应该忽视文档说明，因为读者很可能不知道词语`canonical name`指的是什么。

#### 7.3.2 例外：重载

如果一个方法重载了超类中的方法，那么`Javadoc`并非必需的。

#### 7.3.3 可选的Javadoc

对于包外不可见的类和方法，如有需要，也是要使用`Javadoc`的。如果一个注释是用来定义一个类，方法，字段的整体目的或行为， 那么这个注释应该写成`Javadoc`，这样更统一更友好。

原文地址: [Google Java Style Guide](http://checkstyle.sourceforge.net/reports/google-java-style-20170228.html)