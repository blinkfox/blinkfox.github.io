---
layout:   post
title:      "Vulnhub - Symfonos"
date:       2019-07-06 
author:     "D4mianwayne"
tag:      nmap, ctf, vulnhub
category: Vulnhub
---

Today, we are going to pwn Symfonos from Vulnhub.

### Machine Setup

Nothing much to setup in the machine, just import it to Virtualbox and choose the networking setting as per your ease, I chose the Bridge Networking as it's easy to setup.

## Methodology

* Nmap scan of the machine.
* Checking the SMB server for files.
* Checking Helios folder for important files.
* Discovering the WordPress website in `/h3l105` route.
* Scanning website for the vulnerabilities via wpscan.
* Exloiting Mail Masta 1.0 Plugin for LFI(Local File Inclusion).
* Enumerating SMTP server.
* Using SMTP for getting a reverse shell thrrough `/var/mail/helios` as Helios on machine.
* Gaining root peivileges by exploiting `/opt/statuscheck` binary.


# Nmap

Let's get started, using `nmap -sV -sC -A 192.168.43.12` reveals HTTP, SSH, SMTP, and SMB. 

![](/img/symfonos/symfonosnmap.png)

# SMB service enumeration

Honestly speaking, I've had some problems using `smbclient` so I usually take the help of inbulit File application connect to server feature.
So moving to File, I connectedto the server and there were two folders, first **Anonymous** and second **Helios**, Helios folder was password protected and Anonymous was avilable to everyone so I checked **Anonymous** folder and there was a file named as `attention.txt`. 

![](/img/symfonos/symfonossmb.png)

It has following information:-

***

>Can users please stop using passwords like 'epidioko', 'qwerty' and 'baseball'! Next person I find using one of these passwords will be fired!

-Zeus

***

So, using those password one by one for helios folder reveals folder's files with password `qwerty`. 
It has 2 files `research.txt` and `todo.txt`.

***

research.txt

>Helios (also Helius) was the god of the Sun in Greek mythology. He was thought to ride a golden chariot which brought the Sun across the skies each day from the east (Ethiopia) to the west (Hesperides) while at night he did the return journey in leisurely fashion lounging in a golden cup. The god was famously the subject of the Colossus of Rhodes, the giant bronze statue considered one of the Seven Wonders of the Ancient World.

***

Not relevant.

***

todo.txt


>1. Binge watch Dexter
>2. Dance
>3. Work on /h3l105

***

There `/h3l105` is our focus.

# WordPress Enumeration

So, accesing to `http://192.168.43.12/h3l105` reveals a wordpress website with login, post, castegories and other link.

***

Note

>When you try to acces `/wp-login.php`, you'll recieve a error so to resolve that add machine IP Address to your `/etc/hosts` file as `symfonos.local`.

***

Well, it's wordpress so `wpscan` to rescuse.
Using `wpscan --url http://symfonos.local/h3l105 --no-banner --no-update` reveals two CVEs, one is SQL Injection and other LFI vulnerability both in Mail Masta Plugin 1.0.

![](/img/symfonos/symfonoswpscan.png)

# Exploiting LFI Vulnerabilty 
[Here](https://www.exploit-db.com/exploits/40290), shows LFI PoC. So implementing the PoC on `http://symfonos.local/h3l105`,

Visiting `http://symfonos.local/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd` reveals the content of `/etc/passwd`.

![](/img/symfonos/symfonosetc.png)

We can also access `/wp-config.php`(this was not relevant) as well as some other files including `/var/mail/helios`.

I was completely clueless so I asked [@DCAU7](https://twitter.com/DCAU7) for hint, he told me that I need to use SMTP to progress from here.

# SMTP Enumeration and Reverse Shell via SMTP

Using telnet we can connect to SMTP and we can send data to `/var/mail/helios` since it's accessible through LFI vulnerability.

>telnet symfonos.local 25

Once connected we can use it to execute our php code by accessing to `http://symfonos.local/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/var/mail/helios`.

![](/img/symfonos/symfonosvarmail.png)

First let's send a email to helios with our reverse shell

Using telnet we can connect to SMTP and from there we can send our reverse shell.

>telnet symfonos.local 25

```

MAIL FROM: Robin
RCPT TO: helios
DATA
<PHP Reverse Shell>
.
quit

```

![](/img/symfonos/symfonossmtp.png)

# Getting User and Root

So, once email will be sent we can route to `http://symfonos.local/h3l105/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/var/mail/helios` and that will execute our reverse shell.

![](/img/symfonos/symfonosshell.png)

>find / -perm -u=s -type f 2>/dev/null

![](/img/symfonos/symfonosfind.png)

Reveals a interesting file `/opt/statuscheck`, so analyzing that binary reveals that it is scraping the document info of that website. Using 
`strings statuscheck` reveals that it's using curl i.e. `curl -I http://localhost.com` which basically gives the head information of a website. I tried to search for curl privilege escalation but sadly it wasn't useful.
I was stuck at this but [@mzfr](https://twitter.com/0xmzfr) gave me a hint that it's something that we have done before.

![](/img/symfonos/symfonosstrings.png)

So, that helped me a lot. I chnaged directory to `/tmp` folder and ceated a binary named curl by using following commands:-

```

$ echo $"#!/bin/sh\n/bin/sh" > curl
$ chmod 777 curl
$ export PATH=:$(pwd)
$ /opt/statuscheck

```

![](/img/symfonos/symfonosroot.png)


That was it folks.

# Note

So, [@mzfr](https://twittwer.com/0xmzfr) gained that reverse shell in a different way than mine. So, there are two ways to gain reverse shell to the symfonos machine.
Here's his [writeup](https://mzfr.github.io/symfonos).
