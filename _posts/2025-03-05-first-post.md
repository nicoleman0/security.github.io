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

![the IP location](/images/ip_geolocation.png)

Although the IP address belonging to someone in China was already a bit suspicious, I needed more evidence of it being malicious. My next plan of action? To find out the User-Agent associated with this address. Since attackers often spoof User-Agent strings to mask the tools they use as legitimate browsers/applications, finding the User-Agent string was imperative.

Since I was already in Wireshark, that wouldn't be too difficult. All that needs to be done is to inspect a captured HTTP GET request. Easily done by following an HTTP stream.

`*Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0*`

It looks like the attacker was using a Linux-based Firefox browser, which could help him blend in with the rest of the traffic. This is valuable information, because it can help detect future attempts easily.

Next, I needed to see if the attacker exploited any vulnerabilities on the company web server. They most likely uploaded malicious code to allow themselves remote control over the server. This is otherwise known as the *web shell*. 

To do that, I needed to isolate all of the HTTP POST requests coming from the attacker's IP address. 

`ip.src == 117.11.88.124 and http.request.method == "POST"`

![The first attempt]

Here the attacker is trying to upload a malicious script, innocently named, "image.php". 

Attackers will often name malicious files with innocuous names, so as to avoid suspicion. But the very fact that a mysterious IP address is attempting to inject a PHP script into the company web server is enough by itself.

![Part 2]

See their IP address nestled in that PHP script? That alone is suspicious, and supported my conclusion that the attacker was trying to establish a reverse shell. Clearly, they are with the `system` function. It seems this first attempt was a failure however. The company web server's filters were able to stop the malicious file from executing. 

![sucess]

The second attempt was successful though, and the malicious PHP file was uploaded succesfully. It was able to bypass the server's filtering, most likely because the attacker tricked the filtering mechanism by adding a .jpg extension before the .php extension. This effectively made the server bug out, because it read the .jpg extension first, and probably thouugh it was an innocent photo. 

Well, it wasn't. The input validation mechanism of this server should definitely be updated!

---

To be continued...
