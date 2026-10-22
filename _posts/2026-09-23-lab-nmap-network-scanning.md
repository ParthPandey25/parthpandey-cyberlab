---
title: "Lab: Nmap Network Scanning Fundamentals"
author: parthpandey
categories: [Labs]
tags: [nmap, network-scanning, enumeration, network]
render_with_liquid: false
---

## Objective

Master the fundamental Nmap scanning techniques used in the reconnaissance phase of penetration testing. This lab covers host discovery, port scanning, service/version detection, OS detection, and NSE scripting.

## Environment Setup

| Component | Details |
|---|---|
| **Attack Machine** | Kali Linux 2024.x |
| **Target Machine** | Metasploitable 2 (Ubuntu) |
| **Network** | 10.10.10.0/24 |
| **Tools Required** | Nmap 7.x |

```console
$ nmap --version
Nmap version 7.94 ( https://nmap.org )
```

## Procedure

### Step 1: Host Discovery

Discovering live hosts on the network without performing a port scan:

```console
$ nmap -sn 10.10.10.0/24
Starting Nmap 7.94 ( https://nmap.org )
Nmap scan report for 10.10.10.1
Host is up (0.00025s latency).
Nmap scan report for 10.10.10.5
Host is up (0.0013s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 2.54 seconds
```

**Key flags:**
- `-sn` — Ping scan only, no port scan (formerly `-sP`)
- Useful for mapping out live hosts before deeper scanning

### Step 2: Basic TCP Port Scan

```console
$ nmap -T4 -p- 10.10.10.5
Starting Nmap 7.94 ( https://nmap.org )
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
23/tcp    open  telnet
25/tcp    open  smtp
80/tcp    open  http
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
512/tcp   open  exec
513/tcp   open  login
514/tcp   open  shell
1099/tcp  open  rmiregistry
1524/tcp  open  ingreslock
2121/tcp  open  ccproxy-ftp
3306/tcp  open  mysql
5432/tcp  open  postgresql
5900/tcp  open  vnc
6000/tcp  open  X11
8009/tcp  open  ajp13
8180/tcp  open  unknown
```

**Key flags:**
- `-T4` — Aggressive timing for faster scans
- `-p-` — Scan all 65,535 ports (not just the default top 1000)

**Observation:** The target has a very large attack surface with 20+ open ports, typical of Metasploitable.

### Step 3: Service and Version Detection

```console
$ nmap -sV -sC -p 21,22,80,445,3306 10.10.10.5
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian
80/tcp   open  http        Apache httpd 2.2.8 ((Ubuntu) DAV/2)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian
3306/tcp open  mysql       MySQL 5.0.51a-3ubuntu5
```

**Key flags:**
- `-sV` — Probe open ports to determine service/version info
- `-sC` — Run default NSE scripts (equivalent to `--script=default`)

**Observation:** `vsftpd 2.3.4` is a notoriously vulnerable version with a known backdoor (CVE-2011-2523).

### Step 4: OS Detection

```console
$ nmap -O 10.10.10.5
OS details: Linux 2.6.9 - 2.6.33
Network Distance: 1 hop
```

**Key flags:**
- `-O` — Enable OS detection using TCP/IP fingerprinting

### Step 5: NSE Scripting

Running specific vulnerability detection scripts:

```console
$ nmap --script=vuln -p 21,445 10.10.10.5
PORT    STATE SERVICE
21/tcp  open  ftp
| ftp-vsftpd-backdoor:
|   VULNERABLE:
|   vsFTPd version 2.3.4 backdoor
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2011-2523

445/tcp open  microsoft-ds
| smb-vuln-ms08-067:
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution
```

**Key flags:**
- `--script=vuln` — Run all vulnerability detection scripts
- Can also target specific scripts: `--script=ftp-vsftpd-backdoor`

### Step 6: Output Formats

Saving scan results in multiple formats for documentation:

```console
$ nmap -sV -sC -p- -oA scan_results 10.10.10.5
```

**Key flags:**
- `-oA` — Output in all formats (.nmap, .xml, .gnmap)
- `-oN` — Normal output only
- `-oX` — XML output (useful for tools like Metasploit)
- `-oG` — Grepable output

## Findings

| Finding | Severity | Description |
|---|---|---|
| vsftpd 2.3.4 Backdoor | **Critical** | Known backdoor CVE-2011-2523, allows remote code execution |
| Telnet Enabled | **High** | Transmits credentials in plaintext |
| MySQL Exposed | **High** | Database accessible on network, potential for data exfiltration |
| Outdated OpenSSH | **Medium** | Version 4.7p1 has known vulnerabilities |
| Multiple Unnecessary Services | **Medium** | Large attack surface increases risk |

## Nmap Cheat Sheet

```
# Quick reference for common scan types

# Host Discovery
nmap -sn 10.10.10.0/24              # Ping sweep
nmap -Pn 10.10.10.5                 # Skip host discovery

# Port Scanning
nmap -p 80,443 10.10.10.5           # Specific ports
nmap -p- 10.10.10.5                 # All ports
nmap -F 10.10.10.5                  # Fast (top 100 ports)
nmap --top-ports 1000 10.10.10.5    # Top N ports

# Service/Version Detection
nmap -sV 10.10.10.5                 # Version detection
nmap -sC 10.10.10.5                 # Default scripts
nmap -A 10.10.10.5                  # Aggressive (OS + version + scripts + traceroute)

# Scan Types
nmap -sT 10.10.10.5                 # TCP connect scan
nmap -sS 10.10.10.5                 # SYN scan (stealth, requires root)
nmap -sU 10.10.10.5                 # UDP scan

# NSE Scripts
nmap --script=vuln 10.10.10.5       # Vulnerability scripts
nmap --script=smb-enum-shares 10.10.10.5  # Specific script

# Output
nmap -oA results 10.10.10.5         # All formats
nmap -oN results.txt 10.10.10.5     # Normal text
nmap -oX results.xml 10.10.10.5     # XML
```

## Lessons Learned

1. **Always scan all 65,535 ports** (`-p-`) — services often run on non-standard ports that the default top 1000 scan misses
2. **Service version detection** (`-sV`) is critical — knowing the exact version allows you to search for known CVEs
3. **NSE scripts** are incredibly powerful for automated vulnerability detection — the `vuln` category alone found two critical issues
4. **Save scan results** in multiple formats (`-oA`) — XML output integrates with Metasploit and other tools for the exploitation phase
5. **Timing matters** — `T4` is good for local networks, but use `T2` or `T3` on real engagements to avoid detection and IDS alerts

## References

- [Nmap Official Documentation](https://nmap.org/book/)
- [Nmap Scripting Engine (NSE)](https://nmap.org/nsedoc/)
- [CVE-2011-2523 — vsftpd 2.3.4 Backdoor](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)
