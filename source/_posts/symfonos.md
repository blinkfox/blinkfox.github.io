---
layout:   post
title:      "Vulnhub - Symfonos-2"
date:       2019-07-20
author:     "D4mianwayne"
tag:      nmap, ctf, vulnhub
category: Vulnhub
---

Today, we are going to pwn Symfonos 2 from Vulnhub.

# Nmap

Starting off with Nmap, using `nmap -sV -sC -A -p- -T5 192.168.43.85` shows that we have FTP, SSH, HTTP and SMB port. 

![](/img/symfonos2/nmap.png)

# HTTP and FTP Enumeration

Going to the HTTP port shows a similar image that we encountered in previous symfonos machine. Nothing is there, running gobuster, dirb and other web crawlers gave nothing. So moving over to SMB, sice it has `Backup` folder which is publicly accessible, we can see a `log.txt`. It has following data:


SAO, we get know thatthere is user aeolus and the directory where backup is being stored.
So, as we knew FTP service is available which is ProFTP 1.3.5, searching for the exploit on ExploitDB. It has a exloit of mod_copy which allows you to copy files from one folder to another. [@mzfr](https://twitter.com/0xmzfr) discovered it.
We tried it and we were almost close to making it work but alas, we couldn't. It was just a silly mistake and from this point we accidentaly did the machine in a **very** unintended way. 

```

site cpfr /etc/passwd
350 File or directory exists, ready for destination name
site cpto <?php phpinfo(); ?>
550 cpto: Permission denied
site cpfr /proc/self/fd/3
350 File or directory exists, ready for destination name
site cpto /var/www/test.php

```


#### Note

I will explain the other method at last which [@zayotic](https://twitter.com/zayotic) told  [@mzfr](https://twitter.com/0xmzfr) after completion of machine.

# FTP Bruteforce and SSH Login

Well, from this point using **THC-Hydra** and **rockyou.txt** we got the password of user aeolus. `hydra -l aeolus -P rockyou.txt 192.168.43.85 ftp -t 40` reveals that the password is `sergioteamo`. Now, we tried it to SSH into the machine with the same creentials.

![](/img/synfonos2/hydra.png)

**SSH Login**

![](/img/symfonos2/ssh.png)

# Privilge Escalation

With some enumeration, I foud that there are 2 users **aeolus** and **cronus**. I tried it enumerate everything I can but at the it was a wild goose chase. So [@mzfr](https://twitter.com/0xmzfr) told to me check `/etc/apache2` folder and checking some files from that I cme to know that there is a local web server running on the machine, to confirm this I used **curl** which confirms it. So using `socat TCP-LISTEN:5000;fork,reuseaddr 127.0.0.1:8080` we just forked te actual port i.e. 8080 to 5000 of the actual web server, now visiting to `http://192.168.43.85:5000` shows that it is a **LibreNMS** based web application. I checked ExploitDB and it has exploit for adding a hostname that will give reverse shell, using metasploit I added the module to the `~/.ms4/exploits` directory.

![](/img/symfonos2/website.img)

As the exploit will start working it will give a reverse shell as the user **cronus**. Using `sudo -L` we can see that **mysql** can be run as root. [GTFOBins](https://gtfobins.github.io) to the rescue.

That has shown that using `sudo mysql '\! /bin/sh` we can spawn root shell.

![](/img/symfonos2/sql.png)

![](/img/symfonos2/root.png)

**The Flag**

![](/img/symfonos2/flag.png)

# Intended Way

As much as I was happy for pwning this machine but it was temporry the main learning part is missed in it by using hydra.
So as I mentioned eariler we were supposed to use that ProFTP 1.3.5 vulnerbaility which can be found here to copy files from one folder to another. From that exploit we can copy the files from any folder and copy it to `/home/aeolus/share/` that will make that file publicly accessible to us via SMB. So, copying the file `/var/backups/shadow.bak` to `/home/aeolus/share/shadow.txt`. After that it will be available to SMB service which can be used to dump the users password and then cracked with **john** to crack the hash of the user **aeolus**.

# Resources

ProFTP 1.3.5 - File Copy: https://www.exploit-db.com/exploits/36742
GTFOBins: https://gtfobins.github.io
LibreNMS Exploit: https://www.exploit-db.com/exploits/46970

#### Thanks to

[@mzfr](https://twitter.com/0xmzfr) 
[@DCAU7](https://twitter.com/DCAU7) 

#### Detailed version og intended way.

The very detail version of the intended way is covered by [@mzfr](https://twitter.com/0xmzfr) in his [writeup](https://mzfr.github.io/symfonos2).
