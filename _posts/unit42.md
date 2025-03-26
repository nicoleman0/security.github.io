---
layout: post
title: "Unit 42"
author: "Nicholas Coleman"
date: 2025-03-26
tags: hackthebox
---

*In this Sherlock, you will familiarize yourself with Sysmon logs and various useful EventIDs for identifying and analyzing malicious activities on a Windows system. Palo Alto's Unit42 recently conducted research on an UltraVNC campaign, wherein attackers utilized a backdoored version of UltraVNC to maintain access to systems. This lab is inspired by that campaign and guides participants through the initial access stage of the campaign.*

**How many Event logs are there with Event ID 11?**

The first thing I did was create a Python script to parse the Sysmon logs into a readable XML format. 
