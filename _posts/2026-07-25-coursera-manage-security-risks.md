---
title: "Google Cybersecurity: Play It Safe — Manage Security Risks"
author: parthpandey
categories: [Coursera, Google-Cybersecurity]
tags: [course-notes, risk-management, nist, security-frameworks, cissp-domains]
render_with_liquid: false
date: 2026-07-25 11:30:00 +0530
---

## Course Overview

| Detail | Value |
|---|---|
| **Platform** | Coursera |
| **Certificate** | Google Cybersecurity Professional Certificate |
| **Course** | 2 of 8 — Play It Safe: Manage Security Risks |
| **Duration** | ~11 hours |
| **Status** | ✅ Completed |

---

## Executive Summary

Course 2 of the Google Cybersecurity Professional Certificate establishes the operational bedrock of enterprise security: risk governance, security architecture alignment, and control frameworks. In this course, I mastered the **8 CISSP Security Domains**, analyzed compliance requirements across industry standards (GDPR, HIPAA, PCI-DSS), mapped defense-in-depth controls to **NIST CSF 2.0** and **NIST SP 800-37 (RMF)**, and practiced formal security auditing and incident response triage.

---

## Core Frameworks & Governance Methodologies

### 1. The 8 CISSP Security Domains
Security is not merely a collection of firewall rules; it spans eight interconnected domains that govern an organization:
1. **Security and Risk Management**: Establishing enterprise governance, compliance, threat modeling, and business continuity.
2. **Asset Security**: Data classification (Public, Internal, Confidential, Restricted), data handling lifecycle, and secure data sanitization (crypto-shredding, degaussing, physical disintegration).
3. **Security Architecture and Engineering**: Implementing models such as Bell-LaPadula (Confidentiality: *No Read Up, No Write Down*) and Biba (Integrity: *No Read Down, No Write Up*), Zero Trust Architecture (ZTA), and cryptographic subsystems.
4. **Communication and Network Security**: Network segmentation, DMZ design, multi-zone perimeter defense, and secure communications protocols (TLS 1.3, IPsec).
5. **Identity and Access Management (IAM)**: Authentication (MFA, FIDO2/WebAuthn), authorization models (DAC, MAC, RBAC, ABAC), and lifecycle identity governance.
6. **Security Assessment and Testing**: Vulnerability scanning, penetration testing, configuration audits, and code security evaluations.
7. **Security Operations (SecOps)**: Centralized SIEM monitoring, log ingestion, threat hunting, forensic acquisition, and incident response playbooks.
8. **Software Development Security**: Secure SDLC, DevSecOps pipelines, OWASP Top 10 mitigation, and API security verification.

---

### 2. The CIA Triad & Quantitative Risk Formulas

Every security control protects one or more pillars of the **CIA Triad**:
* **Confidentiality**: Ensuring data is accessible only to authorized entities via AES-256 encryption at rest, TLS in transit, and least-privilege ACLs.
* **Integrity**: Guaranteeing data accuracy and completeness without unauthorized tampering, verified through SHA-256 hashing, HMACs, and digital signatures.
* **Availability**: Providing reliable access to services and systems through redundant clustering, cloud load balancing, DDoS mitigation, and robust disaster recovery plans.

#### Quantitative Risk Assessment Math:
$$\text{Single Loss Expectancy (SLE)} = \text{Asset Value (AV)} \times \text{Exposure Factor (EF)}$$
$$\text{Annualized Loss Expectancy (ALE)} = \text{SLE} \times \text{Annualized Rate of Occurrence (ARO)}$$

By calculating ALE before and after implementing a defensive control, security analysts determine the **Cost-Benefit Analysis (CBA)**:
$$\text{CBA} = \text{ALE}_{\text{prior}} - \text{ALE}_{\text{post}} - \text{Annual Cost of Control}$$

---

### 3. NIST CSF 2.0 & NIST RMF Comparison

```text
┌────────────────────────────────────────────────────────────────────────┐
│                        NIST CSF 2.0 CORE FUNCTIONS                     │
├────────────┬───────────────────────────────────────────────────────────┤
│ GOVERN     │ Organizational context, risk management strategy, policies│
│ IDENTIFY   │ Physical & software asset inventory, risk assessments     │
│ PROTECT    │ Identity management, access control, data protection, trng│
│ DETECT     │ Continuous monitoring, anomalous activity detection (SIEM)│
│ RESPOND    │ Incident analysis, mitigation, containment, communications│
│ RECOVER    │ Restoration of operations, post-incident lessons learned  │
└────────────┴───────────────────────────────────────────────────────────┘
```

#### NIST SP 800-37 Risk Management Framework (RMF) Lifecycle:
1. **Prepare**: Set organizational context and risk tolerances.
2. **Categorize**: Classify systems and processed data based on impact levels (Low, Moderate, High).
3. **Select**: Choose baseline NIST SP 800-53 security controls.
4. **Implement**: Deploy technical, operational, and managerial controls.
5. **Assess**: Test controls to verify correct operation and intended outcomes.
6. **Authorize**: Official senior management sign-off (Authorization to Operate - ATO).
7. **Monitor**: Continuous assessment of controls and environmental changes.

---

## Practical Labs & Case Studies

### Lab 1: Comprehensive Internal Security Audit
* **Scenario**: Conducted a simulated security compliance audit for an e-commerce retailer migrating customer payment records to the cloud.
* **Identified Vulnerabilities**:
  1. Unencrypted cardholder data stored in log files (violating PCI-DSS Req 3.4).
  2. Broad administrative permissions assigned without MFA challenge (violating NIST CSF PR.AC-1).
  3. Absence of centralized log aggregation preventing intrusion visibility (violating NIST CSF DE.CM-1).
* **Remediation Delivered**: Created a prioritized corrective action plan detailing automated tokenization, hardware MFA enforcement, and CloudWatch log forwarding to a central SIEM.

### Lab 2: Incident Response Playbook Development
* **Framework**: NIST SP 800-61 Rev. 2 (Computer Security Incident Handling Guide).
* **Target Scenario**: Unauthorized external brute force targeting cloud administrative accounts.
* **Execution**: Drafted step-by-step triage playbooks:
  1. *Detection*: Trigger alert when $>5$ authentication failures occur in 5 minutes.
  2. *Containment*: Temporarily suspend compromised IAM access keys and revoke active STS sessions.
  3. *Eradication*: Audit CloudTrail logs for persistence mechanisms (unauthorized IAM users, backdoored security groups).
  4. *Recovery*: Rotate master credentials, enforce hardware FIDO2 keys, and verify system integrity.
  5. *Lessons Learned*: Incorporate geo-blocking and automated IP blacklisting into WAF rules.

---

## Key Takeaways & Practical Impact

1. **Risk Is a Business Decision**: Technical vulnerabilities must be translated into business risk (SLE/ALE) so leadership can allocate appropriate budgets for remediation.
2. **Defense-in-Depth**: Relying solely on perimeter firewalls fails against insider threats or compromised credentials; layered controls (technical, administrative, physical) are mandatory.
3. **Continuous Compliance**: Security is an ongoing cycle, not an annual checklist; continuous telemetry and SIEM auditing form the backbone of modern SecOps.
