---
title: "Google Cybersecurity: Automate Cybersecurity Tasks with Python (Log Parsing, Regex & IP Filter Automation)"
author: parthpandey
categories: [Coursera, Google-Cybersecurity]
tags: [python, automation, scripting, regex, file-io, ip-filtering, soc, devsecops]
render_with_liquid: false
date: 2026-09-10 14:30:00 +0530
---

**Automate Cybersecurity Tasks with Python** is the 7th course in the Google Cybersecurity Professional Certificate. In an enterprise SOC, manual log review across millions of events is mathematically impossible. This course focuses on developing robust Python automation scripts to parse security logs, extract network artifacts via regular expressions, and programmatically maintain firewall allowlists and blocklists.

---

## Course Information

| Attribute | Specification |
|:---|:---|
| **Platform** | Coursera |
| **Certificate** | Google Cybersecurity Professional Certificate |
| **Course** | 7 of 8 — Automate Cybersecurity Tasks with Python |
| **Status** | ✅ Completed |
| **Core Topics** | Python Data Structures, File I/O (`with open`), String Manipulation, Regex (`re` module), Algorithm Optimization |

---

## 1. Project Case: Automated IP Access Control List (ACL) Updater

### The Problem Statement
A critical corporate server maintains an approved IP allowlist file (`allow_list.txt`). Due to employees rotating off assignments or suspicious behavior flagged by our SIEM, security analysts must regularly remove a list of revoked IP addresses (`remove_list`) from the file without manual copy-pasting or file corruption risks.

### Python Algorithm Implementation

```python
"""
Security Automation Script: Ingests an existing IP allowlist, filters out
revoked IP addresses, and safely overwrites the production file.
"""

def update_allow_list(import_file: str, remove_list: list[str]) -> None:
    # 1. Safely open the file using a context manager
    with open(import_file, "r") as file:
        # 2. Read entire contents into a string variable
        ip_addresses = file.read()

    # 3. Convert whitespace-separated string into a mutable Python list
    ip_list = ip_addresses.split()

    # 4. Iterate through the revoked IP list
    for ip in remove_list:
        # 5. Remove matching occurrences while checking membership
        while ip in ip_list:
            ip_list.remove(ip)

    # 6. Reconstruct the clean, newline-delimited string
    updated_ips = "\n".join(ip_list)

    # 7. Atomically write the sanitized data back to the file
    with open(import_file, "w") as file:
        file.write(updated_ips)
    
    print(f"[+] Successfully purged revoked entries. Clean count: {len(ip_list)} IPs.")

# Production Execution Example
if __name__ == "__main__":
    revoked_ips = ["192.168.1.105", "10.0.4.15", "172.16.2.88"]
    update_allow_list("allow_list.txt", revoked_ips)
```

### Key Python Concepts Leveraged:
- **`with open()` Context Manager**: Guarantees file descriptors are cleanly closed even if an unexpected exception occurs during runtime.
- **`.split()` and `.join()`**: Translates raw file text into iterable array tokens and reconstructs clean formatting with zero memory leaks.
- **Defensive Membership Check (`while ip in ip_list`)**: Prevents `ValueError` exceptions when attempting to remove elements that may appear multiple times or not exist.

---

## 2. Advanced Threat Hunting with Regular Expressions (`re`)

Log files (such as Apache, Nginx, or syslog) are rarely structured as neat single tokens. Extracting Indicators of Compromise (IoCs) requires regex pattern compilation.

### Extracting Malicious IP Addresses & Status Codes

```python
import re

# Simulated Apache Access Log Sample
raw_log = """
192.168.1.50 - - [10/Sep/2026:12:01:14] "GET /admin/login.php HTTP/1.1" 401 234
203.0.113.195 - - [10/Sep/2026:12:01:15] "POST /api/v1/auth HTTP/1.1" 200 452
198.51.100.82 - - [10/Sep/2026:12:01:19] "GET /../../etc/passwd HTTP/1.1" 403 180
10.0.0.1 - - [10/Sep/2026:12:01:22] "GET /index.html HTTP/1.1" 200 2315
"""

# Regex pattern for IPv4 addresses: 4 octets (0-255)
ip_pattern = r"\b(?:[0-9]{1,3}\.){3}[0-9]{1,3}\b"
extracted_ips = re.findall(ip_pattern, raw_log)
print("Discovered Network Endpoints:", extracted_ips)

# Regex pattern to identify directory traversal attack attempts
traversal_pattern = r"(?:\.\.\/|\.\.\\)"
flagged_lines = [line for line in raw_log.strip().split("\n") if re.search(traversal_pattern, line)]

print("\n[!] Directory Traversal Incidents Detected:")
for alert in flagged_lines:
    print(" ->", alert)
```

---

## 3. Engineering Best Practices for Security Scripts

When authoring automation tools for security operations:
1. **Never Hardcode Credentials**: Store API tokens, database URIs, and webhook secrets in environment variables (`os.getenv()`) or secrets managers.
2. **Implement Structured Logging**: Replace raw `print()` statements with Python's native `logging` module to output timestamped, leveled logs (`INFO`, `WARNING`, `CRITICAL`).
3. **Handle Exceptions Gracefully**: Wrap network socket calls and file parsing in `try...except` blocks to prevent daemon crashes during active incidents.

Python is the universal glue of modern cybersecurity. Writing concise, maintainable automation scripts separates reactive analysts from proactive security engineers.
