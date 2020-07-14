---
layout:    post
title:      "Hack The Box - Bastion"
date:       2019-09-10
author:     "D4mianwayne"
category: Hack The Box
tag: windows, htb
---


Write-up for Bastion from hackthebox.eu 

# Nmap

Starting off with nmap we use `nmap -sV -sC -A 10.10.10.134` shows `22`, `139`, `445`, `135` are open. 
Basically, we gonna use **smbclient** for further enumeration.

![](/img/bastion/nmap.png)

# SMB Service Enumeration

Using `smbclient -L //10.10.10.134` shows there are 4 folders on the network drive. Again, using smbclinet for that result in error for every folder excepyt **Backups**.

Using, `smbclient //10.10.10.134/Backups -U root` we can access the files which typically contains `notes.txt` , `nmap-test-file` , `SDTC56B` and a folder named `WindowsImageBackup`.

![](/img/bastion/notes.png)

#### Checking Files

The file named `notes.txt` has following contents:-

>Sysadmins: please don't transfer the entire backup file locally, the VPN to the subsidiary office is too slow.

#### Mounting the Shared Drive's Folder

Since we want to take a closer look in it **Backups** folder, the best way to do that is to mount that drive via inbuilt `mount` program.
Using `mount -t cifs //10.10.10.134/Backups -o user=guest,password= /tmp/mnt` we can mount the drive it took sometime since the network was slow.

![](/img/bastion/mountdrive.png)

#### Mounting the .vhd files

Looking for files in Backups shows that there was a **.vhd** file, since as previously suggested in `notes.txt` we not gonna get the whole .vhd file instead of that we will mount it using `guestmount` package tool.
Using `guestmount --add /mnt/backups/WindowsImageBackup/L4mpje-PC/Backup\ 2019-02-22\ 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd --inspector --ro /mnt/vhd` we ca succesfully mount the vhd file for further enumeration.
Network was kinda slow it took ~15 minutes.

# SSH Password Recovery and SSH Login

There were a lot of **.xml** files but as we do in linux machine enumeration we head over to `System32/config` and using one of the known tools for dumping the password is `samdump2`and with `samdump2 SYSTEM SAM` we can dump the passwords aka NTLM hash of users password.

![](/img/bastion/passwordfile.png)

Since we got the NTLM hash, usinh [Hashkiller](https://hashkiller.co.uk/Cracker/NTLM) we found the hash value that is `buraeulampje`.

>26112010952d963c8dc4217daec986d9                                                    bureaulampje

Using above password for user **L4mpje** we successfully logged into the machine.

![](/img/bastion/sshlogin.png)

# User Flag and Root Password

Though, I spent quite a few time in finding user flag thrn I used `dir /s *user.txt*`, I saw the password is in `/L4mpje/Desktop`, hence we obtain the user flag.

![](/img/bastion/user.png)

Time to enumerate for root, I saw mRemoteNG and speaking of it, mRemoteng stores user's password in a config file so time to get it.

So, I went to `L4mpje/AppData/mRemoteng` to et the `config.ini` we can see administartor password looks base64 encoded encrypted data. 
After obtaining that ciphertext I started searching for tools for cracking that ciphertext. Lckly, I found a github repo with [mremoteng-decrypt](https://github.com/kmahyyg/mremoteng-decrypt/), using the python file and `python3 mremoteng-decrypt -s <hash>` we can get the password of Administrator i.e. `thXLHM96BeKL0ER2`.

![](/img/bastion/password.png)

After that I tried `rnas /users:Administrator CMD.exe` but it didn't worked so I tried ssh again and we were logged in as Administrator in the machine. So going to `Users/Administrator/Desktop` we obtained our `root.txt`.

![](/img/bastion/root.png)
