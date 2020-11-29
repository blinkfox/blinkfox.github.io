---
title: JavaScript基础教程
date: 2018-09-14 00:12:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/09/14/js.jpg
categories: 前端
tags:
  - JavaScript
---

## 一、JavaScript介绍

[JavaScript][1]是目前所有主流浏览器上唯一支持的脚本语言，这也是早期`JavaScript`的唯一用途。其主要作用是在不与服务器交互的情况下修改`HTML`页面内容，因此其最关键的部分是`DOM`（文档对象模型），也就是`HTML`元素的结构。通过`Ajax`可以使`HTML`页面通过`JavaScript`，在不重新加载页面的情况下从服务器上获取数据并显示，大幅提高用户体验。通过`JavaScript`，使`Web`页面发展成胖客户端成为可能。

### 语言的性质

本节对`JavaScript`的性质做简要介绍，以帮你理解一些疑问。

`JavaScript`和`ECMAScript`（JavaScript versus ECMAScript）
编程语言称为`JavaScript`，语言标准被称为`ECMAScript`。他们有不同名字的原因是因为“Java”已经被注册为商标（属于Oracle）。目前，只有`Mozilla`被正式允许使用“JavaScript”名称，因为很久以前他们得到一份许可。因此，开放的语言标准拥有不同的名字。当前的`JavaScript`版本是`ECMAScript 6`，`ECMAScript 7`当前是开发版。

`JavaScript`之父，`Brendan Eich`[迅速了创建一门编程语言][2]。（否则，Netscape将使用其他技术）。他借鉴了几门其他语言的一些特性：

- JavaScript借鉴了Java的语法和如何区分原始值和对象。
- JavaScript的函数设计受Scheme和AWK的启发——他们（的函数）都是第一类（first-class）对象，并且在语言中广泛使用。闭包使他们（函数）变成强大的工具。
- Self影响了JavaScript独一无二的面向对象编程(OOP)风格。它的核心思想（在这里我们没有提到）非常优雅，基于此创建的语言非常少。但后面会提到一个简单的模式照顾大部分用例。JavaScript面向对象编程的杀手级特性是你可以直接创建对象。不需要先创建类或其他类似的东西。
- Perl和Python影响了JavaScript字符串，数组和正则表达式的操作。

`JavaScript`在最初的时候并不是一个完善的语言，因此也导致`JavaScript`遗留了很多令人诟病的问题。在开发稍大规模的应用时会显得力不从心，但是由于`JavaScript`本身是一种非常灵活的语言，因此在它的基础上开发程序库比较容易，因此出现了一大批非常优秀的第三方库，如[jQuery][3]，[ExtJS][4]，[underscorejs][5]，[backbone][6]等等，由于这些第三方库，`JavaScript`变得非常简单。其中`jQuery`的使用非常广泛，它大幅简化了`DOM`和`Ajax`，已经成为了很多网站的标配。`jQuery`虽然基于`JavaScript`，但它提供了另外一种编程范式，也就是逻辑式编程，与`SQL`和正则表达式类似。

### JavaScript能做什么

![JavaScript项目在Github所占比例][7]

如上图，`JavaScript`作为[Github][8]上最流行、最火的编程语言，几乎无所不能。这里是[PuYart][9]的关于[`JavaScript`就要统治世界了][10]的文章，可以让我们了解`JavaScript`到底能做什么的一些介绍。

1. Web前端(各种前端工具类库、前端框架、动画效果、数据可视化等)
2. 服务端开发([Node.js][11])
3. 移动应用或者`Hybrid App`(Cordova)
4. 桌面应用([NW.js][12]、[Electron][13])
5. 游戏([Unity3D][14]、[Cocos2d-js][15]、[Pomelo][16])
6. VR([JavaScript在VR世界的应用][17])
7. 硬件、嵌入式物联网等([Tessel：用JavaScript做嵌入式开发][18])
8. 操作系统([NodeOS][19])

> Atwood's Law: any application that can be written in JavaScript, will eventually be written in JavaScript.(Atwood定律：凡是能用JavaScript写出来的，最终都会用JavaScript写出来。)

## 二、 JavaScript语法

### 语句和表达式

了解`JavaScript`的语法，先来了解两个主要的语法类型：语句和表达式。

- 语句通常是“做某些事情”。程序是一组语句的序列。举个例子，下面声明（创建）一个变量 `foo`： 

```javascript
var foo;
```

- 表达式是产生“值”。他们通常位于赋值操作的右边、函数参数等。举个例子： 

```javascript
3 * 7
```

语句和表达式之间的区别最好通过实例说明，`JavaScript`（像Java）有两种不同的方式实现`if-then-else`。一种是用语句：

```javascript
var x;
if (y >= 0) {
    x = y;
} else {
    x = -y;
}
```

另一种是表达式：

```javascript
var x = y >= 0 ? y : -y;
```

你可以将后者作为函数参数（但前者不行）：

```javascript
myFunction(y >= 0 ? y : -y)
```

最后，每当`JavaScript`期待一个语句，你也可以用一个表达式代替。例如：

```javascript
foo(bar(7, 1));
```

`foo(...);`是一个语句（也叫做表达式语句），`bar(7, 1)`则是一个表达式。他们都实现函数调用。

### 流程控制语句和语句块

流程控制语句，其语句体可以是单条语句。举两个例子：

```javascript
if (obj !== null) obj.foo();

while (x > 0) x--;
```

然而，任何语句总能被语句块代替，花括号包含零或多条语句。因此，你也可以这样写：

```javascript
if (obj !== null) {
    obj.foo();
}

while (x > 0) {
    x--;
}
```

为便于程序的阅读和维护，推荐使用后一种方式，即语句块方式。

### 分号

`JavaScript`中的分号是[可选的][20]。但省略（分号）可能会带来意想不到的结果，所以我建议还是写上分号。

正如上面所看到的，分号作为语句的结尾，但语句块不需要。仅有一种情况下你能看到语句块后面有分号——**函数表达式后面的函数体块**。**表达式作为语句的结尾，后面是分号**：

```javascript
var x = 3 * 7;
var f = function () { };
```

### 注释

`JavaScript`的注释有两种形式：单行注释和多行注释。单行注释以`//`开头，以换行符结尾：

```javascript
x++; // 单行（single-line）注释
```

多行注释用`/**/`包裹

```javascript
/* 
 这是多行注释
 多行哦
 */
```

## 三、变量和赋值

`JavaScript`中的变量在使用前必须先声明，否则会报错引用错误（Reference Error）：

```javascript
var foo;  // 声明变量“foo”
```

### 赋值

你可以在声明变量的同时为其赋值：

```javascript
var foo = 6;
```

你也可以给已经存在的变量重新赋值：

```javascript
foo = 4;  // 更改变量的值
```

### 复合赋值操作符

有很多复合赋值操作符，例如+=。下面的两个赋值操作等价：

```javascript
x += 1;
x = x + 1;
```

### 标识符和变量名

标识符就是事物的名字，在`JavaScript`中他们扮演不同的语法角色。例如，变量的名称是一个标识符。

大体上，标识符的第一个字符可以是任何`Unicode`字符、美元标志符（$）或下划线（_）。后面可以是任意字符和数字。因此，下面全是合法的标识符：

```javascript
arg0
_tmp
$elem
π
```

> **注意**：首字符不能是数字，如果是数字的话，该如何区分是数字还是变量呢？

一些标识符是“保留关键字”——他们是语法的一部分，不能用作变量名。从技术上讲，下面三个标识符不是保留字，但也不应该作为变量名：

```javascript
Infinity NaN undefined
```

## 四、值

`JavaScript`有所有我们期待的编程语言值类型：布尔，数字，字符串，数组等。`JavaScript`中的所有值都有属性。每个属性有一个键（或名字）和一个值。你可以使用点（.）操作符读取属性：

```javascript
value.propKey
```

举个例子：字符串`abc`有属性`lenght`（长度）

```javascript
var str = 'abc';
console.log(str.length); // 得到3
```

上面的代码也可以写成下面这样：

```javascript
'abc'.length // 得到3
```

点操作符也可以用来给属性赋值：

```javascript
var obj = {};  // 空对象
obj.foo = 123; // 创建属性“foo”，设置它为123
console.log(obj.foo); // 得到123
```

你也可以通过它（.）调用方法：

```javascript
'hello'.toUpperCase(); // 得到HELLO
```

上面，我们在值`hello`上面调用方法`toUpperCase()`。

### 原始类型值和对象

JavaScript定义了不同值之间的区别：

- 原始值包括：`boolean`，`number`，`string`，`null`和`undefined`。
- 所有其他的值都是对象。实际上对象被定义为——所有不为原始值的值。

两者之间的主要区别在于他们是如何被比较的：每一个对象有一个独一无二的标志，并且仅和自己相等：

```javascript
var obj1 = {};  // 一个空对象
var obj2 = {};  // 另一个空对象
obj1 === obj2   // false
obj1 === obj1   // true
```

相反，所有原始值只要编码值相同就被认为是相同的：

```javascript
var prim1 = 123;
var prim2 = 123;
prim1 === prim2 // true
```

### 原始类型值

下面全是原始类型值（简称：原始值）：

- 布尔类型：true，false
- 数字类型：1736，1.351
- 字符串类型: ‘abc’，”abc”
- 两个“无值（non-values）”：undefined，null
原始值的特征：

- **值做比较时,“内容”做比较**。

```javascript
3 === 3 // true
'abc' === 'abc' // true
```

- **无法更改**：值的属性无法更改，无法添加和移除属性，获取未知属性总返回undefined。

```javascript
var str = 'abc';
str.foo = 3; // try to create property `foo` ⇒ no effect
str.foo  // unknown property ⇒  undefined
```

### 对象

#### 对象的类型

所有非原始值的值都是对象。最常见的几种对象类型是：

- 简单对象（类型是`Object`）能通过对象字面量创建：

```javascript
{
    firstName: ‘Jane’, 
    lastName: ‘Doe’
}
```

上面的对象有两个属性：`firstName`属性的值是“Jane”，`lastName`属性的值是“Doe”。

- 数组（类型是`Array`）能通过数组字面量创建：

```javascript
[ ‘apple’, ‘banana’, ‘cherry’ ]
```

上面的数组有三个元素，可以通过数字索引访问。例如“apple”的索引是0。

- 正则表达式对象（类型是`RegExp`）能通过正则表达式字面量创建。

```javascript
/^a+b+$/
```

#### 对象的特征

- **比较的是引用**：比较的是标识符，每个值有自己的标识符。

```javascript
{} === {}  // 两个不同的空对象, false
var obj1 = {};
var obj2 = obj1;
obj1 === obj2   // true
```

- **默认可以更改**。

```javascript
var obj = {};
obj.foo = 123;
obj.foo //123
```

所有的数据结构（如数组）都是对象，但并不是所有的对象都是数据结构。例如：正则表达式是对象，但不是数据结构。

### undefined 和 null

`JavaScript`有两个“无值）”：`undefined`和`null`。

`undefined`的意思是“没有值”。未初始化的变量是`undefined`：

```javascript
var foo;
foo // undefined
```
  
读取不存在的属性时，将返回`undefined`：

```javascript
  > var obj = {}; // 空对象
  > obj.foo // undefined
```

缺省的参数也是`undefined`：

```javascript
function f(x) {
    return x;
}
f(); //undefined
```
    
`null`的意思是“没有对象”。它被用来表示对象的无值（参数，链上的对象等）。

通常情况下你应该把`undefined`和`null`看成是等价的，如果他们代表相同意义的无值的话。检查他们的一种方式是通过严格比较：

```javascript
if (x === undefined || x === null) {
    ...
}
```

另一种在实际中使用的方法是认为undefined 和 null 都是false：

```javascript
if (!x) {
    ...
}
```

> **警告**：false，0，NaN 和 “” 都被当作false。

### 包装类型

对象类型的实例`Foo`（包括内建类型，例如Array和其他自定义类型）从对象`Foo.prototype`上获取方法。你可以通过读取这个方法的方式（不是调用）验证这点：

```javascript
[].push === Array.prototype.push  // true
```

相反，**原始类型是没有类型的，所以每个原始类型有一个关联类型，称之为包装类型**：

- 布尔值的包装类型是 Boolean。布尔值从Boolean.prototype上获取方法：

```javascript
  > true.toString === Boolean.prototype.toString    //true
```

> 注意：包装类型名字的首字母是大写的B。如果在JavaScript中布尔值的类型可以访问，那么它可能会被转换为布尔对象。

- 数字值的包装类型是`Number`。
- 字符串值的包装类型是`String`。

包装类型也有实例（他们的实例是对象），但不常用。相反，包装类型有其他用处：**如果你将他们作为函数调用，他们可以将值转换为原始类型**。

```javascript
Number('123') //123
String(true)  //'true'
```

### 通过typeof和instanceof将值分类

有两个操作符可以用来将值分类：`typeof`主要用于原始值，`instanceof`主要用于对象。

#### typeof 使用方法如下：

`typeof «value»`

`typeof`返回描述`value`“类型”的一个字符串。例如：

```javascript
typeof true //'boolean'
typeof 'abc' //'string'
typeof {} // 空对象字面量,'object'
typeof [] // 空数组字面量,'object'
```
  
下面列出了`typeof`操作的所有结果：

```
操作数 结果
undefined	'undefined'
null	'object'
Boolean value	'boolean'
Number value	'number'
String value	'string'
Function	'function'
All other values	'object'
```

有两个结果和我们上面说的的原始值与对象是矛盾的：

- 函数的类型是`function`而不是`object`。因为函数（类型为“function”）是对象（类型是对象）的子类型，这不是一个错误。
- `null`的类型是`object`。这是一个bug，但从没被修复，因为修复后会破坏现有的代码。

#### instanceof使用方法如下：

`«value» instanceof «Constr»`

如果`value`是一个对象，并且`value` 是由构造函数`Constr`创建的（参考：类）。例如：

```javascript
var b = new Bar();  // 通过构造函数Bar创建对象
b instanceof Bar    //true
{} instanceof Object    //true
[] instanceof Array //true
```

### 深入阅读

- [探索JavaScript中Null和Undefined的深渊][21]

## 五、布尔

布尔类型原始值包括`true`和`false`。下面的操作符会得到布尔值：

- 二元逻辑运算符：&&（与），||（或）
- 前缀逻辑运算符：!（非）
- 等值运算符：=== !== == !=
- 比较运算符（字符串或数字）：> >= < <=

### 真值和假值

每当`JavaScript`希望一个布尔值时（例如：if语句的条件），可以使用任何值。它将被理解（转换）为`true`或`false`。下面的值被理解为`false`：

- undefined, null
- 布尔: false
- 数字: 0, NaN
- 字符串: ‘’

所有其他值被认为`true`。被理解为`false`的值称为假值，被理解为`true`的值称为真值。可以使用`Boolean`作为函数，测试值被理解为什么。

```javascript
Boolean(undefined)  //false
Boolean(0)    //false
Boolean(3)    //true
```

### 二元逻辑运算符

`JavaScript`中的**二元逻辑运算符是短路运算**——如果第一个操作数可以确定结果，第二个操作数将不被验证（运算）。例如，在下面的代码中，函数`foo()`永远不会被调用。

```javascript
false && foo()
true || foo()
```

此外，**二元逻辑运算符会返回操作数中的一个**，可能是一个布尔值，也可能不是。

- **与**：如果第一个操作数是假值，返回第一个。否则返回第二个操作数。

```javascript
NaN && 'abc'    //NaN
123 && 'abc'    //'abc'
```

- **或**：如果第一个操作数是真值，返回第一个。否则，返回第二个操作数。

```javascript
'abc' || 123    //'abc'
'' || 123   //123
```

### 等值运算符

在`JavaScript`中检测相等，你可以使用严格相等（`===`）和严格不等（`!==`）。或者你也可以使用非严格相等（`==`）和非严格不等（`!=`）。

> **经验规则：总是用严格运算符，假装非严格运算符不存在。严格相等更安全。**

### 深入阅读

- [在JavaScript中什么时候使用==是正确的？][22]

## 六、数字

`JavaScript`中的**所有数字都是浮点型**（虽然大部分的JavaScript引擎内部也使用整数）。至于为什么这样设计，查看这里（[每一个JavaScript开发者应该了解的浮点知识][23]）。

```javascript
1 === 1.0   //true
```

特殊数字：

- `NaN` (“不是一个数字 not a number”): 错误值。

```javascript
Number('xyz')  // 'xyz' 不能被转换为数字得到:NaN
```
    
- `Infinity`：也是最大错误值（无穷大）

```javascript
3 / 0   //Infinity
Math.pow(2, 1024)  // 数字太大了,得到Infinity
```

`Infinity`有时很有用，因为它比任何其他数字都大。同样，`-Infinity` 比其他任何数字都小。

- `JavaScript`有两个零，`+0`和`-0`。它（js引擎）通常不让你看到，并简单将两个零都显示为0：

```javascript
+0  //0
-0  //0
```

因此最好假装只有一个零（正如我们看到假值时所做的那样：**-0 和 +0 都是假值**）。

### 运算符

`JavaScript`中有下列算数运算符：

```javascript
加: number1 + number2
减: number1 - number2
乘: number1 * number2
除: number1 / number2
模: number1 % number2
自增: ++variable, variable++
自减: –variable, variable–
负值: -value
正值（转换为数字）: +value
```

全局对象`Math`通过函数提供更多算数运算操作。

`JavaScript`中也有位运算符（例如：&）。

## 七、字符串

字符串可以直接通过字符串字面量创建。这些字面量被单引号或双引号包裹。反斜线（\）转义字符并且产生一些控制字符。例如：

```javascript
'abc'
"abc"

'Did she say "Hello"?'
"Did she say \"Hello\"?"

'That\'s nice!'
"That's nice!"

'Line 1\nLine 2'  // 换行
'Backlash: \\'
```

可以通过方括号访问单个字符：

```javascript
var str = 'abc';
str[1]    //'b'
```
`length`属性是字符串的字符数量。

```javascript
'abc'.length  //3
```

> **提醒**：字符串是不可变的，如果你想改变现有字符串，你需要创建一个新的字符串。

### 字符串运算符

字符串可以通过加号操作符（+）拼接，如果其中一个操作数为字符串，会将另一个操作数也转换为字符串。

```javascript
var msgCount = 3;
'You have '+ msgCount + ' messages' //'You have 3 messages'
```

连续执行拼接操作可以使用`+=`操作符：

```javascript
var str = '';
str += 'Multiple ';
str += 'pieces ';
str += 'are concatenated.';
console.log(str); //'Multiple pieces are concatenated.'
```

### 字符串方法

字符串有许多有用的方法。例如：

```javascript
'abc'.slice(1)  // 复制子字符串,得到索引1及其之后的字符串，即：'bc'
'abc'.slice(1, 2)   //得到索引1和2之间的字符串，即：'b'

'\t xyz  '.trim()  // 移除空白字符，即：'xyz'

'mjölnir'.toUpperCase()   //转成大写，即：'MJÖLNIR'

'abc'.indexOf('b')  // 查找第一个b的索引，即：1
'abc'.indexOf('x')    //没有返回-1
```

## 八、语句

### 条件（Conditionals）
`if`语句通过布尔条件决定执行那个分支：

```javascript
if (myvar === 0) {
    // then
}

if (myvar === 0) {
    // then
} else {
    // else
}

if (myvar === 0) {
    // then
} else if (myvar === 1) {
    // else-if
} else if (myvar === 2) {
    // else-if
} else {
    // else
}
```

下面的`switch`语句，furit的值决定那个分支被执行。

```javascript
switch (fruit) {
    case 'banana':
        // ...
        break;
    case 'apple':
        // ...
        break;
    default:  // 所有其他情况
        // ...
}
```

### 循环（Loops）

for 循环的格式如下：

```javascript
for(初始化; 当条件成立时循环; 下一步操作)
```

例子：

```javascript
for (var i=0; i < arr.length; i++) {
    console.log(arr[i]);
}
```

当条件成立时`while`循环继续循环它的循环体。

```javascript
// 和上面的for循环相等
var i = 0;
while (i < arr.length) {
    console.log(arr[i]);
    i++;
}
```

当条件成立时，`do-while`循环继续循环。由于条件位于循环体之后，所以循环体总是被至少至少执行一次。

```javascript
do {
    // ...
} while(条件);
```

在所有的循环中：

- break中断循环
- continue开始一个新的循环迭代

## 九、函数

定义函数的一种方法是通过函数声明：

```javascript
function add(param1, param2) {
    return param1 + param2;
}
```

上面的代码定义一个名称叫做`add`的函数，有两个参数`param1`和`param2`，并且返回参数的和。下面是如何调用这个函数：

```javascript
add(6, 1)   //7
add('a', 'b')   //'ab'
```

另一种定义`add()`函数的方法是通过函数表达式：

```javascript
var add = function (param1, param2) {
    return param1 + param2;
};
```

函数表达式产生一个值，因此可以直接将函数作为参数传递给其他函数：

```javascript
someOtherFunction(function (p1, p2) { ... });
```

### 函数声明提升

函数声明会被提升，他们全被移动到当前作用域开始之处。这允许你在函数声明之前调用它们：

```javascript
function foo() {
    bar();  // 没问题，bar被提升
    function bar() {
        ...
    }
}
```

> **注意**：虽然变量声明也会被提升，但赋值的过程不会被提升：

```javascript
function foo() {
    bar();  // 有问题，bar是undefined
    var bar = function () {
        // ...
    };
}
```

### 特殊变量参数

**在`JavaScript`中你可以调用任意函数并传递任意数量的参数**——语言绝不会“抱怨”（参数检测）。都可以正常工作，然而，使所有参数可访问需要通过特殊变量`arguments`。`arguments`看起来像数组，但它没有数组的方法（称为类数组 array-like）。

```javascript
function f() { return arguments }
var args = f('a', 'b', 'c');
args.length //3
args[0]  // 获取索引为0的元素,'a'
```

### 太多或太少参数

让我们通过下面的函数探索`JavaScript`中传递太多或太少参数时如何处理

```javascript
function f(x, y) {
    console.log(x, y);
}
```

多出的参数将被忽略（可以通过`arguments`访问）：

```javascript
f('a', 'b', 'c')    //a b
```

缺少的参数将会是`undefined`：

```javascript
f('a')    //a undefined
f() //undefined undefined
```

### 可选参数

下面是一个常见模式，给参数设置默认值：

```javascript
function pair(x, y) {
    x = x || 0;  // (*)
    y = y || 0;
    return [ x, y ];
}
```

在`（*）`这行，如果x是真值（除了：`null`，`undefined` 等），	 	操作符返回x。否则，它返回第二个操作数。

```javascript
pair()  //[ 0, 0 ]
pair(3) //[ 3, 0 ]
pair(3, 5)  //[ 3, 5 ]
```

### 强制数量

如果你想强制参数的数量，你可以检测`arguments.length`：

```javascript
function pair(x, y) {
    if (arguments.length !== 2) {
        throw new Error('Need exactly 2 arguments');
    }
    ...
}
```

### 将arguments 转换为数组

`arguments`不是一个数组，它仅仅是类数组（array-like）：它有一个`length`属性，并且你可以通过方括号索引方式访问它的元素。然而，你不能移除元素，或在它上面调用任何数组方法。因此，有时你需要将其转换为数组。这就是下面函数的作用。

```javascript
function toArray(arrayLikeObject) {
    return [].slice.call(arrayLikeObject);
}
```

## 十、异常处理

[异常处理][24]最常见的方式像下面这样：

```javascript
function throwException() {
    throw new Error('Problem!');
}

try {
    throwException();
} catch (e) {
    console.log(e);  // 错误：信息
    console.log(e.stack);  // 非标准，但大部分浏览器支持
}
```

try分支包裹易出错的代码，如果try分支内部抛出异常，catch分支将会执行。

## 十一、严格模式

严格模式开启检测和一些其他措施，使`JavaScript`变成更整洁的语言。推荐使用严格模式。为了开启严格模式，只需在`JavaScript`文件或`script`标签第一行添加如下语句：

```javascript
'use strict';
```

你也可以在每个函数上选择性开启严格模式，只需将上面的代码放在函数的开头：

```javascript
function functionInStrictMode() {
    'use strict';
}
```

下面的两小节看下严格模式的三大好处。

### 明确错误

让我们看一个例子，严格模式给我们明确的错误，否则`JavaScript`总是静默失败：下面的函数`f()` 执行一些非法操作，它试图更改所有字符串都有的只读属性——`length`：

```javascript
function f() {
    'abc'.length = 5;
}
```

当你调用上面的函数，它静默失败，赋值操作被简单忽略。让我们将`f()`在严格模式下运行：

```javascript
function f_strict() {
    'use strict';
    'abc'.length = 5;
}
```

现在浏览器报给我们一些错误：

```javascript
f_strict()  // TypeError: Cannot assign to read only property 'length' of abc
```

### 不是方法的函数中的this

在严格模式下，不作为方法的函数中的`this`值是`undefined`：

```javascript
function f_strict() {
    'use strict';
    return this;
}
console.log(f_strict() === undefined);  // true
```

在非严格模式下，`this`的值是被称作全局对象（`global object`）（在浏览器里是`window`）：

```javascript
function f() {
    return this;
}
console.log(f() === window);  // true
```

### 不再自动创建全局变量

在非严格模式下，如果你给不存在的变量赋值，`JavaScript`会自动创建一个全局变量：

```javascript
function f() { foo = 5 }
f()  // 不会报错
foo // 5
```
在严格模式下，这会产生一个错误：

```javascript
function f_strict() { 'use strict'; foo2 = 4; }
f_strict()  // ReferenceError: foo2 is not defined
```

### 深入阅读

- [揭秘javascript中谜一样的this][25]
- [JavaScript中的this关键字][26]

## 十二、变量作用域和闭包

在`JavaScript`中，你必须使用变量之前，通过`var`声明变量：

```javascript
var x;
x = 3;
y = 4;  // ReferenceError: y is not defined
```

你可以用一条`var`语句声明和初始化多个变量：

```javascript
var x = 1, y = 2, z = 3;
```

但我建议每个变量使用一条语句。因此，我将上面的语句重写为：

```javascript
var x = 1;
var y = 2;
var z = 3;
```

由于提升（见下文），最好在函数顶部声明变量。

### 变量和函数作用域

变量的作用域总是整个函数（没有块级作用域）。例如：

```javascript
function foo() {
    var x = -3;
    if (x < 0) {  // (*)
        var tmp = -x;
        ...
    }
    console.log(tmp);  // 3
}
```

我们可以看到tmp变量不仅在（*）所在行的语句块存在，它在整个函数内都存在。

### 变量提升

变量声明会被提升：声明会被移到函数的顶部，但赋值过程不会。举个例子，在下面的函数中`（*）`行位置声明了一个变量。

```javascript
function foo() {
    console.log(tmp); // undefined
    if (false) {
        var tmp = 3;  // (*)
    }
}
```

在内部，上面的函数被执行像下面这样：

```javascript
function foo() {
    var tmp;  // declaration is hoisted
    console.log(tmp);
    if (false) {
        tmp = 3;  // assignment stays put
    }
}
```

### 闭包

每个函数保持和函数体内部变量的连接，甚至离开创建它的作用域之后。例如：

```javascript
function createIncrementor(start) {
    return function () {  // (*)
        return start++;
    }
}
```

在`（*）`行开始的函数在它创建时保留上下文，并在内部保存一个`start`活动值：

```javascript
var inc = createIncrementor(5);
inc()   // 5
inc() // 6
inc()   // 7
```

闭包是一个函数加上和其作用域链的链接。因此，`createIncrementor()`返回的是一个闭包。

### IIFE：模拟块级作用域

有时你想模拟一个块，例如你想将变量从全局作用域隔离。完成这个工作的模式叫做 `IIFE`(立即执行函数表达式(`Immediately Invoked Function Expression`))：

```javascript
(function () {  // 块开始
    var tmp = ...;  // 非全局变量
}());  // 块结束
```

上面你会看到函数表达式被立即执行。外面的括号用来阻止它被解析成函数声明；只有函数表达式能被立即调用。函数体产生一个新的作用域并使`tmp`变为局部变量。

### 闭包实现变量共享

下面是个经典问题，如果你不知道，会让你费尽思量。因此，先浏览下，对问题有个大概的了解。

闭包保持和外部变量的连接，有时可能和你想像的行为不一致：

```javascript
var result = [];
for (var i=0; i < 5; i++) {
    result.push(function () { return i });  // (*)
}
console.log(result[1]()); // 5 (不是 1)
console.log(result[3]()); // 5 (不是 3)
```

`(*)`行的返回值总是当前的i值，而不是当函数被创建时的i值。当循环结束后，i的值是5，这是为什么数组中的所有函数的返回值总是一样的。如果你想捕获当前变量的快照，你可以使用`IIFE`：

```javascript
for (var i=0; i < 5; i++) {
    (function (i2) {
        result.push(function () { return i2 });
    }(i));  // 复制当前的i
}
```

深入阅读

- [认识javascript中的作用域和上下文][27]
- [JavaScript的作用域和提升机制][28]
- [了解JavaScript的执行上下文][29]

## 十三、对象和继承

和所有的值类型一样，对象有属性。事实上，你可以将对象当作一组属性的集合，每个属性都是一对（键和值）。键是字符串，值可以是任意`JavaScript`值。到目前为止，我们仅仅见过键是标识符的属性，因为点操作符处理的键必须为标识符。在这节，你讲见到另一种访问属性的方法，能将任意字符串作为键。

### 单个对象
在`JavaScript`中，你可以直接创建对象，通过对象字面量：

```javascript
var jane = {
    name: 'Jane',

    describe: function () {
        'use strict';
        return 'Person named '+this.name;
    }
};
```

上面的对象有两个属性：`name`和`describe`。你能读（“get”）和 写（“set”）属性：

```javascript
jane.name  // get，'Jane'
jane.name = 'John';  // set
jane.newProperty = 'abc';  // 自动创建
```

属性是函数如`describe`可以被当作方法调用。当调用他们时可以在它们内部通过this引用对象。

```javascript
jane.describe()  // 调用方法,'Person named John'
jane.name = 'Jane';
jane.describe() // 'Person named Jane'
```

`in`操作符用来检测一个属性是否存在：

```javascript
'newProperty' in jane   // true
'foo' in jane   // false
```

若读取一个不存在的属性，将会得到`undefined`值。因此上面的两个检查也可以像下面这样：

```javascript
jane.newProperty !== undefined  // true
jane.foo !== undefined  // false
```

`delete`操作符用来删除一个属性：

```javascript
delete jane.newProperty //true
'newProperty' in jane   //false
```

### 任意键属性

属性的键可以是任意字符串。到目前为止，我们看到的对象字面量中的和点操作符后的属性关键字。按这种方法你只能使用标识符。如果你想用其他任意字符串作为键名，你必须在对象字面量里加上引号，并使用方括号获取和设置属性。

```javascript
var obj = { 'not an identifier': 123 };
obj['not an identifier']    //123
obj['not an identifier'] = 456;
```

方括号允许你动态计算属性关键字：

```javascript
var x = 'name';
jane[x]; // 'Jane'
jane['na'+'me']; // 'Jane'
```

### 引用方法

如果你引用一个方法，它将失去和对象的连接。就其本身而言，函数不是方法，其中的`this`值为`undefined`（严格模式下）。

```javascript
var func = jane.describe;
func()  // TypeError: Cannot read property 'name' of undefined
```

解决办法是使用函数内置的`bind()`方法。它创建一个新函数，其`this`值固定为给定的值。

```javascript
var func2 = jane.describe.bind(jane);
func2() // 'Person named Jane'
```

### 方法内部的函数

每个函数都有一个特殊变量`this`。如果你在方法内部嵌入函数是很不方便的，因为你不能从函数中访问方法的`this`。下面是一个例子，我们调用`forEach`循环一个数组：

```javascript
var jane = {
    name: 'Jane',
    friends: [ 'Tarzan', 'Cheeta' ],
    logHiToFriends: function () {
        'use strict';
        this.friends.forEach(function (friend) {
            // 这里的“this”是undefined
            console.log(this.name + ' says hi to ' + friend);
        });
    }
}
```

调用`logHiToFriends`会产生错误：

```javascript
jane.logHiToFriends()   // TypeError: Cannot read property 'name' of undefined
```

有两种方法修复这问题。

- 将`this`存储在不同的变量。

```javascript
logHiToFriends: function () {
    'use strict';
    var that = this;
    this.friends.forEach(function (friend) {
        console.log(that.name + ' says hi to ' + friend);
    });
}
```

- forEach的第二个参数允许提供`this`值。

```javascript
logHiToFriends: function () {
    'use strict';
    this.friends.forEach(function (friend) {
        console.log(this.name + ' says hi to ' + friend);
    }, this);
}
```

在`JavaScript`中函数表达式经常被用作函数参数。时刻小心函数表达式中的`this`。

### 构造函数：对象工厂

除了作为“真正”的函数和方法，函数还在JavaScript中扮演第三种角色：**如果通过new操作符调用，他们会变为构造函数，对象的工厂**。构造函数是对其他语言中的类的粗略模拟。约定俗成，构造函数的第一个字母大写。例如：

```javascript
// 设置实例数据
function Point(x, y) {
    this.x = x;
    this.y = y;
}
// 方法
Point.prototype.dist = function () {
    return Math.sqrt(this.x*this.x + this.y*this.y);
};
```

我们看到构造函数分为两部分：首先，`Point`函数设置实例数据。其次，`Point.prototype`属性包含对象的方法。前者的数据是每个实例私有的，后面的数据是所有实例共享的。

我们通过new操作符调用`Point`：

```javascript
var p = new Point(3, 5);
p.x //3
p.dist();    //5.830951894845301
```

p是`Point`的一个实例：

```javascript
p instanceof Point  //true
typeof p    //'object'
```

### 深入阅读

- [Javascript继承 原型的陷阱][30]
- [Javascript 封装问题][31]

## 十四、数组

数组是数组元素的序列，能通过整数索引方法数组元素，数组索引从0开始。

### 数组字面量
数组字面量创建数组很方便：

```javascript
> var arr = [ 'a', 'b', 'c' ];
```

上面的数组有三个元素：分别是字符串“a”，“b”， “c”。你可以通过整数索引访问它们：

```javascript
arr[0]  //'a'
arr[0] = 'x';
arr
// [ 'x', 'b', 'c' ]
```

`length`属性总表示一个数组有多少项元素。

```javascript
arr.length    //3
```

除此之外它也可以用来从数组上移除尾部元素：

```javascript
arr.length = 2; 
arr // [ 'x', 'b' ]
```

`in`操作符也可以在数组上工作。

```javascript
1 in arr // arr在索引为1处是否有元素？,true
5 in arr // arr在索引为5处是否有元素？false
```

值得注意的是数组是对象，因此可以有对象属性：

```javascript
arr.foo = 123;
arr.foo   // 123
```

### 数组方法

数组有许多方法。举些例子：

```javascript
var arr = [ 'a', 'b', 'c' ];

arr.slice(1, 2)  // 复制元素，[ 'b' ]
arr.slice(1)    // [ 'b', 'c' ]

arr.push('x')  // 在末尾添加一个元素，4
arr // [ 'a', 'b', 'c', 'x' ]

arr.pop()  // 移除最后一个元素，'x'
arr   // [ 'a', 'b', 'c' ]

arr.shift()  // 移除第一个元素，'a'
arr // [ 'b', 'c' ]

arr.unshift('x')  // 在前面添加一个元素，3
arr // [ 'x', 'b', 'c' ]

arr.indexOf('b')  // 查找给定项在数组中的索引，若不存在返回-1，
// 1
arr.indexOf('y')  // -1

arr.join('-')  // 将元素拼接为一个字符串，'x-b-c'
arr.join('')    // 'xbc'
arr.join()  // 'x,b,c'
```

### 遍历数组

有几种方法可以遍历数组元素。其中两个最重要的是`forEach`和`map`。

`forEach`遍历整个数组，并将当前元素和它的索引传递给一个函数：

```javascript
[ 'a', 'b', 'c' ].forEach(function (elem, index) {  // (*)
    console.log(index + '. ' + elem);
});
```

上面代码的输出

```javascript
0. a
1. b
2. c
```

注意`（*）`行的函数参数是可省略的。例如：它可以只有一个参数`elem`。

`map`创建一个新数组，通过给每个存在数组元素应用一个函数：

```javascript
[1,2,3].map(function (x) { 
    return x*x 
});
// [ 1, 4, 9 ]
```

### 深入阅读

- [有趣的javascript原生数组函数][32]

## 十五、正则表达式

`JavaScript`内建支持正则表达式。他们被双斜线分隔：

```javascript
/^abc$/
/[A-Za-z0-9]+/
```

### 方法 test()：测试是否匹配

```javascript
/^a+b+$/.test('aaab')   // true
/^a+b+$/.test('aaa')    // false
```

### 方法 exec()：匹配和捕获组

```javascript
/a(b+)a/.exec('_abbba_aba_')    // [ 'abbba', 'bbb' ]
```

返回的数组第一项（索引为0）是完整匹配，捕获的第一个分组在第二项（索引为1），等。有一种方法可以反复调用获取所有匹配。

### 方法 replace()：搜索并替换

```javascript
'<a> <bbb>'.replace(/<(.*?)>/g, '[$1]') // '[a] [bbb]'
```

`replace`的第一个参数必须是正则表达式，并且开启全局搜索（`/g`标记），否则仅第一个匹配项会被替换。有一种方法使用一个函数来计算替换项。

## 十六、数学

[Math][33]是一个有算数功能的对象。例如：

```javascript
Math.abs(-2) // 2
Math.pow(3, 2) // 3^2 = 9
Math.max(2, -1, 5) //5
Math.round(1.9) // 2
Math.cos(Math.PI)  // 预定义常量π，-1
```

## 十七、标准库的其他功能

`JavaScript`标准库相对简单，但有很多其他东西你可以使用：

[Date][34]：日期构造函数，主要功能有转换和创建日期字符串，访问日期组成部分（年，小时等）。
[JSON][35]：一个对象，功能是转换和生成`JSON`数据。
[console.*][36]方法：浏览器的具体方法，不是语言成分的部分，但他们也可以在[Node.js][37]中工作。

## 十八、下一步学什么？

在你学会了这篇文章的基础教程后，你可以转到大部分章节末尾提到的高级教程。此外，我建议你看下面的资源：

- Style guides: I have written [a guide to style guides][38]
- [Underscore.js][39]: 一个弥补JavaScript标准库缺少的功能的库
- [JSbooks – free JavaScript books][40]
- [Frontend rescue: how to keep up to date on frontend technologies][41]
- [http://yanhaijing.com][42] 当然还有我的博客也非常不错哦
- [http://yanhaijing.com/es5][43] 如果你想成为高手，我建议阅读`ecmascript`规范
- [给javascript初学者的24条最佳实践][44]
- [我希望我知道的七个JavaScript技巧][45]

参考自原文：http://www.2ality.com/2013/06/basic-javascript.html
参考自译文：http://yanhaijing.com/basejs/

  [1]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript
  [2]: http://yanhaijing.com/javascript/2013/06/22/javascript-designing-a-language-in-10-days/
  [3]: http://jquery.com/
  [4]: http://extjs.org.cn/
  [5]: http://underscorejs.org/
  [6]: http://backbonejs.org/
  [7]: https://statics.sh1a.qingstor.com/2020/11/29/js.png
  [8]: https://github.com/
  [9]: https://segmentfault.com/u/puyart
  [10]: https://segmentfault.com/a/1190000003767058
  [11]: https://nodejs.org/
  [12]: http://nwjs.io/
  [13]: http://electron.atom.io/
  [14]: http://unity3d.com/cn/
  [15]: http://www.cocos.com/doc/article/index?type=cocos2d-x&url=/doc/cocos-docs-master/manual/framework/cocos2d-js/catalog/../1-about-cocos2d-js/1-1-a-brief-history/zh.md
  [16]: http://pomelo.netease.com/
  [17]: https://www.phodal.com/blog/why-javascript-will-use-vr-world/
  [18]: http://blog.jobbole.com/46055/
  [19]: http://node-os.com/
  [20]: http://www.2ality.com/2011/05/semicolon-insertion.html
  [21]: http://yanhaijing.com/javascript/2014/01/05/exploring-the-abyss-of-null-and-undefined-in-javascript/
  [22]: http://yanhaijing.com/javascript/2014/04/25/strict-equality-exemptions/
  [23]: http://yanhaijing.com/javascript/2014/03/14/what-every-javascript-developer-should-know-about-floating-points
  [24]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch
  [25]: http://yanhaijing.com/javascript/2013/12/28/demystifying-this-in-javascript
  [26]: http://yanhaijing.com/javascript/2014/04/30/javascript-this-keyword
  [27]: http://yanhaijing.com/javascript/2013/08/30/understanding-scope-and-context-in-javascript
  [28]: http://yanhaijing.com/javascript/2014/04/30/JavaScript-Scoping-and-Hoisting
  [29]: http://yanhaijing.com/javascript/2014/04/29/what-is-the-execution-context-in-javascript
  [30]: http://yanhaijing.com/javascript/2013/08/23/javascript-inheritance-how-to-shoot-yourself-in-the-foot-with-prototypes
  [31]: http://yanhaijing.com/javascript/2013/08/30/encapsulation-of-javascript
  [32]: http://yanhaijing.com/javascript/2014/01/17/fun-with-javascript-native-array-functions
  [33]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math
  [34]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date
  [35]: http://www.2ality.com/2011/08/json-api.html
  [36]: https://developer.mozilla.org/en-US/docs/Web/API/console
  [37]: https://nodejs.org/
  [38]: http://www.2ality.com/2013/07/meta-style-guide.html
  [39]: http://underscorejs.org/
  [40]: http://jsbooks.revolunet.com/
  [41]: http://uptodate.frontendrescue.org/
  [42]: http://yanhaijing.com/
  [43]: http://yanhaijing.com/es5
  [44]: http://yanhaijing.com/javascript/2013/12/11/24-JavaScript-best-practices-for-beginners
  [45]: http://yanhaijing.com/javascript/2014/04/23/seven-javascript-quirks-i-wish-id-known-about