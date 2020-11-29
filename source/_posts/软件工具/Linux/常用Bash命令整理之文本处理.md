---
title: 常用Bash命令整理之文本处理
date: 2018-10-11 22:05:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/10/11/bash.jpg
categories: 软件工具
tags:
  - Linux
---

## 1. sort - 文本排序

`sort`命令用于将文本文件的行排序。默认情况下，`sort`命令是按照字符串的字母顺序排序。

sort 的常用命令如下：

```bash
# 将文本内容按字母顺序排序
sort example.txt

# 使用 -u 选项，移除所有重复行后排序
sort -u example.txt

# 使用 -n 选项，将令数字按数值的大小排序
sort -n example.txt

# 使用 -r 选项，以倒序方式排序
sort -n -r example.txt

# 同时将 file1、file2 的内容排序
sort file1 file2
```

## 2.uniq - 文本去重

`uniq`命令用于移除或发现文件中重复的条目。

```bash
# 它将移除文件中重复的行并显示单一行
uniq example.txt

# 可以统计重复行出现的次数
uniq -c example.txt

# 使用 -d 选项，只显示文件中有重复的行并只显示一次
uniq -d example.txt

# 使用 -D 选项，显示文件中所有重复的行
uniq -D example.txt

# 使用 -u 选项，只显示文件中不重复的行
uniq -u example.txt

# 使用 -w 选项，限制 uniq 命令只比较每行的前 3 个字符是否重复
uniq -w 3 example.txt

# 使用 -s 选项，避免 uniq 命令比较每行的前 3 个字符，只比较后面的字符是否重复
uniq -s 3 example.txt

# 使用 -f 选项，避免 uniq 命令比较第一列的内容，只比较后面的字符是否重复
uniq -f 1 example.txt
```

## 3.tr - 替换或删除字符

`tr`命令主要用于删除文件中控制字符或进行字符转换。使用`tr`时要转换两个字符串：字符串 1 用于查询，字符串 2 用于处理各种转换。`tr`刚执行时，字符串 1 中的字符被映射到字符串 2 中的字符，然后转换操作开始。

`tr`命令的语法如下所示：

```bash
tr [OPTION]... SET1 [SET2]
```

常用命令示例：

```bash
# 若要将大括号转换为小括号
tr '{}' '()' < textfile > newfile

# 若要将大括号转换成方括号
tr '{}' '\[]' < textfile > newfile

# 若要将小写字符转换成大写，请输入：
tr 'a-z' 'A-Z' < textfile > newfile

# 若要创建一个文件中的单词列表
tr -cs '[:lower:][:upper:]' '[\n*]' < textfile > newfile

# 若要从某个文件中删除所有空字符
tr -d '\0' < textfile > newfile

# 若要用单独的换行替换每一序列的一个或多个换行，请输入：
tr -s '\n' < textfile > newfile

# 要以单个“#”字符替换 <space> 字符类中的每个字符序列
tr -s '[:space:]' '[#*]'
```

## 4.grep - 查找字符串

`grep`命令用于搜索文本或指定的文件中与指定的字符串或模式相匹配的行。默认情况下，`grep`命令只显示匹配的行。

`grep`命令的语法如下所示：

```bash
grep [OPTION]... PATTERN [FILE]...
grep [OPTION]... [-e PATTERN | -f FILE] [FILE]...
```

```bash
# `grep`命令查找文件/etc/passwd 中帐号 blinkfox 的信息
grep blinkfox /etc/passwd

# 使用 -i 选项，强制 grep 命令忽略搜索关键字的大小写
grep -i blinkfox /etc/passwd

# 使用 -r 选项，可以递归搜索指定目录下的所有文件
grep -r blinkfox /etc/

# 使用 -w 选项，只匹配包含指定单词的行
grep -w blinkfox /etc/

# 使用 -c 选项，报告文件或文本中模式被匹配的次数
grep -c blinkfox /etc/passwd

# 使用 -n 选项，显示每一个匹配的行的行号
grep -n blinkfox /etc/passwd

# 使用 -v 选项，可以输出除匹配指定模式的行以外的其他所有行
grep -v blinkfox /etc/passwd

# 使用 --color 选项，在输出中将匹配的字符串以彩色的形式标出
grep --color blinkfox /etc/passwd
```

## 5.diff - 比较两个文件

`diff`命令用于比较两个文件，并找出它们之间的不同。`diff`命令的语法如下所示：

```bash
diff [OPTION]... from-file to-file
```

常用使用方式如下：

```bash
# 比较两个文件
diff nsswitch.conf nsswitch.conf.org

# 使用 -w 选项，比较时忽略空格
diff -w nsswitch.conf nsswitch.conf.org

# 使用 -y 选项，以并排的格式输出两个文件的比较结果
diff -y nsswitch.conf nsswitch.conf.org

使用 -c 选项，以上下对比的格式输出两个文件的比较结果
diff -c nsswitch.conf nsswitch.conf.org
```