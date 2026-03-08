# TryHackMe – Basic Pentesting Writeup

## Overview

This lab demonstrates the **basic penetration testing methodology** including reconnaissance, enumeration, exploitation, and privilege escalation.

The objective is to simulate a real-world vulnerable server and compromise it using common security testing techniques.

---

# 1. Objective

In this room the goals were:

- Scan a target machine
- Enumerate services
- Discover hidden directories
- Identify usernames
- Crack weak passwords
- Gain unauthorized access
- Escalate privileges

This lab helps understand how attackers compromise poorly configured systems.

---

# 2. Connecting to TryHackMe VPN

Before interacting with the target machine, the system must be connected to the **TryHackMe VPN**.

### Command

```bash
sudo openvpn myfile.ovpn
```

### Why This Step Is Important

The target machine exists on a **private internal network**.  
Without VPN access, the attacker machine cannot communicate with the target.

---

# 3. Port Scanning using Nmap

## Objective

Identify open ports and running services on the target machine.

### Command

```bash
nmap -sC -sV -p- <TARGET_IP>
```

### Explanation

- **-sC** → Default scripts  
- **-sV** → Service version detection  
- **-p-** → Scan all ports  

### Result

The scan revealed the following open ports:

| Port | Service |
|-----|------|
| 22 | SSH |
| 80 | HTTP |
| 139 | SMB |
| 445 | SMB |

### Conclusion

These services provide multiple attack surfaces:

- **HTTP** → Web enumeration  
- **SMB** → User enumeration  
- **SSH** → Remote login  

---

# 4. Web Directory Enumeration (Gobuster)

## Objective

Find hidden directories on the web server.

### Command

```bash
gobuster dir -u http://<TARGET_IP>/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

### Why Gobuster?

Web applications often contain hidden directories:

- Developer folders  
- Backup files  
- Sensitive resources  

### Result

```
**
```

### Answer

```
**
```

This confirms the existence of a hidden directory.

---

# 5. SMB Enumeration (enum4linux)

## Objective

Enumerate SMB service to find system usernames.

### Command

```bash
enum4linux -a <TARGET_IP>
```

### Why enum4linux?

SMB services often leak:

- usernames  
- shares  
- system details  

### Result

Discovered users:

```
**
**
```

These usernames can be used for login attempts.

---

# 6. SSH Brute Force Attack (Hydra)

## Objective

Attempt password brute force on SSH.

### Command

```bash
hydra -l ** -P /usr/share/wordlists/rockyou.txt ssh://<TARGET_IP>
```

### Why Hydra?

- SSH port was open  
- A valid username was known  
- Weak passwords are common  

### Result

```
login: **
password: **
```

---

# 7. Accessing the Server

Using the discovered credentials to log into the machine.

### Command

```bash
ssh **@<TARGET_IP>
```

### Question

Which service was used to access the server?

### Answer

```
**
```

---

# 8. Privilege Escalation Enumeration

After logging in, enumerate users on the system.

### Command

```bash
ls /home
```

### Result

```
**
**
```

### Question

Which user can you switch to?

### Answer

```
**
```

---

# 9. SSH Key Discovery

Investigating the `.ssh` directory of the discovered user.

### Command

```bash
ls /home/**/.ssh
```

### Result

```
**
```

This private key may allow login as that user.

---

# 10. Cracking the SSH Private Key

Convert the SSH key to a crackable format.

### Commands

```bash
ssh2john id_rsa > key_hash
john --wordlist=/usr/share/wordlists/rockyou.txt key_hash
```

### Result

```
**
```

### Final Answer

```
**
```

---

# Conclusion

This lab demonstrates several key penetration testing concepts:

- Enumeration is the most critical step in hacking
- Weak passwords can lead to full system compromise
- SMB services may leak usernames
- SSH private keys must be properly protected
- Misconfigurations allow privilege escalation

---

# Disclaimer

This writeup is intended for **educational purposes only**.  
Sensitive answers and flags have been intentionally hidden to respect the platform's learning policy.