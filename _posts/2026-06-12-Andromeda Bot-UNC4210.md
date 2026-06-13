---
title: CyberDefenders - Andromeda Bot - UNC4210
date: 2026-06-12 00:00:00 +0800
categories:
  - CyberDefenders
  - Endpoint Forensics
  - Windows
tags:
  - MemProcFS
  - Timeline-Explorer
  - EvtxECmd
difficulty: Medium
pin: false
---

---

**Scenario:** As a member of the DFIR team at SecuTech, you're tasked with investigating a security breach affecting multiple endpoints across the organization. Alerts from different systems suggest the breach may have spread via removable devices. You’ve been provided with a memory image from one of the compromised machines. Your objective is to analyze the memory for signs of malware propagation, trace the infection’s source, and identify suspicious activity to assess the full extent of the breach and inform the response strategy.

## Questions

> Q1: Tracking the serial number of the USB device is essential for identifying potentially unauthorized devices used in the incident, helping to trace their origin and narrow down your investigation. What is the serial number of the inserted USB device?

In this lab, we are provided with a `memory.dmp` file. Therefore, we need to mount it using `MemProcFS` so that it can be accessed and analyzed as a browsable file system.

```
memprocfs.exe -device "C:\Users\Administrator\Desktop\Start Here\Artifacts\memory.dmp" -forensic 3
```

![](assets/cyberdef/Pasted%20image%2020260613174500.png)

![](assets/cyberdef/Pasted%20image%2020260613174948.png)

The `SYSTEM`hive is missing from the memory dump, which means we cannot analyze `USBSTOR`keys. In addition, USB registries can be found in `M:\py\reg\usb` which contain information about connected devices. We will specifically look into `usb_storage.txt` as it contained various important information.


>✅ **Answer: 7095411056659025437&0**

---

> Q2: Tracking USB device activity is essential for building an incident timeline, providing a starting point for your analysis. When was the last recorded time the USB was inserted into the system?

![](assets/cyberdef/Pasted%20image%2020260613175509.png)

To obtain the timeline of USB, we can analyze `usb_devices.txt` by filtering the `Serial Number`.
This file recorded information like Device Name, First Insert, Last Insert & Last Removal.

>✅ **Answer: 2024-10-04 13:48**

---

> Q3: Identifying the full path of the executable provides crucial evidence for tracing the attack's origin and understanding how the malware was deployed. What is the full path of the executable that was run after the PowerShell commands disabled Windows Defender protections?

![](assets/cyberdef/Pasted%20image%2020260613212626.png)

![](assets/cyberdef/Pasted%20image%2020260613212937.png)

In order to identify the executed powershell command, we can analyze the windows event logs. The logs can be retrieve at `M:\forensic\files\ROOT\Windows\System32\winevt\Logs`. Instead of reading the logs 1 by 1, we can parse it with `EvtxECmd` and view it with `Timeline Explorer`.

```
EvtxECmd.exe -d "M:\forensic\files\ROOT\Windows\System32\winevt\Logs" --csv C:\Users\Administrator\Desktop\logs --csvf logs.csv
```

To identify configuration made to Microsoft Defender, we can filter `Set-MpPreference`. There are few common parameters used by attacker such as `-DisableRealtimeMonitoring`, `ExclusionPath` and etc.

![](assets/cyberdef/Pasted%20image%2020260613214022.png)

We can see that the attacker disabled the real time monitoring using `Set-MpPreference` and run `E:\hidden\Trusted Installer.exe`.

>✅ **Answer: `E:\hidden\Trusted Installer.exe`**

---

> Q4: Identifying the bot malware’s C&C infrastructure is key for detecting IOCs. According to threat intelligence reports, what URL does the bot use to download its C&C file?

![](assets/cyberdef/Pasted%20image%2020260613214858.png)

![](assets/cyberdef/Pasted%20image%2020260613215146.png)

Next, we can analyze the hash of the previous executable by uploading it to VirusTotal. The URL the bot used to download its C&C file is `http://anam0rph.su/in.php`

>✅ **Answer: `http://anam0rph.su/in.php`**

---

> Q5: Understanding the IOCs for files dropped by malware is essential for gaining insights into the various stages of the malware and its execution flow. What is the MD5 hash of the dropped **.exe** file?

![](assets/cyberdef/Pasted%20image%2020260613215521.png)

![](assets/cyberdef/Pasted%20image%2020260613220102.png)

![](assets/cyberdef/Pasted%20image%2020260613220146.png)

To identify the file dropped by the malware, we can filter Event ID `11`. In Sysmon, it means file creation operations. The executable dropped by the malware is `Sahofivizu.exe`. To obtain its MD5 hash we can search its name.

>✅ **Answer: 7FE00CC4EA8429629AC0AC610DB51993**

---

> Q6: Having the full file paths allows for a more complete cleanup, ensuring that all malicious components are identified and removed from the impacted locations. What is the full path of the first **DLL** dropped by the malware sample?

![](assets/cyberdef/Pasted%20image%2020260613220516.png)

Looking back at the dropped file, the first DLL dropped by the malware is `Gozekeneka.dll`

>✅ **Answer: Gozekeneka.dll**

---

> Q7: Connecting malware to APT groups is crucial for uncovering an attack's broader strategy, motivations, and long-term goals. Based on IOCs and threat intelligence reports, which APT group reactivated this malware for use in its campaigns?

![](assets/cyberdef/Pasted%20image%2020260613220848.png)

>✅ **Answer: Turla**









