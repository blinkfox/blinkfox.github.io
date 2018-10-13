---
title: 常用Bash命令整理之其他常用命令
date: 2018-10-13 18:15:00
author: blinkfox
categories: 软件工具
tags: Linux
---

## 1. hostname - 查看主机名

`hostname`命令用于查看系统的主机名，或是修改系统的主机名。

`hostname`的常用命令如下：

```bash
# 显示系统的当前主机名
hostname

# 修改你系统的主机名
hostname blinkfox-system

# 使用 -F 选项，从指定的文件中读取主机名
hostname -F /root/hostname.txt
```

## 2. uptime - 查看系统运行时间

`uptime`命令用于打印系统的运行时间等信息。使用如下：

```bash
uptime
```

## 3. w、who - 列出登录的用户

`w`命令用于显示登录用户及他们当前运行的进程。输入的内容格式如下：

```bash
w

# 打印如下
22:42  up 18 days, 1 hr, 2 users, load averages: 1.23 1.79 1.75
USER     TTY      FROM              LOGIN@  IDLE WHAT
blinkfox console  -                日19   6days -
blinkfox s000     -                五23       - w
```

`who`命令有与`w`命令类似的用途，但它的功能比`w`命令更强大一些。语法格式如下：

```bash
who [OPTION]... [FILE | ARG1 ARG2]
```

`who`常用命令如下：

```bash
# 显示当前登录的所有用户信息
who

# 显示系统的启动时间
who -b

# 显示系统登录进程
who -l

# 显示与当前标准输入关联的用户信息
who -m

# 显示系统的运行级别
who -r

# 显示所有登录用户的用户名和登录用户数
who -q
```

## 4. uname - 查看系统信息

`uname`命令用于打印内核名称和版本、主机名等系统信息。命令的语法如下所示：

```bash
uname [OPTION]...
```

常用使用方式如下：

```bash
# 只打印内核的名称
uname

# 使用 -n 选项，只打印系统的主机名
uname -n

# 使用 -r 选项，打印内核版本信息
uname -r

# 使用 -m 选项，打印系统的硬件名称
uname -m

# 使用 -p 选项，打印系统的处理器类型信息
uname -p

# 使用 -i 选项，打印系统的硬件平台信息
uname -i

# 使用 -a 选项，打印上述所有示例中的信息
uname -a
```

## 5. date - 显示和设置系统日期和时间

`date`命令用于以多种格式显示日期和时间，或设置系统的日期和时间。`date`命令的语法如下所示：

```bash
date [OPTION]... [+FORMAT]
date [-u|--utc|--universal] [MMDDhhmm[[CC]YY][.ss]]
```

常用使用命令如下：

```bash
# 以默认格式显示系统的当前日期时间
date

# 格式化当前日期
date +"%Y-%m-%d"

# 格式化输出昨天的日期
date -d "1 day ago" +"%Y-%m-%d"

# 2秒后格式化输出
date -d "2 second" +"%Y-%m-%d %H:%M.%S"

# 普通格式化转出
date -d "2009-12-12" +"%Y/%m/%d %H:%M.%S"

# apache格式转换
date -d "Dec 5, 2009 12:00:37 AM" +"%Y-%m-%d %H:%M.%S"

# 日期加减操作
date +%Y%m%d #显示前天年月日
date -d "+1 day" +%Y%m%d #显示前一天的日期
date -d "-1 day" +%Y%m%d #显示后一天的日期
date -d "-1 month" +%Y%m%d #显示上一月的日期
date -d "+1 month" +%Y%m%d #显示下一月的日期
date -d "-1 year" +%Y%m%d #显示前一年的日期
date -d "+1 year" +%Y%m%d #显示下一年的日期

# 设定时间
date -s # 设置当前时间，只有root权限才能设置，其他只能查看
date -s 20160816 # 设置成20160816，这样会把具体时间设置成空00:00:00
date -s 01:01:01 # 设置具体时间，不会对日期做更改
date -s "01:01:01 2012-05-23" # 这样可以设置全部时间 
date -s "01:01:01 20120523" # 这样可以设置全部时间
date -s "2012-05-23 01:01:01" # 这样可以设置全部时间 
date -s "20120523 01:01:01" # 这样可以设置全部时间
```

## 6. id - 显示用户属性

`id`命令用于打印输出用户`uid`、`gid`、用户名和组名等用户身份信息。`id`命令的语法如下所示：

```bash
id [OPTION]... [USERNAME]
```

常见使用命令如下：

```bash
# 输出当前用户的uid、用户名、gid、组名及用户属于的群组信息
id

# 使用 -u 选项，输出用户的 uid
id -u

#-u 选项和 -n 选项结合使用，输出账户的用户名
id -un

# 使用 -g 选项，输出帐号当前起作用的gid
id -g

# -g 与 -n 选项结合使用，输出帐号当前起作用的用户组名
id -gn

# 使用 -G 选项，输出帐号所属的所有群组id
id -G root

# -G 与 -n 选项结合使用，输出账号所属的所有群组的名称
id -Gn root
```