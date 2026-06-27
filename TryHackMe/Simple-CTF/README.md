# TryHackMe – Simple CTF Write-up

## Room Information

* **Platform:** TryHackMe
* **Room:** Simple CTF
* **Difficulty:** Easy
* **Category:** Web Exploitation, Password Cracking, Linux Privilege Escalation

---

# Objective

The objective of this room was to gain initial access to the target machine, enumerate the available services, exploit a vulnerable web application to obtain remote code execution, recover user credentials, and finally escalate privileges to obtain the root flag.

---

# Enumeration

The first step was to perform an Nmap scan to identify open ports and running services.

The scan revealed:

* FTP
* SSH
* HTTP

Since the web server was accessible, further enumeration focused on the HTTP service.

---

# Web Enumeration

Directory enumeration was performed to discover hidden files and directories.

During enumeration, a CMS Made Simple installation was discovered.

The CMS version was identified and searched for known vulnerabilities.

---

# Exploitation

A public exploit for the vulnerable CMS Made Simple version was available.

The exploit was used to gain code execution on the server.

To verify command execution, a simple payload was executed:

```php
system("id");
```

The server returned:

```
uid=33(www-data) gid=33(www-data)
```

This confirmed successful remote command execution as the **www-data** user.

A reverse shell payload was then executed to obtain an interactive shell.

---

# Shell Stabilization

After receiving the reverse shell, it was upgraded for better usability.

Commands used included:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

This provided a more stable shell for further enumeration.

---

# Credential Discovery

While enumerating the system, configuration files containing database credentials were discovered.

The recovered password was reused successfully with:

```bash
su mitch
```

Access to the **mitch** user account was obtained.

---

# User Flag

After switching to the **mitch** account, the home directory was explored.

The user flag was located inside the user's home directory and successfully captured.

---

# Privilege Escalation

Running:

```bash
sudo -l
```

revealed:

```
(root) NOPASSWD: /usr/bin/vim
```

This indicated that the **mitch** user could execute Vim as root without requiring a password.

Using the GTFOBins technique for Vim, a root shell was obtained.

After successful privilege escalation, the root flag was retrieved from the root user's directory.

---

# Skills Practiced

* Service Enumeration
* Directory Enumeration
* CMS Fingerprinting
* Public Exploit Research
* Remote Code Execution
* Reverse Shell Handling
* Linux Enumeration
* Credential Reuse
* Shell Stabilization
* GTFOBins Privilege Escalation

---

# Lessons Learned

This room demonstrated the importance of proper enumeration before exploitation. Identifying the CMS version allowed the use of a known exploit to achieve remote code execution. After gaining an initial shell, careful system enumeration led to credential discovery and lateral movement to another user. Finally, understanding Linux privilege escalation techniques and GTFOBins made it possible to obtain root access.

---

# Tools Used

* Nmap
* Gobuster
* Searchsploit
* Netcat
* Python PTY
* Vim
* GTFOBins
* Linux Command Line

---

# Conclusion

Simple CTF is an excellent beginner-friendly machine that covers a complete penetration testing workflow, including reconnaissance, exploitation, post-exploitation, and privilege escalation. It reinforces the importance of systematic enumeration, understanding public vulnerabilities, handling reverse shells, and leveraging Linux misconfigurations to gain administrative access.

