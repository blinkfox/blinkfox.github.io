---
layout:   post
title:      "Vulnhub - PumpkinRaising"
date:       2019-07-17
author:     "D4mianwayne"
tag:      nmap, ctf, vulnhub
category: Vulnhub
---

Today, we are going to pwn PumpkinrRaising from Vulnhub.

# Nmap

Starting off with nmap and using `nmap -sV -sC -A -p- -T5 192.168.43.92` shows that only 2 ports are opren `22`, `80`. 

![](/img/pumpkin-garden/nmap.png)

# Surfing on HTTP and Enumeration

So from above nmap scan we can see there is a `robots.txt` file with most diasllowed enteries but first off we should check off the source of the index page.

![](/img/pumpkin-garden/robots.png)
#### First ID


Now, moving on further we can see `robots.txt` has a gpg file path so using curl we get it on system `curl http://192.168.43.92/seeds/seed.txt.gpg > seeds.txt.gpg`. Upon checking it it turns out to be AES-256 encrypted ciphertext, we need a key in ordwe to decrypt it. Finding the key was kind of guessy and more like connecting dots. I tried several passphrases but it fails everytime so I started checking index page again for some hint, there were3 words `SEED - WATER - SUNLIGHT`, so I tried `SEEDWATERSUNLIGHT` turns out to be the right key which gives `seeds.txt` which has morse code which decodes to another 5 digit ID.

> YIPPEE! YOU ARE ON THE RIGHT PATH... BIGMAXPUMPKIN SEEDS ID: 69507 


#### Second ID

 Upon analyzing it, we can see there is a `pumpkin.html` so going thre didn't give any hint so I checked source of that page which guves base32 encoded data.
Decoding that data with `echo -n | base32 -d` gives path for a pcap file.
This one was pcap forensics challenge but easy one, as we saw the pcap file in one of the disallowed enteries. Opening it in wireshark and following TCP stream we can see there is a conversation and hence we can see our third ID.

***

Hey Jack, Robert has given me your contact. I'm sure I have the seeds that you want
Hi Mark, I'm greatful that you have the seeds
Please share the seed ID so that I can get you exact seeds
Sure, 50609 is the ID
Thank you, I have the seeds. You'll get your seeds in a couple of days
Thank you so much Mark
You're welcome

***

![](/img/pumpkin-garden/second.png)

#### Third ID

Checking the source code of index page shows a route for `pumpkin.html` which was also a pumpkin page so checking the source again shows bunch of hex characters at the very bottom of the page which decode to:

***

59 61 79 21 20 41 70 70 72 65 63 69 61 74 65 20 79 6f 75 72 20 70 61 74 69 65 6e 63 65 20 3a 29 0a 41 6c 6c 20 74 68 69 6e 67 73 20 61 72 65 20 64 69 66 66 69 63 75 6c 74 20 62 65 66 6f 72 65 20 74 68 65 79 20 62 65 63 6f 6d 65 20 65 61 73 79 2e 0a 41 63 6f 72 6e 20 50 75 6d 70 6b 69 6e 20 53 65 65 64 73 20 49 44 3a 20 39 36 34 35 34 0a 0a 44 6f 2c 20 72 65 6d 65 6d 62 65 72 20 74 6f 20 69 6e 66 6f 72 6d 20 4a 61 63 6b 20 74 6f 20 70 6c 61 6e 74 20 61 6c 6c 20 34 20 73 65 65 64 73 20 69 6e 20 74 68 65 20 73 61 6d 65 20 6f 72 64 65 72 2e


>Yay! Appreciate your patience :)                                                                                                                                All things are difficult before they become easy.                                                                                                             Acorn Pumpkin Seeds ID: 96454                                                                                                                                                                                                                                                                                                     Do, remember to inform Jack to plant all 4 seeds in the same order.

***

#### Fourth ID


This took sometime since we had already check all the things and I checked way too much time every single page and I started checking `underconstruction.html` wecan see there is a gif file so I started checking it, I was cluless here so [@mzfr](https://twitter.com/0xmzfr) and he told me to use stegosuite for it. So, using `stegosuite -x jackolantern.gif -k <password>`, so we need a password for it as well. I tried using everything for that but nothing. So as we saw i `robots.txt` it has a disallowed entry for `/hidden/note.txt` file.
Upon opening it we can see credentials combos, 

***

Robert : C@43r0VqG2=
Mark : Qn@F5zMg4T
goblin : 79675-06172-65206-17765

***

I tried every password of the above users and got a success with Mark's which gives `decorative.txt` which has fourth and last 5 digit ID.

![](/img/pumpkin-garden/stegosuite.png)

***

Fantastic!!! looking forward for your presence in pumpkin party.
Lil' Pump-Ke-Mon Pumpkin seeds ID : 86568

***

# SSH Login and Root Flag

So, as we got hint from last seed "it has to planteded in same order", so that means we need to get them in correct order which is

***

69507 
50609 
96454  
86568

***

So, using this passord which is `69507506099645486568` for user `jack` we looged into pumpkin machine.

![](/img/pumpkin-garden/ssh.png)

So using `sudo -l` to find which binary we can use as sudo which gives that strace can be used as sudo user.
Few weeks ago, I tried `unknowndevices64` which had a similar type of root privilege.
So, `sudo strace -o /dev/null /bin/sh` spawns a root shell for us and hence we can use read root flag.

![](/img/pumpkin-garden/root.png)


That was it folks, we got it. It was great CTF based machine for beginners. Kudos to [@mzfr](https://twitter.com/0xmzfr) for helping me out.

