---
title: Fenix — 比 MyBatis 更加强大的 Spring Data JPA 扩展库
date: 2019-08-20 00:10:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2019/08/20/jpa.png
cover: true
categories: 后端
tags:
  - Java
  - JPA
  - Fenix
---

> [Fenix](https://github.com/blinkfox/fenix)（菲尼克斯）是一个比 MyBatis 更加强大，为解决复杂、动态 SQL (`JPQL`) 而生的 `Spring Data JPA` 扩展库，目的是辅助开发者更方便、快捷的书写复杂、动态且易于维护的 SQL，支持 `XML` 和 Java 链式 `API` 两种方式来书写动态 SQL。

- [使用文档: https://blinkfox.github.io/fenix](https://blinkfox.github.io/fenix)

## 特性

- 简单、轻量级、无副作用的集成和使用；
- 作为 JPA 的扩展和增强，兼容 Spring Data JPA 的各种特性；
- 提供了 `XML` 和纯 Java API 两种方式来书写 SQL；
- `XML` 的方式功能强大，让 SQL 和 Java 代码解耦，易于维护；
- 也可以采用 Java 链式 `API` 来书写动态 SQL；
- 具有动态性、极致的可复用性和可调试性的优点；
- 具有可扩展性，可自定义 `XML` 语义标签和对应的标签处理器来生成自定义逻辑的 SQL 片段和参数；

## 初衷

随着 [Spring Data JPA](https://spring.io/projects/spring-data-jpa) 越来越流行，极大的方便了数据的“增删改”和简单查询的场景，但是在复杂、动态查询方面就显得有些“糟糕”了，相比 `MyBatis` 的 `XML` 动态 SQL 而言，缺少了一定优雅和可维护性。

所以，为了能使开发人员能像在 `MyBatis` 中那样在 `XML` 中书写 `JPQL` 语句，Fenix 中引入了 [MVEL](http://mvel.documentnode.com/) 表达式和模板引擎的语法来书写和渲染 XML 中的动态 SQL。通俗的说，就是支持使用表达式、`if/else`、`foreach` 等来达到跟 MyBatis 类似的动态 SQL 能力。但是，仅靠这些“灵活”的动态能力，仍然会书写出大量相似或重复的 SQL。

因此，为了更加极致的解决 SQL 片段“**相似或重复**”的问题，Fenix 中引入了 SQL 片段的“**语义化标签**”，将大多数常见的 SQL 片段做成 `XML` 标签，通过传递的字段和动态的参数值就可以生成对应的 SQL 片段和命名参数。语言化的 `XML` 标签可以在各个需要的地方复用，也支持自定义你自己的 XML SQL 语义标签。

为了便于开发人员书写一般中短长度的动态 SQL，Fenix 还提供了 Java 链式 `API` 书写动态 SQL 的方式，使 SQL 可读性和紧凑性更好，如果要书写静态或动态的中、长 SQL，则推荐使用 `XML` 方式，便于集中阅读、调试和维护 SQL。

> **注**：本 `Fenix` 扩展库开发的核心思想来源于我几年前写的动态 SQL 拼接库 [Zealot](https://github.com/blinkfox/zealot)。如果你熟悉《星际争霸》的话，大概能理解其中的关系。

## 与 MyBatis 的 SQL 比较

### 假设业务查询场景

下面将通过一个多条件查询**操作日志**的功能，来初步了解和比较 `MyBatis` 与 `Fenix` 在写“**多条件模糊分页**”查询时 SQL 写法的一些差异。

![查询页面](https://statics.sh1a.qingstor.com/2019/08/20/search.png)

由于是查询的场景，上面的几个查询条件都是**非必填**的，字段含义解释如下：

- **操作名称**：数据库字段类型为 `String` 型，根据输入的名称来进行**模糊查询**（`LIKE`）；
- **操作类型**：数据库字段类型为 `int` 型，可以下拉选择多个选项来进行**范围查询**（`IN`）；
- **操作结果**：数据库字段类型为 `int` 型，只能下拉选择一个选项值来进行**等值查询**（`=`）；
- **操作时间**：数据库字段类型为 `datetime` 型，可以选择开始时间或者结束时间来进行**区间查询**（`BETWEEN ? AND ?`、`>=`、`<=`）；

### MyBatis 的 SQL 写法

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.blinkfox.example.repository.mapper.OperationLogMapper">

    <!-- MyBatis 映射字段为 Bean 的 resultMap. -->
    <resultMap id="operationLogMap" type="com.blinkfox.example.repository.pojo.OperationLog">
        <id column="c_id" property="id"/>
        <result column="c_title" property="title"/>
        <result column="n_type" property="type"/>
        <result column="n_result" property="result"/>
        <result column="dt_create_time" property="createTime"/>
        <result column="c_description" property="description"/>
    </resultMap>

    <!-- MyBatis 动态查询操作日志的 SQL. -->
    <select id="queryOperationLogs" resultMap="operationLogMap">
        SELECT
            ol.c_id,
            ol.c_title,
            ol.n_type,
            ol.n_result,
            ol.dt_create_time,
            ol.c_description
        FROM
            t_operation_log AS ol
        <trim prefix="WHERE" suffix="" suffixOverrides="AND">
            <if test="log.result != null and log.result != 0">
                ol.n_result = #{log.result} AND
            </if>
            <if test="log.title != null and log.title != ''">
                ol.c_title like CONCAT('%', #{log.title}, '%') AND
            </if>
            <if test="log.typeList != null">
                ol.n_type in
                <foreach collection="log.typeList" index="index" item="item" open="(" separator="," close=")">
                    #{item}
                </foreach>
                AND
            </if>
            <if test="log.startTime != null and log.endTime != null">
                ol.dt_create_time BETWEEN #{log.startTime} AND #{log.endTime} AND
            </if>
            <if test="log.startTime != null and log.endTime == null">
                ol.dt_create_time &gt;= #{log.startTime} AND
            </if>
            <if test="log.startTime == null and log.endTime != null">
                ol.dt_create_time &lt;= #{log.endTime} AND
            </if>
        </trim>
    </select>

</mapper>
```

### Fenix 的 SQL 写法

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 操作日志的 SQL 仓库. -->
<fenixs namespace="OperationLogRepository">

    <!-- 多条件模糊分页查询操作日志的示例 SQL. -->
    <fenix id="queryOperationLogs" removeIfExist="1 = 1 AND ">
        SELECT
            ol.id,
            ol.title,
            ol.type,
            ol.result,
            ol.createTime,
            ol.description
        FROM
            OperationLog AS ol
        WHERE
            1 = 1
        <andLike field="ol.title" value="log.title" match="log.title != empty"/>
        <andIn field="ol.type" value="log.typeList" match="log.typeList != empty"/>
        <andEqual field="ol.result" value="log.result" match="log.result != empty"/>
        <andBetween field="ol.createTime" start="log.startTime" end="log.endTime" match="(log.startTime != empty) || (log.endTime != empty)"/>
    </fenix>

</fenixs>
```

### MyBatis 与 Fenix 的比较总结

`MyBatis` 和 `Fenix` 的 SQL 有以下几个差异点：

- MyBatis 只能写原生 SQL，无法享受跨数据库时的兼容性；由于 Fenix 是基于 Spring Data JPA 的扩展，即可以写 `JPQL` 语句，也可以写原生 `SQL` 语句，上述示例中写的是 `JPQL` 语句，SQL 的字段表达上更简洁。
- MyBatis 书写动态 SQL 依赖只能 `if/else`、`foreach` 等分支循环操作，灵活性高，但是代码量和重复性较高；而 Fenix 也有 `if/else`、`foreach` 等分支循环操作，但内置了大量的更加简单、强大和语义化的 XML [SQL 标签](xml/xml-tags)，使用语义化的 SQL 标签，使得 SQL 的语义简单明了，再通过 `match` 属性的值来确定是否生成此条 SQL，来达到动态性。
- MyBatis 通过 `trim` 标签消除 `WHERE` 语句后的 `1 =1 AND`，而 `Fenix` 是通过在 `<fenix />` 节点中声明 `removeIfExist` 属性（非必填）来声明式的消除。
- MyBatis 的动态 SQL 解析引擎是 [OGNL](http://commons.apache.org/proper/commons-ognl/)，而 Fenix 的解析引擎是 [MVEL](http://mvel.documentnode.com/)，功能和性能上都更优一些。

> **总结**：通过以上 MyBatis 和 Fenix 的各自 SQL 写法比较来看，`Fenix` 的 SQL 在**动态性**、**简介性**和**SQL 语义化**等方面，都更加强大。

## 支持场景

适用于 Java `Spring Data JPA` 项目，`JDK 1.8` 及以上。

## Spring Boot 项目集成

如果你是 Spring Boot 项目，那么直接集成 `fenix-spring-boot-starter` 库，并激活 `FenixJpaRepositoryFactoryBean`。

> **注**：如果不是 Spring Boot 项目，请参看[这里](https://blinkfox.github.io/fenix/#/quick-install?id=not-spring-boot-project)。

### Maven

```xml
<dependency>
    <groupId>com.blinkfox</groupId>
    <artifactId>fenix-spring-boot-starter</artifactId>
    <version>1.0.1</version>
</dependency>
```

### Gradle

```bash
compile 'com.blinkfox:fenix-spring-boot-starter:1.0.1'
```

### 激活 Fenix FactoryBean

然后需要在你的 Spring Boot 应用的 `@EnableJpaRepositories` 注解中，配置
`repositoryFactoryBeanClass` 的属性值为 `FenixJpaRepositoryFactoryBean.class`。

```java
/**
 * 请在 Spring Boot 应用中配置 {@link EnableJpaRepositories#repositoryFactoryBeanClass}
 * 的值为 {@link FenixJpaRepositoryFactoryBean}.
 *
 * @author blinkfox on 2019-08-15.
 */
@EnableJpaRepositories(repositoryFactoryBeanClass = FenixJpaRepositoryFactoryBean.class)
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

> **注**： `FenixJpaRepositoryFactoryBean` 继承自 Spring Data JPA 默认的 `JpaRepositoryFactoryBean`。所以，Fenix 与 JPA 的各种注解和特性完全兼容，并提供了更加强大的 `@QueryFenix` 注解。

### application.yml 配置项

要修改 Fenix 的配置信息，你需要在你的 Spring Boot 项目中，在 `application.yml` 或者 `application.properties` 中去修改配置信息。

以下通过 `application.yml` 文件来说明 Fenix 中的几个配置项、默认值和说明信息，供你参考。

```yaml
# Fenix 的几个配置项、默认值及详细说明，通常情况下你不需要填写这些配置信息.
fenix:
  # 成功加载 Fenix 配置信息后，是否打印启动 banner，默认 true.
  print-banner: true
  # 是否打印 Fenix 生成的 SQL 信息，默认为空.
  # 当该值为空时，会读取 'spring.jpa.show-sql' 的值，为 true 就打印 SQL 信息，否则不打印.
  # 当该值为 true 时，就打印 SQL 信息，否则不打印. 生产环境不建议设置为 true.
  print-sql:
  # 扫描 Fenix XML 文件的所在位置，默认是 fenix 目录及子目录，可以用 yaml 文件方式配置多个值.
  xml-locations: fenix
  # 扫描你自定义的 XML 标签处理器的位置，默认为空，可以是包路径，也可以是 Java 或 class 文件的全路径名
  # 可以配置多个值，不过一般情况下，你不自定义自己的 XML 标签和处理器的话，不需要配置这个值.
  handler-locations:
```

## 开源许可证

本 `Fenix` 的 Spring Data JPA 扩展库遵守 [Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0) 许可证。
