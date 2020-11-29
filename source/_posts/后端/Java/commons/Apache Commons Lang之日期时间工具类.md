---
title: Apache Commons Lang之日期时间工具类
date: 2018-09-29 22:10:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/09/29/commons-datetime.jpg
categories: 后端
tags:
  - Java
  - Apache
---

> 码农不识Apache，码尽一生也枉然。

## FastDateFormat

`FastDateFormat`是一个快速且线程安全的时间操作类，它完全可以替代`SimpleDateFromat`。因为是线程安全的，所以你可以把它作为一个类的静态字段使用。构造方法为protected，不允许直接构造它的对象，可以通过工厂方法获取。FastDateFormat之所以是线程安全的，是因为这个类是无状态的：内部的成员在构造时就完成了初始化，并在对象存活期，不提供任何API供外界修改他们。

### getInstance(String pattern)

获取指定日期时间格式的`FastDateFormat`实例。

### format(Date date)

将日期时间格式化为字符串。

```java
FastDateFormat.getInstance("yyyy-MM-dd HH:mm:ss").format(new Date()); // 2017-06-03 23:32:31
```

### format(long millis)

同`format(Date date)`相似。

### format(Calendar calendar)

同`format(Date date)`相似。

## DateFormatUtils

将时间转化为字符串的工具类。不可实例化对象且线程安全，依赖于`FastDateFormat`。

### 预定义的日期格式

`DateFormatUtils`预定义的日期格式有如下几种：

```java
public static final FastDateFormat ISO_8601_EXTENDED_DATETIME_FORMAT = FastDateFormat.getInstance("yyyy-MM-dd'T'HH:mm:ss");
public static final FastDateFormat ISO_8601_EXTENDED_DATETIME_TIME_ZONE_FORMAT = FastDateFormat.getInstance("yyyy-MM-dd'T'HH:mm:ssZZ");
public static final FastDateFormat ISO_8601_EXTENDED_DATE_FORMAT = FastDateFormat.getInstance("yyyy-MM-dd");
public static final FastDateFormat ISO_8601_EXTENDED_TIME_FORMAT = FastDateFormat.getInstance("HH:mm:ss");
public static final FastDateFormat ISO_8601_EXTENDED_TIME_TIME_ZONE_FORMAT = FastDateFormat.getInstance("HH:mm:ssZZ");
public static final FastDateFormat SMTP_DATETIME_FORMAT = FastDateFormat.getInstance("EEE, dd MMM yyyy HH:mm:ss Z", Locale.US);
```

### format(Date date, String pattern)

将日期格式化为字符串。

```java
DateFormatUtils.format(new Date(), "yyyy-MM-dd HH:mm:ss"); // 2017-06-03 23:03:53
DateFormatUtils.ISO_8601_EXTENDED_DATETIME_FORMAT.format(new Date()); // 2017-06-03T23:09:52
DateFormatUtils.format(System.currentTimeMillis(), "yyyy-MM-dd HH:mm:ss"); // 2017-06-03 23:16:59
```

### format(long millis, String pattern)

同`format(Date date, String pattern)`相似。

### format(Calendar calendar, String pattern)

同`format(Date date, String pattern)`相似。

## DateUtils

`DateUtils`提供了很多很方便的功能，减轻了使用Date的复杂性。把原来需用`Calendar`才能完成的功能统一集中了起来，也就是说没有对应的`CalendarUtils`类。在JDK中，Date与Calendar概念本身就有些混淆，只是为了保持兼容性才引入的Calendar。相对于Calendar提供的方法，DateUtils提供了更加合理的方法，对时间的单个字段操作变得更加的容易。

### 常量

```java
public static final long MILLIS_PER_SECOND = 1000; // 1秒钟的毫秒数
public static final long MILLIS_PER_MINUTE = 60 * MILLIS_PER_SECOND; // 1分钟的毫秒数
public static final long MILLIS_PER_HOUR = 60 * MILLIS_PER_MINUTE; // 1小时的毫秒数
public static final long MILLIS_PER_DAY = 24 * MILLIS_PER_HOUR; // 1天的毫秒数
```

### boolean isSameDay(Date date1, Date date2)

判断两个日期是否是同一天。

```java
DateUtils.isSameDay(new Date(), new Date()); // true
```

### boolean isSameDay(Calendar cal1, Calendar cal2)

同`isSameDay(Date date1, Date date2)`相似。

### Date parseDate(String str, String... parsePatterns)

解析日期时间字符串日期时间Date对象，通过尝试各种不同的解析器来解析表示日期的字符串。

```java
DateUtils.parseDate("2017-06-03 23:51:44", "yyyy-MM-dd HH:mm:ss"); // 2017-06-03 23:51:44
DateUtils.parseDate("2017年06月03日 23时51分44秒", "yyyy-MM-dd HH:mm:ss", "yyyy年MM月dd日 HH时mm分ss秒");
```

### Date addYears(Date date, int amount)

得到`date`日期时间后（前）`amount`年后的日期时间。

```java
Date d3 = DateUtils.addYears(new Date(), 3); // 2020-06-04 00:06:21
Date d3 = DateUtils.addYears(new Date(), -2); // 2015-06-04 00:06:21
```

### Date addMonths(Date date, int amount)

同`addYears(Date date, int amount)`相似，对月份数进行加减。

### Date addWeeks(Date date, int amount)

同`addYears(Date date, int amount)`相似，对周数进行加减。

### Date addDays(Date date, int amount)

同`addYears(Date date, int amount)`相似，对天数进行加减。

### Date addHours(Date date, int amount)

同`addYears(Date date, int amount)`相似，对小时数进行加减。

### Date addMinutes(Date date, int amount)

同`addYears(Date date, int amount)`相似，对分钟数进行加减。

### Date addSeconds(Date date, int amount)

同`addYears(Date date, int amount)`相似，对秒数进行加减。

### Date addMilliseconds(Date date, int amount)

同`addYears(Date date, int amount)`相似，对毫秒数进行加减。

### Date setYears(Date date, int amount)

对给定的日期时间设置年份。

```java
Date d4 = DateUtils.setYears(new Date(), 2028); // 2028-06-04 00:16:48
```

### Date setMonths(Date date, int amount)

同`setYears(Date date, int amount)`相似，对月数进行设置。

### Date setDays(Date date, int amount)

同`setYears(Date date, int amount)`相似，对天数进行设置。

### Date setHours(Date date, int amount)

同`setYears(Date date, int amount)`相似，对小时数进行设置。

### Date setMinutes(Date date, int amount)

同`setYears(Date date, int amount)`相似，对分钟数进行设置。

### Date setSeconds(Date date, int amount)

同`setYears(Date date, int amount)`相似，对秒钟数进行设置。

### Date setMilliseconds(Date date, int amount)

同`setYears(Date date, int amount)`相似，对毫秒数进行设置。

### toCalendar(Date date)

将日期转为`Calendar`实例。

### Date round(Date date, int field)

对日期时间进行四舍五入。filed指定取整的字段，可以取的值为

- Calendar.SECOND
- Calendar.MINUTE
- Calendar.HOUR_OF_DAY
- Calendar.DAY_OF_MONTH
- Calendar.MONTH
- Calendar.YEAR
... 

```java
// 当前时间为'2017-06-04 00:44:41'，则执行以下代码
DateUtils.round(new Date(), Calendar.YEAR); // 2017-01-01 00:00:00
DateUtils.round(new Date(), Calendar.MONTH); // 2017-06-01 00:00:00
DateUtils.round(new Date(), Calendar.HOUR_OF_DAY); // 2017-06-04 01:00:00
DateUtils.round(new Date(), Calendar.DAY_OF_MONTH); // 2017-06-04 00:00:00
DateUtils.round(new Date(), Calendar.HOUR); // 2017-06-04 01:00:00
DateUtils.round(new Date(), Calendar.MINUTE); // 2017-06-04 00:45:00
DateUtils.round(new Date(), Calendar.SECOND); // 2017-06-04 00:44:43
```

### Date truncate(Date date, int field)

从给定字段开始格式化截取日期。对一个时间对象的某个字段进行截断。

```java
// 当前时间为'2017-06-04 00:56:05'，则执行以下代码
DateUtils.truncate(new Date(), Calendar.YEAR); // 2017-01-01 00:00:00
DateUtils.truncate(new Date(), Calendar.MONTH); // 2017-06-01 00:00:00
DateUtils.truncate(new Date(), Calendar.HOUR_OF_DAY); // 2017-06-04 00:00:00
DateUtils.truncate(new Date(), Calendar.DAY_OF_MONTH); // 2017-06-04 00:00:00
DateUtils.truncate(new Date(), Calendar.HOUR); // 2017-06-04 00:00:00
DateUtils.truncate(new Date(), Calendar.MINUTE); // 2017-06-04 00:56:00
DateUtils.truncate(new Date(), Calendar.SECOND); // 2017-06-04 00:56:05
```

### Date ceiling(Date date, int field)

从给定字段开始“向上”格式化日期。

```java
// 当前时间为'2017-06-04 01:02:31'，则执行以下代码
DateUtils.ceiling(new Date(), Calendar.YEAR); // 2018-01-01 00:00:00
DateUtils.ceiling(new Date(), Calendar.MONTH); // 2017-07-01 00:00:00
DateUtils.ceiling(new Date(), Calendar.HOUR_OF_DAY); // 2017-06-04 02:00:00
DateUtils.ceiling(new Date(), Calendar.DAY_OF_MONTH); // 2017-06-05 00:00:00
DateUtils.ceiling(new Date(), Calendar.HOUR); // 2017-06-04 02:00:00
DateUtils.ceiling(new Date(), Calendar.MINUTE); // 2017-06-04 01:03:00
DateUtils.ceiling(new Date(), Calendar.SECOND); // 2017-06-04 01:02:32
```

### long getFragmentInDays(Date date, int fragment)

返回一个指定时间的天数。关键的是参数`fragment`，它的作用非常重要。它的值必须是Calendar的时间常量字段。

**注意**：小时必须用24小时制的，即`Calendar.HOUR_OF_DAY`，而不能用`Calendar.HOUR`字段。

```java
// 当前时间为'2017-06-04 01:12:31'，则执行以下代码
DateUtils.getFragmentInDays(new Date(), Calendar.YEAR); // 155
DateUtils.getFragmentInDays(new Date(), Calendar.MONTH); // 4
```

### long getFragmentInMilliseconds(Date date, int fragment)

同`getFragmentInDays(Date date, int fragment)`相似。

### long getFragmentInSeconds(Date date, int fragment)

同`getFragmentInDays(Date date, int fragment)`相似。


### long getFragmentInMinutes(Date date, int fragment)

同`getFragmentInDays(Date date, int fragment)`相似。

### long getFragmentInHours(Date date, int fragment)

同`getFragmentInDays(Date date, int fragment)`相似。

### boolean truncatedEquals(Date date1, Date date2, int field)

比较日历对应字段是否相等。

## StopWatch

`StopWatch`是一个方便的计时器。

### 使用示例

```java
StopWatch stopWatch = new StopWatch();
stopWatch.start();
...
stopWatch.stop();
System.out.println(stopWatch.getTime());
```

### 主要方法：

- `start()`: 开始计时
- `stop()`: 停止计时
- `reset()`: 重置计时
- `suspend()`: 暂停计时
- `resume()`: 继续计时
- `getTime()`: 获取消耗的毫秒数
- `getNanoTime()`: 获取消耗的纳秒数
- `getStartTime()`: 获取开始的毫秒数
- `isStarted()`: 是否开始
- `isSuspended()`: 是否暂停
- `isStopped()`: 是否停止