---
title: CyberDefenders - PacketDetective
date: 2026-06-03 00:00:00 +0800
categories:
  - CyberDefenders
  - Network Forensics
tags:
  - WireShark
difficulty: Easy
pin: false
---

---

**Scenario:** In September 2020, your SOC detected suspicious activity from a user device, flagged by unusual SMB protocol usage. Initial analysis indicates a possible compromise of a privileged account and remote access tool usage by an attacker.

## Questions

> Q1: The attacker’s activity showed extensive SMB protocol usage, indicating a potential pattern of significant data transfer or file access.
> What is the total number of bytes of the SMB protocol?

![](assets/cyberdef/Pasted%20image%2020260603220836.png)

Statistics of the protocol can be viewed at **Statistics** -> **Protocol Hierarchy** and look for SMB.


>✅ **Answer: 4406

---

> Q2: Authentication through SMB was a critical step in gaining access to the targeted system. Identifying the username used for this authentication will help determine if a privileged account was compromised.  
> Which username was utilized for authentication via SMB?

![](assets/cyberdef/Pasted%20image%2020260603221113.png)

SMB used NTLM for authentication, and NTLM authentication consists of three messages:
1. NTLMSSP_NEGOTITATE: Client -> Server (Determines supported options)
2. NTLMSSP_CHALLENGER: Server -> Client (Server Challenge)
3. NTLMSSP_AUTH: Client -> Server (Sends username, domain, workstation, and challenge response)

Filter SMB packet and look for NTLMSSP_AUTH, in this scenario it is authenticating with username Administrator

>✅ **Answer: Administrator**

---

> Q3: During the attack, the adversary accessed certain files. Identifying which files were accessed can reveal the attacker's intent.  
> What is the name of the file that was opened by the attacker?

The SMB flow when someone trying to access a file:

```
Client                    Server
------                    ------
Tree Connect      ---->
                  <----  Success

NT Create AndX    ---->  Open/Create file
                  <----  FID returned

Read/Write        ---->  Use returned FID
                  <----  Data

Close             ---->
                  <----  Success

```

![](assets/cyberdef/Pasted%20image%2020260603222212.png)

In this pcap file, the adversary accessed the file `eventlog`inside `IPC$`share

>✅ **Answer: eventlog**

---

> Q4: Clearing event logs is a common tactic to hide malicious actions and evade detection. Pinpointing the timestamp of this action is essential for building a timeline of the attacker’s behavior.  
> What is the timestamp of the attempt to clear the event log? (24-hour UTC format)

![](assets/cyberdef/Pasted%20image%2020260603223051.png)

Look for ClearEventLogW packet or filter `dcerpc.opnum == 0`, because 0 is the first method in RPC, and it is also refer to Event Log Clearing

>✅ **Answer: 2020-09-23 16:50**

---

> Q5: The attacker used "named pipes" for communication, suggesting they may have utilized Remote Procedure Calls (RPC) for lateral movement across the network. RPC allows one program to request services from another remotely, which could grant the attacker unauthorized access or control.  
> What is the name of the service that communicated using this named pipe?

![](assets/cyberdef/Pasted%20image%2020260603230323.png)

I used the search feature and changed the filter to Packet Bytes and String. Then, I search for PIPE, it returned a result at packet 25 and the name of the service is `atsvc`

>✅ **Answer: atsvc**

---

> Q6: Measuring the duration of suspicious communication can reveal how long the attacker maintained unauthorized access, providing insights into the scope and persistence of the attack.
> What was the duration of communication between the identified addresses 172.16.66.1 and 172.16.66.36?

![](assets/cyberdef/Pasted%20image%2020260603230738.png)

The statistics of conversation can be viewed at Statistics -> Conversation -> IPV4. Tick Absolute Start Time.

>✅ **Answer: 11:7247**

---

> Q7: The attacker used a non-standard username to set up requests, indicating an attempt to maintain covert access. Identifying this username is essential for understanding how persistence was established.  
> Which username was used to set up these potentially suspicious requests?

![](assets/cyberdef/Pasted%20image%2020260603231010.png)

In packet 5, we can see that the attacker used username `backdoor`to authenticated to the `ADMIN$`share.

>✅ **Answer: backdoor**

---

> Q8: The attacker leveraged a specific executable file to execute processes remotely on the compromised system. Recognizing this file name can assist in pinpointing the tools used in the attack.  
> What is the name of the executable file utilized to execute processes remotely?

By referring to the previous screenshot, the attacker is using a tool called `PSEXESVC.EXE` aka PsExec. It is a tool that access the system remotely via SMB protocol. For more information can refer to [PsExec-Hunt]({% link _posts/2026-05-26-PsExec-Hunt%})

>✅ **Answer: PSEXEC.exe**






