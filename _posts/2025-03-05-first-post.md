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

![Screenshot](assets/images/ip_geolocation.png)

