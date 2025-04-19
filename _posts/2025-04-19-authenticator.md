---
layout: post
title: "Malicious Google Authenticator Report"
author: "Nicholas Coleman"
date: 2025-04-19
tags: tryhackme
---

### Malicious Ads on Bing Search Leads to Malware

*You work as an analyst at a Security Operation Center (SOC). Someone contacts your team to report a coworker has downloaded a suspicious file after searching for Google Authenticator. The caller provides some information similar to social media posts at:*

- [https://www.linkedin.com/posts/unit42_2025-01-22-wednesday-a-malicious-ad-led-activity-7288213662329192450-ky3V/](https://www.linkedin.com/posts/unit42_2025-01-22-wednesday-a-malicious-ad-led-activity-7288213662329192450-ky3V/)
- [https://x.com/Unit42_Intel/status/1882448037030584611](https://x.com/Unit42_Intel/status/1882448037030584611)

*Based on the caller's initial information, you confirm there was an infection.  You retrieve a packet capture (pcap) of the associated traffic.  Reviewing the traffic, you find several indicators matching details from a Github page referenced in the above social media posts.  After confirming an infection happened, you begin writing an incident report.*

#### LAN Segment Details
- LAN segment range:  **`10.1.17[.]0/24`**   (**`10.1.17[.]0`** through **`10.1.17[.]255`**)
- Domain:  **`bluemoontuesday[.]com`**
- Active Directory (AD) domain controller:  **`10.1.17[.]2 - WIN-GSH54QLW48D`**
- AD environment name:  **`BLUEMOONTUESDAY`**
- LAN segment gateway:  **`10.1.17[.]1`**
- LAN segment broadcast address:  **`10.1.17[.]255`**

#### Indicators of Compromise

**Files**

| Hash          | Filename                     | Type       | Suspicion                                   |
| ------------- | ---------------------------- | ---------- | ------------------------------------------- |
| `a833f27c...` | `pas.ps1`                    | PowerShell | Likely a payload script                     |
| `9634ecaf...` | `Teamviewer_Resource_fr.dll` | DLL        | Abused DLL                                  |
| `904280f2...` | `TeamViewer.exe`             | EXE        | Possibly modified/packed remote access tool |
| `3448da03...` | `TV.dll`                     | DLL        | Possibly injected                           |
| `fd045fce...` | `skqllz.ps1`                 | PowerShell | Possibly secondary script                   |

**From Malicious Ad on Bing**

- `hxxps[:]//www.bing[.]com/aclk?ld=e8PH8aLxSiJxjw4Si9lgLztzVUCUwCJ7LeV4z4DsU61Sx3HWK9X1fxNGVCWc4jKyspIeWPFeqVejCDavG1lRWD4Ukf127WLw1hUPnGntv_1Y1z30t5JNXJyKZ986BV2aP3kDwSnS0DDaXYX4hQcab6syHfzjtxZLUNJD5oG8MEhJwV-_N_vpfcrfaGeRQCbjbYwL3zeQ&u=aHR0cHMlM2ElMmYlMmZtaWNyb3NvZnQtdGVhbXMtZG93bmxvYWQuYnVybGVzb24tYXBwbGlhbmNlLm5ldCUzZm1zY2xraWQlM2Q5ZTYxNDgwMjZjMzIxNTJlM2ZkYzJmOTMwZDQ5MjNiYw&rlid=9e6148026c32152e3fdc2f930d4923bc&ntb=1`
- `Cloudflare IP:443 - hxxps[:]//microsoft-teams-download.burleson-appliance[.]net/?msclkid=9e6148026c32152e3fdc2f930d4923bc`
- `82.221.136.26:443 - hxxps[:]//microsoft.teams-live[.]com/en/index.html?msclkid=9e6148026c32152e3fdc2f930d4923bc`
- `82.221.136.26:443 - hxxps[:]//microsoft.teams-live[.]com/en/download.php`

### Task
- #### What is the IP address of the infected Windows client?
	- Repeated beaconing from: `5.252.153[.]241` using the `/8182020` URI.
- ![2ip](/security.github.io/images/auth/2_ip.png)
- The IP address of the infected Windows client is `10.1.17[.]215`

- #### What is the mac address of the infected Windows client?
	- Search query: `arp && (arp.src.proto_ipv4 == 10.1.17.215 || arp.dst.proto_ipv4 == 10.1.17.215)`
	- ![3](/security.github.io/images/auth/3.png)
	- Sender MAC address: **`Intel_26:4a:74 (00:d0:b7:26:4a:74)`**

- #### What is the host name of the infected Windows client?
	- `DESKTOP-L8C5GSJ<00>` is the host name.
	- <00> indicates that this is a Workstation Service (the base name used for identification on the LAN)

- #### What is the user account name from the infected Windows client?
	- ![5](/security.github.io/images/auth/5.png)
	- It is `shutchenson` -> for Steve Hutchenson

- #### What is the likely domain name for the fake Google Authenticator page?
	- For this, have to look for any weird Google-like pages.
	- ![6](/security.github.io/images/auth/6.png)
	- `google-authenticator.burleson-appliance[.]net`
	- `authenticatoor.org`
- #### What are the IP addresses used for C2 servers for this infection?
	- ![7](/security.github.io/images/auth/7.png)
	- The C2 Servers for this infection are:
		- `5.252.153[.]241`
		- `45.125.66[.]32`
		- `45.125.66[.]252`
	- ![8](/security.github.io/images/auth/8.png)
	- The IP for the C2 Server comes out of *Frankfurt* in *Germany*.

