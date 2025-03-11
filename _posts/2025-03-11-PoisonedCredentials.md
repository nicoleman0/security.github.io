---
layout: post
title: "LLMNR/NBT-NS Poison Attack"
author: "Nicholas Coleman"
date: 2025-03-11
tags: cyberdefenders
---
The client believes that there are LLMNR/NBT-NS poisoning attacks occurring within their network. 

With this knowledge, I opened the pcap file given, and began searching with the Wireshark filter: `llmnr and ip.src == 192.168.232.162` which helped narrow down possible poisoned queries. In a short amount of time, I noticed the mistyped query: "fileshaare". 

To find the potential rogue machine that was poisoning queries, I analyzed responses to the mistyped query. 

I used the victim IP address as the destination in my search filter, and looked for any NetBIOS Name Service (`NBNS`) query responses, and found what I needed at entry #51.
