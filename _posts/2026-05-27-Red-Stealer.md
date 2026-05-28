---
title: CyberDefenders - Red Stealer
date: 2026-05-27 00:00:00 +0800
categories:
  - CyberDefenders
  - Threat Intel
tags:
difficulty: Easy
pin: false
---

---

**Scenario:** You are part of the Threat Intelligence team in the SOC (Security Operations Center). An executable file has been discovered on a colleague's computer, and it's suspected to be linked to a Command and Control (C2) server, indicating a potential malware infection.
## Questions

>Q1: Categorizing malware enables a quicker and clearer understanding of its unique behaviors and attack vectors. What category has Microsoft identified for that malware in VirusTotal?

![](assets/cyberdef/Pasted%20image%2020260528152807.png)

>✅ **Answer: Trojan**

---

> Q2: Clearly identifying the name of the malware file improves communication among the SOC team. What is the file name associated with this malware?

![](assets/cyberdef/Pasted%20image%2020260528152916.png)

>✅ **Answer: Wextract**

---

> Q3: Knowing the exact timestamp of when the malware was first observed can help prioritize response actions. Newly detected malware may require urgent containment and eradication compared to older, well-documented threats. What is the UTC timestamp of the malware's first submission to VirusTotal?

![](assets/cyberdef/Pasted%20image%2020260528152941.png)

>✅ **Answer: 2023-10-06 04:41**

---

> Q4: Understanding the techniques used by malware helps in strategic security planning. What is the MITRE ATT&CK technique ID for the malware's data collection from the system before exfiltration?

![](assets/cyberdef/Pasted%20image%2020260528153012.png)

>✅ **Answer: T1005**

---

> Q5: Following execution, which social media-related domain names did the malware resolve via DNS queries?

![](assets/cyberdef/Pasted%20image%2020260528153032.png)

>✅ **Answer: facebook.com**

---

> Q6: Once the malicious IP addresses are identified, network security devices such as firewalls can be configured to block traffic to and from these addresses. Can you provide the IP address and destination port the malware communicates with?

![](assets/cyberdef/Pasted%20image%2020260528153110.png)

>✅ **Answer: 77.91.124.55:19071**

---

> Q7: YARA rules are designed to identify specific malware patterns and behaviors. Using MalwareBazaar, what's the name of the YARA rule created by "Varp0s" that detects the identified malware?

![](assets/cyberdef/Pasted%20image%2020260528153143.png)

>✅ **Answer: detect_Redline_Stealer**

---

> Q8: Understanding which malware families are targeting the organization helps in strategic security planning for the future and prioritizing resources based on the threat. Can you provide the different malware alias associated with the malicious IP address according to ThreatFox?

![](assets/cyberdef/Pasted%20image%2020260528153212.png)

>✅ **Answer: RECORDSTEALER**

---

> Q9: By identifying the malware's imported DLLs, we can configure security tools to monitor for the loading or unusual usage of these specific DLLs. Can you provide the DLL utilized by the malware for privilege escalation?

![](assets/cyberdef/Pasted%20image%2020260528153237.png)

>✅ **Answer: ADVAPI32.dll**

