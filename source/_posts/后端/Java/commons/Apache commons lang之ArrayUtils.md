---
title: Apache Commons Lang之ArrayUtils
date: 2018-09-28 23:09:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/09/28/feature.jpg
categories: 后端
tags:
  - Java
  - Apache
---

> 码农不识Apache，码尽一生也枉然。

## 常量数组

```java
public static final Object[] EMPTY_OBJECT_ARRAY = new Object[0];
public static final Class<?>[] EMPTY_CLASS_ARRAY = new Class[0];
public static final String[] EMPTY_STRING_ARRAY = new String[0];
public static final long[] EMPTY_LONG_ARRAY = new long[0];
public static final Long[] EMPTY_LONG_OBJECT_ARRAY = new Long[0];
public static final int[] EMPTY_INT_ARRAY = new int[0];
public static final Integer[] EMPTY_INTEGER_OBJECT_ARRAY = new Integer[0];
```

## 转换为Map

### toMap(Object[] array)

将二维数组转换为Map。

```java
Map colorMap = ArrayUtils.toMap(new String[][] {
    {"RED", "#FF0000"},
    {"GREEN", "#00FF00"},
    {"BLUE", "#0000FF"}
});
```

## 生成数组

### T[] toArray(final T... items)

将不定参数转换为数组。

```java
String[] array = ArrayUtils.toArray("1", "2");
String[] emptyArray = ArrayUtils.<String>toArray();
```

## null转空数组

### Object[] nullToEmpty(Object[] array)

将null数组转为对应类型的空数组，如果array不是null，则返回array。

```java
String[] arr = ArrayUtils.nullToEmpty((String[]) null);
```

## 数组操作

### <T> T[] subarray(T[] array, int startIndexInclusive, int endIndexExclusive)

截取数组开始索引位置和结束索引位置的数组为子数组

```java
Object[]s1=ArrayUtils.subarray(newObject[]{"aa","bb","cc"},0,1); // ["aa"]
Object[]s2=ArrayUtils.subarray(newObject[]{"aa","bb","cc"},0,2); // ["aa", "bb"]
```

### reverse(long[] array)

反转数组。

```java
ArrayUtils.reverse(new String[]{"aa","bb"});//结果是：{"bb"，"aa"}
```

### swap(Object[] array, int offset1, int offset2)

交换数组中的元素。

```java
ArrayUtils.swap(["1", "2", "3"], 0, 2) -> ["3", "2", "1"]
ArrayUtils.swap(["1", "2", "3"], 0, 0) -> ["1", "2", "3"]
ArrayUtils.swap(["1", "2", "3"], 1, 0) -> ["2", "1", "3"]
ArrayUtils.swap(["1", "2", "3"], 0, 5) -> ["1", "2", "3"]
ArrayUtils.swap(["1", "2", "3"], -1, 1) -> ["2", "1", "3"]
```

## 数组元素查找

### int indexOf(Object[] array, Object objectToFind)

数组元素所在的索引位置,如果没有则返回-1,可指定起始搜索位置。

```java
ArrayUtils.indexOf(new String[]{"aa","bb","cc"},"cc"); // 2
ArrayUtils.indexOf(new String[]{"aa","bb","bb"},"bb",2); // 2
ArrayUtils.indexOf(newObject[]{"aa","bb","cc"},"cc",3); // -1
```

### int lastIndexOf(Object[] array, Object objectToFind, int startIndex)

同`indexOf(Object[] array, Object objectToFind)`相反。反向查询某个object在数组中的位置，可以指定起始搜索位置。

### contains(Object[] array, Object objectToFind)

判断数组中是否包含某个元素。

```java
ArrayUtils.contains(new String[]{"a", "b", "c"}, "a"); // true
```

## 数组判空

### boolean isEmpty(Object[] array)

判断数组是否为空。

```java
ArrayUtils.isEmpty(new String[]{"21","是"}); // false
ArrayUtils.isEmpty(new String[]{""}); // false
ArrayUtils.isEmpty(new String[]{null}); // false
ArrayUtils.isEmpty(new String[]{}); // true
```

### <T> boolean isNotEmpty(T[] array)

同``相反。判断数组是否不为空。

## 合并数组元素

### <T> T[] addAll(T[] array1, T... array2)

合并多个数组到某一个数组中。

```java
ArrayUtils.addAll(null, null)     = null
ArrayUtils.addAll(array1, null)   = cloned copy of array1
ArrayUtils.addAll(null, array2)   = cloned copy of array2
ArrayUtils.addAll([], [])         = []
ArrayUtils.addAll([null], [null]) = [null, null]
ArrayUtils.addAll(["a", "b", "c"], ["1", "2", "3"]) = ["a", "b", "c", "1", "2", "3"]
```

### <T> T[] add(T[] array, T element)

将单个元素合并到数组中。

```java
ArrayUtils.add(null, null)      = IllegalArgumentException
ArrayUtils.add(null, "a")       = ["a"]
ArrayUtils.add(["a"], null)     = ["a", null]
ArrayUtils.add(["a"], "b")      = ["a", "b"]
ArrayUtils.add(["a", "b"], "c") = ["a", "b", "c"]
```

### <T> T[] add(T[] array, int index, T element)

将单个元素合并到指定索引位置的数组中。

```java
ArrayUtils.add(null, 0, null)      = IllegalArgumentException
ArrayUtils.add(null, 0, "a")       = ["a"]
ArrayUtils.add(["a"], 1, null)     = ["a", null]
ArrayUtils.add(["a"], 0, "b")      = ["b", "a"]
ArrayUtils.add(["a", "b"], 1, "c") = ["a", "c", "b"]
```

## 移除数组元素

### <T> T[] remove(T[] array, int index)

移除数组中指定索引位置的元素。

```java
ArrayUtils.remove(["a"], 0)           = []
ArrayUtils.remove(["a", "b"], 0)      = ["b"]
ArrayUtils.remove(["a", "b"], 1)      = ["a"]
ArrayUtils.remove(["a", "b", "c"], 1) = ["a", "c"]
```

### <T> T[] removeAll(T[] array, int... indices)

同`<T> T[] remove(T[] array, int index)`相似，移除数组中所有指定索引位置的元素。

```java
ArrayUtils.removeAll(["a", "b", "c"], 0, 2) = ["b"]
ArrayUtils.removeAll(["a", "b", "c"], 1, 2) = ["a"]
```

### <T> T[] removeElement(T[] array, Object element)

移除数组中的第一个element元素。

```java
ArrayUtils.removeElement(null, "a")            = null
ArrayUtils.removeElement([], "a")              = []
ArrayUtils.removeElement(["a"], "b")           = ["a"]
ArrayUtils.removeElement(["a", "b"], "a")      = ["b"]
ArrayUtils.removeElement(["a", "b", "a"], "a") = ["b", "a"]
```