---
title: CyberDefenders - REvil - XXE Infiltration
date: 2026-06-08 00:00:00 +0800
categories:
  - CyberDefenders
  - Network Forensics
tags:
  - WireShark
  - PortScan
  - XXE
difficulty: Easy
pin: false
---

---

**Scenario:** An automated alert has detected unusual XML data being processed by the server, which suggests a potential XXE (XML External Entity) Injection attack. This raises concerns about the integrity of the company's customer data and internal systems, prompting an immediate investigation.
## Questions

> Q1: Identifying the open ports discovered by an attacker helps us understand which services are exposed and potentially vulnerable. Can you identify the highest-numbered port that is open on the victim's web server?

![](assets/cyberdef/Pasted%20image%2020260608190628.png)

![](assets/cyberdef/Pasted%20image%2020260608191005.png)

To identify port scan activity, we need to first identify the attacker's IP. There is only one result return in the Conversation Statistics. The IP `210.106.114.183` is scanning the ports of `50.229.151.185`. Filter `tcp.flags == 0x0004 && ip.src == 210.106.114.183` to identify open ports. The highest-numbered port opened is `3306` which is used by msql.

>✅ **Answer: 3306**

---

> Q2: By identifying the vulnerable PHP script, security teams can directly address and mitigate the vulnerability. What's the complete URI of the PHP script vulnerable to XXE Injection?

XXE injection allows attacker to interfere with application's processing of XML data. To identify XXE injection, we can search `XML` or POST request. 

![](assets/cyberdef/Pasted%20image%2020260608192557.png)

![](assets/cyberdef/Pasted%20image%2020260608193036.png)

By following the TCP stream, it shows that the attacker injected XXE payload via `upload.php`, the payload is trying to read the file in `/var/www/html/index.php`.

>✅ **Answer: /review/upload.php**

---

> Q3: To construct the attack timeline and determine the initial point of compromise. What's the name of the first malicious XML file uploaded by the attacker?

![](assets/cyberdef/Pasted%20image%2020260608193317.png)

![](assets/cyberdef/Pasted%20image%2020260608193413.png)

The first malicious XML file uploaded was `TheGreatGatsby.xml`, the payload is reading the `/etc/password`of the system.

>✅ **Answer: TheGreatGatsby.xml**

---

> Q4: Understanding which sensitive files were accessed helps evaluate the breach's potential impact. What's the name of the web app configuration file the attacker read?

![](assets/cyberdef/Pasted%20image%2020260608193821.png)

The attacker read the `config.php` which commonly stored credentials such as db password.

>✅ **Answer: config.php**

---

> Q5: To assess the scope of the breach, what is the password for the compromised database user?

![](assets/cyberdef/Pasted%20image%2020260608194013.png)

>✅ **Answer: Winter2024**

---

> Q6: Following the database user compromise. What is the timestamp of the attacker's initial connection to the MySQL server using the compromised credentials after the exposure?

![](assets/cyberdef/Pasted%20image%2020260608194848.png)

By following the sequence of the packets, after the attacker read the `config.php` there is a mysql login at packet `88348`. The timestamp of the attacker's initial connection is `2024-05-31 12:08`

---

> Q7: To eliminate the threat and prevent further unauthorized access, can you identify the name of the web shell that the attacker uploaded for remote code execution and persistence?

![](assets/cyberdef/Pasted%20image%2020260608195143.png)

The webshell the attacker uploaded is `booking.php`, the payload is directing to an external IP and php wrapped is used to bypass restrictions.