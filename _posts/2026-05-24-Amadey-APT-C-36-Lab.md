---
title: CyberDefenders - Amadey - APT-C-36 Lab
date: 2026-05-24 00:00:00 +0800
categories:
  - CyberDefenders
  - Endpoint Forensics
tags:
  - Volatility3
difficulty: Medium
pin: false
---

---

**Scenario:** An after-hours alert from the Endpoint Detection and Response (EDR) system flags suspicious activity on a Windows workstation. The flagged malware aligns with the Amadey Trojan Stealer. Your job is to analyze the presented memory dump and create a detailed report for actions taken by the malware.

## Questions

>Q1: In the memory dump analysis, determining the root of the malicious activity is essential for comprehending the extent of the intrusion. What is the name of the parent process that triggered this malicious behavior?

In this lab, I used Volatility3 to analyze the memory dump. 

`vol.py -f Desktop/Start\ here/Artifacts/Windows\ 7\ x64-Snapshot4.vmem windows.pstree`

Basically, this command is loading the memory dump file and with the plugin `windows.pstree`. This plugin will list the running process in a tree hierarchy.

![](assets/cyberdef/Pasted%20image%2020260524155459.png)

There is a process that drew my attention: `lssass.exe`, which suggests the masquerading of the legitimate process `lsass.exe`.

>✅ **Answer: `lssass.exe`**

---

> Q2: Once the rogue process is identified, its exact location on the device can reveal more about its nature and source. Where is this process housed on the workstation?

Since we already know the PID of rogue process, we can use plugin `windows.cmdline` and filter the pid

`vol.py -f Desktop/Start\ here/Artifacts/Windows\ 7\ x64-Snapshot4.vmem windows.cmdline --pid 2748`

![](assets/cyberdef/Pasted%20image%2020260524160141.png)

>✅ **Answer: `C:\Users\0XSH3R~1\AppData\Local\Temp\925e7e99c5\lssass.exe`**

---

> Q3: Persistent external communications suggest the malware's attempts to reach out C2C server. Can you identify the Command and Control (C2C) server IP that the process interacts with?

Next, I used `windows.netscan` to recover the network connections from the memory dump

`vol.py -f Desktop/Start\ here/Artifacts/Windows\ 7\ x64-Snapshot4.vmem windows.netscan`

![](assets/cyberdef/Pasted%20image%2020260524161002.png)

The results show two closed connections, but the first one does not have a LocalAddr, Port, or Remote Port. It does not strongly indicate C2 communication. However, the second highlighted red box contains complete connection details. In addition, it connects to port 80, likely attempting to disguise itself as legitimate web traffic.

>✅ **Answer: 41.75.84.12**

---

> Q4: Following the malware link with the C2C, the malware is likely fetching additional tools or modules. How many distinct files is it trying to bring onto the compromised workstation?

I don't know how to find the files that the malware dropped onto the compromised workstation. I read through the walkthrough. First, we extract the memory region associated with the PID 2748 and output it into a file with the command below:

`vol.py -f ~/Desktop/Start\ here/Artifacts/Windows\ 7\ x64-Snapshot4.vmem windows.memmap --pid 2748 --dump`

Next, we extract the readable text from the dump file and filter `GET` request with grep

`strings pid.2748.dmp | grep "GET /"`

![](assets/cyberdef/Pasted%20image%2020260524163244.png)

>✅ **Answer: 2**

---

> Q5: Identifying the storage points of these additional components is critical for containment and cleanup. What is the full path of the file downloaded and used by the malware in its malicious activity?

To find the path of the file downloaded, we can use the `windows.cmdline` plugin, and try filter `.dll` with grep

`vol.py -f Desktop/Start\ here/Artifacts/Windows\ 7\ x64-Snapshot4.vmem windows.cmdline | grep .dll`

![](assets/cyberdef/Pasted%20image%2020260524163532.png)

>✅ **Answer: `C:\Users\0xSh3rl0ck\AppData\Roaming\116711e5a2ab05\clip64.dll`**

---

> Q6: Once retrieved, the malware aims to activate its additional components. Which child process is initiated by the malware to execute these files?

![](assets/cyberdef/Pasted%20image%2020260524163652.png)

By looking back the process hierarchy tree, there is a `rundll32.exe` spawned by the malicious process `lssass.exe`

>✅ **Answer: `rundll32.exe`**

---

> Q7: Understanding the full range of Amadey's persistence mechanisms can help in an effective mitigation. Apart from the locations already spotlighted, where else might the malware be ensuring its consistent presence?

After reading Amadey's persistence mechanisms, it will create multiple persistence mechanisms which are Registry Run Key and Scheduled Task. We can use `windows.filescan` plugin to scan and filter the associated malicious file name.

`vol.py -f Desktop/Start\ here/Artifacts/Windows\ 7\ x64-Snapshot4.vmem windows.filescan | grep lssass.exe`

![](assets/cyberdef/Pasted%20image%2020260524165226.png)

It created a scheduled task in `C:\Windows\System32\Tasks\lssass.exe`

>✅ **Answer: `C:\Windows\System32\Tasks\lssass.exe`**



