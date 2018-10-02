---
title: PostgreSQL基础知识整理
date: 2018-09-25 00:33:00
author: blinkfox
img: http://static.blinkfox.com/hexoblog_20180925_sql.jpg
categories: 数据库
tags:
  - PostgreSQL
  - 数据库
---

## 数据库操作DML

### CREATE TABLE

`CREATE TABLE`语句用于创建数据库中的表。语法是：

```sql
CREATE TABLE 表名称 (
列名1 数据类型,
列名2 数据类型,
列名3 数据类型,
....
);
```

例：

```sql
CREATE TABLE user (
id integer,
user_name varchar(255),
email varchar(255),
age integer,
address varchar(255)
)
```

**注意**：字段类型`char`和`varchar`区别：

- 容纳固定长度的字符串。
- 容纳可变长度的字符串。

### CONSTRAINT

约束、限制，常见的约束如下：

- `CHECK`(检查约束)
- `NOT NULL`(非空约束)
- `UNIQUE`(唯一约束)
- `Primary Key`(主键)
- `Foreign Key`(外键)

例：

```sql
-- 各种约束的使用示例
CREATE TABLE user (
    id integer NOT NULL PRIMARY KEY,
    corp_id integer REFERENCES corp (id),
    user_name text UNIQUE,
    age numeric CHECK (age > 0)
);
```

### ALTER TABLE

`ALTER TABLE`用来添加，删除或修改现有表中的列，也可以用来添加和删除现有表上的各种制约因素。语法如下：

```sql
-- 现有表中添加一个新的列
ALTER TABLE table ADD column datatype;

-- 现有表中删除一个新的列
ALTER TABLE table DROP COLUMN column;

-- 现有表中更改数据类型的列
ALTER TABLE table MODIFY COLUMN column datatype;

-- 现有表中一列添加NOT NULL约束
ALTER TABLE table MODIFY column datatype NOT NULL;

-- 现有表中添加唯一约束
ALTER TABLE table ADD CONSTRAINT MyUniqueConstraint UNIQUE(column1, column2...);

-- 现有表中添加CHECK约束
ALTER TABLE table ADD CONSTRAINT MyUniqueConstraint CHECK (CONDITION);

-- 现有表中添加PRIMARY KEY约束
ALTER TABLE table ADD CONSTRAINT MyPrimaryKey PRIMARY KEY (column1, column2...);

-- 现有表中删除约束
ALTER TABLE table DROP CONSTRAINT MyUniqueConstraint;

-- 现有表中删除主键
ALTER TABLE table DROP CONSTRAINT MyPrimaryKey;
```

### DROP TABLE

`DROP TABLE`语句是用来删除表定义及其所有相关的数据表的索引，规则，触发器和约束。语法如下：

```sql
DROP TABLE table;
```

### TRUNCATE TABLE

`TRUNCATE TABLE`命令用于从现有的表删除完整的数据。在每个表上的DELETE（删除）具有相同的效果，但是，因为它没有实际扫描的表，它的速度快。语法如下：

```sql
TRUNCATE TABLE  table;
```

### DELETE FROM

`DELETE FROM`用来从一个表中删除现有的记录。可以使用WHERE子句DELETE查询删除所选行，否则所有的记录会被删除。语法如下：

```sql
DELETE FROM table WHERE [condition];
```

### INSERT INTO

`INSERT INTO`语句允许一个到一个表中插入新行。一个可以作为一个查询的结果，在一个时间或几行插入一行。基本语法如下：

```sql
INSERT INTO table (column1, column2, column3,...columnN)]
VALUES (value1, value2, value3,...valueN);
```

> **注**：
1.这里 column1, column2,...columnN是要插入数据的表中的列名。
2. 可以以任何顺序列出目标列名。 VALUES子句或查询的值都与显式或隐式的列列表从左到右。

如果要添加表中的所有列的值，可能不需要在SQL查询中指定列（次）名称。但要确保表中是在相同的顺序的列值的顺序。INSERT INTO语法如下：

```sql
INSERT INTO table VALUES (value1,value2,value3,...valueN);
```

### UPDATE

`UPDATE`被用来修改现有的表中的记录。可以使用`UPDATE`查询的`WHERE`子句更新选定行，否则会被更新的所有行。基本语法如下：

```
UPDATE table SET column1 = value1, column2 = value2...., columnN = valueN WHERE [condition];
```

## SQL基础

### SELECT

```sql
SELECT column FROM table;
```

### DISTINCT

找出表内的不同值的情况。

```sql
SELECT DISTINCT column FROM table;
```

例：

```sql
SELECT DISTINCT id, email FROM user;
```

### WHERE

```sql
SELECT column FROM table WHERE [condition];
```

### AND / OR

```sql
SELECT column FROM table WHERE [condition1] [AND|OR] [condition2];
```

### IN / NOT IN

```sql
SELECT column FROM table WHERE column IN ('value1', 'value2', ...);
```

例：

```sql
SELECT * FROM user WHERE user_name IN ('张三', '李四');
```

### BETWEEN ... AND ...

```sql
SELECT column FROM table WHERE column BETWEEN 'value1' AND 'value2';
```

例：

```sql
SELECT * FROM user WHERE age BETWEEN 18 AND 25;
```

### LIKE

```sql
SELECT column FROM table WHERE column LIKE {模式};
```

例：

```sql
SELECT * FROM user WHERE user_name LIKE '%张%';
```

### ORDER BY

```sql
SELECT column FROM table [WHERE condition] ORDER BY column [ASC, DESC];
```

> **注**：ASC（默认）代表结果会以由小往大的顺序列出，而DESC代表结果会以由大往小的顺序列出。

例：

```sql
SELECT user_name, email, age FROM user ORDER BY age DESC;
```

### GROUP BY

```sql
SELECT column1, SUM(column2) FROM table GROUP BY column1;
```

例：

```sql
SELECT user_name, SUM(age) FROM user GROUP BY user_name;
```

### HAVING

对函数产生的值来设定条件。

```sql
SELECT column1, SUM(column2) FROM table GROUP BY column1 HAVING [condition];
```

例：

```sql
SELECT user_name, SUM(ages) FROM user GROUP BY user_name HAVING SUM(age) > 1500;
```

### ALIAS

> SELECT '表别名'.'列名' AS '列别名' FROM table AS '表别名';

例：

```sql
SELECT u.user_name AS name, sum(age) ages FROM user AS u GROUP BY u.store_name;
```

### 函数

- `AVG` (平均)
- `COUNT` (计数)
- `MAX` (最大值)
- `MIN` (最小值)
- `SUM` (总合)

```sql
SELECT fun_name(column) FROM table;
```

例：

```sql
SELECT count(u.id) AS user_count FROM user AS u;
```

### 表连接 

INNER JOIN: 如果表中有至少一个匹配，则返回行；
LEFT JOIN: 即使右表中没有匹配，也从左表返回所有的行；
RIGHT JOIN: 即使左表中没有匹配，也从右表返回所有的行；
FULL JOIN: 只要其中一个表中存在匹配，就返回行。

例：

```sql
SELECT u.uesr_name, c.corp_name FROM user AS u LEFT JOIN corp AS c ON c.id = u.corp_id;
```

## SQL进阶

### UNION / UNION ALL

`UNION`用于合并两个或多个SELECT语句的结果，不返回任何重复的行。`UNION ALL`运算符语句，则包括重复行的结果。使用UNION，每个SELECT选择的列数必须具有相同的，相同数目的列表达式相同的数据类型，并让它们在相同的顺序，但它们不必是相同的长度。语法如下：

```sql
SELECT column1 [, column2 ]
FROM table1 [, table2 ]
[WHERE condition]

UNION [UNION ALL]

SELECT column1 [, column2 ]
FROM table1 [, table2 ]
[WHERE condition]
```

### INTERSECT / INTERSECT ALL

和`UNION`指令类似，`INTERSECT`也是对两个SQL语句所产生的结果做处理的。不同的地方是，`UNION`基本上是一个`OR`(如果这个值存在于第一句或是第二句，它就会被选出)，而`INTERSECT`则比较像`AND`(这个值要存在于第一句和第二句才会被选出)。`UNION`是并集，而`INTERSECT`是交集。`INTERSECT ALL`则包含交集后的重复数据。语法如下：

```sql
SELECT column1 [, column2 ]
FROM table1 [, table2 ]
[WHERE condition]

INTERSECT [INTERSECT ALL]

SELECT column1 [, column2 ]
FROM table1 [, table2 ]
[WHERE condition]
```

### EXCEPT / EXCEPT ALL

`EXCEPT`用于求差集，其将查询在前一个结果集中但是不再后面一个结果集中的记录。`EXCEPT ALL`则包含交集后的重复数据。语法如下：

```sql
SELECT column1 [, column2 ]
FROM table1 [, table2 ]
[WHERE condition]

EXCEPT [EXCEPT ALL]

SELECT column1 [, column2 ]
FROM table1 [, table2 ]
[WHERE condition]
```

> **注**：在其他数据库求差集的关键字是：`MINUS`。

### SUBQUERY

`SUBQUERY`即子查询，子查询也是一个普通查询，目的是将用子查询返回的数据将被用来在主查询中作为条件，以进一步限制要检索的数据。可以使用子查询的有SELECT，INSERT，UPDATE和DELETE语句，与运算符如=，<，>，>=，<=，IN等一起使用。有几个子查询必须遵循的规则：

- 必须用括号括起来的子查询。
- 子查询只能有一个在SELECT子句中的列，除非多列在主查询的查询来比较其选定的列。
- ORDER BY不能使用在子查询中，虽然主查询就可以使用ORDER BY。GROUP BY可以用来执行相同的功能在子查询中的ORDER BY。
- 子查询返回多于一行只能用于使用多值的运算符，如为IN，EXISTS，IN，ANY / SOME，ALL运算符。

使用示例如下：

```sql
-- SELECT语句中的子查询
SELECT * FROM COMPANY WHERE ID IN (SELECT ID FROM COMPANY WHERE SALARY > 45000);

-- INSERT语句的子查询
INSERT INTO COMPANY_BKP SELECT * FROM COMPANY WHERE ID IN (SELECT ID FROM COMPANY);

-- UPDATE语句的子查询
UPDATE COMPANY SET SALARY = SALARY * 0.50 WHERE AGE IN (SELECT AGE FROM COMPANY_BKP WHERE AGE >= 27);

-- UPDATE语句的子查询
DELETE FROM COMPANY WHERE AGE IN (SELECT AGE FROM COMPANY_BKP WHERE AGE > 27 );
```

### EXISTS / NOT EXISTS

`EXISTS`用于检查子查询是否至少会返回一行数据，该子查询实际上并不返回任何数据，而是返回值True或False。EXISTS指定一个子查询，检测行的存在。`NOT EXISTS`的作用与`EXISTS`正好相反。如果子查询没有返回行，则满足了`NOT EXISTS`中的`WHERE`子句。语法如下： 

```sql
EXISTS subquery
```

EXISTS 和 IN 的比较：

- EXISTS(包括 NOT EXISTS )子句的返回值是一个BOOLEAN值。EXISTS内部有一个子查询语句(SELECT ... FROM...)，我将其称为EXIST的内查询语句。其内查询语句返回一个结果集。EXISTS子句根据其内查询语句的结果集空或者非空，返回一个布尔值。而IN引导的子查询只能返回一个字段
- EXISTS : 强调的是是否返回结果集，不要求知道返回什么，IN则需要知道返回的字段值。
- EXISTS与IN的使用效率的问题，通常情况下采用exists要比in效率高，因为IN不走索引，但要看实际情况具体使用：IN适合于外表大而内表小的情况；EXISTS适合于外表小而内表大的情况。

### CONCATENATE 

连接字符串。有的时候，我们有需要将由不同列获得的资料串连在一起。每一种数据库都有提供方法来达到这个目的。

- Oracle: CONCAT(), || 
- SQL Server: + 
- MySQL: CONCAT() 
- PostgreSQL: CONCAT(), || 

PostgreSQL的`CONCAT()`的语法如下：

> concat(str "any" [, str "any" [, ...] ])

PostgreSQL的`||`的语法如下：

> string || string
> string || non-string 或 non-string || string

例：

```sql
SELECT u.user_name || ' ' || u.age FROM user AS u;
```

### SUBSTRING

截取字符串。

- Oracle: SUBSTR()
- SQL Server: SUBSTRING()
- MySQL: SUBSTR(), SUBSTRING()
- PostgreSQL: SUBSTRING()

PostgreSQL的SUBSTRING()语法如下：

- substring(string [from int] [for int]) 截取子字符串。
- substring(string from pattern) 截取匹配POSIX正则表达式的子字符串。
- substring(string from pattern for escape) 截取匹配SQL正则表达式的子字符串。

例：

```sql
-- 得到hom
substring('Thomas' from 2 for 3);

-- 得到mas
substring('Thomas' from '...$')

-- 得到oma
substring('Thomas' from '%#"o_a#"_' for '#')
```

### TRIM

SQL中的`TRIM`函数是用来移除掉一个字符串中的字头或字尾。最常见的用途是移除字首或字尾的空白。这个函数在不同的数据库中有不同的名称： 

- Oracle: LTRIM(), RTRIM()
- SQL Server: LTRIM(), RTRIM()
- MySQL: TRIM(), LTRIM(), RTRIM()
- PostgreSQL: TRIM(), BTRIM(), LTRIM(), RTRIM()
 
PostgreSQL的TRIM()语法如下：

- trim([leading | trailing | both] [characters] from string) 从字符串string的开头/结尾/两边删除只包含characters中字符(缺省是空白)的最长的字符串。
- btrim(string text [, characters text]) 从string开头和结尾删除只包含 characters中字符(缺省是空白)的最长字符串。
- ltrim(string text [, characters text]) 从字符串string的开头删除只包含characters 中字符(缺省是一个空白)的最长的字符串。
- rtrim(string text [, characters text]) 从字符串string的结尾删除只包含characters中字符(缺省是个空白)的最长的字符串。

```sql
-- 得到Tom
trim(both 'x' from 'xTomxx')

-- 得到trim
btrim('xyxtrimyyx', 'xy')

-- 得到trim
ltrim('zzzytrim', 'xyz')

-- 得到trim
rtrim('trimxxxx', 'x')
```

### CASE

`CASE`表达式是一种通用的条件表达式，类似于其它编程语言中的`if/else`语句。

```sql
CASE WHEN [condition] THEN result
     [WHEN ...]
     [ELSE result]
END
```

示例如下：

```sql
SELECT sex, 
    CASE WHEN sex = 0 THEN '女' 
         WHEN sex = 1 THEN '男' 
         ELSE '未知' 
    END 
FROM user;
```

### COALESCE

`COALESCE`返回它的第一个非`NULL`的参数值。如果所有参数都是NULL那么返回NULL。它常用于在显示数据时用缺省值替换NULL。语法如下：

```sql
COALESCE(value [, ...])
```

使用示例：

```sql
SELECT COALESCE(NULL, NULL, GETDATE());
```

### NULLIF

当且仅当value1等于value2时，`NULLIF`才返回null。否则它返回value1。这些可以用于执行上面给出的`COALESCE`例子的反例。语法如下：

```sql
NULLIF(value1, value2)
```

### GREATEST / LEAST

GREATEST和LEAST函数从一个任意数字表达式的列表里选取最大或者最小的数值。 这些表达式必须都可以转换成一个普通的数据类型，它将会是结果类型。列表中的NULL值将被忽略。只有所有表达式的结果都是 NULL 的时候，结果才会是 NULL。语法如下：

```sql
GREATEST(value [, ...])
LEAST(value [, ...])
```

> **注意**：GREATEST和LEAST都不是 SQL 标准，但却是很常见的扩展。