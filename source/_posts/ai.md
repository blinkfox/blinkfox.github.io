---
layout:     post
title:      "Vulnhub - AI: Web"
subtitle:   "Write-Up"
date:       2019-08-27
author:     "D4mianwayne"
tag:      nmap, ctf, vulnhub
category: Vulnhub
---

This is a writeup for [AI:Web](https://www.vulnhub.com/entry/ai-web-1,353/) by [arif](https://twitter.com/@arif_xpress) from Vulnhub.

# Nmap

Starting off with the nmap and using `nmap -sV -sC -A -p- -T4 192.168.43.135`, we can see that it has only one port open i.e. HTTP or 80.

![](/img/ai/nmap.png)

# HTTP Enumeration

Checking the HTTP port, we can see that it has a very basic homepage.

![](/img/ai/http.png)

Now, from the nmap scan we knew that it has a `robots.txt`, let's check it.

![](/img/ai/robots.png)

`robots.txt` had following contents:-

```
User-agent: *
Disallow: 
Disallow: /m3diNf0/
Disallow: /se3reTdir777/uploads/
```
Now, let's try the `/m3diNf0` directory but apparently both of them gave the same response **You do not have the permission to view this**.

![](/img/ai/noperm.png)

Now, using uniscan I found that there is a `info.php` but it was not useful since, since it only contains some basic information, but still what worthy of attenton was the following line:-

![](/img/ai/docroot.png)


Going over `/se3reTdir777` shows a input for to retrieve the data/credentials by entering the ID.

![](/img/ai/login.png)

Since after entering something like `' 0R 1=1` didn't worked as I thought it was SQLi, then I tried `1,2,3,4....` that leaked the admin, ID and other data.

![](/img/ai/leak.png)

# SQL Injection and Reverse Shell

I tried to use sqlmap, but the url didn't had any vulnerable parameter so we needed the whole request to attack the database. Thanks to [mzfr](https://twitter.com/0xmzfr) for pointing it out. I captured the request and saved it in a file and used `sqlmap -r new.txt --dbs` and started it.

```
POST /se3reTdir777/ HTTP/1.1
Host: 192.168.43.135
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 38
Connection: close
Referer: http://192.168.43.135/se3reTdir777/
Upgrade-Insecure-Requests: 1

uid=1%2C2%2C3%2C4....&Operation=Submit
```

![](/img/ai/sqldata.png)


Once the scan will over, we can see it has two databases

![](/img/ai/sqlmap.png)

Checking the tables and thier respective columns gave some data, though they were kinda useless in our condition since there was no other port beside 80. 

![](/img/ai/crdentials.png)

Now, as for now I was a little clueless since we successfully exploited the SQL Injection vulnerabilty, so I tried `--os-shell` argument to spawn an instance of the shell. That worked, now we knew that it is work on PHP, selecting php as an option and the bruteforce option, after that entering the root directory we found on earlier **HTTP Enumeration**. 

![](/img/ai/shell.png)

So, there are already uploaded files in the folder let's try to access files, apparently one given a blank page, from other we can see an upload service:-

![](/img/ai/uploaf.png)

As a php backend website, I quickly used the classic old reverse shell script and uploaded it, once it was uploaded, we started the listener and go to `/uploads/shell.php` start the daemon service to reverse callback the shell file spawning a shell.

>Note: Do `python -c 'import pty; pty.spawn("/bin/bash)' to spawn a TTY shell.

![](/img/ai/reverseshell.png)

##### Upload file contents

One file has contents which can be seen and deduce that it is upload service.

![](/img/ai/files.png)

# Root

Now, I started searching for SUID binaries but nothing, so I started searching for writable files and directory by using `find / -user www-data -writable f 2>/dev/null`, which showed us that user `www-data` has write permission on `/etc/passwd`.

![](/img/ai/passwd.png)

![](/img/ai/permissions.png)

Recalling what we have done to MinUV2 from Vulnhub, let's add a sudo user to the file, doing this `echo "robin:sXuCKi7k3Xh/s:0:0::/root:/bin/bash" >> /etc/passwd` which has a password `toor` :P

```
echo "robin:sXuCKi7k3Xh/s:0:0::/root:/bin/bash" >> /etc/passwd
su robin
toor
```

Rooted, time for a flag:-

![](/img/ai/root.png)

##### Leaked Username and Passwords

Though only one user credentials were correct from `systemUser` database that was leaked from it which was `aiweb1pwn` though that user didn't have any **seen** vulnerable configuration so that's why we had to do this from user `www-data`.

![](/img/ai/crdentials.png)
