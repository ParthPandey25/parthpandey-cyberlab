# 🛡️ Parth Pandey's CyberLab & Security Research Blog

> **Personal Cybersecurity Research Journal & CTF Walkthrough Archive**: Built using the [Jekyll Chirpy Theme](https://github.com/cotes2020/jekyll-theme-chirpy), deployed automatically via GitHub Actions to GitHub Pages. Contains detailed vulnerability assessments, penetration testing walkthroughs, defensive telemetry engineering notes, and tool troubleshooting breakdowns.

[![Jekyll](https://img.shields.io/badge/Jekyll-Chirpy-red?logo=jekyll&logoColor=white)](https://jekyllrb.com)
[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Deploy%20Active-brightgreen?logo=github&logoColor=white)](https://parthpandey25.github.io/parthpandey-cyberlab/)
[![TryHackMe](https://img.shields.io/badge/TryHackMe-Top%202%25%20%5BGURU%5D-red?logo=tryhackme&logoColor=white)](https://tryhackme.com/p/parthrspandey25)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## 🌐 Live Blog Website
- **Live URL**: [https://parthpandey25.github.io/parthpandey-cyberlab](https://parthpandey25.github.io/parthpandey-cyberlab)
- **Portfolio**: [https://portfolio-parthpandey.vercel.app](https://portfolio-parthpandey.vercel.app)

---

## 📚 Published Walkthroughs & Technical Notes

### 🎯 TryHackMe & Practical CTF Exploitation
1. **[Confessions of a Top 2% TryHackMe Practitioner: Where Tools Break](_posts/2026-10-22-confessions-of-a-tryhackme-practitioner-where-tools-break.md)**: Deep dive into real-world tool failures (850k dictionary freeze, dumb TTY trap, vanishing Meterpreter sessions, Windows root DACLs).
2. **[TryHackMe: Blue (EternalBlue MS17-010 Exploitation)](_posts/2026-09-24-tryhackme-blue-eternalblue.md)**: Kernel pool spraying, SMBv1 remote code execution, Meterpreter process migration to `spoolsv.exe`, and SAM database dumping.
3. **[TryHackMe: Mr. Robot CTF — Web Recon & SUID Escapes](_posts/2026-09-25-tryhackme-mr-robot-ctf.md)**: Web discovery in `robots.txt`, dictionary pruning via `sort -u`, WordPress theme shell injection, and legacy SUID Nmap interactive escape.
4. **[TryHackMe: Ice — CVE-2014-0759 Buffer Overflow](_posts/2026-09-26-tryhackme-ice-cve-2014-0759.md)**: Exploiting Icecast HTTP header buffer overflow, UAC bypass, and Kiwi (Mimikatz) cleartext LSASS memory credential harvesting.
5. **[TryHackMe: Basic Pentesting Walkthrough](_posts/2026-09-23-tryhackme-basic-pentesting.md)**: End-to-end beginner penetration testing methodology using Gobuster, Hydra, and John the Ripper.
6. **[Windows Privilege Escalation Guide](_posts/2026-09-27-windows-privilege-escalation-mastery.md)**: Service DACLs (`accesschk.exe`), unquoted service paths, `AlwaysInstallElevated`, and `SeImpersonatePrivilege` token abuse (PrintSpoofer).

### 🔍 Vulnerability Assessments & Hands-On Labs
7. **[Metasploitable Port-by-Port Vulnerability Assessment](_posts/2026-09-23-lab-nmap-network-scanning.md)**: Systematic Nmap port sweep, NSE vulnerability auditing, and CVE mapping across legacy protocols (FTP 21, Telnet 23, SMB 139/445).

### 🎓 Google Cybersecurity Professional Certificate Notes
8. **Foundations of Cybersecurity** (`_posts/2026-09-23-coursera-foundations-of-cybersecurity.md`)
9. **Assets, Threats, and Vulnerabilities** (`_posts/2026-09-23-coursera-assets-threats-vulnerabilities.md`)
10. **Networks and Network Security** (`_posts/2026-09-23-coursera-networks-and-network-security.md`)
11. **Tools of the Trade: Linux and SQL** (`_posts/2026-09-23-coursera-linux-and-sql.md`)
12. **Play It Safe: Manage Security Risks** (`_posts/2026-09-23-coursera-manage-security-risks.md`)

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
- **Email**: parthrspandey25@gmail.com
- **LinkedIn**: [linkedin.com/in/parth-pandey-045709258](https://linkedin.com/in/parth-pandey-045709258)
- **TryHackMe Profile**: [tryhackme.com/p/parthrspandey25](https://tryhackme.com/p/parthrspandey25)
