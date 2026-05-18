---
title: "HTB - MachineName"          # Change to your writeup title
date: 2025-01-10 00:00:00 +0800    # Date + your timezone offset (MY = +0800)
categories: [HackTheBox, Linux]     # [Platform, OS] — e.g. [TryHackMe, Windows], [CTF, Web]
tags: [nmap, gobuster, privesc]     # Tools & techniques used (lowercase)
difficulty: Medium                  # Easy / Medium / Hard / Insane
image:
  path: /assets/images/htb-machinename/banner.png   # Optional: machine banner image
pin: false                          # Set to true to pin this post to the top
---

> ⚠️ **Spoiler Warning:** This writeup covers a retired machine. If you haven't solved it yet, try it first!

---

## Overview

Brief 2–3 sentence summary of the machine or challenge.

- **Platform:** HackTheBox / TryHackMe / CTF Name
- **OS:** Linux / Windows
- **Difficulty:** Medium
- **Release Date:** YYYY-MM-DD
- **Retired Date:** YYYY-MM-DD

---

## Reconnaissance

### Port Scan

```bash
nmap -sC -sV -oN nmap/initial 10.10.10.X
```

```
# Paste nmap output here
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.2
80/tcp  open  http    Apache 2.4.41
```

**Open Ports Summary:**

| Port | Service | Version |
|------|---------|---------|
| 22   | SSH     | OpenSSH 8.2 |
| 80   | HTTP    | Apache 2.4.41 |

---

## Enumeration

### Web Enumeration

```bash
gobuster dir -u http://10.10.10.X -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Describe what you found and why it matters.

![Web page screenshot](/assets/images/htb-machinename/webpage.png)

### [Service Name] Enumeration

Add more enumeration sections as needed — one per service or attack surface.

---

## Foothold / Initial Access

Explain your exploitation path step by step. Include commands, tool usage, and reasoning.

```bash
# Example exploit command
python3 exploit.py 10.10.10.X 4444
```

> 💡 **Note:** Explain any non-obvious step so readers understand *why* you did it, not just *what* you did.

```bash
# Caught reverse shell
nc -lvnp 4444
```

```
# Output
connect to [10.10.14.X] from (UNKNOWN) [10.10.10.X] 52134
www-data@machinename:/var/www/html$
```

---

## Lateral Movement *(if applicable)*

Describe any pivoting between users or hosts before reaching root/admin.

```bash
su - anotheruser
# or
ssh user@10.10.10.X -i found_key
```

---

## Privilege Escalation

### Enumeration

```bash
# Always start with these
id
sudo -l
find / -perm -4000 -type f 2>/dev/null    # SUID binaries
cat /etc/crontab                           # Cron jobs
```

Describe what stood out and why it's exploitable.

### Exploitation

Walk through the privesc step by step.

```bash
# Example: SUID binary exploit
/usr/bin/somebinary -e /bin/bash
```

```
root@machinename:/# id
uid=0(root) gid=0(root) groups=0(root)
```

---

## Flags

```
user.txt : [REDACTED]
root.txt : [REDACTED]
```

> Flags are intentionally redacted. The goal is methodology, not shortcuts. 😊

---

## Lessons Learned

Reflect on what was new or interesting about this machine.

- **New technique:** e.g. First time exploiting X vulnerability type
- **Tool tip:** e.g. Discovered that Gobuster flag `-x php,txt,html` speeds up web enum
- **Mistake made:** e.g. Spent 30 min on rabbit hole — next time enumerate all ports first

---

## References

- [CVE-XXXX-XXXXX](https://nvd.nist.gov/)
- [GTFOBins - somebinary](https://gtfobins.github.io/)
- [Tool documentation](https://example.com)

---

*Written by [Your Alias] | [your-github.io](https://your-github.io)*
