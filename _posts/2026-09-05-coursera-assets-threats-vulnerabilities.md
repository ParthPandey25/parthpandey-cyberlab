---
title: "Google Cybersecurity: Assets, Threats, and Vulnerabilities"
author: parthpandey
categories: [Coursera, Google-Cybersecurity]
tags: [course-notes, assets-threats, vulnerability-assessment, threat-modeling, cryptography, authentication]
render_with_liquid: false
date: 2026-09-05 10:30:00 +0530
---

## Course Overview

| Detail | Value |
|---|---|
| **Platform** | Coursera |
| **Certificate** | Google Cybersecurity Professional Certificate |
| **Course** | 5 of 8 — Assets, Threats, and Vulnerabilities |
| **Duration** | ~25 hours |
| **Status** | ✅ Completed |

---

## Executive Summary

Course 5 covers the tactical identification of organizational assets, assessment of vulnerabilities, threat modeling against adversary tactics, and the application of cryptographic algorithms and authentication protocols. In this course, I analyzed the vulnerability management lifecycle, calculated **CVSS v3.1** vector scores, practiced structured threat modeling using **STRIDE** and **PASTA**, examined malware categories and evasion tactics, and evaluated symmetric vs. asymmetric cryptographic primitives.

---

## Technical Foundations

### 1. Asset Classification & The Vulnerability Management Lifecycle

An **asset** is any data, personnel, device, software, or facility that possesses value to an organization.

```text
┌────────────────────────────────────────────────────────────────────────┐
│                   VULNERABILITY MANAGEMENT LIFECYCLE                   │
├────────────────────────────────────────────────────────────────────────┤
│  1. DISCOVER     │ Continuous inventory of all network & cloud assets  │
│  2. PRIORITIZE   │ Categorize assets by business criticality (CVSS)    │
│  3. ASSESS       │ Automated scanning (Nessus, OpenVAS) & manual audits│
│  4. REPORT       │ Document risk level, attack surface & PoC evidence  │
│  5. REMEDIATE    │ Patching, configuration hardening, or compensating  │
│                  │ controls (WAF, network isolation)                   │
│  6. VERIFY       │ Re-scan to validate complete vulnerability closure  │
└────────────────────────────────────────────────────────────────────────┘
```

#### Common Vulnerability Scoring System (CVSS v3.1):
CVSS produces a numerical severity score from **0.0 to 10.0**:
* **None**: `0.0`
* **Low**: `0.1 – 3.9`
* **Medium**: `4.0 – 6.9`
* **High**: `7.0 – 8.9`
* **Critical**: `9.0 – 10.0`

Key Base Metric Metrics:
* **Attack Vector (AV)**: Network (N), Adjacent (A), Local (L), Physical (P).
* **Attack Complexity (AC)**: Low (L), High (H).
* **Privileges Required (PR)**: None (N), Low (L), High (H).
* **User Interaction (UI)**: None (N), Required (R).
* **Scope (S)**: Unchanged (U), Changed (C).
* **Impact**: Confidentiality (C), Integrity (I), Availability (A).

---

### 2. Threat Modeling Frameworks: STRIDE vs. PASTA

Threat modeling systematically identifies what can go wrong before an attacker finds it.

#### STRIDE Methodology (Developed by Microsoft):
| Threat Category | Security Property Violated | Real-World Example | Primary Defense |
|---|---|---|---|
| **S**poofing | Authenticity | Forging email headers or IP addresses | MFA, digital certificates, SPF/DKIM/DMARC |
| **T**ampering | Integrity | Altering data in transit or database fields | SHA-256 HMAC, parameter validation, TLS |
| **R**epudiation | Non-Repudiation | Denying unauthorized wire transfer | Audit logging, append-only logs, signatures |
| **I**nformation Disclosure | Confidentiality | Leaking credit card numbers in error logs | AES-256 encryption, data masking, strict ACLs |
| **D**enial of Service | Availability | SYN flooding or application CPU exhaustion | Rate limiting, Cloud Armor, CDN caching |
| **E**levation of Privilege | Authorization | Standard user escalating to root/admin | PoLP, secure code audits, patched kernels |

#### PASTA (Process for Attack Simulation and Threat Analysis):
A risk-centric 7-stage methodology that aligns technical threat findings directly with corporate business objectives.

---

### 3. Cryptography & Authentication (AAA) Architecture

```text
               CRYPTOGRAPHIC PRIMITIVES TAXONOMY
                               │
         ┌─────────────────────┼─────────────────────┐
         ▼                     ▼                     ▼
   Symmetric Key        Asymmetric Key        Cryptographic
     Encryption           Encryption             Hashing
  (AES-GCM, ChaCha20)    (RSA, ECC/ECDSA)    (SHA-256, SHA-3)
         │                     │                     │
  - Single secret key   - Public/Private pair - One-way function
  - Ultra-fast          - Key exchange,       - Fixed-length digest
  - Bulk data at rest/   digital signatures   - File integrity,
    in transit          - Slower computation    password storage
```

#### Authentication, Authorization, and Accounting (AAA):
1. **Authentication**: Verifying the claimant's identity (Something you know, have, or are).
   * Multi-Factor Authentication (MFA): Combining $\ge 2$ distinct factors (e.g., Password + Hardware FIDO2 Security Key).
2. **Authorization**: Granting specific access privileges based on policy (RBAC, ABAC).
3. **Accounting**: Recording session activity and resource consumption in audit logs.

---

## Practical Labs & Exercises

### Lab 1: Threat Modeling an E-Commerce Web Architecture
* Applied the **STRIDE** methodology to a multi-tier web application consisting of a React client, Node.js API gateway, and PostgreSQL database.
* Identified threats:
  * *Tampering*: Unvalidated client-side cart pricing allowed negative totals. (Remediated via server-side schema validation).
  * *Information Disclosure*: Database error messages leaked internal table structures. (Remediated via global custom error middleware).
  * *Elevation of Privilege*: Standard users could edit `/api/users/<id>` to set `role: "admin"`. (Remediated via strict DTO white-listing and `@PreAuthorize` guards).

### Lab 2: Cryptographic Analysis & Hash Verification
* Computed and verified MD5, SHA-1, and SHA-256 checksums across firmware update packages.
* Analyzed collision attacks on legacy algorithms (MD5/SHA-1) and configured production cipher suites enforcing forward secrecy (ECDHE-RSA-AES256-GCM-SHA384).

---

## Key Takeaways

1. **Prioritization Over Exhaustion**: Thousands of vulnerabilities exist; calculating true CVSS risk and exploitability (EPSS) ensures patching focuses on high-impact exposures first.
2. **Shift Left with Threat Modeling**: Incorporating STRIDE during system design catches structural architecture flaws before a single line of vulnerable code is deployed.
3. **Defense-in-Depth Cryptography**: Encryption at rest protects physical media theft; encryption in transit prevents eavesdropping; salted hashing protects authentication databases.
