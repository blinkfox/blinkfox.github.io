---
layout:     post
title:      "Vulnhub - Tr0ll:1"
subtitle:   "Write-Up"
date:       2019-08-24 
author:     "D4mianwayne"
tag:      nmap, ctf, vulnhub
category: Vulnhub
---

Today, we are going to pwn Troll:1 from [Vulnhub](https://www.vulnhub.com/entry/tr0ll-1,100/) by [maleus](https://twitter.com/@maleus21).

# Nmap

Running a nmap scan on the machine for analysing the services running on the machine. Doing that so, `nmap -sV -sC -A -p- -T5 102.168.43.169`
![](/../img/troll/nmap.png)


# FTP Enumeration

From our nmap scan we knew that it FTP has Anonymous login enabled, so using ftp package we log in to ftp as anonymous user and found there was a `lol.pcap`.

![](../img/troll/ftp.png)

Opening it in wireshark, we can see there is a `secret_txt` somewhat existed in a network from where this traffic was captured. So I tried to follow the TCP stream and found that there is a directory mentioned in that specific file.

![](../img/troll/wireshark.png)

# HTTP Service Enumeration

Beforehand, I tried checking `robots.txt` and found out that we have been trolled again from the mentioned `/secret` page like the index page.

![](../img/troll/web1.png)

Robots.txt's content:-

```
User-agent:*
Disallow: /secret
```

From that pcap file we founded a directory reference so let's try it,that directory listing from the web I found that there is a binary and it was 32 Bit ELF, running it shows the following string:-

![](/../img/troll/hiidendir.png)

```
robin@oracle:~/Vulnhub/Tr0ll$ ./roflmao
Find address 0x0856BF to proceed
```
That was a directory reference, going over there I found 2 more directories.
![](../img/troll/hexxdir.png)

The `good_luck` directory has `which_one_lol.txt` having following data:-

```
maleus
ps-aux
felux
Eagle11
genphlux < -- Definitely not this one
usmc8892
blawrg
wytshadow
vis1t0r
overflow
```
Other one has a text file named `Pass.txt` which says `Good_luck:-)`.

# SSH Login and Root Flag

Using hydra to craft a dictionary attack. since at this point it was way too clear. Now, I tried doing th attack with `hydra -L new.txt -P pass.txt 192.168.43.169 ssh` which was giving a error, I had no idea what is happening sice the machine involved some trolling for the users, aftering spending 15 minutes on the argument being provided to the hydra, I tried `hydra -L user.txt -p Pass.txt 192.168.43.169 ssh` which gave us the password `Pass.txt` for user `overflow`, oh god that was definitely a good troll.

![](../img/troll/sshcreds.png)

Now, there are some commands I usually run in order to find anything interesting, so while trying them, I found the linux kernel is way too old usingg `uname -a` which in our case seems exploitabe. Using [this](https://www.exploit-db.com/download/37292) exploit, I downloaded it on my host machine and started a local HTTP server via `python -m SimpleHTTPServer 1337` in order to transfer te file to the machine.
![](../img/troll/localhttp.png)

Now, I checked whether the machine has gcc installed or not and to our surprise it does. Now,let's compile and run it already. Using `gcc -o root root.c` since I renamed that file to `root.c`. Running that exploit gave us root shell.

![](../img/troll/root.png)

##### The Flag

Time to get flag:-

![](../img/troll/rootflag.png)

That was it, until then enjoy.

##### Fun

While enumerating the system I found `lamo.py` in `/opt` folder which was the cause of our automatic connection close because of timing constraints. Here, it's content:-

![](../img/troll/fun.png)

Just after checking it, I got disconnected. 