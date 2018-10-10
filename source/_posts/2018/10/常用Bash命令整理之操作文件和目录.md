---
title: 常用Bash命令整理之操作文件和目录
date: 2018-10-10 23:10:00
author: blinkfox
categories: 软件工具
tags:
  - Linux
---

## 1. touch - 创建文件

`touch`命令就可用于创建、变更和修改文件的时间戳。它是 Linux 操作系统的标准程序。`touch`命令又如下选项：

```bash
-a: 只改变访问时间 
-c: 不创建任何文件
-m: 只改变修改时间
-r: 使用指定文件的时间替代当前时间
-t: 使用 [[CC]YY]MMDDhhmm[.ss] 替代当前时间
```

touch 命令的常见用法如下：

```bash
# 创建一个名为 effyl 的新空文件
touch effyl

# 同时创建名称分别为 effyl myeffyl lueffyl 的三个文件
touch effyl myeffyl lueffyl

# 使用 -a 选项，可以改变或更新文件的最新访问时间，如果文件 effyl 不存在，则新创建一个
touch -a effyl

# 使用 -c 选项，可以避免创建一个新文件，并用当前时间更新文件的时间戳
touch -c effyl

# 使用 -m 选项，可以只改变文件的修改时间，而访问时间不变
touch -m effyl

# 使用 -c 和 -t 选项，来明确设置文件的时间
touch -c -t YYMMDDHHMM filename

# 如果想使用文件 myeffyl 的时间戳更新文件 effyl 的时间戳，可以使用 -r 选项
touch -r myeffyl effyl
```

## 2.mkdir - 创建目录

`mkdir`命令用于创建一个新目录。最基本的`mkdir`命令的使用方法如下所示：

```bash
# 在当前目录下创建一个给定的目录名
mkdir <dirname>

# 在 backup 中的相对路径创建一个名为 old 的目录
mkdir backup/old

# 在 backup 中的绝对路径中创建一个名为 old 的目录
mkdir /home/blinkfox/backup/old

# 使用 -p 选项，会自动创建所有还不存在的父目录
mkdir -p backup/old

# 使用 -m 选项，可以设置将要创建目录的权限
# 如：创建一个任何人都有读写访问权限的目录
mkdir -p -m 777 backup/old
```

## 3.cp - 复制文件或目录

`cp`命令用于将文件从一个地方复制到另一个地方。原来的文件保持不变，新文件可能保持原名或用一个不同的名字。

使用 cp 命令复制文件和目录的语法有以下几种：

```bash
# 复制源文件到目标文件
cp [OPTION] SOURCE DEST

# 复制一个或多个源文件到一个目录
cp [OPTION] SOURCE... DIRECTORY

# 同上
cp [OPTION] -t DIRECTORY SOURCE... 
```

常用使用示例如下：

```bash
# 在当前目录下，创建一个文件 file.txt 的副本，取名为 newfile.txt
cp file.txt newfile.txt

# 复制当前目录下的 file.txt 文件到 /tmp 目录下
cp file.txt /tmp

# 复制当前目录下的所有文件到 /tmp 目录下
cp * /tmp

# 使用 -p 选项，可以使复制一个文件到新文件时，保留源文件的所有者、权限等信息
cp -p filename /path/to/new/location/myfile

# 使用 -R 或 -r 选项，恶意递归地复制一个目录
# 即将一个目录及其下的所有文件和子目录都复制到另一个目录
cp -R * /home/blinkfox/backup
```

## 4.ln - 链接文件或目录

`ln`命令用于创建软链接或硬链接。使用 -s 选项，可以创建一个软链接：

```bash
# 在目录 lib 下创建一个软链接 library.so，链接到 /home/blinkfox/src/library.so
ln -s /home/blinkfox/src/library.so /home/blinkfox/lib

# 创建目录的软链接
ln -s /home/blinkfox/src source
```

## 5. mv - 移动文件或目录

`mv`命令用于将文件和目录从一个位置移到另外一个位置。除了移动文件，`mv`命令还可用于修改文件或目录的名字。

mv 命令的基本语法如下所示：

```bash
mv SOURCE... DIRECTORY
```

常用命令如下：

```bash
# 将当前目录下的文件 source.txt 移到目录 /tmp 下
mv source.txt /tmp

# 将目录 dir1、dir2 移到目录 dir_dist 下
mv dir1 dir2 dir_dist

# 将当前目录下的 old.txt 文件更名为 new.txt
mv old.txt new.txt

# 使用 -i 选项，在重写覆盖目标文件或目录之前给出提示信息
mv -i old.txt new.txt

# 将当前目录下的所有文件移动到目录 /tmp 下
mv * /tmp/

# 使用 -i 选项，从 dir1 中移动那些在目标目录中不存在的文件到目标目录
mv -u dir1/* dir2/
```

## 6.rm - 删除文件或目录

`rm`命令用于删除指定的文件和目录。其语法如下所示：

```bash
rm [OPTIONS]... FILE...
```

`rm`的常用命令如下：

```bash
# 删除当前目录下的文件 file1.txt、file2.txt、file3.txt
rm file1.txt file2.txt file3.txt

# 删除当前目录下的所有文件
rm *

# 删除你当前帐号主目录下的 temp 目录中的所有文件
rm ~/temp/*

# 使用 -i 选项，可以在删除每个文件或目录前提示用户确认
rm -i *

# 删除当前目录下所有以".doc"结尾的文件
rm *.doc

# 删除当前目录下所有文件名中包含"movie"字符串的文件
rm *movie*

# 删除当前目录下所有以"a"开头的文件
rm a*

# 删除当前目录下整个文件名（包括扩展名）只有 3 个字符的所有文件
rm ???

# 删除当前目录下文件扩展名有两个字符的所有文件
rm *.??

# 删除当前目录下文件名中含有字母 a 或 b 或 c 的所有文件
rm *[abc]*

# 删除当前目录下文件名中包含 0~9 的所有文件
rm *[0-9]*

# 删除当前目录下文件扩展名是字母 c 或 h 的所有文件
rm *.[ch]

# 删除 /tmp 目录下的所有文件及其子目录
rm -rf /tmp/*
```

> -f 删除前不提示用户确认，并忽略不存在的文件

> -r 递归地删除目录及其下的内容