---
layout: post
title:      "Vulnhub - Pumpkin Festival"
date:       2019-07-24
author:     "D4mianwayne"
tag:      nmap, ctf, vulnhub
category: Vulnhub
---

Today, we are going to pwn Pumpkin Festival from Vulnhub.

# Nmap

Starting off with the nmap using `nmap -sV -sC -A -p- -T5 192.168.43.17` reveals that we have FTP, HTTP and SSH at 6880.

![](/img/Pumpkin-Festival/nmap.png) 

# Tokens

From the first discovery I found that flag are named as token and that is our challenge i.e. finding all tokens.

#### First

First one was at the botttom of the HTML page can be viewed upon inspection of the page.

![](/img/Pumpkin-Festival/first.png) 

#### Second

With the result of nmap scan we seen that anaonymous login is open. Using the package `ftp` we logged into the machine and  in the directory named secret we found a file named `token.txt` having the second token.

>PumpkinToken : 2d6dbbae84d724409606eddd9dd71265

![](/img/Pumpkin-Festival/second.png)

#### Third

This one requires bruteforcing the FTP service in order to get token, as we have inspected HTTP client for first we saw that there is a user named harry. On the basis of that assumption, using `hydra -l harry -P rockyou.txt 192.168.43.17 ftp -t 60`, it will take some time so don't get impatient.

Password of user **harry**:-

![](/img/Pumpkin-Festival/harry-ftp.png)

Logging in with user **harry** shows that we have a `token.txt` and a directory named `Donotopen`. Get the token and start changing directories.

>PumpkinToken : ba9fa9abf2be9373b7cbd9a6457f374e

#### Forth          

While logged in FTP, and chanaging bunch **NO** named nested directories, we get another token and a `data.txt`.

>PumpkinToken : f9c5053d01e0dfc30066476ab0f0564c

That `data.txt` was a tar file which can be confirmed using `file data.txt`. 

Extracting it gives a hex encoded OpenSSL Private Key.

![](/img/Pumpkin-Festival/tarfile.png)

#### Fifth

Checking `robots.txt` on a HTTP we get see a `/tarck/track.txt`, going to that webpage we see:-

***

Hey Jack!

Thanks for choosing our local store. Hope you like the services.
Tracking code : 2542 8231 6783 486

-Regards 
admin@pumpkins.local

***

`admin@pumpkins.local` seems interesting, editing the hosts file we add `pumpkia.local`. After this, we get a wordpress temed website. Using wpscan doesn't give any interesting results. So, using dirb we see a lot of pages which can be seen at any normal wordpress website.

![](/img/Pumpkin-Festival/dirb.png)

Going to `readme.html` it shows a base?? encoded data, using CyberChef and trying multiple bases we can see it's a base64 data which decodes to credentails.

***

<!DOCTYPE html>
<html>
<head>
	<meta name="viewport" content="width=device-width" />
</head>
<body>
<h1 id="logo">
	<a href="https://wordpress.org/"><img alt="WordPress" src="wp-admin/images/wordpress-logo.png" /></a>
</h1>
<p style="text-align: center">Semantic Personal Publishing Platform</p>

<center><h2>-- This content is removed because of security purposes --</h2>
<p>K82v0SuvV1En350M0uxiXVRTmBrQIJQN78s</p></center>

</body>
</html>

***

>morse & jack : Ug0t!TrIpyJ

So, logging into the wordpress with as morse we see that there is a flag in biographical info.

>PumpkinToken : 7139e925fd43618653e51f820bc6201b

#### Sixth

Now, so we had a OpenSSL private key, we can use it to get into the machine via SSH as jack with `ssh -i key jack@192.168.43.17 -p 6880` we get into the system.

![](/img/Pumpkin-Festival/ssh.png)

In the home directory we can see a ELF named token, running it we can get our seventh token.

>PumpkinToken : 8d66ef0055b43d80c34917ec6c75f706


# Root

Using wordpress credentials for jack we can use sudo. Using `sudo -l` we can see that creating a file within `/home/jack/pumpkin/alohomora` we can execute any binary. From our previous encounter of privilege escalation we are going to create a binary named alohooa.

```

jack@pumpkin:~/pumpkins$ echo "/bin/sh" > alohomora
jack@pumpkin:~/pumpkins$ chmod +x alohomora
jack@pumpkin:~/pumpkins$ sudo ./alohomora
# id
uid=0(root) gid=0(root) groups=0(root)
# whoami; id; cd root; cat *.txt
root
uid=0(root) gid=0(root) groups=0(root)

```

Hence, checking the root directory we can see a great art of pumpkin.


![](/img/Pumpkin-Festival/root.png)


### Thanks

Thanks to [@mzfr](https://twitter.com/0xmzfr) and [Andre](https://twitter.com/mrhenrike) for helping me out.
