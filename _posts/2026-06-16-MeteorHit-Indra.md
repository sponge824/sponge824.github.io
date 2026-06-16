---
title: CyberDefenders - MeteorHit - Indra
date: 2026-06-16 00:00:00 +0800
categories:
  - CyberDefenders
  - Endpoint Forensics
  - Windows
tags:
  - Registry-Explorer
  - Event-Log-Explorer
difficulty: Medium
pin: false
---

---

**Scenario:** A critical network infrastructure has encountered significant operational disruptions, leading to system outages and compromised machines. Public message boards displayed politically charged messages, and several systems were wiped, causing widespread service failures. Initial investigations reveal that attackers compromised the Active Directory (AD) system and deployed wiper malware across multiple machines.

Fortunately, during the attack, an alert employee noticed suspicious activity and immediately powered down several key systems, preventing the malware from completing its wipe across the entire network. However, the damage has already been done, and your team has been tasked with investigating the extent of the compromise.

You have been provided with forensic artifacts collected via KAPE SANS Triage from one of the affected machines to determine how the attackers gained access, the scope of the malware's deployment, and what critical systems or data were impacted before the shutdown.

## Questions

> Q1: The attack began with using a Group Policy Object (GPO) to execute a malicious batch file. What is the name of the malicious GPO responsible for initiating the attack by running a script?

![](assets/cyberdef/Pasted%20image%2020260616174611.png)

![](assets/cyberdef/Pasted%20image%2020260616174938.png)

GPO can be found by loading `SOFTWARE`hive with Registry Explorer and navigate to `Microsoft\Windows\CurrentVersion\Group Policy`. Under script, there is a script named `setup.bat`which is likely associated with the attack. The name of the GPO that is relevant to the malicious script execution is `DeploySetup`


>✅ **Answer: DeploySetup** 

---

> Q2: During the investigation, a specific file containing critical components necessary for the later stages of the attack was found on the system. This file, expanded using a built-in tool, played a crucial role in staging the malware. What is the name of the file, and where was it located on the system? Please provide the full file path.

![](assets/cyberdef/Pasted%20image%2020260616175904.png)

![](assets/cyberdef/Pasted%20image%2020260616180557.png)

At 2024/09/24 16:04, the suspicious script `setup.bat` was executed with CMD. Later on, `expand.exe` a Windows utility used to extract contents was used. It extract the files inside `env.cab`to `C:\ProgramData\Microsoft\env`

>✅ **Answer: `C:\ProgramData\Microsoft\env\env.cab`** 

---

> Q3: The attacker employed password-protected archives to conceal malicious files, making it important to uncover the password used for extraction. Identifying this password is key to accessing the contents and analyzing the attack further. What is the password used to extract the malicious files?

![](assets/cyberdef/Pasted%20image%2020260616181114.png)

The flag `-p`is used specific the password for the archive

>✅ **Answer: hackemall** 

---

> Q4: Several commands were executed to add exclusions to Windows Defender, preventing it from scanning specific files. This behavior is commonly used by attackers to ensure that malicious files are not detected by the system's built-in antivirus. Tracking these exclusion commands is crucial for identifying which files have been protected from antivirus scans. What is the name of the first file added to the Windows Defender exclusion list?

![](assets/cyberdef/Pasted%20image%2020260616182122.png)

Next, we can use powershell log `Windows PowerShell.evtx`to to identify the files added to the exclusion list. We can either filter `ExclusionPath` or `MpPreference`to identify it. 

>✅ **Answer: `update.bat`**

---

> Q5: A scheduled task has been configured to execute a file after a set delay. Understanding this delay is important for investigating the timing of potential malicious activity. How many seconds after the task creation time is it scheduled to run?

![](assets/cyberdef/Pasted%20image%2020260616182632.png)

To identify logs associated with schedule tasks, we can filter `schtasks`in sysmon log. We can see there is a scheduled task created scheduled to run at 09:08:13. Based on the log, I assumed that the log time is the same timezone. `17+180+13=210`

>✅ **Answer: `210`**

---

> Q6: After the malware execution, the wmic utility was used to unjoin the computer system from a domain or workgroup. Tracking this operation is essential for identifying system reconfigurations or unauthorized changes. What is the Process ID (PID) of the utility responsible for performing this action?

![](assets/cyberdef/Pasted%20image%2020260616183432.png)

Filter `wmic`in sysmon log, we can identify the command `unjoindomainworkgroup`

>✅ **Answer: `7492`**

---

> Q7: The malware executed a command to delete the Windows Boot Manager, a critical component responsible for loading the operating system during startup. This action can render the system unbootable, leading to serious operational disruptions and making recovery more difficult. What command did the malware use to delete the Windows Boot Manager?

![](assets/cyberdef/Pasted%20image%2020260616184358.png)

Windows Boot manager is managed by `bcdedit.exe`. We can filter `bcdedit`to identify the command used to delete the Windows Boot Manager

>✅ **Answer: `C:\Windows\Sysnative\bcdedit.exe  /delete {9dea862c-5cdd-4e70-acc1-f32b344d4795} /f`**

---

> Q8: The malware created a scheduled task to ensure persistence and maintain control over the compromised system. This task is configured to run with elevated privileges every time the system starts, ensuring the malware continues to execute. What is the name of the scheduled task created by the malware to maintain persistence?

![](assets/cyberdef/Pasted%20image%2020260616184715.png)

This scheduled task was scheduled to run once the system start with highest privileges as `NT AUTHORITY\SYSTEM`. 

>✅ **Answer: Aa153!EGzN**

---

> Q9: A malicious program was used to lock the screen, preventing users from accessing the system. Investigating this malware is important to identify its behavior and mitigate its impact. What is the name of this malware? (not the filename)

![](assets/cyberdef/Pasted%20image%2020260616185632.png)

![](assets/cyberdef/Pasted%20image%2020260616185452.png)

![](assets/cyberdef/Pasted%20image%2020260616185803.png)

I filter`lock`in sysmon and noticed a familiar file that was added to exclusion list previously. I upload the hash at MalwareBazaar and it appears to be `BreakWin`

>✅ **Answer: BreakWin**

---

> Q10: The disk shows a pattern where malware overwrites data (potentially with zero-bytes) and then deletes it, a behavior commonly linked to Wiper malware activity. The USN (Update Sequence Number) is vital for tracking filesystem changes on an NTFS volume, enabling investigators to trace when files are created, modified, or deleted, even if they are no longer present. This is critical for building a timeline of file activity and detecting potential tampering. What is the USN associated with the deletion of the file msuser.reg?

![](assets/cyberdef/Pasted%20image%2020260616190248.png)


![](assets/cyberdef/Pasted%20image%2020260616190336.png)

![](assets/cyberdef/Pasted%20image%2020260616190918.png)

To find the USN, we can parse `$Extend`located in `C:`. It houses USN journal where it records every file operation of the volume. We can parse `$Extend`with NTFS Log Tacker and view the log in Timeline Explorer. Then, we can filter `msuser`in the File/Directory field.

>✅ **Answer: 11721008**










