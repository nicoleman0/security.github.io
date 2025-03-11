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

![IP DST](/security.github.io/images/poison/ipdst.png)

Here we can see that the source address is `192.168.232.215`. This is the rogue machine that is responsible for poisoning the queries made by the victim. 

![rogue machine](/security.github.io/images/poison/nbns_rogue.png)

Looking through the details, we can see that this machine is claiming to have resolved the query (`FILESHAARE<20>: type NB, class IN`) and is trying to spoof the legitimate host's identity. This is more proof of an LLMNR/NBT-NS poisoning attack. 

Following this revelation, the next order of business was to assess whether user accounts on the network were compromised. 

To do that, first I had to isolate all of the NBNS traffic coming from the rogue machine/attacker, using the filter: `nbns.addr==192.168.232.215`. 

![nbns addr](/security.github.io/images/poison/nbns_addr.png)

From this, we can see that the rogue machine is also resolving queries to `192.168.232.176`.

These poisoned responses are intended to redirect network data by falsely resolving queries for names that do not exist. 

This is intended to trick target machines into interacting with the rogue machine. 

Now we know the IP address of the rogue machine. From this, we can filter our search with `ip.dst==192.168.232.215` to look for any evidence of compromised usernames. 

By examining one of the captured SMB2 packets sent to the rogue machine, the compromised username is clearly visible: `janesmith`.

![account name](/security.github.io/images/poison/account_name.png)

Lastly, finding out the hostname of the rogue machine was the final step. Filtering for all of the SMB2 packets, I looked into packet 453, which indicated a session setup response error. Under the session response information, the NetBIOS name was listed as `ACCOUNTINGPC`. This is the rogue machine that was being used by the threat actor to exfiltrate usernames and potentially other sensitive information from the client. 

![host name](/security.github.io/images/poison/hostname.png)

---

### Security Reccomendations

It is *strongly* recommended to isolate the rogue machine from the rest of the network as soon as possible. Doing so will allow the IT team to further diagnose, and resolve the issue. 

However, this does not solve the root cause of this issue, which are NetBIOS vulnerabilities. If NetBIOS is not necessary for business operations, the recommended action is to disable the service entirely. Using DNS and SMB protocols instead would provide a much safer network environment and minimize the attack surface, possibly preventing future attacks like this.

If this is not possible, blocking NetBIOS traffic inbound/outbound to the internet would be the next best step. If that is not possible, limiting NetBIOS' use to specific whitelisted IP addresses is also an option. However, this is far less effective than simply disabling NetBIOS entirely, especially since it is an outdated legacy service. 

### Summary
1. WHO: Rogue machine: ACCOUNTINGPC with IP addr: 192.168.232.215
2. WHERE: NetBIOS service
3. WHAT: Query poisoning to redirect traffic to attacker's machine
4. WHY: To exfiltrate PII (usernames)
