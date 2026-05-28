---
title: CyberDefenders - PsExec Hunt
date: 2026-05-26 00:00:00 +0800
categories:
  - CyberDefenders
  - Network Forensics
tags:
  - WireShark
difficulty: Easy
pin: false
---

---

**Scenario:** An alert from the Intrusion Detection System (IDS) flagged suspicious lateral movement activity involving PsExec. This indicates potential unauthorized access and movement across the network. As a SOC Analyst, your task is to investigate the provided PCAP file to trace the attacker’s activities. Identify their entry point, the machines targeted, the extent of the breach, and any critical indicators that reveal their tactics and objectives within the compromised environment.

## Questions

>Q1: To effectively trace the attacker's activities within our network, can you identify the IP address of the machine from which the attacker initially gained access?

In order to understand better we have to know what is PsExec and how it works. It is a tool used to access the system remotely via SMB protocol. According to [Red Canary](https://redcanary.com/blog/threat-detection/threat-hunting-psexec-lateral-movement/), PSExec follows a simple pattern:

1. First, it will establishes an SMB network connection to a target system using administrator's credentials.
2. Then, a copy of receiver process named `PSEXECSVC.EXE` will be uploaded to the target system's ADMIN$ share
3. Launches `PSEXECSVC.EXE` which sends input and output to a named pipe `\\.\pipe\psexesvc`

![](assets/cyberdef/Pasted%20image%2020260526181915.png)

From this screenshot, I can see it matched the pattern of PSExec, where it authenticate to 2 smb share server which are `\\10.0.0.133\IPC$` & `\\10.0.0.133\ADMIN$`. At packet 144/145, `PSEXESVC.exe` was uploaded to the file server


>✅ **Answer: 10.0.0.130**

---


> Q2: To fully understand the extent of the breach, can you determine the machine's hostname to which the attacker first pivoted?

![](assets/cyberdef/Pasted%20image%2020260526185939.png)

By following the TCP stream, I can see that the attacker pivot from HR-PC to SALES-PC

> ✅ **Answer: SALES-PC**

---

> Q3: Knowing the username of the account the attacker used for authentication will give us insights into the extent of the breach. What is the username utilized by the attacker for authentication?

Based on the information on top, the attacker is authenticating with the username `\ssales`

> ✅ **Answer: ssales**

---

> Q4: After figuring out how the attacker moved within our network, we need to know what they did on the target machine. What's the name of the service executable the attacker set up on the target?

After the SMB authentication, I can see that `PSEXESVC.exe` has dropped on the target machine

> ✅ **Answer: PSEXESVC.exe**

---

Q5: We need to know how the attacker installed the service on the compromised machine to understand the attacker's lateral movement tactics. This can help identify other affected systems. Which network share was used by PsExec to install the service on the target machine? 

![](assets/cyberdef/Pasted%20image%2020260526190709.png)

> ✅ **Answer: ADMIN$**

---

> Q6: We must identify the network share used to communicate between the two machines. Which network share did PsExec use for communication?

![](assets/cyberdef/Pasted%20image%2020260526190905.png)

Apart from `ADMIN$`, the attacker also communicate with `IPC$` which is commonly used in SMB communications.

> ✅ **Answer: IPC$**

---

> Q7: Now that we have a clearer picture of the attacker's activities on the compromised machine, it's important to identify any further lateral movement. What is the hostname of the second machine the attacker targeted to pivot within our network?

![](assets/cyberdef/Pasted%20image%2020260526191508.png)

By filtering `ntlmssp.challenge.target_name`, this filter focuses on authentication. I can easily find the next machine the attacker pivot to.

> ✅ **Answer: MARKETING-PC**

