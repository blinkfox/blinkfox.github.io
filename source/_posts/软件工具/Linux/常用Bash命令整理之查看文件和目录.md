---
title: 常用Bash命令整理之查看文件和目录
date: 2018-10-09 22:45:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/10/09/linux.jpg
categories: 软件工具
tags:
  - Linux
---

## 1. ls - 列出文件名和目录

`ls`命令是`Linux`中最常用的命令之一，其作用就是列出文件名和目录。在命令行提示符下，直接输入`ls`命令，不带任何选项，将列出当前目录下所有文件和目录，但不会显示详细的信息，比如，文件类型、大小、修改日期和时间、权限等。

以下便是`ls`命令及其选项的作用说明：

```bash
# 仅列出当前目录下所有文件和目录
ls

# 每行显示一条记录，每条记录包括文件类型、大小、修改日期和时间、权限等
ls -l

# 将文件大小显示符合人类阅读习惯的格式
ls -lh

# 将使用不同的特殊字符归类不同的文件类型
ls -F

# 以长列表格式列出某个目录的信息
ls -ld /var/log

# 将递归地列出子目录的内容
ls -R /etc/sysconfig/

# 以长列表格式按文件或目录的修改时间倒序地列出文件和目录
ls -ltr

# 以长列表格式按文件大小顺序列出文件和目录
ls -ls

# 列出包括隐藏文件或目录在内的所有文件和目录，包括“.”（当前目录）和“..”（父目录）
ls -a

# 列出包括隐藏文件或目录在内的所有文件和目录，不包括“.”（当前目录）和“..”（父目录）
ls -A

输出的内容类似于-l选项，指示显示uid和gid，替代显示所有者和用户组
ls -n
```

## 2. cat - 连接显示文件内容

`cat` 命令也是Linux系统中最常用的命令之一。`cat`命令让我们可以看看文件的内容、连接文件、创建一个或多个文件和重定向输出到终端或文件。

`cat`命令的语法如下所示：

```bash
cat [OPTION] [FILE]...
```

`cat`常用命令如下：

```bash
# 使用 cat 命令查看文件 /etc/group 的内容
cat /etc/group

# 显示多个文件的内容
cat /etc/redhat-release /etc/issue

# -n 选项，可以显示文件内容的行号
cat -n /etc/fstab

# -b 选项和 -n 选项类似，但只标识非空白行的行号
cat -b /etc/fstab

# -e 选项，将在每一行的结尾显示“$”字符
cat -e /etc/fstab
```

> 当你只输入 cat 命令，而没有任何参数时，它只是接收标准输入的内容并在标准输出中显示。所以你在输入一行内容并回车后，会在接下来的一行显示相同的内容。你也可以重定向标准输出到一个新文件。

## 3.less、more - 分屏显示文件

`more`命令在你使用小的xterm窗口时，或是想不使用文本编辑器而只是简单地阅读一个文件时是很有用的。more命令是一个用于一次翻阅一整屏文件的过滤器。

```bash
# 查看一个文件，自动清空屏幕并显示文件开头部分
more /etc/inittab

# 指定一次显示num行
more -num /etc/inittab
```

与`more`命令相比，我个人更喜欢`less`命令来查看文件。`less`命令与`more`命令类似，但`less`命令向前和向后翻页都支持，而且`less`命令不需要在查看前加载整个文件，即`less`命令查看文件更快速。

`less`常用命令参数如下：

```bash
-b  <缓冲区大小> 设置缓冲区的大小
-e  当文件显示结束后，自动离开
-f  强迫打开特殊文件，例如外围设备代号、目录和二进制文件
-g  只标志最后搜索的关键词
-i  忽略搜索时的大小写
-m  显示类似more命令的百分比
-N  显示每行的行号
-o  <文件名> 将less 输出的内容在指定文件中保存起来
-Q  不使用警告音
-s  显示连续空行为一行
-S  行过长时间将超出部分舍弃
-x  <数字> 将“tab”键显示为规定的数字空格
/字符串：向下搜索“字符串”的功能
?字符串：向上搜索“字符串”的功能
n： 重复前一个搜索（与 / 或 ? 有关）
N： 反向重复前一个搜索（与 / 或 ? 有关）
b  向后翻一页
d  向后翻半页
h  显示帮助界面
Q  退出less 命令
u  向前滚动半页
y  向前滚动一行
空格键 滚动一行
回车键 滚动一页
[pagedown]： 向下翻动一页
[pageup]：   向上翻动一页
```

## 4.head - 显示文件头部

`head`命令用于打印指定输入的开头部分内容。默认情况下，打印每个指定输入的前10行内容。

使用`-n`选项可以指定打印文件的前N行：

```bash
# 指定打印文件的前5行
head -n 5 /etc/inittab
（或）head -5 /etc/inittab

# 打印文件的前N个字节的数据
head -c 10 /etc/inittab
```

## 5.tail - 显示文件尾部

`tail`命令和`head`命令相反，它打印指定输入的结尾部分的内容。默认情况下，它打印指定输入的最后10行内容。

使用`-n`选项可以指定打印文件的最后N行：

```bash
# 指定打印文件的后10行
tail -n 10 /etc/inittab
tail -10 /etc/inittab

# 即时打印文件中新写入的行
tail -f /var/log/messages

# --retry选项表示持续尝试打开某个文件，当你想打开一个稍后才会创建或即使不可用的文件
tail -f /tmp/debug.log --retry
```

## 6.file - 查看文件类型

`file`命令用于接收一个文件作为参数并执行某些测试，已确定正确的文件类型。

```bash
# 查看文件类型
file /etc/inittab

# 可以MIME类型的格式显示文件类型的信息
file -i  /etc/inittab

# 使用-N 选项，输出的队列可以以在文件名之后无空白填充的形式显示
file -N *
```

## 7.wc - 查看文件统计信息

`wc`命令用于查看文件的行数、单词数和字符数等信息。语法类似如下所示：

```bash
wc filename
X Y Z /etc/inittab
```

其中X表示行数，Y表示单词数，Z表示字节数，filename表示文件名。

```bash
# -l选项，可以只统计文件的行数信息
wc -l /etc/inittab

# -w选项，可以只统计文件的单词数信息
wc -w /etc/inittab

# -c选项，可以只统计文件的字节数信息
wc -c /etc/inittab

# -L选项，可以只统计文件中最长的行的长度
wc -L /etc/inittab
```

## 8.find - 查找文件或目录

`find`命令用于根据你指定的参数搜索和定位文件和目录的列表。`find`命令可以在多种情况下使用，比如你可以通过权限、用户、用户组、文件类型、日期、大小和其他可能的条件来查找文件。

`find`命令常用使用和说明如下：

```bash
# 查找指定目录下的某个文件
find /etc/ -name inittab

# 在当前目录下查找名称为 inittab 的文件
find . -name inittab

# 在当前目录下，文件不区分大小写是example的所有文件
find . -iname example

# 找出当前目录下所有以 sh 结尾的文件
find . -type f -name "*.sh"

# 找出当前目录下，文件权限是 777 的所有文件
find . -type f -perm 777

# 找出当前目录下，文件权限不是 777 的所有文件
find . -type f ! -perm 777

# 找出当前目录下所有只读文件
find . -type f ! -perm /a+w

# 找出你帐号主目录下的所有可执行文件
find ~ -type f -perm /a+w

# 找出 /tmp 目录下的.log文件并将其删除：
find /tmp/ -type f -name "*.log" -exec rm -f {} \;

# 找出当前目录下的所有空文件
find . -type f -empty

# 找出当前目录下的所有空目录
find . -type d -empty

# 找出 /tmp 目录下的所有隐藏文件
find /tmp/ -type f -name ".*"

# 找出 /tmp 目录下，所有者是 root 的文件和目录
find /tmp/ -user root

# 找出 /tmp 目录下，用户组是 developer 的文件和目录
find /tmp/ -group root

# 找出你账号的主目录下，3 天前修改的文件
find ~ -type f -mtime 3

# 找出你账号的主目录下，30 天以前修改的所有文件
find ~ -type f -mtime +30

# 找出你账号的主目录下，3 天以内修改的所有文件
find ~ -type f -mtime -3

# 找出你账号的主目录下，30 天以前，60 天以内修改的所有文件
find ~ -type f -mtime +30 -mtime -60

# 找出 /etc 目录下，一小时以内变更过的文件
find /etc -type f -cmin -60

# 找出 /etc 目录下，一小时以内访问过的文件
find /etc -type f -amin -60

# 找出你账号主目录下，大小是50MB的所有文件
find ~ -type f -size 50MB

# 找出你账号主目录下，大于50MB小于100MB的所有文件
find ~ -type f -size +50MB -size -100MB

# 找出你账号主目录下，大于100MB的文件并将其删除
find ~ -type f -size +100MB -exec rm -rf {} \;
```