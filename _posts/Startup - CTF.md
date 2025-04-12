---
layout: post
title: "Startup"
author: "Nicholas Coleman"
date: 2025-04-12
tags: tryhackme
---

## Startup

My task is to do a penetration test on a food-based start-up. They have provided me with their IP address. This is a black box penetration test.

### Nmap Scan

```
# Nmap 7.95 scan initiated Sat Apr 12 14:36:35 2025 as: /usr/lib/nmap/nmap --privileged -sC -sV -oA startup 10.10.225.222
Nmap scan report for 10.10.225.222
Host is up (0.095s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
| -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
|_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.6.25.18
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b9:a6:0b:84:1d:22:01:a4:01:30:48:43:61:2b:ab:94 (RSA)
|   256 ec:13:25:8c:18:20:36:e6:ce:91:0e:16:26:eb:a2:be (ECDSA)
|_  256 a2:ff:2a:72:81:aa:a2:9f:55:a4:dc:92:23:e6:b4:3f (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Maintenance
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Apr 12 14:36:49 2025 -- 1 IP address (1 host up) scanned in 14.10 seconds
```

## FTP

It looks like the FTP server used by the startup has anonymous login enabled. That makes it a lot easier for me to snoop around.

FTP:
![[FTP.png]]

I couldn't really find anything useful, but maybe I can upload a malicious file that I can then execute from the browser. This FTP server is most likely for their website that is currently being built. 

After doing a gobuster search:
![[gobuster.png]]

Bingo - there is a `/files` path. When navigating to this page, I notice that it is the same directory as when we accessed by FTP. This means I can upload a malicious file through the FTP connection and execute it via the browser!

For this I can reuse Pentestmonkey's php-reverse-shell script, that I used for another CTF, to help me gain access.

![[ftp_upload.png]]

Just like that - the reverse shell script is now showing up on the `/files` page. I can execute it by simply double-clicking. However before executing it, I made sure to be listening in using netcat on the same port that I specified in the php script:

![[ftp_php.png]]

Once inside, I am able to find their secret ingredient quite easily:
![[recipe.png]]

Using `bash -i` will allow us to use a stable shell in the machine.

From here I found a suspicious Wireshark capture. I copied it to the ftp folder so that I could download it and view it on Wireshark on my VM.

![[wireshark_capture.png]]

After looking through the different conversations, I noticed that in this TCP stream there were a few password attempts that, although failed, may indicate something important. Although the password doesn't seem to work for root, it may be important still.

After all, they did try it 3 times:
`c4ntg3t3n0ughsp1c3`

Maybe it works for ssh'ing into the user (lenny)'s machine? 

![[ssh_login.png]]

There you go! Finding the user.txt was easy after this. It was located in the home directory.

Now, we have to escalate privileges to obtain the root flag. A bit harder.

Looking around the machine, I was able to find a directory called scripts:
![[scripts.png]]

Good news -> planner.sh is owned by root:

`-rwxr-xr-x 1 root   root     77 Nov 12  2020 planner.sh`

Abusing this privilege would be the most logical attack vector. 

To get more information, I installed pspy64 so I could see what processes are running under root. Doing so will allow me to determine more about what exactly is being executed here.

From this, I can see that the planner.sh is running print.sh every minute. I can take advantage of this by changing print.sh so that it gives me a reverse shell. Before this, I set up netcat to listen in.

![[planner.png]]

After running that, I checked to see if I had gained root privileges. Bingo!

![[Startup - Photos/Final.png]]

Abusing these types of scripts is easy, and a good reminder of why you must be very careful when using scripts that take advantage of sudo privileges. 
