---
title: "Google Cybersecurity: Tools of the Trade — Linux and SQL"
author: parthpandey
categories: [Coursera, Google-Cybersecurity]
tags: [course-notes, linux, sql, command-line, bash, file-permissions]
render_with_liquid: false
date: 2026-08-22 16:45:00 +0530
---

## Course Overview

| Detail | Value |
|---|---|
| **Platform** | Coursera |
| **Certificate** | Google Cybersecurity Professional Certificate |
| **Course** | 4 of 8 — Tools of the Trade: Linux and SQL |
| **Duration** | ~25 hours |
| **Status** | ✅ Completed |

---

## Executive Summary

Course 4 focuses on the primary investigative and administrative tools used by security operations analysts daily: the **Linux operating system environment** and **Structured Query Language (SQL)**. Through dozens of hands-on interactive terminal labs, I honed practical fluency in Bash navigation, pipeline filtering (`grep`, `awk`, `sed`), POSIX user and file permission administration, audit log forensics in `/var/log`, and SQL database querying to pinpoint unauthorized access patterns, failed authentications, and data exfiltration indicators.

---

## Technical Deep Dive: Linux Command-Line Mastery

### 1. Linux Directory Architecture & Forensics Artifacts
Security analysts must navigate the Linux Filesystem Hierarchy Standard (FHS) instinctively:
* `/etc`: Host configurations, user accounts (`/etc/passwd`), password hashes (`/etc/shadow`), network settings (`/etc/network/interfaces`), and scheduled jobs (`/etc/crontab`).
* `/var/log`: Centralized system event logs:
  * `/var/log/auth.log` (Debian/Ubuntu) or `/var/log/secure` (RHEL/CentOS): User authentication logs, SSH sessions, sudo executions, and failed login bursts.
  * `/var/log/syslog` / `/var/log/messages`: Operating system kernel events and service status messages.
  * `/var/log/apache2/` or `/var/log/nginx/`: Web server access and error logs.
* `/proc`: Virtual pseudo-filesystem exposing real-time kernel memory, running process states (`/proc/<PID>/cmdline`, `/proc/<PID>/exe`), and network connections (`/proc/net/tcp`).

---

### 2. POSIX File Permissions & Access Controls

Every file and directory in Linux has an ownership model consisting of **User (Owner)**, **Group**, and **Others**:

```text
- r w x r - x r - -   (Octal: 754)
┬ ────┬──── ────┬──── ────┬────
│     │         │         └── Others (Read: 4)
│     │         └──────────── Group (Read: 4 + Execute: 1 = 5)
│     └────────────────────── User / Owner (Read: 4 + Write: 2 + Execute: 1 = 7)
└──────────────────────────── File Type (- = Regular file, d = Directory, l = Symlink)
```

#### Special Permission Bits:
* **SUID (Set User ID - Octal 4000)**: Executes the binary with the file owner's permissions (e.g., `chmod u+s /bin/ping`). *Risk*: Misconfigured SUID binaries are high-priority targets for local privilege escalation.
* **SGID (Set Group ID - Octal 2000)**: Executes with group permissions or enforces directory inheritance.
* **Sticky Bit (Octal 1000)**: Prevents users from deleting or renaming files in shared directories unless they own them (e.g., `/tmp` has permissions `drwxrwxrwt` or `1777`).

#### Auditing SUID Binaries via Terminal:
```bash
find / -perm -u=s -type f 2>/dev/null
```

---

### 3. Pipeline Filtering & Log Parsing Workflows

Real-world incident investigation requires chaining standard UNIX utilities:
```bash
# Extract the top 10 external IP addresses generating failed SSH logins
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr | head -n 10

# Search for world-writable directories that could indicate malicious staging
find / -type d -perm -0002 -ls 2>/dev/null

# Monitor live authorization attempts in real time
tail -f /var/log/auth.log | grep -E "Accepted|Failed"
```

---

## Technical Deep Dive: SQL for Security Analysts

Security analysts frequently query relational databases (PostgreSQL, MySQL, SQLite) to investigate transaction histories, user access privileges, and security event logs.

### 1. Essential SQL Query Construction

```sql
-- Query all failed login attempts from external networks over the past 24 hours
SELECT 
    login_id,
    user_name,
    ip_address,
    login_timestamp,
    failure_reason
FROM 
    employee_auth_logs
WHERE 
    status = 'FAILED'
    AND login_timestamp >= NOW() - INTERVAL '24 HOURS'
    AND ip_address NOT LIKE '192.168.%'
ORDER BY 
    login_timestamp DESC;
```

### 2. Aggregation & Anomaly Detection Queries

```sql
-- Detect potential brute-force attacks: group by IP and flag >= 10 failures
SELECT 
    ip_address,
    COUNT(*) AS total_failed_attempts,
    MIN(login_timestamp) AS first_attempt,
    MAX(login_timestamp) AS last_attempt
FROM 
    user_logins
WHERE 
    login_status = 'FAIL'
GROUP BY 
    ip_address
HAVING 
    COUNT(*) >= 10
ORDER BY 
    total_failed_attempts DESC;
```

### 3. Multi-Table Relational Correlation (JOINs)

```sql
-- Correlate employees with high-privilege administrative access rights
SELECT 
    e.employee_id,
    e.full_name,
    e.department,
    r.role_name,
    r.permission_level
FROM 
    employees e
INNER JOIN 
    user_roles ur ON e.employee_id = ur.employee_id
INNER JOIN 
    roles r ON ur.role_id = r.role_id
WHERE 
    r.permission_level = 'SUPERADMIN'
    AND e.employment_status = 'TERMINATED'; -- Flags orphaned accounts!
```

---

## Hands-On Labs Completed

1. **Investigating Unauthorized File Access in Linux**:
   * Evaluated access permissions across `/var/secure_docs/`.
   * Discovered insecure `777` permissions permitting global read/write on sensitive financial files.
   * Remediated file modes using `chmod 640` and updated group ownership to `finance_auditors` using `chown`.
2. **Database Forensic Extraction**:
   * Analyzed simulated server access records in a relational database.
   * Constructed multi-clause SQL queries combining `LIKE`, `IN`, and date range filters to isolate exfiltrated customer records and map them back to compromised session tokens.

---

## Key Takeaways

1. **CLI Speed & Precision**: The Linux command line enables immediate triage of gigabytes of log files using lightweight pipelines without relying on heavy graphical software.
2. **Principle of Least Privilege (PoLP)**: System security begins with strict file ownership; auditing permissions and SUID bits eliminates trivial attack paths.
3. **Data Literacy**: SQL enables security analysts to interrogate enterprise databases directly, correlating disparate security events into coherent incident timelines.
