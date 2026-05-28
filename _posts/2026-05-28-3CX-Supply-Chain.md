---
title: CyberDefenders - 3CX Supply Chain
date: 2026-05-28 00:00:00 +0800
categories:
  - CyberDefenders
  - Threat Intel
tags:
  - VirusTotal
difficulty: Easy
pin: false
---

---

**Scenario:** A large multinational corporation heavily relies on the 3CX software for phone communication, making it a critical component of their business operations. After a recent update to the 3CX Desktop App, antivirus alerts flag sporadic instances of the software being wiped from some workstations while others remain unaffected. Dismissing this as a false positive, the IT team overlooks the alerts, only to notice degraded performance and strange network traffic to unknown servers. Employees report issues with the 3CX app, and the IT security team identifies unusual communication patterns linked to recent software updates.

## Questions

>Q1: Understanding the scope of the attack and identifying which versions exhibit malicious behavior is crucial for making informed decisions if these compromised versions are present in the organization. How many versions of 3CX **running on Windows** have been flagged as malware?

![](assets/cyberdef/Pasted%20image%2020260528141658.png)

There are 2 versions of 3CX affected that are running on Windows.


>✅ **Answer: 2**

---

> Q2: Determining the age of the malware can help assess the extent of the compromise and track the evolution of malware families and variants. What's the UTC creation time of the `.msi` malware?

![687](assets/cyberdef/Pasted%20image%2020260528142552.png)

![](assets/cyberdef/Pasted%20image%2020260528142705.png)

I obtained the hash from the `.msi`file provided, and searched it on VirusTotal. Under Details -> History, we can see the creation time.

>✅ **Answer: 2023-03-13 06:33**

---

> Q3: Executable files (`.exe`) are frequently used as primary or secondary malware payloads, while dynamic link libraries (`.dll`) often load malicious code or enhance malware functionality. Analyzing files deposited by the Microsoft Software Installer (`.msi`) is crucial for identifying malicious files and investigating their full potential. Which malicious DLLs were dropped by the `.msi` file?

![](assets/cyberdef/Pasted%20image%2020260528143457.png)

There are 2 `.dll` files dropped by the malicious file.

>✅ **Answer: ffmpeg.dll,d3dcompiler_47.dll**

---

> Q4: Recognizing the persistence techniques used in this incident is essential for current mitigation strategies and future defense improvements. What is the MITRE Technique ID employed by the `.msi` files to load the malicious DLL?

![](assets/cyberdef/Pasted%20image%2020260528144315.png)

The techniques used by 3CX Supply Chain Attack was recorded in [MITRE ATT&CK](https://attack.mitre.org/campaigns/C0057/). 

>✅ **Answer: T1574**

---

> Q5: Recognizing the malware type (`threat category`) is essential to your investigation, as it can offer valuable insight into the possible malicious actions you'll be examining. What is the threat category of the two malicious DLLs?

![](assets/cyberdef/Pasted%20image%2020260528144854.png)

![](assets/cyberdef/Pasted%20image%2020260528144942.png)

Both `.dll` categorized as trojan in virustotal

>✅ **Answer: trojan**

---

> Q6: As a threat intelligence analyst conducting dynamic analysis, it's vital to understand how malware can evade detection in virtualized environments or analysis systems. This knowledge will help you effectively mitigate or address these evasive tactics. What is the MITRE ID for the virtualization/sandbox evasion techniques used by the two malicious DLLs?

![](assets/cyberdef/Pasted%20image%2020260528145531.png)

>✅ **Answer: T1497**

---

> Q7: When conducting malware analysis and reverse engineering, understanding anti-analysis techniques is vital to avoid wasting time. Which hypervisor is targeted by the anti-analysis techniques in the `ffmpeg.dll` file?

![](assets/cyberdef/Pasted%20image%2020260528150646.png)

>✅ **Answer: VMWare**

---

> Q8: Identifying the cryptographic method used in malware is crucial for understanding the techniques employed to bypass defense mechanisms and execute its functions fully. What encryption algorithm is used by the `ffmpeg.dll` file?

![](assets/cyberdef/Pasted%20image%2020260528151153.png)

According to [Unit42](https://unit42.paloaltonetworks.com/3cxdesktopapp-supply-chain-attack/), the `.dll` was encrypted using RC4

>✅ **Answer: RC4**

---

> Q9: As an analyst, you've recognized some TTPs involved in the incident, but identifying the APT group responsible will help you search for their usual TTPs and uncover other potential malicious activities. Which group is responsible for this attack?

![](assets/cyberdef/Pasted%20image%2020260528151749.png)

>✅ **Answer: Lazarus**

