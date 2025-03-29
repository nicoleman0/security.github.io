---
layout: post
title: "XSS Attack"
author: "Nicholas Coleman"
date: 2025-03-28
tags: letsdefend
---

#### Log Alert

![XSS alert](/security.github.io/images/xss_attack/XSS_alert.png)
#### Investigation

Source IP: 112.85.42.13
- Country: CN
- City: Nanjing
- ISP: China Unicom Jiangsu province network
- E-Mail: `zhaoyz3@chinaunicom.cn`


![china](/security.github.io/images/xss_attack/china_pic.png)

A poorly rated IP address. This adds to the suspicion that this is an attack.

Decoded Requested URL: 
`https://172.16.17.17/search/?q=<$script>javascript:$alert(1)<$/script>`

Using my Log Management tool, I queried for any logs relating to the Source IP.

![logs](/security.github.io/images/xss_attack/logs.png)

**Timeline**:
```
06:34 PM: https://172.16.17.17/
06:35 PM: https://172.16.17.17/about-us/
06:45 PM: https://172.16.17.17/search/?q=test
06:46 PM: https://172.16.17.17/search/?q=<$img src =q onerror=prompt(8)$>
06:46 PM: https://172.16.17.17/search/?q=prompt(8)
06:50 PM: https://172.16.17.17/search/?q=<$script>$for((i)in(self))eval(i)(1)<$/script>
06:53 PM: https://172.16.17.17/search/?q=<$svg><$script ?>$alert(1)
06:56 PM: https://172.16.17.17/search/?q=<$script>javascript:$alert(1)
```

1. **06:34 PM - 06:35 PM**:
	1. The attacker starts by accessing the main page and then the "About Us" page.
	2. Alone this obviously does not cause any alarms, but this also could be part of the Reconnaissance stage.
2. **06:45 PM**:
	1. The attacker tests the search functionality using a benign query (`?q=test`).
	2. This is often done to inspect how search parameters are handled.
	3. The attackers are looking for whether or not the input is reflected in the response.
3. **06:46 PM**:
	1. The first XSS attempt: `?q=<$img src=q onerror=prompt(8)$>`
		1. This uses an `<img>` tag with a `onerror` attribute to execute JavaScript (`prompt(8)`).
		2. This is a common XSS test to see if the input is reflected without proper sanitization.
	2. **Second XSS attempt**: `?q=prompt(8)`
		1. If the previous payload was successful, the attacker may be checking to see if the input directly triggered a `prompt()` function.
		2. This is most likely a confirmation attempt.
4. **06:50 PM**:
	1. **Third XSS attempt**: `?q=<$script>$for((i)in(self))eval(i)(1)<$/script>`
		1. The third attempt is more aggressive, using the `eval()` function, which can execute arbitrary JavaScript. 
		2. This indicates the attacker is most likely probing for severe XSS vulnerabilities.
5. **06:53 PM**:
	1. **Fourth XSS attempt**: `?q=<$svg><$script ?>$alert(1)`
		1. Here, the attacker is attempting to exploit XSS using an SVG tag. 
		2. Some insecure sites may assume SVG tags are safe, but they can contain embedded scripts (like this one).
6. **06:56 PM**:
	1. **Fifth XSS attempt**: `?q=<$script>javascript:$alert(1)`
		1. This is a direct JavaScript execution using an `alert()` function. 
		2. It’s a simple yet effective payload to test if script tags are blocked or sanitized.

**Endpoint Information**
![end](/security.github.io/images/xss_attack/endpoint.png)

Looks like the WebServer that was targeted's last login was 24 days before the XSS attack.

Double checking either way, there is nothing that raises the alarm. This means that the XSS attack was unsuccessful!

#### Final Report

This XSS attack coming out of China ultimately was unsuccessful in its attempt to breach our WebServer1002. They tried multiple times with various techniques. 

The attack was not successful, and WebServer1002 was not breached. The attack was staged by a threat actor operating out of Nanjing, China. 

The ISP of the IP was 'China Unicom Jiangsu province network'.

This alert was a **True Positive**. 

#### Extracted Artifacts

| Value                                                        | Comment     | Type          |
| ------------------------------------------------------------ | ----------- | ------------- |
| 112.85.42.13                                                 | Attacker IP | IP Address    |
| https://172.16.17.17/search/?q=<$script>javascript:$alert(1) | Decoded URL | E-mail Domain |
