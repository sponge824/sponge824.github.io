---
title: CyberDefenders - JetBrains
date: 2026-06-04 00:00:00 +0800
categories:
  - CyberDefenders
  - Network Forensics
tags:
  - WireShark
difficulty: Easy
pin: false
---

---

**Scenario:** During a recent security incident, an attacker successfully exploited a vulnerability in our web server, allowing them to upload webshells and gain full control over the system. The attacker utilized the compromised web server as a launch point for further malicious activities, including data manipulation.

## Questions

> Q1: Identifying the attacker's IP address helps trace the source and stop further attacks. What is the attacker's IP address?

![](assets/cyberdef/Pasted%20image%2020260604204746.png)

By reviewing the HTTP conversation statistics, there are multiple IP addresses with a high number of packets.

To reduce our scope, we can check if there is any enumeration or file upload. To achieve that, we can filter `GET` or `POST`request, `http.request.method == <METHODS>`. We observed there is a suspicious file upload, it uploaded a `.jsp`file

![](assets/cyberdef/Pasted%20image%2020260604210236.png)

![](assets/cyberdef/Pasted%20image%2020260604210047.png)

By following the TCP stream, we observed that he attacker uploaded a zip file that contained of  a `.jsp` webshell.

>✅ **Answer: 23.158.56.196**

---

> Q2: To identify potential vulnerability exploitation, what version of our web server service is running?

![](assets/cyberdef/Pasted%20image%2020260604210509.png)

The version of our web server service can also be found in the TCP stream.

>✅ **Answer: 2023.11.3**

---

> Q3: After identifying the version of our web server service, what CVE number corresponds to the vulnerability the attacker exploited?

![](assets/cyberdef/Pasted%20image%2020260604211231.png)

Based on the TCP stream, the web server is running on TeamCity 2023.11.3, it is vulnerable to CVE-2024-27198. This vulnerability allows attacker to bypass authentication.

>✅ **Answer: CVE-2024-27198**

---

> Q4: The attacker exploited the vulnerability to create a user account. What credentials did he set up?

![](assets/cyberdef/Pasted%20image%2020260604211327.png)

The credentials of the user account created by the attacker can also be found in the TCP stream

>✅ **Answer: c91oyemw:CL5vzdwLuK**

---

> Q5: The attacker uploaded a webshell to ensure his access to the system. What is the name of the file that the attacker uploaded?

![](assets/cyberdef/Pasted%20image%2020260604211645.png)

As mentioned earlier, the attacker uploaded a zip file that consists of a jsp webshell

>✅ **Answer: NSt8bHTg.zip**

---

> Q6: When did the attacker execute their first command via the web shell?

![](assets/cyberdef/Pasted%20image%2020260604212045.png)

The first command executed by the attacker via web shell is `ls` at 2024-06-30 08:03.

>✅ **Answer: 2024-06-30 08:03**

---

> Q7: The attacker tampered with a text file that contained the credentials of the admin user of the webserver. What new username and password did the attacker write in the file?

![](assets/cyberdef/Pasted%20image%2020260604212439.png)

The attacker wrote the username and password to `Creds.txt`file. To read it clearer, we can decode the data with [urldecoder](https://www.urldecoder.org/). 

![](assets/cyberdef/Pasted%20image%2020260604212558.png)

The decode url data shown that the attacker used echo command to write the username and password into `/tmp/Creds.txt`.

>✅ **Answer: a1l4m:youarecompromised**

---

> Q8: What is the MITRE Technique ID for the attacker's action in the previous question (Q7) when tampering with the text file?

![](assets/cyberdef/Pasted%20image%2020260604212931.png)

>✅ **Answer: T1565.001**

---

> Q9: The attacker tried to escape from the container but he didn’t succeed, What is the command that he used for that?

![](assets/cyberdef/Pasted%20image%2020260604213536.png)

![](assets/cyberdef/Pasted%20image%2020260604213550.png)

>✅ **Answer: docker run --rm -it -v /:/host ubuntu chroot host**