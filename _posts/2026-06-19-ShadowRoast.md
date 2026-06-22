---
title: CyberDefenders - ShadowRoast
date: 2026-06-20 00:00:00 +0800
categories:
  - CyberDefenders
  - Threat Hunting
  - Splunk
tags:
  - Splunk
difficulty: Medium
pin: false
---

---

## Scenario

As a cybersecurity analyst at TechSecure Corp, you have been alerted to unusual activities within the company's Active Directory environment. Initial reports suggest unauthorized access and possible privilege escalation attempts.

---

## Key Findings

| Indicator | Detail                                                                      |
| --------- | --------------------------------------------------------------------------- |
| Process   | AdobeUpdater.exe                                                            |
| Process   | BackupUtility.exe (Rubeus)                                                  |
| Process   | DefragTool.exe (Mimikatz)                                                   |
| User      | CORPNET\sanderson                                                           |
| User      | COPRNET\tcooper                                                             |
| Path      | C:\Users\Default\AppData\Local\Temp\BackupUtility.exe                       |
| Path      | C:\Users\sanderson\Downloads\AdobeUpdater.exe                               |
| Path      | C:\Users\Default\AppData\Local\Temp\DefragTool.exe                          |
| SHA256    | 1bfbefa4ff4d0df3ee0090b5079cf84ed2e8d5377ba5b7a30afd88367d57b9ff (Rubeus)   |
| SHA256    | 92804faaab2175dc501d73e814663058c78c0a042675a8937266357bcfb96c50 (Mimikatz) |


---
## Tools

- Splunk

---

## Methodology

### Initial Access

![](assets/cyberdef/Pasted%20image%2020260620220702.png)

![](assets/cyberdef/Pasted%20image%2020260620223357.png)

I started by looking at the `source`field to get an overview what logs available in this environment. There are total of 3 type of logs available in this environment (`DC01`,`FileServer`,`Office-PC`). Firstly, I will be filtering `Office-PC` Sysmon Event ID 1, this event id will records the creation of new process. 

I noticed `CORPNET\sanderson` opened `AdobeUpdater.exe`from Windows Explorer, which stood out because it spawned `cmd.exe`which is unusual. 

![](assets/cyberdef/Pasted%20image%2020260620230128.png)

![](assets/cyberdef/Pasted%20image%2020260620231617.png)

To investigate, I look further into the logs and noticed the compromised user executed `powershell -ep bypass`which is used to bypass policy restrictions, it means any scripts can be run on that machine with no restrictions. It followed by the execution `C:\Users\Default\AppData\Local\Temp\BackupUtility.exe` with command `asreproast /format:hashcat`. It is a renamed copy of `Rubeus` which used to perform AS-REP roasting. It is used to exploits account that have "Kerberos pre-authentication" disabled, allowing attackers to request crackable password hashes
### Pivot

![](assets/cyberdef/Pasted%20image%2020260620233325.png)

![](assets/cyberdef/Pasted%20image%2020260620233658.png)

Later on, there's another account `COPRNET\tcooper` ran the similar powershell command but with a different executable `C:\Users\Default\AppData\Local\Temp\DefragTool.exe`. It is also a renamed copy but `Mimikatz.exe` which are used to extract plaintexts passwords, hash, PIN code and kerberos tickets.

```bash
# Enumerate process creation

index=shadowroast source="C:\\Logs\\Office-PC.ndjson" "event.provider"="Microsoft-Windows-Sysmon" event.code=1 earliest="06/08/2024:01:05:11"
| table _time, winlog.event_data.User, winlog.event_data.ParentCommandLine, winlog.event_data.CommandLine, winlog.event_data.ParentProcessId, winlog.event_data.ProcessId
| sort +_time
```

### Persistence

![](assets/cyberdef/Pasted%20image%2020260622145802.png)

Next, I will be finding potential persistence on the host by filtering Event ID 13 (RegistryValue Set) and Run keyword. The reason I filter Run because `Run`and `RunOnce`registry keys is commonly used by attacker to run specific scripts or programs when a user logs in the system. A `Run`registry key was created with the name `wyW5PZyF`.


```bash
# Enumerate persistence run key created 

index=shadowroast source="C:\\Logs\\Office-PC.ndjson" "event.provider"="Microsoft-Windows-Sysmon" event.code=13 earliest="06/08/2024:01:05:11" Run
| sort +_time
```

---

![](assets/cyberdef/Pasted%20image%2020260622153756.png)

We able to identify the attacker's used `Mimikatz`but its objective remain unclear. By reviewing the question, this is related to DCShadow and we can filter event.code 4929 to detect Active Directory schema changes.

```bash
# DCShadow

index=shadowroast event.code=4929
```

---

![](assets/cyberdef/Pasted%20image%2020260622154549.png)

![](assets/cyberdef/Pasted%20image%2020260622154745.png)

When we are trying to enable RDP connection remotely, the registry value `fDenyTSConnections` will set to `0`to enable rdp access. By filtering `fDenyTSConnections`, I able to identified the attacker set the value to `0` to enable RDP access.

```bash
index=shadowroast fDenyTSConnections
```

### Exfiltration

![](assets/cyberdef/Pasted%20image%2020260622155220.png)

Attacker would usually compressed the files into an archive before exfiltrate because it reduce the volume of outbound connection to avoid detection. We can filter some known archive extension such as `zip, 7z, winrar`. We can identify there's zip file created `CrashDump.zip` on `FileServer`by compromised user `tcooper`.

---
## MITRE ATT&CK

| Technique                                                             | ID        | Description                                                                                                 |
| --------------------------------------------------------------------- | --------- | ----------------------------------------------------------------------------------------------------------- |
| Masquerading                                                          | T1036     | Used `Rebeus.exe`, renamed it as `BackupUtility.exe`<br>Used `Mimikatz.exe`, renamed it as `DefragTool.exe` |
| User Execution                                                        | T1204     | Executed `AdobeUpdater.exe`                                                                                 |
| Steal or Forge Kerberos Tickets: AS-REP Roasting                      | T1558.004 | `asreproast /format:hashcat`                                                                                |
| Rogue Domain Controller                                               | T1207     | mimikatz DCShadow                                                                                           |
| Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder | T1547.001 | `Run` registry key set                                                                                      |


---

## Questions

> Q1: What's the malicious file name utilized by the attacker for initial access?

>✅ **Answer: `94.140.112.73`**

---

> Q2: What's the registry run key name created by the attacker for maintaining persistence?

>✅ **Answer: wyW5PZyF**

---

> Q3: What's the full path of the directory used by the attacker for storing his dropped tools?

>✅ **Answer: `C:\Users\Default\AppData\Local\Temp\`**

---

> Q4: What tool was used by the attacker for privilege escalation and credential harvesting?

>✅ **Answer: Rubeus**

---

> Q5: Was the attacker's credential harvesting successful? If so, can you provide the compromised domain account username?

>✅ **Answer: tcooper**

---

> Q6: What's the tool used by the attacker for registering a rogue Domain Controller to manipulate Active Directory data?

>✅ **Answer: mimikatz**

---

> Q7: What's the first command used by the attacker for enabling RDP on remote machines for lateral movement?

>✅ **Answer: `reg add "hklm\system\currentcontrolset\control\terminal server" /f /v fDenyTSConnections /t REG_DWORD /d 0`**

---

> Q8: What's the file name created by the attacker after compressing confidential files?

>✅ **Answer: CrashDump.zip**












