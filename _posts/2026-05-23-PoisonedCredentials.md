---
title: CyberDefenders - PoisonedCredentials
date: 2026-05-23 00:00:00 +0800
categories:
  - CyberDefenders
  - Network Forensics
tags:
  - WireShark
  - LLMNR_Poisoning
difficulty: Easy
pin: false
---

---

**Scenario:** Your organization's security team has detected a surge in suspicious network activity. There are concerns that LLMNR (Link-Local Multicast Name Resolution) and NBT-NS (NetBIOS Name Service) poisoning attacks may be occurring within your network. These attacks are known for exploiting these protocols to intercept network traffic and potentially compromise user credentials. Your task is to investigate the network logs and examine captured network traffic.

## Questions

>Q1: In the context of the incident described in the scenario, the attacker initiated their actions by taking advantage of benign network traffic from legitimate machines. Can you identify the specific mistyped query made by the machine with the IP address 192.168.232.162?

![](assets/cyberdef/Pasted%20image%2020260523132104.png)

In this network traffic, I can see there is a query made by `192.168.232.162`, FILESHAARE.

>✅ **Answer: fileshaare**

---

> Q2: We are investigating a network security incident. To conduct a thorough investigation, We need to determine the IP address of the rogue machine. What is the IP address of the machine acting as the rogue entity?

![](assets/cyberdef/Pasted%20image%2020260523133134.png)

I can see that this IP `192.168.232.215` is responding to all the mistyped query.

>✅ **Answer: 192.168.232.215**

---

> Q3: As part of our investigation, identifying all affected machines is essential. What is the IP address of the second machine that received poisoned responses from the rogue machine?

Based on the previous screenshot, `192.168.232.176` has also received the poisoned responses from the rogue machine.

> ✅ **Answer: 192.168.232.176**

---

> Q4: We suspect that user accounts may have been compromised. To assess this, we must determine the username associated with the compromised account. What is the username of the account that the attacker compromised?

So we already know that `192.168.232.215` is the rogue machine, after performing LLMNR poisoning attack, usually the attacker will try to authenticate with the stolen account.

![](assets/cyberdef/Pasted%20image%2020260523134425.png)

> ✅ **Answer: janesmith**

---

> Q5: As part of our investigation, we aim to understand the extent of the attacker's activities. What is the hostname of the machine that the attacker accessed via SMB?

![](assets/cyberdef/Pasted%20image%2020260523135351.png)

The hostname can be found in Session Setup Request -> SMB2 -> Session Setup Request (0x01) -> Security Blob -> GSS-API Generic Security Service Application Program Interface -> Simple Protected Negotiation -> negTokenTarg -> NTLM Secure Service Provider -> NTLM Reponse -> NTLMv2 Response