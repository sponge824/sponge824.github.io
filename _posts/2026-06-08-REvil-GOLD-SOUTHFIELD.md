---
title: CyberDefenders - REvil - GOLD SOUTHFIELD
date: 2026-06-08 00:00:00 +0800
categories:
  - CyberDefenders
  - Threat Hunting
tags:
  - Elastic
difficulty: Easy
pin: false
---

---

**Scenario:** You are a Threat Hunter working for a cybersecurity consulting firm. One of your clients has been recently affected by a ransomware attack that caused the encryption of multiple of their employees' machines. The affected users have reported encountering a ransom note on their desktop and a changed desktop background. You are tasked with using Splunk SIEM containing Sysmon event logs of one of the encrypted machines to extract as much information as possible.

## Questions

> Q1: To begin your investigation, can you identify the filename of the note that the ransomware left behind?

![](assets/cyberdef/Pasted%20image%2020260608164516.png)

Filter `eventcode: 11 and winlog.event_data.TargetFilename: *txt*`. The reason for filtering `.txt`file is that most of the ransom notes are written in txt format. The returned results show that there are many `5uizv5660t-readme.txt` scattered throughout the system.

>✅ **Answer: 5uizv5660t-readme.txt**

---

> Q2: After identifying the ransom note, the next step is to pinpoint the source. What's the process ID of the ransomware that's likely involved

![](assets/cyberdef/Pasted%20image%2020260608165348.png)

The process ID that created this file is `5348`and the process is `facebook assistant.exe`that is located in the Administrator's Downloads folder.

>✅ **Answer: 5348**

---

> Q3: Having determined the ransomware's process ID, the next logical step is to locate its origin. Where can we find the ransomware's executable file?

>✅ **Answer: C:\Users\Administrator\Downloads\facebook assistant.exe**

---

> Q4: Now that you've pinpointed the ransomware's executable location, let's dig deeper. It's a common tactic for ransomware to disrupt system recovery methods. Can you identify the command that was used for this purpose?

![](assets/cyberdef/Pasted%20image%2020260608171434.png)

![](assets/cyberdef/Pasted%20image%2020260608171524.png)

Since we already found the process id of the ransomware, we can filter it as the parent process id `winlog.event_data.ParentProcessId: 5348` to look for any other processes created by it. There is one result returned, and it executed an encoded powershell command. After decoding it with CyberChef, this command is deleting all the shadow copies on the system.

>✅ **Answer: `Get-WmiObject Win32_Shadowcopy | ForEach-Object {$_.Delete();}`**

---

> Q5: As we trace the ransomware's steps, a deeper verification is needed. Can you provide the sha256 hash of the ransomware's executable to cross-check with known malicious signatures?

![](assets/cyberdef/Pasted%20image%2020260608172201.png)

![](assets/cyberdef/Pasted%20image%2020260608172218.png)

>✅ **Answer: B8D7FB4488C0556385498271AB9FFFDF0EB38BB2A330265D9852E3A6288092AA**

---

> Q6: One crucial piece remains: identifying the attacker's communication channel. Can you leverage threat intelligence and known Indicators of Compromise (IoCs) to pinpoint the ransomware author's onion domain?

![](assets/cyberdef/Pasted%20image%2020260608172852.png)

By searching the hashes on Google, `tria.ge` report shown that the malware is communicating with an onion domain `http://aplebzu47wgazapdqks6vrcv6zcnjppkbxbr6wketf56nf6aq2nmyoyd.onion/C08285A007995BAB` which is commonly used as C2 server.

>✅ **Answer: aplebzu47wgazapdqks6vrcv6zcnjppkbxbr6wketf56nf6aq2nmyoyd.onion**


