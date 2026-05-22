---
title: CyberDefenders - WebStrike
date: 2026-05-21 00:00:00 +0800
categories:
  - CyberDefenders
  - Network Forensics
tags:
  - WireShark
difficulty: Easy
pin: false
---

---

**Scenario:** A suspicious file was identified on a company web server, raising alarms within the intranet. The Development team flagged the anomaly, suspecting potential malicious activity. To address the issue, the network team captured critical network traffic and prepared a PCAP file for review.

## Questions

>Q1: Identifying the geographical origin of the attack facilitates the implementation of geo-blocking measures and the analysis of threat intelligence. From which city did the attack originate?

![webstrike_01](/assets/cyberdef/webstrike_01.png)

Based on the pcap file, there are only 2 IP found. By looking at the pcap file, `117.11.88.124` is probably the attacker.

![webstrike_02](/assets/cyberdef/webstrike_02.png)

In order to know where the attack originate, I used `ip.info`. The attack is from Tianjin, China

> ✅ **Answer: Tianjin**

---

>Q2: Knowing the attacker's User-Agent assists in creating robust filtering rules. What's the attacker's Full User-Agent?

Follow the TCP stream of the attacker's IP, we can see the user agent

![webstrike_03|696](/assets/cyberdef/webstrike_03.png)

>✅ **Answer: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0**

---

>Q3: We need to determine if any vulnerabilities were exploited. What is the name of the malicious web shell that was successfully uploaded?

We can filter POST request to identified the uploaded webshell. By following the TCP stream, I can see that the first upload attempt was failed because of invalid format.

![webstrike_04](/assets/cyberdef/webstrike_04.png)
![webstrike_05](/assets/cyberdef/webstrike_05.png)

The second attempt uploaded successfully, the attacker uploaded a reverse shell listening on port 8080.

![webstrike_01](/assets/cyberdef/webstrike_06.png)

>✅ **Answer: image.jpg.php**

---

>Q4: Identifying the directory where uploaded files are stored is crucial for locating the vulnerable page and removing any malicious files. Which directory is used by the website to store the uploaded files?

When an attacker wanted wanted to locate the upload folder, they will perform enumeration bruteforce the directories. We can filter `GET`request and identify the pattern.

![webstrike_07](/assets/cyberdef/webstrike_07.png)

> ✅ **Answer: /reviews/upload**

---

>Q5: Which port, opened on the attacker's machine, was targeted by the malicious web shell for establishing unauthorized outbound communication?

By looking at the php reverse shell the attacker uploaded, I can see that port 8080 is used by the attacker to obtain shell session

> ✅ **Answer: 8080**

---

>Q6: Recognizing the significance of compromised data helps prioritize incident response actions. Which file was the attacker attempting to exfiltrate?

Looking back at the POST request, there is a packet from web server to attacker's IP

![webstrike_01](/assets/cyberdef/webstrike_08.png)

> ✅ **Answer: passwd**

