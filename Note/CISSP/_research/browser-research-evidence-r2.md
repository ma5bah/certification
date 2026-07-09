---
title: "Browser Research Evidence — Round 2 (Domains 2-8)"
layer: 0
type: source-evidence
date: 2026-07-08
sources_visited: 4
status: complete
---

# CISSP Browser Research — Round 2

## Sources Visited (Round 2)

| # | Source | URL | Key Finding |
|---|--------|-----|-------------|
| 5 | NIST SP 800-30 Rev 1 | `csrc.nist.gov/pubs/sp/800/30/r1/final` | Risk assessment guide, 3-tier hierarchy, supersedes 2002 version |
| 6 | OWASP Top 10:2025 | `owasp.org/Top10/2025/` | **OWASP 2025 is current** (not 2021). A10 changed from SSRF → "Mishandling of Exceptional Conditions" |
| 7 | NIST SP 800-61 Rev 3 | `csrc.nist.gov/pubs/sp/800/61/r3/final` | **Rev 2 withdrawn April 2025.** Rev 3 is a CSF 2.0 Community Profile |
| 8 | MITRE ATT&CK | `attack.mitre.org/` | **15 Enterprise tactics** (Stealth TA0005 and Defense Impairment TA0112 are now separate) |

## Finding 6: NIST SP 800-30 Rev 1 — Risk Assessment Guide

- **Published:** September 2012
- **Supersedes:** SP 800-30 (July 2002)
- **DOI:** 10.6028/NIST.SP.800-30r1
- **Purpose:** Guide for conducting risk assessments of federal information systems. Amplifies SP 800-39.
- **Risk assessment tiers:** Tier 1 (Organization), Tier 2 (Mission/Business), Tier 3 (Information System)
- **Keywords:** Cost-benefit analysis, residual risk, risk assessment, risk mitigation, security controls, threat vulnerability
- **Control Families:** Assessment/Authorization/Monitoring, Planning, Program Management, Risk Assessment, System and Services Acquisition

## Finding 7: OWASP Top 10:2025 — Current Version

**The most current released version is OWASP Top 10:2025**, not 2021.

### Top 10:2025 List (Verified)

| Rank | ID | Title | Change from 2021 |
|------|----|-------|-----------------|
| 1 | A01 | Broken Access Control | Same |
| 2 | A02 | Security Misconfiguration | Up from #5 |
| 3 | A03 | Software Supply Chain Failures | Renamed from "Vulnerable and Outdated Components" |
| 4 | A04 | Cryptographic Failures | Same |
| 5 | A05 | Injection | Down from #3 |
| 6 | A06 | Insecure Design | Same |
| 7 | A07 | Authentication Failures | Same |
| 8 | A08 | Software or Data Integrity Failures | Same |
| 9 | A09 | Security Logging and Alerting Failures | Same |
| 10 | A10 | **Mishandling of Exceptional Conditions** | **NEW — replaces SSRF from 2021** |

**Impact:** The existing note `owasp-top-10-and-web-app-testing.md` references 2021 categories and needs updating.

## Finding 8: NIST SP 800-61 Rev 3 — Incident Response

- **Rev 2 WITHDRAWN** on April 3, 2025
- **Rev 3 published:** April 2025, DOI 10.6028/NIST.SP.800-61r3
- **Title change:** "Incident Response Recommendations and Considerations for Cybersecurity Risk Management: A CSF 2.0 Community Profile"
- **Major shift:** Rev 3 is explicitly a **CSF 2.0 Community Profile**, not a standalone guide
- Maps IR activities to CSF 2.0 functions: Govern, Identify, Protect, Detect, Respond, Recover
- **Keywords:** cyber threat information sharing, CSF, cybersecurity risk management, incident handling, incident management, incident response
- **Authors:** Alexander Nelson (NIST), Sanjay Rekhi (NIST), Murugiah Souppaya (NIST), Karen Scarfone

**Impact:** The existing note `incident-response-lifecycle-nist.md` may reference the old 6-phase Rev 2 model. Rev 3 is CSF 2.0-aligned.

## Finding 9: MITRE ATT&CK — 15 Enterprise Tactics

**The Enterprise matrix now has 15 tactics** (previously 14 in older versions):

| # | Tactic | TA ID | Technique Count |
|---|--------|-------|-----------------|
| 1 | Reconnaissance | TA0043 | 12 |
| 2 | Resource Development | TA0042 | 9 |
| 3 | Initial Access | TA0001 | 11 |
| 4 | Execution | TA0002 | 20 |
| 5 | Persistence | TA0003 | 22 |
| 6 | Privilege Escalation | TA0004 | 13 |
| 7 | **Stealth** | **TA0005** | 30 |
| 8 | **Defense Impairment** | **TA0112** | 18 |
| 9 | Credential Access | TA0006 | 17 |
| 10 | Discovery | TA0007 | 34 |
| 11 | Lateral Movement | TA0008 | 9 |
| 12 | Collection | TA0009 | 17 |
| 13 | Command and Control | TA0011 | 18 |
| 14 | Exfiltration | TA0010 | 9 |
| 15 | Impact | TA0040 | 15 |

**Key change:** "Stealth" (TA0005) and "Defense Impairment" (TA0112) are now separate tactics. Previously these were combined under "Defense Evasion." This reflects the increased sophistication of adversary techniques — hiding (stealth) is distinct from actively breaking security controls (defense impairment).

**AI-related techniques:** "Query Public AI Services" (T1682) appears under Reconnaissance — adversaries are using public AI services to gather intel.

---

## Cumulative Research Summary (Rounds 1-2)

| Source | Domain(s) | Finding |
|--------|-----------|--------|
| ISC2 Exam Outline (2024) | All | Official domain weights, subtopics, AI integration across 8 domains |
| NIST RMF | D1 | 7-step RMF (Prepare through Monitor) |
| NIST CSF 2.0 | D1 | 6 functions with Govern; CSF 1.1 archived |
| Wikipedia BLP | D3 | Simple Security, *-Property, DAC, Strong Star, tranquility |
| NIST SP 800-30 | D1 | Risk assessment at 3 tiers, amplifying SP 800-39 |
| OWASP Top 10:2025 | D6, D8 | **Current is 2025** not 2021; A10 = "Mishandling of Exceptional Conditions" |
| NIST SP 800-61 Rev 3 | D7 | **Rev 2 withdrawn**; Rev 3 is CSF 2.0 Community Profile |
| MITRE ATT&CK | D6, D7 | **15 Enterprise tactics**; Stealth + Defense Impairment split |
