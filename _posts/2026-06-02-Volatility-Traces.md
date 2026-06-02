---
title: CyberDefenders - Volatility Traces
date: 2026-06-02 00:00:00 +0800
categories:
  - CyberDefenders
  - Endpoint Forensics
  - Memory Forensics
tags:
  - Volatility3
difficulty: Easy
pin: false
---

---

**Scenario:** On May 2, 2024, a multinational corporation identified suspicious PowerShell processes on critical systems, indicating a potential malware infiltration. This activity poses a threat to sensitive data and operational integrity.

## Questions

> Q1: Identifying the parent process reveals the source and potential additional malicious activity. What is the name of the suspicious process that spawned two malicious PowerShell processes?

![](assets/cyberdef/Pasted%20image%2020260602132013.png)

In this lab, I used volatility3 to analyze the memory dump. By using the `windows.pstree`plugin. I noticed that there were 2 PowerShell processes in the memory dump. However, I was unable to locate the parent process.

Next, we can use a plugin called `windows.psscan` it scans memory directly for EPROCESS structures, which can identify active, hidden, and recently terminated process that may not appear in `pslist`or `pstree`.

![](assets/cyberdef/Pasted%20image%2020260602133407.png)

By matching the parent process id that spawned the Powershell process `4596`, `InvoiceCheckLi` is the parent process that spawned the powershell. However, the full process name is not visible. 

![](assets/cyberdef/Pasted%20image%2020260602133606.png)

We can use plugin `windows.cmdline`to view the full path of the malicious process.

>✅ **Answer: InvoiceCheckList.exe**

---

> Q2: By determining which executable is utilized by the malware to ensure its persistence, we can strategize for the eradication phase. Which executable is responsible for the malware's persistence?

![](assets/cyberdef/Pasted%20image%2020260602133819.png)

Looking back at the `psscan` otuput and filtering for the parent process ID `4596`shows the processes spawned by the malicious process.

The process `schtasks.exe`is commonly used in persistence techniques. It is used to create and manage scheduled tasks in Windows.

>✅ **Answer: schtasks.exe**

---

> Q3: Understanding child processes reveals potential malicious behavior in incidents. Aside from the PowerShell processes, what other active suspicious process, originating from the same parent process, is identified?

Referring back to the screenshot above, the remaining process spawned by the suspicious process is `RegSvcs.exe`

>✅ **Answer: RegSvcs.exe**

---

> Q4: Analyzing malicious process parameters uncovers intentions like defense evasion for hidden, stealthy malware. What PowerShell cmdlet used by the malware for defense evasion?

![](assets/cyberdef/Pasted%20image%2020260602134741.png)

The malicious process used `Add-MpPreference`to modifies settings for Microsoft Defender. It added the `InvoiceCheckList.exe` to exclusion list which means Microsoft Defender will not block/quarantine this process.

>✅ **Answer: Add-MpPreference**

---

> Q5: Recognizing detection-evasive executables is crucial for monitoring their harmful and malicious system activities. Which two applications were excluded by the malware from the previously altered application's settings?

![](assets/cyberdef/Pasted%20image%2020260602140429.png)

The malicious processes that were excluded by Microsoft Defender were `InvoiceCheckList.exe`and `HcdmIYYf.exe`.

>✅ **Answer: InvoiceCheckList.exe,HcdmIYYf.exe**

---

> Q6: What is the specific MITRE sub-technique ID associated with PowerShell commands that aim to disable or modify antivirus settings to evade detection during incident analysis?

![](assets/cyberdef/Pasted%20image%2020260602141219.png)

>✅ **Answer: T1562.001**

---

> Q7: Determining the user account offers valuable information about its privileges, whether it is domain-based or local, and its potential involvement in malicious activities. Which user account is linked to the malicious processes?

![](assets/cyberdef/Pasted%20image%2020260602141327.png)

>✅ **Answer: Lee**

