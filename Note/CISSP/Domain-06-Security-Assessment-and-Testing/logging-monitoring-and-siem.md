---
title: Logging, Monitoring, and SIEM
layer: 2
domain: 6
tags: [cissp, domain-6, logging, siem, soar, monitoring, mttd, mttr, correlation, use-case]
related_domains: [1, 3, 4, 5, 7]
parent: domain-06-security-assessment-and-testing
siblings: [vulnerability-management-lifecycle, penetration-testing-methodology, security-audits-and-compliance-testing]
---

# Logging, Monitoring, and SIEM

## Feynman Explanation

If a pen test is a scheduled fire drill, monitoring is the smoke detector that runs 24/7. Logs are the *evidence* the system produces about what it's doing — every login, every file change, every network connection. A SIEM (Security Information and Event Management) is the brain that takes thousands of log streams and asks "is any of this bad?" A SOAR (Security Orchestration, Automation, and Response) is the hands that respond. The two KPIs every board cares about — *MTTD* (Mean Time To Detect) and *MTTR* (Mean Time To Respond/Remediate) — are the security program's vital signs. If you can't measure them, you can't manage them.

## Technical Details

### Log sources (the universe of telemetry)

| Domain | Example sources | Why it matters |
|---|---|---|
| Host / endpoint | Windows Event Log, Sysmon, EDR (CrowdStrike, Defender), macOS Unified Log, Linux auditd | Process execution, file change, privilege use |
| Network | NetFlow, firewall logs, DNS logs, proxy / WAF, NDR (Vectra, Darktrace), packet capture | Lateral movement, C2 beacon, exfil |
| Identity | AD domain controllers, AAD sign-in logs, IdP (Okta, Auth0), VPN concentrator | AuthN/AuthZ, MFA fatigue, impossible travel |
| Application | Web server access/error, app audit (DB, ERP), API gateway logs | Web attacks (OWASP), business-logic abuse |
| Cloud | CloudTrail (AWS), Azure Activity, GCP Audit, Cloud Audit/Data Plane | IAM, S3, KMS, compute changes |
| Email | MTA, gateway (Proofpoint, Mimecast), EDR mail telemetry | Phishing, BEC, malware delivery |
| Physical | Badge readers, CCTV, NVR, environmental | Tailgating, unauthorized access |
| Threat intel | TIP feeds (MISP, Anomali), OSINT, ISAC sharing | IOC matching, TTP context |

> **CISSP gotcha:** the most common cause of "we didn't detect the breach" is *not* a missing log source — it's a log source whose events are *not being parsed / correlated*. Coverage is not the same as utilization.

### The five log integrity properties

For a log to be admissible as evidence (legal, regulatory, audit):

| Property | Definition | How to ensure |
|---|---|---|
| **Authenticity** | Provenance can be verified | Cryptographic signing (e.g., RFC 3161 trusted timestamping) |
| **Integrity** | Not altered after creation | Write-once storage, hash chain, immutability (e.g., S3 Object Lock) |
| **Confidentiality** | Protected from unauthorized access | TLS in transit, encryption at rest, RBAC on log store |
| **Availability** | Accessible when needed | Replication, retention tier, hot/warm/cold storage |
| **Non-repudiation** | Cannot be denied | Combination of authenticity + integrity + chain of custody |

NIST SP 800-92 (*Guide to Computer Security Log Management*) is the authoritative US reference for log handling.

### SIEM architecture (logical)

```
  ┌────────────┐   ┌────────────┐   ┌────────────┐
  │   Logs     │   │   Logs     │   │   Logs     │   ...thousands
  │ (endpoint) │   │ (network)  │   │ (cloud)    │
  └─────┬──────┘   └─────┬──────┘   └─────┬──────┘
        │                │                │
        └────────────────┴────────────────┘
                         │   collectors / agents
                         ▼
              ┌─────────────────────────┐
              │  Parsing / Normalization │   (CEF, LEEF, ECS)
              └─────────────┬───────────┘
                            ▼
              ┌─────────────────────────┐
              │  Correlation Engine      │   (rules, ML, UEBA)
              └─────────────┬───────────┘
                            ▼
              ┌─────────────────────────┐
              │  Alert / Case Mgmt       │   (tier-1 queue)
              └─────────────┬───────────┘
                            ▼
              ┌─────────────────────────┐
              │  SOAR (response)         │   (auto-containment)
              └─────────────────────────┘
```

**Key terms:**

| Term | Meaning |
|---|---|
| **SIEM** | Security Information and Event Management — log collection, correlation, alerting, dashboards |
| **SOAR** | Security Orchestration, Automation, and Response — playbook execution, case management |
| **EDR** | Endpoint Detection and Response — endpoint-native telemetry + response |
| **NDR** | Network Detection and Response — network telemetry + ML detection |
| **XDR** | "Extended" DR — vendor-consolidated EDR + NDR + cloud + identity |
| **UEBA / UEBAM** | User & Entity Behavior Analytics — behavioral baseline + anomaly detection |
| **TIP** | Threat Intelligence Platform — IOC/TTP management, sharing |

### SIEM vs. SOAR vs. log manager

| Function | Log Manager | SIEM | SOAR |
|---|---|---|---|
| Collect & store | ✓ | ✓ | (consumes SIEM) |
| Search & retain | ✓ | ✓ | (limited) |
| Correlate & alert | ✗ | ✓ | (consumes SIEM alerts) |
| Case management | ✗ | basic | ✓ |
| Run response playbook | ✗ | ✗ | ✓ |
| Threat intel ingestion | ✗ | ✓ | ✓ |

The mature SOC stack is **SIEM + SOAR + EDR + NDR + TIP**, integrated.

### Correlation types (how SIEMs turn noise into signal)

| Correlation | Example | False-positive risk |
|---|---|---|
| **Threshold** | ">10 failed logins in 5 min" | Medium |
| **Trend / behavioral** | "User downloads 100× normal data volume" | Medium-high |
| **Cross-source** | "Firewall block + EDR process + DNS query within 60s" | Low (high-fidelity) |
| **Stateful / sequence** | "Login from new country → mailbox rule created → mail forward to external" | Low |
| **Time-window / pattern** | "Compromise pattern matching MITRE T1566 → T1078 → T1041" | Low |
| **Asset-based** | "Critical asset X observed unusual port scan" | Low |

### Detection engineering — the lifecycle of a use case

```
   ┌────────────┐   ┌────────────┐   ┌────────────┐
   │  Identify  │──▶│   Develop  │──▶│    Tune    │
   │   threat   │   │   rule /   │   │   (FP/N)   │
   │   (ATT&CK) │   │   analytic  │   └─────┬─────┘
   └────────────┘   └────────────┘         │
                                           ▼
                                     ┌────────────┐
                                     │   Measure  │
                                     │   (KPIs)   │
                                     └─────┬──────┘
                                           │
                                           ▼
                                     ┌────────────┐
                                     │  Retire /  │
                                     │  replace   │
                                     └────────────┘
```

> **A SIEM without curated, measured use cases is a data landfill.** CISSP expects the candidate to know that *detection coverage* is the explicit gap between adversary techniques (ATT&CK) and SIEM rules.

### KPIs — the math (and what good looks like)

#### Mean Time To Detect (MTTD)

$$
MTTD = \frac{1}{n}\sum_{i=1}^{n}(t_{detect,i} - t_{occur,i})
$$

Time from when the event *actually happened* to when the SOC *first detected* it (alert fired, ticket opened).

#### Mean Time To Respond / Remediate (MTTR)

$$
MTTR = \frac{1}{n}\sum_{i=1}^{n}(t_{resolve,i} - t_{detect,i})
$$

Time from detection to resolution (containment + eradication + recovery).

> "MTTR" is overloaded in the industry:
> - **MTTR-Contain** — detection → containment only
> - **MTTR-Full** — detection → full recovery
> - **MTTR-Fix** — detection → root-cause fix (often used in vuln mgmt)
> Be explicit which one you mean.

#### Mean Time Between Failures (MTBF) — for control reliability

$$
MTBF = \frac{\text{Total operational time}}{\text{Number of failures}}
$$

#### Detection coverage (fraction of ATT&CK covered)

$$
Coverage = \frac{|\text{Techniques with detection rule}|}{|\text{Techniques in-scope for environment}|}
$$

A mature program targets **≥ 80%** of relevant ATT&CK techniques with at least one detection rule.

#### Alert fidelity

$$
Precision = \frac{TP}{TP + FP} \quad\quad Recall = \frac{TP}{TP + FN} \quad\quad F1 = 2 \cdot \frac{P \cdot R}{P + R}
$$

A SOC target is typically **Precision ≥ 0.7** (i.e., ≤ 30% of alerts are false positives) to keep analyst sanity.

### Industry benchmarks (2024)

| Metric | Median | Top quartile | Source |
|---|---|---|---|
| MTTD (dwell time) | ~10 days (Mandiant) to ~200 days (IBM) | < 24 hours | Mandiant M-Trends 2024, IBM CDB 2024 |
| MTTR (containment) | ~30–60 days median | < 1 day | IBM / Ponemon |
| Log source coverage | 30–50% of available telemetry | > 80% | ESG / Gartner |
| SIEM use-case count | 50–200 | 500+ with curated, measured | Vendor benchmarks |
| Analyst alert load | 1000+ alerts/day per analyst (saturated) | < 100 actionable/day | Target via SOAR + tuning |

> CISSP often tests the *direction* of these numbers (faster = better) and the *relative* comparison (MTTD < MTTR always).

### SIEM use-case library (typical structure)

| Use case family | Example rule |
|---|---|
| Authentication | Brute-force, impossible travel, MFA fatigue, golden-ticket indicators |
| Privilege abuse | New admin group add, sudo misuse, lateral Kerberos |
| Malware / EDR | Ransomware canary file change, LOLBin execution, suspicious child process |
| Data exfiltration | Large DNS query, unusual S3 download, USB use on classified host |
| Cloud | Root key use, IAM policy change, KMS disable, public S3 exposure |
| Insider | Off-hours bulk download, mailbox rule to external, resignation + download |
| Web (OWASP) | SQLi, XSS, deserialization, SSRF, auth bypass — see [[owasp-top-10-and-web-app-testing]] |

### Continuous monitoring (NIST SP 800-137 + 800-137A)

NIST defines ISCM (Information Security Continuous Monitoring) as a 6-step process:

1. Define the monitoring strategy (what assets, what frequencies)
2. Establish metrics (KPI trees: is the control operating? are we improving?)
3. Establish monitoring frequencies
4. Implement monitoring (collect, correlate, alert)
5. Analyze and report
6. Respond to findings (remediation → feeds [[vulnerability-management-lifecycle]])

> ISCM is the operational incarnation of Domain 6 — the *always-on* testing layer.

## CISO / Risk Manager View

For the board, monitoring is the **"are we safe right now"** system. The two numbers that matter are MTTD and MTTR, and the only thing that matters more than the absolute value is the **trend over the last 8 quarters**. A program that improved MTTD from 200 days to 5 days in 24 months has a *measurable* security story. A program at "we don't know" does not.

**The "we have a SIEM" fallacy.** SIEM is a tool, not a capability. A SIEM that collects 10,000 EPS but has 20 unmaintained rules, no SOAR, and 4,000 daily alerts no one triages is a liability — analysts burn out, the real attack hides in the noise. The mature CISO invests in **detection engineering** (people, process, use-case library) at *equal or greater* budget than the SIEM license.

**Log retention is a board topic.** Retention drives SIEM cost (storage is the #1 SIEM expense). The defensible retention policy balances forensic value (typically 1 year hot, 7 years cold for regulated industries) against cost. PCI DSS requires 1 year of audit trail with 90 days immediately available. HIPAA = 6 years. SOX = 7 years.

**Budget framing:**
- SIEM (Splunk Enterprise Security, Microsoft Sentinel, Chronicle, QRadar, Elastic): $50K–$2M+/year, dominated by ingestion volume.
- SOAR (Splunk SOAR, Tines, Torq, Palo Alto XSOAR): $50K–$500K/year.
- EDR ($30–$80/endpoint/year) and NDR ($50K–$500K/year) are now standard line items.
- SOC (24×7 follow-the-sun): $2M–$10M/year if fully internal; lower with MSSP co-sourcing.

**Outsourcing decision** (build vs. buy): the *CISO* function stays internal (you cannot outsource accountability), but tier-1 monitoring is commonly MSSP-co-sourced, and after-hours is universally so. The board topic is "what is the *handoff* protocol between MSSP and our tier-2/3, and is there a quarterly purple-team exercise to validate it?"

**Regulatory pressure is increasing.** SEC's 2023 cybersecurity disclosure rules (Form 8-K Item 1.05) require public companies to disclose material breaches within 4 business days — making MTTD a *financial-disclosure* metric, not just a security one.

## Related Connections

### Sibling L2 deep-dives
- [[vulnerability-management-lifecycle]] (vuln tickets appear in SIEM; MTTR is shared KPI)
- [[penetration-testing-methodology]] (purple team measures detection rate from pen-test traffic)
- [[security-audits-and-compliance-testing]] (SIEM logs = SOC 2 / ISO / PCI evidence)

### L3 specifics
- [[owasp-top-10-and-web-app-testing]] (web attack use cases map to OWASP Top 10)
- [[mitre-att-and-ck]] (detection engineering maps to ATT&CK)
- [[red-team-vs-blue-team-vs-purple-team]] (BAS measures detection coverage)

### Cross-domain
- [[Domain 1 — Security and Risk Management]] (KPIs feed risk register)
- [[Domain 3 — Security Architecture and Engineering]] (secure logging is an architectural decision)
- [[Domain 4 — Communication and Network Security]] (network telemetry is core feed)
- [[Domain 5 — Identity and Access Management]] (identity events are highest-signal logs)
- [[Domain 7 — Security Operations]] (SIEM/SOAR is the heart of the SOC)

## References

- NIST SP 800-92, *Guide to Computer Security Log Management* (2006)
- NIST SP 800-137, *Information Security Continuous Monitoring (ISCM) for Federal Information Systems and Organizations* (2011)
- NIST SP 800-137A, *Assessing Information Security Continuous Monitoring (ISCM) Programs* (2020)
- NIST SP 800-61 Rev. 2, *Computer Security Incident Handling Guide* (2012; rev. 3 in draft)
- CIS Critical Security Controls v8 — Control 13 (Network Monitoring and Defense), Control 8 (Audit Log Management), Control 17 (Incident Response)
- MITRE ATT&CK Enterprise v15+ — use-case library backbone
- MITRE D3FEND — defensive technique mapping (counterpart to ATT&CK)
- Mandiant *M-Trends 2024* — industry MTTD benchmark
- IBM *Cost of a Data Breach Report 2024* — MTTR and cost-per-record benchmarks
- SANS *2019 SIEM Survey* (most recent comprehensive; later vendor-specific reports exist)
- Gartner *Market Guide for Security Information and Event Management* (annual)
- PCI DSS v4.0 §10 (Log and Monitor All Access)
- HIPAA §164.312(b) (Audit controls) and §164.316(b) (Documentation retention 6 years)
- SOX §404 IT general controls (audit log retention 7 years)
- SEC Cybersecurity Disclosure Rule (2023) — 4-business-day breach disclosure
- ENISA *Reference Incident Classification Taxonomy* (EU)
- RFC 3161 — Trusted Timestamping
- RFC 5424 — syslog protocol
- CEF (Common Event Format) — ArcSight standard
- LEEF (Log Event Extended Format) — IBM QRadar standard
- ECS (Elastic Common Schema) — Elastic standard
- OpenTelemetry — modern log/metric/trace collection
- OWASP *Logging Cheat Sheet*
