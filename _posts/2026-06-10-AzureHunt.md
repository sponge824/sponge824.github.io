---
title: CyberDefenders - AzureHunt
date: 2026-06-10 00:00:00 +0800
categories:
  - CyberDefenders
  - Cloud Forensics
  - Azure
tags:
  - ELK
difficulty: Easy
pin: false
---

---

**Scenario:** A finance company's Azure environment has flagged multiple failed login attempts from an unfamiliar geographic location, followed by a successful authentication. Shortly after, logs indicate access to sensitive Blob Storage files and a virtual machine start action. Investigate authentication logs, storage access patterns, and VM activity to determine the scope of the compromise.

## Questions

> Q1: As a US-based company, the security team has observed significant suspicious activity from an unusual country. What is the name of the country from which the attack originated?

![](assets/cyberdef/Pasted%20image%2020260610180033.png)

![](assets/cyberdef/Pasted%20image%2020260610180839.png)

This Azure environment consist of thee types of log: `azureadlogs`,`bloblogs`, and `activitylogs`. Next, we can group all the logon country and identify which country the attack originated. Since, this is a US-base company, high volume of logon from another country increase the suspicion.

>✅ **Answer: Germany**

---

> Q2: To establish an accurate incident timeline, what is the timestamp of the initial activity originating from the country?

![](assets/cyberdef/Pasted%20image%2020260610184259.png)

By filtering the log originating from Germany. `source.geo.country_name.keyword: Germany`. The timestamp of the initial activity originating from Germany is `2023-10-05 15:09`

>✅ **Answer: 2023-10-05 15:09**

---

> Q3: To assess the scope of compromise, we must determine the attacker's entry point. What is the display name of the compromised user account?

![](assets/cyberdef/Pasted%20image%2020260610184641.png)

To identify the attacker's entry point, we can filter the logon username into a table. Reviewing the results, we can see that the initial logon username was `alice`, whcih was later followed by `it.admin1`.

>✅ **Answer: alice**

---

> Q4: To gain insights into the attacker's tactics and enumeration strategy, what is the name of the script file the attacker accessed within blob storage?

![](assets/cyberdef/Pasted%20image%2020260610185737.png)

Filter `azure.eventhub.category: StorageRead` which represents read operations performed on objects within Azure storage and `event.operationName: GetBlob` refers to API operation read or downloads a blob. Reviewing the results, we can see that there is a powershell script `ps1`accessed by the attacker

>✅ **Answer: service-config.ps1**

---

> Q5: For a detailed analysis of the attacker's actions, what is the name of the storage account housing the script file?

![](assets/cyberdef/Pasted%20image%2020260610190128.png)

The name of the storage account is `cactusstorage2023`

>✅ **Answer: cactusstorage2023**

---

> Q6: Tracing the attacker's movements across our infrastructure, what is the User Principal Name (UPN) of the second user account the attacker compromised?

![](assets/cyberdef/Pasted%20image%2020260610190600.png)

>✅ **Answer: `it.admin1@cybercactus.onmicrosoft.com`**

---

> Q7: Analyzing the attacker's impact on our environment, what is the name of the Virtual Machine (VM) the attacker started?

![](assets/cyberdef/Pasted%20image%2020260610192753.png)

To reduce the scope, we can configure start date/time to the timestamp of initial activity which is around `Oct 5, 2023 @ 15:00`. Then, we can filter the log type to `activitylogs`and the event action to `MICROSOFT.COMPUTE/VIRTUALMACHINES/START/ACTION`

```
azure-eventhub.eventhub: activitylogs and event.action.keyword : MICROSOFT.COMPUTE/VIRTUALMACHINES/START/ACTION
```
 The query above filters the logs to display events related to the activity performed and the action of starting the virtual machines. In addtion, we can filter add `azure.resource.name`column to table, it will display the name of the virtual machine

>✅ **Answer: DEV01VM**

---

> Q8: To assess the potential data exposure, what is the name of the database exported?

![](assets/cyberdef/Pasted%20image%2020260610193125.png)

![](assets/cyberdef/Pasted%20image%2020260610193225.png)

![](assets/cyberdef/Pasted%20image%2020260610193335.png)

To find the database exported, we can filter with the query below. This query filter the logs to display events related to database export. Reviewing the results, there is only 1 database exported during the timestamp which is `CustomerDataDB`.

```
azure-eventhub.eventhub: activitylogs and event.action.keyword: MICROSOFT.SQL/SERVERS/DATABASES/EXPORT/ACTION
```

>✅ **Answer: CustomerDataDB**

---

> Q9 : In your pursuit of uncovering persistence techniques, what is the display name associated with the user account you have discovered?

![](assets/cyberdef/Pasted%20image%2020260610194353.png)

![](assets/cyberdef/Pasted%20image%2020260610194453.png)

Filter `event.action: "Add User"` display event related to create new user. Reviewing the logs, the create user action was initiated by the compromised account `it.admin1@cybercactus.onmicrosoft.com`. The new user created is `IT_Support@cybercactus.onmicrosoft.com`

>✅ **Answer: IT Support**

---

> Q10: The attacker utilized a compromised account to assign a new role. What role was granted?

![](assets/cyberdef/Pasted%20image%2020260610194934.png)

Filter `event.action.keyword: "MICROSOFT.AUTHORIZATION/ROLEASSIGNMENTS/WRITE"` to display event related new role assignment. The results shown that the new role granted by the compromised account is `Owner`

>✅ **Answer: Owner**

---

> Q11: For a comprehensive timeline and understanding of the breach progression, What is the timestamp of the first successful login recorded for this user account?

![](assets/cyberdef/Pasted%20image%2020260610200223.png)

This query filters the logs to display events related to IT support's sign-in activities. Upon, examining the output, the first sign-in activities related to IT support was occurred on `Oct 6, 2023 @ 07:30:43`.

>✅ **Answer: 2023-10-06 07:30**











