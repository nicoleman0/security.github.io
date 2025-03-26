---
layout: post
title: "Phishing Investigation"
author: "Nicholas Coleman"
date: 2025-03-26
tags: letsdefend
---

![pv1](/security.github.io/images/phishing_investigation/pv1.png)

I had received an alert on my SIEM tool, which prompted me to investigate. 

- When was it sent?
	- Jun, 13, 2021, 02:13 PM
- What is the email's SMTP address?
	- 24.213.228.54
- What is the sender address?
	- trenton@tritowncomputers.com
- What is the recipient address?
	- lars@letsdefend.io
- Is the mail content suspicious?
	- The filename is suspicious, but the premise of the email is not.
- Are there any attachment?
	- Yes there is an attachment. What looks like an Excel spreadsheet.

Scans:

![pv2](/security.github.io/images/phishing_investigation/pv2.png)

![pv3](/security.github.io/images/phishing_investigation/pv3.png)

MD5 Hash: `0cbede8a169ecbbabd533aa9202d9015`
SHA-256: `38b01a12b8dcd39ebdcf9e97772e848237330eb227e1ccee80125564b27377e5`

![pv4](/security.github.io/images/phishing_investigation/pv4.png)

One of the bundled files was flagged as suspicious by VirusTotal. Looking more into it, it seems like BitDefender and a couple of other big-name AVs are flagging this. The bundled file name is `11f44531fb088d31307d87b01e8eabff.zip`. Its MD5 is `9458859abfd384f38362af01fb306f14`.

This could be something. Looking more into the bundled file, it seems like it is in contact with two IP addresses: `188.209.214.83` and `204.79.197.203`. These are the C2 servers.

![pv5](/security.github.io/images/phishing_investigation/pv5.png)

Delving deeper, I can see all sorts of nasty behavior coming from this bundle file. I have all but confirmed the maliciousness of this attachment. 

Looking at the Device Action tab in my SIEM, I can see that the email was allowed by the receiving user's mail service. I moved to proactively delete the email before the user could potentially interact with the malicious file.

It looks like, from looking at logs, that the user did not click/download the attachment. I could not find any evidence of communication to these C2 servers by the user's system. 

The bundled file contains a Trojan virus that interacts with the IP address: `188.209.214.83` which is tied to the domain: nws.visionconsulting.ro

The bundled Trojan drops multiple suspicious files, as well as starting processes on the victim machine. The use of temporary directories to store unusual DLL files is a clear indication of malicious activity. Furthermore, the use of regsvr32.exe with those DLL files suggests a potential attempt to register a malicious or unknown DLL.

