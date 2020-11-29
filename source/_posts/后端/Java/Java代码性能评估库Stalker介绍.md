---
title: Java代码性能评估库Stalker介绍
date: 2019-02-14 12:00:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2019/02/14/stalker.jpg
cover: true
categories: 后端
tags:
  - Java
  - 性能测试
---

[English Document](https://github.com/blinkfox/stalker/blob/master/README.md)

> 这是一个简单的用来对Java代码做性能评估的工具库。

## 特性

- 轻量级（jar包仅`26kb`）
- API简单易用
- 易于集成或扩展

## Maven集成

```xml
<dependency>
    <groupId>com.blinkfox</groupId>
    <artifactId>stalker</artifactId>
    <version>1.0.0</version>
</dependency>
```

## API 介绍和使用

### 预先准备

在对Java方法做性能测试之前，先准备好待测试的类和方法：

```java
/**
 * 用于测量（仅测试使用）该类中的方法的执行耗时的类.
 *
 * @author blinkfox on 2019-02-03.
 */
public class MyTestService {

    private static final Logger log = LoggerFactory.getLogger(MyTestService.class);

    /**
     * 测试方法1，模拟业务代码耗时 2~5 ms，且会有约 1% 的几率执行异常.
     */
    public void hello() {
        // 模拟运行时抛出异常.
        if (new Random().nextInt(100) == 5) {
            throw new MyServiceException("My Service Exception.");
        }

        // 模拟运行占用约 2~5 ms 的时间.
        this.sleep(2L + new Random().nextInt(3));
    }

    /**
     * 测试方法2，模拟业务代码运行占用约 2 ms 的时间.
     */
    public void fastHello() {
        this.sleep(2L);
    }

    /**
     * 本线程调用该方法时，睡眠指定时间，用来模拟业务耗时.
     *
     * @param time 时间
     */
    private void sleep(long time) {
        try {
            Thread.sleep(time);
        } catch (InterruptedException e) {
            log.info("InterruptedException", e);
            Thread.currentThread().interrupt();
        }
    }

}
```

### Stalker类

#### 1. 最简示例

以下代码将会预热`5`次，然后在单线程下正式执行`10`次，从而将运行结果计算统计并输出出来：

```java
public static void main(String[] args) {
    Stalker.run(() -> new MyTestService().hello());
}
```

以上结果将默认在控制台输出：

```bash
+-----------------------------------------------------------------------------------------------------------------------------------------+
|                                  threads: 1, concurrens: 1, warmups:5, runs: 10, printErrorLog: false                                   |
+---+----------+-------+---------+---------+----------+---------+---------+---------+---------+---------------------+---------------------+
|   |  Costs   | Total | Success | Failure |   Sum    |   Avg   |   Min   |   Max   | StdDev  | 95% LowerConfidence | 95% UpperConfidence |
+---+----------+-------+---------+---------+----------+---------+---------+---------+---------+---------------------+---------------------+
| 1 | 35.33 ms |  10   |   10    |    0    | 35.29 ms | 3.53 ms | 2.56 ms | 4.81 ms | 0.85 ms |       3.0 ms        |       4.06 ms       |
+---+----------+-------+---------+---------+----------+---------+---------+---------+---------+---------------------+---------------------+
```

#### 2. 更全示例

以下代码表示，两个方法`hello()`和`fastHello()`将会预热`1000`次，在`1000`个线程`200`个并发下，每次执行`10`次：

```java
Stalker.run(Options.of(1000, 200).warmups(1000).runs(10),
        () -> new MyTestService().hello(),
        () -> new MyTestService().fastHello());
```

以上结果将默认在控制台输出：

```bash
+------------------------------------------------------------------------------------------------------------------------------------------+
|                               threads: 1000, concurrens: 200, warmups:1000, runs: 10, printErrorLog: false                               |
+---+-----------+-------+---------+---------+---------+---------+---------+----------+---------+---------------------+---------------------+
|   |   Costs   | Total | Success | Failure |   Sum   |   Avg   |   Min   |   Max    | StdDev  | 95% LowerConfidence | 95% UpperConfidence |
+---+-----------+-------+---------+---------+---------+---------+---------+----------+---------+---------------------+---------------------+
| 1 | 454.33 ms | 10000 |  9900   |   100   | 36.79 s | 3.72 ms | 2.01 ms | 11.89 ms | 1.31 ms |       3.69 ms       |       3.74 ms       |
| 2 | 159.94 ms | 10000 |  10000  |    0    | 21.72 s | 2.17 ms | 2.01 ms | 3.24 ms  | 0.15 ms |       2.17 ms       |       2.18 ms       |
+---+-----------+-------+---------+---------+---------+---------+---------+----------+---------+---------------------+---------------------+
```

结果说明：

- `Costs`: 实际正式运行所消耗的总时间
- `Total`: 正式运行的总次数
- `Success`: 正式运行的成功次数
- `Failure`: 正式运行的失败次数
- `Sum`: 每次运行的耗时结果求和之后的值
- `Avg`: 所有运行耗时结果的算术平均数
- `Min`: 所有运行耗时结果中最小值
- `Max`: 所有运行耗时结果中最大值
- `StdDev`: 所有运行耗时结果的标准方差
- `95% LowerConfidence`: 95%置信区间的最小边界值
- `95% LowerConfidence`: 95%置信区间的最大边界值

#### 3. 主要方法

- `void run(Runnable... runnables)`: 对若干个要执行的代码做性能测量评估.
- `void run(Options options, Runnable... runnables)`: 通过自定义的`Options`对若干个要执行的代码做性能测量评估.

### Options类

Options表示做性能测量时的选项参数

#### 主要属性如下

- `name`: 选项参数的名称
- `threads`: 正式执行的线程数，默认为1。
- `concurrens`: 正式多线程下执行的并发数，默认为1。
- `warmups`: 单线程下的预热次数，默认5。
- `runs`: 每个线程正式执行的次数，默认10。
- `printErrorLog`: 是否打印错误日志，默认false。
- `outputs`: 将测量结果通过多种方式(集合)输出出来，默认为输出到控制台，可自定义实现`MeasureOutput`接口。

#### 主要方法

以下是构造`Options`实例的若干重载方法：

- `Options of(String name)`
- `Options of(int runs)`
- `Options of(String name, int runs)`
- `Options of(int threads, int concurrens)`
- `Options of(String name, int threads, int concurrens)`
- `Options of(String name, int threads, int concurrens, int runs)`

其他方法：

- `boolean valid()`: 校验Options相关参数是否合法
- `Options named(String name)`: 设置 Options 实例的 name 属性
- `Options threads(int threads)`: 设置 Options 实例的 threads 属性
- `Options concurrens(int concurrens)`: 设置 Options 实例的 concurrens 属性
- `Options warmups(int warmups)`: 设置 Options 实例的 warmups 属性
- `Options runs(int runs)`: 设置 Options 实例的 runs 属性
- `Options printErrorLog(boolean printErrorLog)`: 设置 Options 实例的 printErrorLog 属性
- `Options outputs(MeasureOutput... measureOutputs)`: 自定义设置 Options 实例的 MeasureOutput 输出通道

### Assert类

Assert类主要用来做断言使用。

#### 示例

```java
Assert.assertFaster(Options.of(),
        () -> new MyTestService().fastHello(),
        () -> new MyTestService().hello());
```

## 许可证

本 [stalker](https://github.com/blinkfox/stalker) 类库遵守 [Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0) 许可证。