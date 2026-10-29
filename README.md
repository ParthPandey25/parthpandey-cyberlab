# 🛡️ Parth Pandey's CyberLab & Security Research Blog

> **Personal Cybersecurity Research Journal, Threat Hunting Case Studies & CTF Walkthrough Archive**: Built using the [Jekyll Chirpy Theme](https://github.com/cotes2020/jekyll-theme-chirpy), deployed automatically via GitHub Actions to GitHub Pages. Contains 23 hands-on penetration testing case studies, SOC/SIEM threat hunting queries, cryptographic breakdowns, and practical offensive security research.

[![Jekyll](https://img.shields.io/badge/Jekyll-Chirpy-red?logo=jekyll&logoColor=white)](https://jekyllrb.com)
[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Deploy%20Active-brightgreen?logo=github&logoColor=white)](https://parthpandey25.github.io/parthpandey-cyberlab/)
[![TryHackMe](https://img.shields.io/badge/TryHackMe-Top%202%25%20%5BGURU%5D-red?logo=tryhackme&logoColor=white)](https://tryhackme.com/p/parthrspandey25)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## 🌐 Live Blog Website
- **Live URL**: [https://parthpandey25.github.io/parthpandey-cyberlab](https://parthpandey25.github.io/parthpandey-cyberlab)
- **Categories**: [https://parthpandey25.github.io/parthpandey-cyberlab/categories/](https://parthpandey25.github.io/parthpandey-cyberlab/categories/)
- **Main Portfolio**: [https://portfolio-parthpandey.vercel.app](https://portfolio-parthpandey.vercel.app)

---

## 📚 Published Walkthroughs & Technical Notes (23 Articles)

### 🔵 SOC Operations, SIEM & Defensive Security
1. **[SOC Operations: Splunk Enterprise Threat Hunting — Correlating MITRE ATT&CK & Authoring Production SPL](_posts/2026-10-24-soc-splunk-siem-live-incident-investigation.md)**: Production SPL queries for brute force (4625), encoded PowerShell (4104), LSASS memory dumping (Sysmon 10), and DNS C2 tunneling.
2. **[Google Cybersecurity: Sound the Alarm — Detection and Response](_posts/2026-09-08-coursera-sound-the-alarm-detection-and-response.md)**: SIEM fundamentals, Chronicle UDM, Suricata rule authoring, and NIST SP 800-61 Rev. 2 incident handling.
3. **[Google Cybersecurity: Automate Cybersecurity Tasks with Python](_posts/2026-09-10-coursera-automate-cybersecurity-tasks-with-python.md)**: Automating log parsing, regex pattern matching, and automated firewall IP allowlist/blocklist algorithms.
4. **[Metasploitable Port-by-Port Vulnerability Assessment](_posts/2026-09-12-lab-nmap-network-scanning.md)**: Systematic Nmap port sweep, NSE vulnerability auditing, and CVE mapping across legacy services.
5. **[Applied Cryptography: AES-GCM, RSA Math, Diffie-Hellman Key Exchange & TLS 1.3](_posts/2026-09-15-applied-cryptography-symmetric-asymmetric-ciphers.md)**: Mathematical foundations of public-key cryptography, block cipher modes, and forward secrecy.

### 🔴 TryHackMe Offensive CTF Case Studies & Walkthroughs
6. **[Confessions of a Top 2% TryHackMe Practitioner: Where Tools Break](_posts/2026-10-26-confessions-of-a-tryhackme-practitioner-where-tools-break.md)**: Deep dive into real-world tool failures (850k dictionary freeze, dumb TTY trap, vanishing Meterpreter sessions, Windows root DACLs).
7. **[TryHackMe: Steel Mountain — Rejetto HFS RCE & Unquoted Service Path](_posts/2026-10-22-tryhackme-steel-mountain-rejetto-unquoted-path.md)**: OSINT employee analysis, null-byte macro execution, PowerUp.ps1 audit, and weaponizing unquoted service paths to `SYSTEM`.
8. **[TryHackMe: Pickle Rick — Command Injection Filter Bypasses & Sudo PrivEsc](_posts/2026-10-18-tryhackme-pickle-rick-ctf-case-study.md)**: HTML source recon, `robots.txt` credential exposure, command blacklist circumvention with `less`/`awk`, and `(ALL) NOPASSWD` root escalation.
9. **[TryHackMe: Vulnversity — File Upload Bypass & systemctl PrivEsc](_posts/2026-10-14-tryhackme-vulnversity-file-upload-systemctl-privesc.md)**: Directory bruteforce with GoBuster, bypassing upload filters via `.phtml`, and weaponizing SUID `/bin/systemctl` with custom systemd service units.
10. **[TryHackMe: Ignite — Weaponizing Fuel CMS 1.4.1 RCE (CVE-2018-16763)](_posts/2026-10-11-tryhackme-ignite-fuel-cms-rce.md)**: Pre-auth PHP code evaluation in Fuel CMS, database configuration credential harvesting, and root password reuse.
11. **[TryHackMe: Alfred — Jenkins Groovy RCE & Token Impersonation](_posts/2026-10-09-tryhackme-alfred-jenkins-token-impersonation.md)**: Jenkins administrative compromise, Nishang PowerShell reverse shell, and `SeImpersonatePrivilege` token theft via Incognito to `SYSTEM`.
12. **[TryHackMe: RootMe — Apache File Upload Bypass & SUID Python PrivEsc](_posts/2026-10-06-tryhackme-rootme-file-upload-python-privesc.md)**: Apache file extension fuzzing (`.phtml`), Netcat ingress, and GTFOBins SUID `/usr/bin/python` privilege escalation.
13. **[TryHackMe: Kenobi — Samba, ProFTPD mod_copy (CVE-2015-3306) & SUID Path Hijacking](_posts/2026-10-04-tryhackme-kenobi-samba-proftpd-privesc.md)**: Enumerating Samba/NFS exports, unauthenticated file copying with ProFTPD, and `$PATH` hijacking on custom SUID `/usr/bin/menu`.
14. **[Windows Privilege Escalation Guide](_posts/2026-09-30-windows-privilege-escalation-mastery.md)**: Service DACLs (`accesschk.exe`), unquoted service paths, `AlwaysInstallElevated`, and `SeImpersonatePrivilege` token abuse (PrintSpoofer).
15. **[TryHackMe: Mr. Robot CTF — Web Recon & SUID Escapes](_posts/2026-09-28-tryhackme-mr-robot-ctf.md)**: Web discovery in `robots.txt`, dictionary pruning via `sort -u`, WordPress theme shell injection, and legacy SUID Nmap interactive escape.
16. **[TryHackMe: Ice — CVE-2014-0759 Buffer Overflow](_posts/2026-09-26-tryhackme-ice-cve-2014-0759.md)**: Exploiting Icecast HTTP header buffer overflow, UAC bypass, and Kiwi (Mimikatz) cleartext LSASS memory credential harvesting.
17. **[TryHackMe: Blue (EternalBlue MS17-010 Exploitation)](_posts/2026-09-22-tryhackme-blue-eternalblue.md)**: Kernel pool spraying, SMBv1 remote code execution, Meterpreter process migration to `spoolsv.exe`, and SAM database dumping.
18. **[TryHackMe: Basic Pentesting Walkthrough](_posts/2026-09-18-tryhackme-basic-pentesting.md)**: End-to-end beginner penetration testing methodology using Gobuster, Hydra, and John the Ripper.

### 🎓 Google Cybersecurity Professional Certificate Curriculum
19. **Assets, Threats, and Vulnerabilities** (`_posts/2026-09-05-coursera-assets-threats-vulnerabilities.md`)
20. **Tools of the Trade: Linux and SQL** (`_posts/2026-08-22-coursera-linux-and-sql.md`)
21. **Networks and Network Security** (`_posts/2026-08-08-coursera-networks-and-network-security.md`)
22. **Play It Safe: Manage Security Risks** (`_posts/2026-07-25-coursera-manage-security-risks.md`)
23. **Foundations of Cybersecurity** (`_posts/2026-07-12-coursera-foundations-of-cybersecurity.md`)

---

## 🚀 Local Development & Testing

```bash
# Clone the repository
git clone https://github.com/ParthPandey25/parthpandey-cyberlab.git
cd parthpandey-cyberlab

# Install ruby dependencies
bundle install

# Run local development server
bundle exec jekyll serve
```

Access the local server at `http://127.0.0.1:4000/parthpandey-cyberlab/`.

---

## 📬 Contact & Links

- **Author**: Parth Pandey ([@ParthPandey25](https://github.com/ParthPandey25))
- **TryHackMe**: [https://tryhackme.com/p/parthrspandey25](https://tryhackme.com/p/parthrspandey25) (Top 2% [GURU], 115+ Rooms, 122-Day Streak)
- **LinkedIn**: [https://www.linkedin.com/in/parth-pandey-045709258](https://www.linkedin.com/in/parth-pandey-045709258)
