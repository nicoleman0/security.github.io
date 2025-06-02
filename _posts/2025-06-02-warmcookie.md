---
layout: post
title: "WarmCookie"
author: "Nicholas Coleman"
date: 2025-06-02
tags: malwareanalysis
---

## Victim Details

| Username | `ghost` |
| :---- | :---- |
| **Hostname** | `DESKTOP-H8ALZBV` |
| **IP Addr.** | `10.8.15[.]133` |
| **FQDN** | `desktop-h8alzbv.lafontainebleu.org` |
| **MAC Addr.** | `00:1c:bf:03:54:82` |

## Indicators of Compromise

| C2 | `72.5.43[.]29` |
| :---- | :---- |
| **Domain** | `quote.checkfedexexp[.]com` |
| **Dropper** | `dab9...6dc2c` |
| **Malware Binary** | `ad8a...6f053` |
| **Incident Time** | `134.757901` |

## Summary

### JS-based backdoor

**Invoice-876597035-003-8331775-8334138.js**  
`dab98819d1d7677a60f5d06be210d45b74ae5fd8cf0c24ec1b3766e25ce6dc2c`

* This file is known to be a malicious backdoor dropper that masquerades as an invoice.
* 22 different security vendors flagged this file as malicious. 

`var x = "https://stap[.]top/admin/cp.php";`

* This script connects to a suspicious domain with the TLD of .top

`var shell = new ActiveXObject("Wscript.Shell");`

* This Windows command is sometimes abused by malware to execute system-level commands without being noticed.   
* Additionally, a simple invoice should never be using **ActiveXObject** or **Wscript.Shell**.

`var stream = new ActiveXObject("ADODB.Stream");`
`...`  
`stream.SaveToFile(filename, 2);`  
`shell.Run(filename, 0);`

* Here the script opens an HTTP Request to download a file. It then writes the file to a disk and runs it. This is all done **without any user interaction** or input.  
  * This backdoor also makes use of various obfuscation techniques.   
* The variable names are **generic and potentially misleading**.  
* The HTTP request content and flow are segmented to avoid static detection.  
* There are **unnecessary string operations** likely meant to bypass signature detection.

In general, if a downloaded file makes use of `WScript` objects and filesystem operations like `ADODB.Stream`, it is more than likely malicious. Especially if the file claims to be an invoice, such as this one, which should not be doing anything. 

### Command & Control (C2)

Host IP has been shown to be interacting with this known malicious Romanian IP address (72.5.43\[.\]29). It is being used as a C2 Server.  

![pic1](/security.github.io/images/warmcookie/pic1.png)

*Here is an example of a suspicious GET Request made by the host to the suspected C2 Server.*  
`http://72.5.43[.]29/data/0f60a3e7baecf2748b1c8183ed37d1e4`

This URL has been shared among the security community as a known infection source.

The malicious communications began at: `134.757901`

A sample of the HTTP Stream involving the C2:

![p2](/security.github.io/images/warmcookie/pic2.png)

*Within this HTTP Stream there are PE file headers, which indicate the transfer of Windows executables over HTTP.* 

`MZ......................@............................................. .!..L.!This program cannot be run in DOS mode.` 

* This is indicative of a Windows binary.

`release.dll.DllGetClassObject.DllRegisterServer.DllRegisterServerEx.DllUnregisterServer.Start`

`GetProcAddress, LoadLibraryA, CreateRemoteThread`

* These are highly suspicious commands, and show that this binary has the capability to register itself, resolve functions, and inject itself into other system processes.

API calls like `IsDebuggerPresent, CheckRemoteDebuggerPresent, GetTickCount, Sleep` show that this binary uses evasion techniques, which is common in malware.

Commands like `GetCommandLineA, SetEnvironmentVariableA, GetEnvironmentVariableA` show that this binary is most likely receiving commands from an outside source (most likely the C2 Server that was identified.)

`2 server pkts, 2 client pkts, 3 turns`

* This indicates some kind of beaconing or the malware downloading its payload.

In general, HTTP should not be transmitting binary payloads that are disguised with ASCII. That is a clear sign that something is amiss. The attacker does this to make the binary data seem innocuous. It is more of the same obfuscation tactics.

![pic3](/security.github.io/images/warmcookie/pic3.png)

*The fact that the infected host makes such regular calls to the IP furthers my suspicion that this is a C2 Server that is being communicated with. This pattern of communication is known as beaconing.* 

### Malicious Domain

![4](/security.github.io/images/warmcookie/pic4.png)

*The domain that the infected host is interacting with, `quote.checkfedexexp[.]com` is a known suspected malicious website and infection source with a VT score of \-14.* 

Evidence of the host interacting with this site furthers the suspicion that something is wrong here. It is masquerading as a FedEx website, so it was probably associated with the malicious "invoice" from before…

### Alert Logs

![5](/security.github.io/images/warmcookie/pic5.png)

**ET POLICY Binary Download Smaller than 1 MB Likely Hostile**

* **Source IP**: 72.5.43\[.\]29 → **Destination**: 10.8.15\[.\]133  
* This suggests a small executable (typical size for a loader) was downloaded from the suspected C2 (72.5.43\[.\]29).

**ET TROJAN Possible Windows Executable sent when remote host claims to send HTML**

* This is an indicator of content-type spoofing, where the C2 server may claim it's sending a web page but actually delivers a binary. This is a classic behavior of malware droppers/backdoors.

**ET POLICY PE EXE or DLL Windows file download HTTP**

* This unfortunately confirms that a Windows executable was downloaded over HTTP. This is *highly suspicious* when unsolicited, as HTTP generally does not transmit code but rather data.

**ETPRO SUSPICIOUS Dotted Quad Host MZ Response**

* "MZ" is the header of a Windows PE (executable) file. This means the server responded with *actual binary content*. Even though it was hidden in ASCII…

**ET INFO Terse Request for .txt \- Likely Hostile**

* Possibly a loader checking in with the C2 by requesting a `.txt` file. This is a common tactic to mask payloads.

## Conclusory Findings

The infected host (`10.8.15[.]133`) was:

* Infected via a JavaScript backdoor (**Invoice-876597035-003-8331775-8334138.js**).  
* Connected to the C2 server (`72.5.43[.]29`).  
* Downloaded and executed a Windows binary payload. (`ad8addf68fdc964220a6e30831720ef8813a27c392030a1d3a0ec63a5536f053`)  
* Attempted POST communications using fake browser headers, likely C2 check-ins or data exfiltration.

### POST Communication!

![6](/security.github.io/images/warmcookie/pic6.png)

| IOC Type | Value |
| :---- | :---- |
| IP Addr. | `72.5.43[.]29` |
| UA String | IE 6.0 on WinXP (`MSIE 6.0…`) |
| URL Path | `/` (for both GET and POST) |
| Encoding | Suspicious `Cookie` field |

* Base64/encrypted cookie in GET and POST  
* Fake User-Agent used  
  * It is spoofing an old IE browser  
  * The very same UA string has been seen in many **AsyncRAT, njRAT, or Nemucod infections**  
* Binary data in the POST body  
  * Non-printable data, likely a payload

### C2 Communication Flow

1\. `GET /` with beacon or check-in cookie  
2\. Server replies: `200 OK` \+ small binary payload  
3\. Client POSTs to `/` — likely reporting host info or responding to command  
4\. Server returns `400 Bad Request`, possibly a protocol desync or stub C2 response

**Evidence of:**

* Check-in via GET with encoded cookie  
* Command/payload sent back  
* POST back to the server — likely with host or task data

"WarmCookie, observed being used for initial access and persistence, offers a means for continuous long-term access to compromised environments and is used to facilitate delivery of additional malware such as [CSharp-Streamer-RAT](https://malpedia.caad.fkie.fraunhofer.de/details/win.csharpstreamer) and [Cobalt Strike](https://malpedia.caad.fkie.fraunhofer.de/details/win.cobalt_strike)”

This excerpt from Talos' report on WarmCookie corroborates with the actions that I observed. 

This leads me to believe that the alerts claiming this to be a WarmCookie attack are in fact **true positives**. Furthermore, everything I have seen leads me to believe that the backdoor is **operational**.

## Review & Further Information

**WARMCOOKIE Execution Flow**

![][image7]

Here we can see that the WARMCOOKIE execution flow matches the activity that occurred against the infected Host IP. 

Beginning with a phishing email, it then redirects to a landing page: (`quote.checkfedexexp[.]com`). 

The landing page contains the JavaScript dropper (**Invoice-876597035-003-8331775-8334138.js**), which then executes PowerShell commands, contained within a binary payload: (`ad8ad…f053`) to install the WARMCOOKIE backdoor. 

The backdoor then begins to communicate with the C2 Server (`72.5.43[.]29`).

With this, I can confidently say that this incident is indeed a WARMCOOKIE backdoor attack. Further analysis is warranted in order to dispel any chance that the attacker was able to exfiltrate data from the compromised host. 
