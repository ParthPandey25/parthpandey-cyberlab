---
title: "TryHackMe: ROOM_NAME_HERE"
author: parthpandey
categories: [TryHackMe]
tags: [linux, web, nmap, enumeration, exploitation, privilege-escalation]
render_with_liquid: false
media_subpath: /images/tryhackme_ROOM_NAME_LOWERCASE/
image:
  path: room_image.webp
---

<!-- 
  ╔══════════════════════════════════════════════════════════╗
  ║  TRYHACKME WRITE-UP TEMPLATE                            ║
  ║  By Parth Pandey                                         ║
  ║                                                          ║
  ║  Instructions:                                           ║
  ║  1. Replace ROOM_NAME_HERE with the room name            ║
  ║  2. Replace ROOM_NAME_LOWERCASE with lowercase room name ║
  ║  3. Update tags based on tools/techniques used           ║
  ║  4. Place room images in /images/tryhackme_roomname/     ║
  ║  5. Save as: YYYY-MM-DD-tryhackme-roomname.md            ║
  ╚══════════════════════════════════════════════════════════╝

  Common tags to choose from:
  OS:         linux, windows
  Web:        web, php, python, javascript, sql, xss, sqli, lfi, rfi, ssrf, ssti, csrf
  Network:    nmap, wireshark, openvpn, smb, ftp, ssh, dns, http
  Tools:      metasploit, burpsuite, gobuster, nikto, hashcat, john, hydra, sqlmap
  Techniques: enumeration, exploitation, privilege-escalation, brute-force,
              reverse-shell, file-upload, command-injection, buffer-overflow,
              password-cracking, lateral-movement, persistence
  Misc:       docker, container-escape, active-directory, cryptography, steganography, forensics
-->

**ROOM_NAME_HERE** brief description of what the room involves and the key techniques/vulnerabilities explored.

[![Tryhackme Room Link](room_card.webp){: width="300" height="300" .shadow}](https://tryhackme.com/room/ROOM_SLUG_HERE){: .center }

## Room Information

| Detail | Value |
|---|---|
| **Platform** | TryHackMe |
| **Difficulty** | Easy / Medium / Hard |
| **Room Link** | [ROOM_NAME_HERE](https://tryhackme.com/room/ROOM_SLUG_HERE) |
| **Date Completed** | YYYY-MM-DD |

## Initial Enumeration

### Nmap Scan

```console
$ nmap -T4 -n -sC -sV -Pn -p- TARGET_IP
Nmap scan report for TARGET_IP
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH X.Xp1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd X.X.XX
```

There are X open ports:

* **22** (`SSH`)
* **80** (`HTTP`)

### Web Enumeration

```console
$ gobuster dir -u http://TARGET_IP -w /usr/share/wordlists/dirb/common.txt
```

## Exploitation

<!-- Describe the vulnerability found and how you exploited it -->

### Foothold

```console
$ # Commands used to gain initial access
```

### User Flag

```
user_flag_here
```

## Privilege Escalation

<!-- Describe the privilege escalation technique -->

```console
$ sudo -l
$ # or other privesc enumeration
```

### Root Flag

```
root_flag_here
```

## Lessons Learned

- Key takeaway 1
- Key takeaway 2
- Key takeaway 3

## Tools Used

- **Nmap** — Port scanning and service enumeration
- **Gobuster** — Directory brute-forcing
- **Tool3** — Purpose
