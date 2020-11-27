---
title: JavaScript之再学习
date: 2018-10-30 00:30:00
author: blinkfox
categories: 前端
tags:
  - JavaScript
---

## 概览

JavaScript 是一种面向对象的动态语言，它包含类型、运算符、标准内置（ built-in）对象和方法。它的语法来源于 Java 和 C，所以这两种语言的许多语法特性同样适用于 JavaScript。需要注意的一个主要区别是 JavaScript 不支持类，类这一概念在 JavaScript 通过对象原型（object prototype）得到延续。另一个主要区别是 JavaScript 中的函数也是对象，JavaScript 允许函数在包含可执行代码的同时，能像其他对象一样被传递。

## 数据类型和结构

### 1. 动态类型

`JavaScript`是一种弱类型或者说动态语言。这意味着你不用提前声明变量的类型，在程序运行过程中，类型会被自动确定。这也意味着你可以使用同一个变量保存不同类型的数据：

```javascript
var foo = 42;    // foo is a Number now
var foo = "bar"; // foo is a String now
var foo = true;  // foo is a Boolean now
```

### 2. 数据类型

最新的`ECMAScript`标准定义了 7 种数据类型:

- 6 种原始类型
  - `Null` (空, 只有一个值`null`)
  - `Undefined` (未定义, 一个没有被赋值的变量的默认值是`undefined`):
  - `Boolean` (布尔, 可以有两个值：`true` 和 `false`)
  - `Number` (数字)
  - `String` (字符串)
  - `Symbol` (符号, ECMAScript 6 新定义的类型，表示独一无二的值)
- 和 `Object` (对象)
  - `Function` (函数)
  - `Array` (数组)
  - `Date` (日期)
  - `JSON` (JS对象标识,来序列化对象、数组、数值、字符串、布尔值和 `null`)
  - `Math` (数学方面的计算)
  - `RegExp` (正则表达式)
  - `Error` (错误)
  - `Map`
  - `Set`

### 内置对象

这里的**内置对象**指的是在全局作用域(`global scope`)中的对象，由于很多，不再一一列出说明，更全面的解释在[这里](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects)。

全局对象本身可通过`this`操作符在全局作用域中获得。实际上，全局作用域就是由全局对象的各个属性组成的（包括继承来的属性）。

## 严格模式

除了正常运行模式，ECMAscript 5添加了第二种运行模式："严格模式"（`strict mode`）。顾名思义，这种模式使得Javascript在更严格的条件下运行。

严格模式可以应用到整个script标签或个别函数中。设立"严格模式"的目的，主要有以下几个：

- 消除 Javascript 语法的一些不合理、不严谨之处，减少一些怪异行为;
- 消除代码运行的一些不安全之处，保证代码运行的安全；
- 提高编译器效率，增加运行速度；
- 为未来新版本的 Javascript 做好铺垫。

### 为某个script标签开启严格模式

进入严格模式的标志，是下面这行语句：

```javascript
'use strict'
```

### 为某个函数开启严格模式

```javascript
function strict(){
  // 函数级别严格模式语法
  'use strict';
  return "I'm a strict mode function!  " + nested();
}

function notStrict() {
    return "I'm not strict.";
}
```

## 相等性判断

JavaScript提供三种不同的值比较操作：

- 严格相等 ("triple equals" 或 "identity")，使用`===`
- 宽松相等 ("double equals") ，使用`==`
- 以及`Object.is` (ECMAScript 2015/ ES6 新特性)

简而言之，在比较两件事情时，双等号将执行类型转换; 三等号将进行相同的比较，而不进行类型转换 (如果类型不同, 只是总会返回 false );  而`Object.is`的行为方式与三等号相同，但是对于NaN和-0和+0进行特殊处理，所以最后两个不相同，而`Object.is(NaN，NaN)`将为 true。

![各原始类型值的相等比较对照表](https://statics.sh1a.qingstor.com/2020/11/27/js_equals.png)

## 作用域

作用域就是变量与函数的可访问范围，即作用域控制着变量与函数的可见性和生命周期。在JavaScript中，变量的作用域有**全局作用域**和**局部作用域**两种。

### 全局作用域

在代码中任何地方都能访问到的对象拥有全局作用域。一般来说以下几种情形：

- 最外层函数和在最外层函数外面定义的变量拥有全局作用域。
- 所有未定义而直接赋值的变量自动声明为拥有全局作用域。
- 所有window对象的属性拥有全局作用域。如：`window.name`、`window.location`等。

> **注**：全局变量存在于程序的整个生命周期。没有块级作用域。

### 局部作用域

局部作用域一般只在固定的代码片段内可访问到，最常见的是在函数内部，所有在一些地方也会看到有人把这种作用域称为**函数作用域**。

### 作用域链

JavaScript里一切都是对象。函数对象和其它对象一样，拥有可以通过代码访问的属性和一系列仅供JavaScript引擎访问的内部属性。其中一个内部属性是`Scope`，该内部属性包含了函数被创建的作用域中对象的集合，这个集合被称为函数的作用域链，它决定了哪些数据能被函数访问。

因为全局变量总是存在于**运行时上下文**作用域链的最末端。所以，在标识符解析的时候，查找全局变量是最慢的。所以，在编写代码的时候应尽量少使用全局变量，尽可能使用局部变量。一个好的经验法则是：**如果一个跨作用域的对象被引用了一次以上，则先把它存储到局部变量里再使用**。

`with`语句主要用来临时扩展作用域链，将语句中的对象添加到作用域的头部。

```javascript
person = {name: "yhb", age: 22, height:175, wife: {name: "lwy", age: 21}};
with (person.wife) {
    console.log(name);
}
```

with语句将`person.wife`添加到当前作用域链的头部，所以输出的就是：`lwy`；with语句结束后，作用域链恢复正常。

> 当代码运行到with语句时，运行期上下文的作用域链临时被改变了。一个新的可变对象被创建，它包含了参数指定的对象的所有属性。这个对象将被推入作用域链的头部，这意味着函数的所有局部变量现在处于第二个作用域链对象中，因此访问代价更高了。
> **注**：在程序中应避免使用with语句。

## 闭包(Closures)

### 一个示例

如何从外部读取局部变量？

```javascript
function f1() {
    var n=999;
    function f2() {
        alert(n); // 999
    }
}
```

在上面的代码中，函数f2就被包括在函数f1内部，这时f1内部的所有局部变量，对f2都是可见的。但是反过来就不行，f2内部的局部变量，对f1就是不可见的。这就是Javascript语言特有的"链式作用域"结构（`chain scope`），子对象会一级一级地向上寻找所有父对象的变量。所以，**父对象的所有变量，对子对象都是可见的，反之则不成立**。

既然f2可以读取f1中的局部变量，那么只要把f2作为返回值，我们不就可以在f1外部读取它的内部变量了吗！

```javascript
function f1() {
    var n=999;
    function f2() {
        alert(n); // 999
    }
    return f2;
}
var result=f1();
result(); // 999
```

### 闭包解释

> **闭包定义**：闭包是一个函数和函数所声明的词法环境的结合。

在上面的代码中，f2函数就是闭包。**闭包**（`closure`）定义非常抽象，很难看懂。我的理解是，**闭包就是能够读取其他函数内部变量的函数**。在本质上，闭包就是将函数内部和函数外部连接起来的一座桥梁。

闭包最大用处有两个，一个是可以读取函数内部的变量，另一个就是让这些变量的值始终保持在内存中，不会在调用结束后被垃圾回收机制（`garbage collection`）回收。

### 立即执行函数表达式

有时你想模拟一个模拟块级作用域，例如你想将变量从全局作用域隔离。完成这个工作的模式叫做`IIFE`(立即执行函数表达式(`Immediately Invoked Function Expression`))：

```javascript
(function () {  // 块开始
    var tmp = ...;  // 非全局变量
}());  // 块结束
```

### 用闭包模拟私有方法

JavaScript 并不提供原生的支持私有方法，但是可以使用闭包模拟私有方法。私有方法不仅仅有利于限制对代码的访问：还提供了管理全局命名空间的强大能力，避免非核心的方法弄乱了代码的公共接口部分。

```javascript
var Counter = (function() {
    var privateCounter = 0;

    function changeBy(val) {
        privateCounter += val;
    }

    return {
        increment: function() {
            changeBy(1);
        },
        decrement: function() {
            changeBy(-1);
        },
        value: function() {
            return privateCounter;
        }
    }
})();

console.log(Counter.value()); /* logs 0 */
Counter.increment();
Counter.increment();
console.log(Counter.value()); /* logs 2 */
Counter.decrement();
console.log(Counter.value()); /* logs 1 */
```

上面创建了一个环境，为三个函数所共享：`Counter.increment`, `Counter.decrement`和`Counter.value`。该共享环境创建于一个匿名函数体内，该函数一经定义立刻执行。环境中包含两个私有项：名为`privateCounter`的变量和名为`changeBy`的函数。这两项都无法在匿名函数外部直接访问。必须通过匿名包装器返回的三个公共函数访问。

**注意**：

- 由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在IE中可能导致内存泄露。解决方法是，在退出函数之前，将不使用的局部变量全部删除。
- 闭包会在父函数外部，改变父函数内部变量的值。所以，如果你把父函数当作对象（object）使用，把闭包当作它的公用方法（Public Method），把内部变量当作它的私有属性（private value），这时一定要小心，不要随便改变父函数内部变量的值。

## 内存机制

首先JavaScript中的变量分为**基本类型**和**引用类型**。

- 基本类型就是保存在栈内存中的简单数据段。基本类型有`Undefined`、`Null`、`Boolean`、`Number`和`String`。这些类型在内存中分别占有固定大小的空间，他们的值保存在**栈空间**，我们通过按值来访问的。
- 引用类型指的是那些保存在堆内存中的对象。引用类型，值大小不固定，栈内存中存放地址指向堆内存中的对象。是按引用访问的。栈内存中存放的只是该对象的访问地址，在堆内存中为这个值分配空间。

### 为什么会有栈内存和堆内存之分？

与垃圾回收机制有关，为了使程序运行时占用的内存最小。

当一个方法执行时，每个方法都会建立自己的内存栈，在这个方法内定义的变量将会逐个放入这块栈内存里，随着方法的执行结束，这个方法的内存栈也将自然销毁了。因此，所有在方法中定义的变量都是放在栈内存中的；

当我们在程序中创建一个对象时，这个对象将被保存到运行时数据区中，以便反复利用（因为对象的创建成本通常较大），这个运行时数据区就是堆内存。堆内存中的对象不会随方法的结束而销毁，即使方法结束后，这个对象还可能被另一个引用变量所引用（方法的参数传递时很常见），则这个对象依然不会被销毁，只有当一个对象没有任何引用变量引用它时，系统的垃圾回收机制才会在核实的时候回收它。

### 垃圾回收机制

Javascript具有自动垃圾回收机制(`GC`:`Garbage Collecation`)，也就是说，执行环境会负责管理代码执行过程中使用的内存。

JavaScript垃圾回收的机制很简单：**找出不再使用的变量，然后释放掉其占用的内存，但是这个过程不是实时的，因为其开销比较大，所以垃圾回收器会按照固定的时间间隔周期性的执行**。

不再使用的变量也就是生命周期结束的变量，当然只可能是局部变量，全局变量的生命周期直至浏览器卸载页面才会结束。局部变量只在函数的执行过程中存在，而在这个过程中会为局部变量在栈或堆上分配相应的空间，以存储它们的值，然后在函数中使用这些变量，直至函数结束，而闭包中由于内部函数的原因，外部函数并不能算是结束。

#### 清除方式

- **标记清除**：垃圾回收器在运行的时候会给存储在内存中的所有变量都加上标记。然后，它会去掉环境中的变量以及被环境中的变量引用的变量的标记（闭包）。而在此之后再被加上标记的变量将被视为准备删除的变量，原因是环境中的变量已经无法访问到这些变量了。最后，垃圾回收器完成内存清除工作，销毁那些带标记的值并回收它们所占用的内存空间。
- **引用计数**：引用计数的含义是跟踪记录每个值被引用的次数。当声明了一个变量并将一个引用类型值赋给该变量时，则这个值的引用次数就是1。如果同一个值又被赋给另一个变量，则该值的引用次数加1。相反，如果包含对这个值引用的变量又取得了另外一个值，则这个值的引用次数减1。当这个值的引用次数变成0时，则说明没有办法再访问这个值了，因而就可以将其占用的内存空间回收回来。这样，当垃圾回收器下次再运行时，它就会释放那些引用次数为0的值所占用的内存。

## 原型(prototype)

原型是一个对象，其他对象可以通过它实现属性继承。JavaScript的对象中都包含了一个`Prototype`内部属性，这个属性所对应的就是该对象的原型。`Prototype`作为对象的内部属性，是不能被直接访问的。所以为了方便查看一个对象的原型，Firefox和Chrome中提供了`__proto__`这个非标准的访问器。

- 所有的对象都有`__proto__`属性，该属性对应着该对象的原型。
- 所有的函数对象都有`prototype`属性，该属性的值会被赋值给该函数创建的对象的`__proto__`属性
- 所有的原型对象都有`constructor`属性，该属性对应创建所有指向该原型的实例的构造函数
- 函数对象和原型对象通过`prototype`和`constructor`属性进行相互关联
- `Object`实例对象的原型`obj.__proto__`就是`Object.prototype`
- `hasOwnProperty`是`Object.prototype`的一个方法，该方法能判断一个对象是否包含自定义属性而不是原型链上的属性，因为"hasOwnProperty" 是 JavaScript 中唯一一个处理属性但是不查找原型链的函数

### 原型链

因为每个对象和原型都有原型，对象的原型指向对象的父，而父的原型又指向父的父，这种原型层层连接起来的就构成了原型链。

当通过原型链查找一个属性的时候，首先查找的是对象本身的属性，如果找不到才会继续按照原型链进行查找。这样一来，如果想要覆盖原型链上的一些属性，我们就可以直接在对象中引入这些属性，达到属性隐藏的效果。

## 对象创建方式

### 1. Object构造函数方式

```javascript
var Person = new Object();
Person.name = 'Nike';
Person.age = 29;
```

这行代码创建了`Object`引用类型的一个新实例，然后把实例保存在变量`Person`中。

### 2. 对象字面量方式

```javascript
var Person = {
 name: 'Nike';
 age: 29;
};
```

对象字面量是对象定义的一种简写形式，目的在于简化创建包含大量属性对象的过程。

> **注**：前两种方法的缺点在于：它们都是用了同一个接口创建很多对象，会产生大量的重复代码，就是如果你有100个对象，那你要输入100次很多相同的代码。那我们有什么方法来避免过多的重复代码呢，就是把创建对象的过程封装在函数体内，通过函数的调用直接生成对象。

### 3. 工厂模式

```javascript
function createPerson(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function() {
        alert(this.name);
    };
    return o;
}

var person1 = createPerson('Nike', 29, 'teacher');
```

在使用工厂模式创建对象的时候，我们都可以注意到，在`createPerson`函数中，返回的是一个对象。但我们就无法判断返回的对象究竟是一个什么样的类型。于是就出现了第四种创建对象的模式。

### 4. 构造函数方式

```javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function() {
        alert(this.name);
    };
}

var person1 = new Person('Nike', 29, 'teacher');
alert(person1 instanceof Object); //ture
```

对比工厂模式，我们可以发现以下区别：

- 没有显示地创建对象
- 直接将属性和方法赋给了`this`对象
- 没有`return`语句
- 终于可以识别的对象的类型。对于检测对象类型，我们应该使用instanceof操作符，我们来进行自主检测：

那么构造函数确实挺好用的，但是它也有它的缺点：就是每个方法都要在每个实例上重新创建一遍，方法指的就是我们在对象里面定义的函数。如果方法的数量很多，就会占用很多不必要的内存。于是出现了第五种创建对象的方法。

### 5. 原型创建对象模式

```javascript
function Person(){}
Person.prototype.name = 'Nike';
Person.prototype.age = 20;
Person.prototype.jbo = 'teacher';
Person.prototype.sayName = function() {
    alert(this.name);
};

var person1 = new Person();
var person2 = new Person();
person1.name = 'Greg';
alert(person1.name); //'Greg' --来自实例
alert(person2.name); //'Nike' --来自原型
```

当为对象实例添加一个属性时，这个属性就会屏蔽原型对象中保存的同名属性。

这时候我们就可以使用构造函数模式与原型模式结合的方式，构造函数模式用于定义实例属性，而原型模式用于定义方法和共享的属性。

### 6. 组合使用构造函数模式和原型模式

```javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
}
Person.prototype = {
    constructor: Person,
    sayName: function(){
        alert(this.name);
    };
}
var person1 = new Person('Nike', 20, 'teacher');
```

### 7. 动态原型模式

```javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;

    if (typeof this.sayName != 'function') {
        Person.prototype.sayName = function() {
            alert(this.name);
        }
    }
}

var person1 = new Person('Nike', 20, 'teacher');
person1.sayName();
```

动态原型模式将所有信息封装在了构造函数中，而通过构造函数中初始化原型（仅第一个对象实例化时初始化原型），这个可以通过判断该方法是否有效而选择是否需要初始化原型。

### 8. 寄生构造函数方式

```javascript
function Person(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function() {
        alert(this.name);
    };
    return o;
}

var person1 = new Person('Nike', 29, 'teacher');
```

寄生模式和工厂模式几乎一样，寄生模式和工厂模式的区别：

- 寄生模式创建对象时使用了`new`关键字
- 寄生模式的外部包装函数是一个构造函数

> **作用**:寄生模式可以在特殊的情况下为对象来创建构造函数,原因在于我们可以通过构造函数重写对象的值，并通过return返回。重写调用构造函数(创建的对象的实例)之后的对象实例的新的值。

### 9. 稳妥构造函数方式

```javascript
function Person(name, age, job) {
    var o = new Object();
    o.sayName = function() {
        alert(this.name);
    };
    return o;
}

var person = new Person('Nike', 29, 'teacher');
person.sayName(); // 使用稳妥构造函数模式只能通过其构造函数内部的方法来获取里面的属性值
```

道格拉斯·克拉克福德发明了JavaScript中的稳妥对象这个概念。所谓稳妥对象，是指没有公共属性，而且其方法也不引用`this`对象。稳妥对象最适合在一些安全环境中（这些环境会禁止使用`this`和`new`），或者在防止数据被其他应用程序改动时使用。稳妥构造函数遵循的与寄生构造函数类似的模式，但又两点不同：

- 一是新创建对象的实例方法不引用`this`；
- 二是不使用`new`操作符调用构造函数。

> **注**：与寄生构造函数模式类似，使用稳妥构造函数模式创建的对象与构造函数之间没有什么关系，因此instanceof操作符对这种对象也没有意义。

## 并发模型和事件循环(event loop)

JavaScript 的并发模型基于**事件循环**。

![Js堆栈队列图](https://statics.sh1a.qingstor.com/2020/11/27/js_event.png)

### 1. 运行时概念

#### 栈

函数调用形成了一个栈帧。

```javascript
function foo(b) {
  var a = 10;
  return a + b + 11;
}

function bar(x) {
  var y = 3;
  return foo(x * y);
}

console.log(bar(7));
```

当调用 bar 时，创建了第一个帧 ，帧中包含了 bar 的参数和局部变量。当 bar 调用 foo 时，第二个帧就被创建，并被压到第一个帧之上，帧中包含了 foo 的参数和局部变量。当 foo 返回时，最上层的帧就被弹出栈（剩下 bar 函数的调用帧 ）。当 bar 返回的时候，栈就空了。

#### 堆

对象被分配在一个堆中，即用以表示一个大部分非结构化的内存区域。

#### 队列

一个 JavaScript 运行时包含了一个待处理的消息队列。每一个消息都与一个函数相关联。当栈拥有足够内存时，从队列中取出一个消息进行处理。这个处理过程包含了调用与这个消息相关联的函数（以及因而创建了一个初始堆栈帧）。当栈再次为空的时候，也就意味着消息处理结束。

### 2. 事件循环

之所以称为**事件循环**，是因为它经常被用于类似如下的方式来实现：

```javascript
while (queue.waitForMessage()) {
  queue.processNextMessage();
}
```

如果当前没有任何消息，queue.waitForMessage 会等待着同步将要到来的消息。

每一个消息完整的执行后，其它消息才会被执行。这个模型的一个缺点在于当一个消息的完成耗时过长，网络应用无法处理用户的交互如点击或者滚动。浏览器用“程序需要过长时间运行”的对话框来缓解这个问题。一个比较好的解决方案是使消息处理变短且如果可能的话，将一个消息拆分成几个消息。

在浏览器里，当一个事件出现且有一个事件监听器被绑定时，消息会被随时添加。如果没有事件监听器，事件会丢失。所以点击一个附带点击事件处理函数的元素会添加一个消息。其它事件亦然。

### 3. 绝不阻塞

事件循环(event loop)模型特性在于它**永不阻塞**。通常由事件或者回调函数进行 I/O (input/output)处理 。

---

参考文档：

- [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript)