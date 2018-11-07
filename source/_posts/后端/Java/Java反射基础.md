---
title: Java反射基础
date: 2018-11-07 21:40:00
author: blinkfox
categories: 后端
tags:
  - Java
---

## 一、概述

### 1. 简介

Java反射(`Reflection`)机制就是在运行状态中，对于任意一个类，都能够知道这个类的属性和方法。对于任意一个对象能够调用它的任意一个属性和方法。这种动态获取的信息和动态调用对象的方法的功能称为Java语言的反射机制。Java程序中一般的对象的类型都是在编译期就确定下来的，而Java反射机制可以动态地创建对象并调用其属性，这样的对象的类型在编译期是未知的。所以我们可以通过反射机制直接创建对象，即使这个对象的类型在编译期是未知的。

反射的核心是JVM在**运行时**才动态加载类或调用方法/访问属性，它不需要事先（写代码的时候或编译期）知道运行对象是谁。反射机制就是通过`java.lang.Class`类来实现的，在Java中，Object 类是所有类的根类，而Class类就是描述Java类的类。

> **注**：因为Class类也是类，所以Object也包括Class类。

### 2. 主要功能

Java反射框架主要提供以下功能：

- 在运行时判断任意一个对象所属的类；
- 在运行时构造任意一个类的对象；
- 在运行时判断任意一个类所具有的成员变量和方法（通过反射甚至可以调用private方法）；
- 在运行时调用任意一个对象的方法；
- 修改构造函数、方法、属性的可见性。

### 3. 主要用途

**反射最重要的用途就是开发各种通用框架**。很多框架（比如Spring）都是配置化的（比如通过XML文件配置JavaBean,Action之类的），为了保证框架的通用性，它们可能需要根据配置文件加载不同的对象或类，调用不同的方法，这个时候就必须用到反射——运行时动态加载需要加载的对象。对与框架开发人员来说，反射虽小但作用非常大，它是各种容器实现的核心。

## 二、反射的使用

### 1. 获取Class对象

反射的各种功能都需要通过Class对象来实现，因此，需要知道如何获取Class对象，主要有以下几种方式。

#### 使用 Class.forName() 的静态方法

`Class.forName(String className)`方法可以通过类或接口的名称（一个字符串或完全限定名）来获取对应的Class对象。

```java
Class<?> cls = Class.forName("com.blinkfox.Zealot");
```

#### 直接获取某个类的class(最安全/性能最好)

```java
Class<String> cls = String.class;
```

#### 调用某个对象的 getClass() 方法

```java
Class<String> cls = str.getClass();
```

### 2. 判断是否为某个类的实例

一般地，我们用`instanceof`关键字来判断是否为某个类的实例。同时我们也可以借助反射中Class对象的`isInstance()`方法来判断是否为某个类的实例，它是一个Native方法：

```java
public native boolean isInstance(Object obj);
```

### 3. 创建实例

通过反射来生成对象主要有两种方式。

#### 使用Class对象的newInstance()方法

```java
Class<?> c = String.class;
Object str = c.newInstance();
```

#### 通过Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance()方法

```java
// 获取String所对应的Class对象
Class<?> c = String.class;
// 获取String类带一个String参数的构造器
Constructor constructor = c.getConstructor(String.class);
// 根据构造器创建实例
Object obj = constructor.newInstance("23333");
System.out.println(obj);
```

> **注**：这种方法可以用指定的构造器构造类的实例。

### 4. 获取方法

获取某个Class对象的方法集合，主要有以下几个方法：

- `getDeclaredMethods()`方法返回类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法。

```java
public Method[] getDeclaredMethods() throws SecurityException
```

- `getMethods()`方法返回某个类的所有公用（public）方法，包括其继承类的公用方法。

```java
public Method[] getMethods() throws SecurityException
```

- `getMethod()`方法返回一个特定的方法，其中第一个参数为方法名称，后面的参数为方法的参数对应Class的对象。

```java
public Method getMethod(String name, Class<?>... parameterTypes)
```

代码示例：

```java
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Test {

    public static void test() throws IllegalAccessException,
            InstantiationException, NoSuchMethodException, InvocationTargetException {
        Class<?> c = MethodClass.class;
        Object object = c.newInstance();
        Method[] methods = c.getMethods();
        Method[] declaredMethods = c.getDeclaredMethods();
        //获取MethodClass类的add方法
        Method method = c.getMethod("add", int.class, int.class);

        //getMethods()方法获取的所有方法
        System.out.println("getMethods获取的方法：");
        for(Method m: methods) {
            System.out.println(m);
        }

        //getDeclaredMethods()方法获取的所有方法
        System.out.println("getDeclaredMethods获取的方法：");
        for(Method m: declaredMethods) {
            System.out.println(m);
        }
    }
}

class MethodClass {

    public final int fuck = 3;

    public int add(int a, int b) {
        return a + b;
    }

    public int sub(int a, int b) {
        return a - b;
    }

}
```

> **注**：通过`getMethods()`获取的方法可以获取到父类的方法,比如`java.lang.Object`下定义的各个方法。

### 5. 获取构造方法

获取类构造器的用法与上述获取方法的用法类似。主要是通过Class类的`getConstructor`方法得到`Constructor`类的一个实例，而Constructor类有一个`newInstance`方法可以创建一个对象实例:

```java
public T newInstance(Object ... initargs)
```

> **注**：此方法可以根据传入的参数来调用对应的Constructor创建对象实例。

### 6. 获取类的成员变量信息

获取的方法同Method相似，主要是这几个方法，在此不再赘述：

- `Field getField(String name)`: 访问公有的成员变量。
- `Field[] getDeclaredFields()`：所有已声明的成员变量。但不能得到其父类的成员变量。
- `Field[] getFields()`和`Field[] getDeclaredFields()`用法同上。

### 7. 调用方法

当我们从类中获取了一个方法后，我们就可以用invoke()方法来调用这个方法。invoke方法的原型为:

```java
public Object invoke(Object obj, Object... args) throws IllegalAccessException, IllegalArgumentException,
 InvocationTargetException
```

代码示例：

```java
public class Test {

    public static void main(String[] args) throws IllegalAccessException,
            InstantiationException, NoSuchMethodException, InvocationTargetException {
        Class<?> klass = MethodClass.class;
        //创建 MethodClass 的实例
        Object obj = klass.newInstance();
        //获取 MethodClass 类的add方法
        Method method = klass.getMethod("add", int.class, int.class);
        //调用 method 对应的方法 => add(1,4)
        Object result = method.invoke(obj, 1, 4);
        System.out.println(result);
    }
}

class MethodClass {

    public final int fuck = 3;

    public int add(int a, int b) {
        return a + b;
    }

    public int sub(int a, int b) {
        return a - b;
    }

}
```

### 8. 利用反射创建数组

数组在Java里是比较特殊的一种类型，它可以赋值给一个`Object Reference`。下面我们看一看利用反射创建数组的例子：

```java
public static void testArray() throws ClassNotFoundException {
    // 使用`java.lang.reflect.Array`反射创建长度为25的字符串数组.
    Class<?> cls = Class.forName("java.lang.String");
    Object array = Array.newInstance(cls, 25);
    // 往数组里添加内容
    Array.set(array,0, "hello");
    Array.set(array,1, "Java");
    Array.set(array,2, "Go");
    Array.set(array,3, "Scala");
    Array.set(array,4, "Clojure");
    // 获取某一项的内容
    System.out.println(Array.get(array, 3));
}
```

## 三、使用反射获取信息

Class类提供了大量的实例方法来获取该Class对象所对应的详细信息，Class类大致包含如下方法，其中每个方法都包含多个重载版本，因此我们只是做简单的介绍，详细请参考[JDK文档](https://docs.oracle.com/javase/8/docs/api/java/lang/Class.html)。

### 1. 获取类内信息

- 构造器: `Constructor<T> getConstructor(Class<?>... parameterTypes)`
- 包含的方法: `Method getMethod(String name, Class<?>... parameterTypes)`
- 包含的属性: `Field getField(String name)`
- 包含的Annotation: `<A extends Annotation> A getAnnotation(Class<A> annotationClass)`
- 内部类: `Class<?>[] getDeclaredClasses()`
- 外部类: `Class<?> getDeclaringClass()`
- 所实现的接口: `Class<?>[] getInterfaces()`
- 修饰符: `int getModifiers()`
- 所在包: `Package getPackage()`
- 类名: `String getName()`
- 简称: `String getSimpleName()`

### 2. 判断类本身信息的方法

- 是否注解类型: `boolean isAnnotation()`
- 是否使用了该Annotation修饰: `boolean isAnnotationPresent(Class<? extends Annotation> annotationClass)`
- 是否匿名类: `boolean isAnonymousClass()`
- 是否数组: `boolean isArray()`
- 是否枚举: `boolean isEnum()`
- 是否原始类型: `boolean isPrimitive()`
- 是否接口: `boolean isInterface()`
- obj是否是该Class的实例: `boolean isInstance(Object obj)`

### 3. 使用反射获取泛型信息

为了通过反射操作泛型以迎合实际开发的需要, Java新增了`java.lang.reflect.ParameterizedType`、`java.lang.reflect.GenericArrayType`、`java.lang.reflect.TypeVariable`、`java.lang.reflect.WildcardType`几种类型来代表不能归一到Class类型但是又和原始类型同样重要的类型。

- `ParameterizedType`: 一种参数化类型, 比如Collection<String>
- `GenericArrayType`: 一种元素类型是参数化类型或者类型变量的数组类型
- `TypeVariable`: 各种类型变量的公共接口
- `WildcardType`: 一种通配符类型表达式, 如`?`、`? extends Number`、`? super Integer`

代码示例：

```java
public class Client {

    private Map<String, Object> objectMap;

    public void test(Map<String, User> map, String string) {
    }

    public Map<User, Bean> test() {
        return null;
    }

    /**
     * 测试属性类型
     *
     * @throws NoSuchFieldException
     */
    @Test
    public void testFieldType() throws NoSuchFieldException {
        Field field = Client.class.getDeclaredField("objectMap");
        Type gType = field.getGenericType();
        // 打印type与generic type的区别
        System.out.println(field.getType());
        System.out.println(gType);
        System.out.println("**************");
        if (gType instanceof ParameterizedType) {
            ParameterizedType pType = (ParameterizedType) gType;
            Type[] types = pType.getActualTypeArguments();
            for (Type type : types) {
                System.out.println(type.toString());
            }
        }
    }

    /**
     * 测试参数类型
     *
     * @throws NoSuchMethodException
     */
    @Test
    public void testParamType() throws NoSuchMethodException {
        Method testMethod = Client.class.getMethod("test", Map.class, String.class);
        Type[] parameterTypes = testMethod.getGenericParameterTypes();
        for (Type type : parameterTypes) {
            System.out.println("type -> " + type);
            if (type instanceof ParameterizedType) {
                Type[] actualTypes = ((ParameterizedType) type).getActualTypeArguments();
                for (Type actualType : actualTypes) {
                    System.out.println("\tactual type -> " + actualType);
                }
            }
        }
    }

    /**
     * 测试返回值类型
     *
     * @throws NoSuchMethodException
     */
    @Test
    public void testReturnType() throws NoSuchMethodException {
        Method testMethod = Client.class.getMethod("test");
        Type returnType = testMethod.getGenericReturnType();
        System.out.println("return type -> " + returnType);

        if (returnType instanceof ParameterizedType) {
            Type[] actualTypes = ((ParameterizedType) returnType).getActualTypeArguments();
            for (Type actualType : actualTypes) {
                System.out.println("\tactual type -> " + actualType);
            }
        }
    }
}
```

---

参考文档：[Java反射基础](http://www.sczyh30.com/posts/Java/java-reflection-1/)