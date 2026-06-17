---
title: CyberDefenders - GoldenSpray
date: 2026-06-17 00:00:00 +0800
categories:
  - CyberDefenders
  - Threat Hunting
  - Elastic
tags:
  - Elastic
difficulty: Medium
pin: false
---

---

**Scenario:** As a cybersecurity analyst at SecureTech Industries, you've been alerted to unusual login attempts and unauthorized access within the company's network. Initial indicators suggest a potential brute-force attack on user accounts. Your mission is to analyze the provided log data to trace the attack's progression, determine the scope of the breach, and the attacker's TTPs.

## Questions

> Q1: What is the attacker's IP address?

![](assets/cyberdef/Pasted%20image%2020260617135526.png)

![](assets/cyberdef/Pasted%20image%2020260617140125.png)

![](assets/cyberdef/Pasted%20image%2020260617141027.png)
The are high volume of failed logon activities `event.code: 4625`from external IP address `77.91.78.115`. By reviewing the log, there are multiple failed logon attempts followed by a successful logon with the username `michaelwilliams` and later on there is an RDP logon to domain `SECURETECH\mwilliams`


>✅ **Answer: `77.91.78.115`**

---

> Q2: What country is the attack originating from?

![](assets/cyberdef/Pasted%20image%2020260617140312.png)

The IP is from Finland

>✅ **Answer: Finland**

---

> Q3: What's the compromised account username used for initial access?

![](assets/cyberdef/Pasted%20image%2020260617141455.png)

The account username used for initial access is `SECURETECH\mwilliams`via RDP.

>✅ **Answer: SECURETECH\mwilliams**

---

> Q4: What's the name of the malicious file utilized by the attacker for persistence on `ST-WIN02`?

![](assets/cyberdef/Pasted%20image%2020260617144205.png)

To identify the malicious file used by the attacker for persistence on the compromised machine, we will analyze Sysmon logs. There are a few columns that I would filter such as computer name, username, and Event ID.  

```
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run /v OfficeUpdater /t REG_SZ /d "C:\Windows\Temp\OfficeUpdater.exe" /f
```

By following the timeline after the brute force, we can identify the attacker creates a Windows startup registry that would automatically run `C:\Windows\Temp\OfficeUpdater.exe`when any user logs on.

>✅ **Answer: OfficeUpdater.exe**

---

> Q5: What is the complete path used by the attacker to store their tools?

![](assets/cyberdef/Pasted%20image%2020260617144733.png)

By filtering event id 11, we can noticed that the attacker dropped few tools on the machine at `C:\Users\Public\Backup_Tools`

>✅ **Answer: `C:\Users\Public\Backup_Tools`**

---

> Q6: What's the process ID of the tool responsible for dumping credentials on `ST-WIN02`?

![](assets/cyberdef/Pasted%20image%2020260617145056.png)

Mimikatz is common tool used by attackers to perform hash dumping and crack it, which justifies how he attacker was able to pivot to systems.

>✅ **Answer: 3708**

---

> Q7: What's the second account username the attacker compromised and used for lateral movement?

![](assets/cyberdef/Pasted%20image%2020260617145817.png)

Later in the log, `jsmith`was logon from the same malicious IP

>✅ **Answer: jsmith**

---

> Q8: Can you provide the scheduled task created by the attacker for persistence on the domain controller?

![](assets/cyberdef/Pasted%20image%2020260617150435.png)

We can search `schtasks.exe` globally to list the logs associated with scheduled tasks. The scheduled task created by `jsmith`on domain controller `ST-DC01.SECURETECH.local` run the file `C:\Windows\Temp\FileCleaner.exe`every hour with SYSTEM privileges.

>✅ **Answer: FilesCheck**

---

> Q9: What type of encryption is used for Kerberos tickets in the environment?

![](assets/cyberdef/Pasted%20image%2020260617151310.png)

![](assets/cyberdef/Pasted%20image%2020260617151357.png)

>✅ **Answer: RC4-HMAC**

---

> Q10: Can you provide the full path of the output file in preparation for data exfiltration?

![](assets/cyberdef/Pasted%20image%2020260617152524.png)

By filtering event id 11, there's a suspicious `zip`file created by powershell and the compromised account. In addition, it was located at public directory, attacker commonly used zip file to reduce the risk of detection.

>✅ **Answer: `C:\Users\Public\Documents\Archive_8673812.zip`**












