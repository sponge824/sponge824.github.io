---
title: CyberDefenders - YellowRat
date: 2026-05-24 00:00:00 +0800
categories:
  - CyberDefenders
  - Threat Intel
tags:
  - VirusTotal
difficulty: Easy
pin: false
---

---

**Scenario:** During a regular IT security check at GlobalTech Industries, abnormal network traffic was detected from multiple workstations. Upon initial investigation, it was discovered that certain employees' search queries were being redirected to unfamiliar websites. This discovery raised concerns and prompted a more thorough investigation. Your task is to investigate this incident and gather as much information as possible.

## Questions

>Q1: Understanding the adversary helps defend against attacks. What is the name of the malware family that causes abnormal network traffic?

There is a hash provided in this lab. I use google to search the hash, and it pointed to a malware called Yellow Cockatoo. It is a .NET remote access trojan (RAT)

![](assets/cyberdef/Pasted%20image%2020260524151134.png)

>✅ **Answer: Yellow Cockatoo Rat**

---

> Q2: As part of our incident response, knowing common filenames the malware uses can help scan other workstations for potential infection. What is the common filename associated with the malware discovered on our workstations?

I browsed through the Red Canary Yellow Cockatoo blog, and found an analysis of the RAT malware. Apparently it only has one component

![](assets/cyberdef/Pasted%20image%2020260524151756.png)

>✅ **Answer: `111bc461-1ca8-43c6-97ed-911e0e69fdf8.dll`**

---

> Q3: Determining the compilation timestamp of malware can reveal insights into its development and deployment timeline. What is the compilation timestamp of the malware that infected our network?

We can find the compilation timestamp in VirusTotal under Details -> History -> Creation Time

![](assets/cyberdef/Pasted%20image%2020260524152229.png)

>✅ **Answer: 2020-09-24 18:26**

---

> Q4: Understanding when the broader cybersecurity community first identified the malware could help determine how long the malware might have been in the environment before detection. When was the malware first submitted to VirusTotal?

First submission can also be found under Details -> History -> First Submission

>✅ **Answer: 2020-10-15 02:47**

---

> Q5: To completely eradicate the threat from Industries' systems, we need to identify all components dropped by the malware. What is the name of the **.dat** file that the malware dropped in the **AppData** folder?

Looking back at the Red Canary blog, it dropped a `.dat` file to the AppData folder which use as a unique identifier for the host.

![](assets/cyberdef/Pasted%20image%2020260524152505.png)

>✅ **Answer: solarmarket.dat**

---

> Q5: It is crucial to identify the C2 servers with which the malware communicates to block its communication and prevent further data exfiltration. What is the C2 server that the malware is communicating with?

>✅ **Answer: `https://gogohid.com`**