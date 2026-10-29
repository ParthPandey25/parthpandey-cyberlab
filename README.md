# 🛡️ Parth Pandey's CyberLab & Security Research Blog

> **Personal Cybersecurity Research Journal, Case Studies & CTF Walkthrough Archive**: Built using the [Jekyll Chirpy Theme](https://github.com/cotes2020/jekyll-theme-chirpy), deployed automatically via GitHub Actions to GitHub Pages. Contains hands-on penetration testing case studies, vulnerability assessments, offensive tooling breakdowns, and security engineering research.

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

## 📚 Published Walkthroughs & Technical Notes

### 🎯 TryHackMe Case Studies & Offensive Walkthroughs
1. **[Confessions of a Top 2% TryHackMe Practitioner: Where Tools Break](_posts/2026-10-26-confessions-of-a-tryhackme-practitioner-where-tools-break.md)**: Deep dive into real-world tool failures (850k dictionary freeze, dumb TTY trap, vanishing Meterpreter sessions, Windows root DACLs).
2. **[TryHackMe: Steel Mountain — Rejetto HFS RCE (CVE-2014-6287) & Unquoted Service Path](_posts/2026-10-22-tryhackme-steel-mountain-rejetto-unquoted-path.md)**: OSINT employee analysis, null-byte macro execution, PowerUp.ps1 audit, and weaponizing unquoted service paths to `SYSTEM`.
3. **[TryHackMe: Pickle Rick — Command Injection Filter Bypasses & Sudo PrivEsc](_posts/2026-10-18-tryhackme-pickle-rick-ctf-case-study.md)**: HTML source recon, `robots.txt` credential exposure, command blacklist circumvention with `less`/`awk`, and `(ALL) NOPASSWD` root escalation.
4. **[TryHackMe: Vulnversity — File Upload Bypass & systemctl PrivEsc](_posts/2026-10-14-tryhackme-vulnversity-file-upload-systemctl-privesc.md)**: Directory bruteforce with GoBuster, bypassing upload filters via `.phtml`, and weaponizing SUID `/bin/systemctl` with custom systemd service units.
5. **[TryHackMe: Alfred — Jenkins Groovy RCE & Token Impersonation](_posts/2026-10-09-tryhackme-alfred-jenkins-token-impersonation.md)**: Jenkins administrative compromise, Nishang PowerShell reverse shell, and `SeImpersonatePrivilege` token theft via Incognito to `SYSTEM`.
6. **[TryHackMe: Kenobi — Samba, ProFTPD mod_copy (CVE-2015-3306) & SUID Path Hijacking](_posts/2026-10-04-tryhackme-kenobi-samba-proftpd-privesc.md)**: Enumerating Samba/NFS exports, unauthenticated file copying with ProFTPD, and `$PATH` hijacking on custom SUID `/usr/bin/menu`.
7. **[Windows Privilege Escalation Guide](_posts/2026-09-30-windows-privilege-escalation-mastery.md)**: Service DACLs (`accesschk.exe`), unquoted service paths, `AlwaysInstallElevated`, and `SeImpersonatePrivilege` token abuse (PrintSpoofer).
8. **[TryHackMe: Mr. Robot CTF — Web Recon & SUID Escapes](_posts/2026-09-28-tryhackme-mr-robot-ctf.md)**: Web discovery in `robots.txt`, dictionary pruning via `sort -u`, WordPress theme shell injection, and legacy SUID Nmap interactive escape.
9. **[TryHackMe: Ice — CVE-2014-0759 Buffer Overflow](_posts/2026-09-26-tryhackme-ice-cve-2014-0759.md)**: Exploiting Icecast HTTP header buffer overflow, UAC bypass, and Kiwi (Mimikatz) cleartext LSASS memory credential harvesting.
10. **[TryHackMe: Blue (EternalBlue MS17-010 Exploitation)](_posts/2026-09-22-tryhackme-blue-eternalblue.md)**: Kernel pool spraying, SMBv1 remote code execution, Meterpreter process migration to `spoolsv.exe`, and SAM database dumping.
11. **[TryHackMe: Basic Pentesting Walkthrough](_posts/2026-09-18-tryhackme-basic-pentesting.md)**: End-to-end beginner penetration testing methodology using Gobuster, Hydra, and John the Ripper.

### 🔍 Vulnerability Assessments & Hands-On Labs
12. **[Metasploitable Port-by-Port Vulnerability Assessment](_posts/2026-09-12-lab-nmap-network-scanning.md)**: Systematic Nmap port sweep, NSE vulnerability auditing, and CVE mapping across legacy protocols (FTP 21, Telnet 23, SMB 139/445).

### 🎓 Google Cybersecurity Professional Certificate Notes
13. **Assets, Threats, and Vulnerabilities** (`_posts/2026-09-05-coursera-assets-threats-vulnerabilities.md`)
14. **Tools of the Trade: Linux and SQL** (`_posts/2026-08-22-coursera-linux-and-sql.md`)
15. **Networks and Network Security** (`_posts/2026-08-08-coursera-networks-and-network-security.md`)
16. **Play It Safe: Manage Security Risks** (`_posts/2026-07-25-coursera-manage-security-risks.md`)
17. **Foundations of Cybersecurity** (`_posts/2026-07-12-coursera-foundations-of-cybersecurity.md`)

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
