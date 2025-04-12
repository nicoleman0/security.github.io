---
layout: post
title: "Startup"
author: "Nicholas Coleman"
date: 2025-04-12
tags: tryhackme
---

My task is to do a penetration test on a food-based start-up. They have provided me with their IP address. This is a black box penetration test.

### Nmap Scan

```
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

### FTP

It looks like the FTP server used by the startup has *anonymous* login enabled. That makes it a lot easier for me to snoop around.

![ftp](/security.github.io/images/startup/FTP.png)

I couldn't really find anything useful, but maybe I can upload a malicious file that I can then execute from the browser. This FTP server is most likely for their website that is currently being built. 

After doing a **gobuster** search:

![gobuster](/security.github.io/images/startup/gobuster.png)

Bingo - there is a `/files` path. When navigating to this page, I notice that it is the same directory as when we accessed by FTP. This means I can upload a malicious file through the FTP connection and execute it via the browser!

For this I can reuse Pentestmonkey's php-reverse-shell script, that I used for another CTF, to help me gain access.

![ftp_upload](/security.github.io/images/startup/ftp_upload.png)

### php reverse shell

Just like that - the reverse shell script is now showing up on the `/files` page. I can execute it by simply double-clicking. However before executing it, I made sure to be listening in using netcat on the same port that I specified in the php script:

![ftp_php](/security.github.io/images/startup/ftp_php.png)

Once inside, I am able to find their secret ingredient quite easily:

![recipe](/security.github.io/images/startup/recipe.png)

Using `bash -i` will allow us to use a stable shell in the machine.

### Suspicious pcap

From here I found a suspicious Wireshark capture. I copied it to the ftp folder so that I could download it and view it on Wireshark on my VM.

![wireshark](/security.github.io/images/startup/wireshark_capture.png)

After looking through the different conversations, I noticed that in this TCP stream there were a few password attempts. They seem to have been attempted by someone that was doing the same thing I was. Maybe another pentester? Probably a malicious actor though... But either way - this password obviously has some kind of meaning. They did try it (`c4ntg3t3n0ughsp1c3`) three times in a row...

Maybe it works for ssh'ing into the user (lenny)'s machine?

### SSH to user machine

![ssh_login](/security.github.io/images/startup/ssh_login.png)

There you go, it worked! Sometimes you just have to try things out.

Finding the user.txt was easy after this. It was located in the home directory.

Now, we have to escalate privileges to obtain the root flag. A bit harder.

Looking around the machine, I was able to find a directory called scripts:

![scripts](/security.github.io/images/startup/scripts.png)

I checked the script, `planner.sh`, and saw that it is owned by root:

`-rwxr-xr-x 1 root   root     77 Nov 12  2020 planner.sh`

Abusing this privilege would be the most logical attack vector. 

## pspy64

To find out more about the system and what processes are going on, I installed **pspy64** (a tool that allows for unprivileged Linux process snooping). Doing this will allow me to determine if this `planner.sh` is doing anything interesting. 

From this, I can see that the planner.sh is running print.sh every minute. I can take advantage of this by changing print.sh to be a reverse shell script. Before this, I set up netcat to listen in.

![planner](/security.github.io/images/startup/planner.png)

After running that very insecure script, I checked to see if I had gained root privileges:

![FINAL](/security.github.io/images/startup/Final.png)

Nice! That wasn't too bad. Not good for this startup though.

### Conclusions

- Leaving anonymous FTP login can be useful in many ways. But it is an unsecured protocol, and should only be used if absolutely necessary and for very specific functions. Otherwise it should be left disabled, as it can provide attackers with the ability to write files and upload malicious scripts.
  - This misconfiguration of anonymous FTP was what ultimately provided me the first 'in'. Without that, it would have been a lot harder to breach the system.
- Another huge issue was the pcap file containing important credentials. Hosting an important file, with a *password* for your system, on your website's remote server (which can be publically FTP'd into) is a terrible idea. That was how I was able to find the password for the lenny user on the target system.
- Last but not least: having a script that can be executed by the user, without sudo priveleges - which uses sudo commands, is a HORRIBLE idea. I was able to abuse this very easily, and gave myself root admin access with a simple reverse shell.
