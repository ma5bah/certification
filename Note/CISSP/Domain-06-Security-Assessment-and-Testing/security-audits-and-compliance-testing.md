---
title: Security Audits and Compliance Testing
layer: 2
domain: 6
tags: [cissp, domain-6, audit, soc2, iso-27001, pci-dss, compliance, sampling, ssae-18]
related_domains: [1, 2, 7, 8]
parent: domain-06-security-assessment-and-testing
siblings: [vulnerability-management-lifecycle, penetration-testing-methodology, logging-monitoring-and-siem]
---

# Security Audits and Compliance Testing

## Feynman Explanation

A security audit is "did you actually do the things you said you did, and does it match the rules someone else wrote?" A pen test tries to *break* things; an audit checks *paperwork and evidence*. A SOC 2 auditor doesn't care that your IDS caught the attack — they care that you have a written policy, an assigned owner, evidence of quarterly review, and a signed attestation. The skill the CISSP tests here is the ability to distinguish *compliance* (you met the rule) from *security* (you are actually safe) — and to know that mature programs need both.

## Technical Details

### Audit vs. assessment vs. test (definitional table)

| Term | Who | Goal | Output | Tone |
|---|---|---|---|---|
| **Audit** | Independent (internal or external CPA) | Verify *conformance* to a standard | Formal opinion / report | Compliance — "yes/no" |
| **Assessment** | Internal / external | Measure *effectiveness* of controls | Risk-rated findings | Risk — "good/bad/ugly" |
| **Test** | Anyone, including target system owner | Validate a specific control works | Pass/fail per case | Operational — "does it work?" |
| **Inspection** | Regulator (FCA, SEC, ECB) | Verify regulatory compliance | Enforcement action | Legal — "penalty / no penalty" |

CISSP uses these terms loosely; the exam cares about the *separation* between *internal* and *external* audit, and between *compliance* and *security*.

### Internal vs. external audit

| Dimension | Internal audit | External audit |
|---|---|---|
| Reports to | Audit committee of the board | Regulator, customer, public |
| Independence | Within org, but separate from line management | Truly independent firm (CPA, certification body) |
| Scope | Risk-based, management's choice | Defined by standard or regulator |
| Frequency | Continuous, risk-based | Annual (most standards) |
| Output | Reports to mgmt + audit committee | Formal opinion (qualified / unqualified) |
| Used for | Improving control design | Public attestation, regulatory filing |

> **Separation of duties (D5) extends here:** the people who *build* controls cannot be the people who *audit* them. Internal audit must report to the audit committee, not the CISO.

### The big three compliance regimes (CISSP exam favorites)

#### 1. SOC 2 (AICPA SSAE-18 / ISAE 3402)

SOC 2 reports on controls at a *service organization* (the company providing the service) against the **AICPA Trust Services Criteria (TSC)**:

| TSC | Full name | What it covers |
|---|---|---|
| **CC** | Common Criteria (mandatory) | General control objectives (security baseline) |
| **A** | Availability | Uptime, DR, capacity |
| **C** | Confidentiality | Protection of confidential info |
| **P** | Processing Integrity | Completeness, accuracy, timeliness |
| **PI** | Privacy | Full privacy framework |

**Two report types:**

| Type | Window | Use |
|---|---|---|
| **Type I** | Point in time | "Controls were suitably designed as of date X" |
| **Type II** | Period (typically 6–12 months) | "Controls operated effectively over the period" |

Type II is the higher-assurance report and is what enterprise customers and most regulators require.

#### 2. ISO/IEC 27001:2022

International ISMS standard. Structure:

- **Clauses 4–10** — ISMS requirements (mandatory, certifiable)
- **Annex A** — 93 controls in 4 themes (Organizational 37, People 8, Physical 14, Technological 34)

The certifiable artifact is the **ISMS** (Information Security Management System). Audit is in two stages: **Stage 1** (documentation review) and **Stage 2** (operating effectiveness). Re-certification every 3 years; surveillance audits annually.

Companion standard **ISO/IEC 27002:2022** is the *how* (implementation guidance) for the Annex A controls.

#### 3. PCI DSS v4.0

Payment Card Industry Data Security Standard — applies to *anyone* that stores, processes, or transmits cardholder data.

12 high-level requirements (v4.0):

| # | Requirement (abbreviated) |
|---|---|
| 1 | Install and maintain network security controls |
| 2 | Apply secure configurations to all system components |
| 3 | Protect stored account data |
| 4 | Protect cardholder data with strong cryptography during transmission |
| 5 | Protect all systems and networks from malicious software |
| 6 | Develop and maintain secure systems and software |
| 7 | Restrict access to system components and cardholder data by business need |
| 8 | Identify users and authenticate access to system components |
| 9 | Restrict physical access to cardholder data |
| 10 | Log and monitor all access to system components and cardholder data |
| 11 | Test security of systems and networks regularly |
| 12 | Support information security with organizational policies and programs |

**Merchant levels (Visa/MC):** Level 1 (>6M tx/yr) requires annual on-site audit by QSA (Qualified Security Assessor) and ASV scan. Level 2–4 may self-assess (SAQ).

### Audit evidence (the "show me" factor)

A control exists in audit terms *only* if you can produce evidence on demand. CISSP expects the candidate to know that "we do MFA" is not evidence; the following *is*:

| Evidence type | Example |
|---|---|
| Policy | "Information Security Policy v3.2, signed by CEO 2024-09-15" |
| Standard / procedure | "Password Standard, version 1.4" |
| Configuration | "AWS Config rule `iam-password-policy` shows min 14 chars, complexity, 90-day rotation" |
| Log / SIEM | "Splunk query result 2024-10-01: 0 failed admin logins from non-approved IPs" |
| Screenshot | "Active Directory GPO `gpo-pw-policy` applied to OU=Domain Controllers" |
| Ticketing | "Jira ticket SEC-1234 closed by jdoe on 2024-10-05 with peer review" |
| Attestation | "Control owner J. Doe quarterly attestation, 2024-09-30" |

> **Three lines of defense** model (common in regulated industries):
> 1. **First line** — operational management owns and runs controls
> 2. **Second line** — risk, compliance, InfoSec sets policy and monitors
> 3. **Third line** — internal audit provides *independent* assurance to the board
> 4. (Sometimes) **Fourth line** — external auditors and regulators

### Audit sampling — the math

Auditors don't test every transaction. They use **sampling**, and the exam expects you to know the difference between *attribute* and *variable* sampling, and how sample size is set.

**Statistical attribute sampling** (most common in IT audits — pass/fail per item):

$$
n = \frac{Z^2 \cdot p \cdot (1 - p)}{e^2}
$$

where:
- $n$ = sample size
- $Z$ = z-score for desired confidence (1.645 for 90%, 1.96 for 95%, 2.576 for 99%)
- $p$ = expected population error rate (often 0.5 for max sample size)
- $e$ = tolerable margin of error (often 0.05)

**Worked example:** 95% confidence ($Z = 1.96$), $p = 0.5$, $e = 0.05$:

$$
n = \frac{1.96^2 \cdot 0.5 \cdot 0.5}{0.05^2} = \frac{3.8416 \cdot 0.25}{0.0025} = \frac{0.9604}{0.0025} \approx 384
$$

So a population with unknown error rate at 95% confidence, ±5% margin → 384 samples.

**AICPA audit sampling guidance** (used in SOC 2):

| Confidence | Sample size (for ≤ 250 items) |
|---|---|
| 90% | 50–60 |
| 95% | 60–90 |
| 99% | 90–150 |

> **CISSP trap:** confusing the formula. You don't subtract or add to $n$ for population size in the basic form; the formula is for *proportion* estimation. The finite population correction is a separate factor: $n_{adj} = n / (1 + (n-1)/N)$, used when $n/N > 5\%$.

### Other audit-relevant standards (alphabet soup)

| Standard | Issuer | Scope |
|---|---|---|
| HIPAA Security Rule | HHS (US) | PHI in healthcare |
| HITRUST CSF | HITRUST | Healthcare hybrid (HIPAA + ISO + NIST) |
| FedRAMP | US GSA | Cloud services to US federal govt |
| CMMC 2.0 | US DoD | Defense industrial base |
| NIST CSF 2.0 | NIST | Voluntary framework, all sectors |
| SOX | US SEC | Financial reporting controls (IT general controls) |
| GDPR Art. 32 | EU | "Appropriate technical and organizational measures" |
| NIS2 | EU | Operators of essential services |
| DORA | EU | Financial-sector ICT risk |
| APRA CPS 234 | Australia | Financial sector info sec |
| Cyber Essentials | UK gov | Baseline UK |

## CISO / Risk Manager View

Audits are expensive *and* essential. The strategic CISO view:

**1. The "single audit" fallacy.** Many CISOs chase certifications sequentially (SOC 2 → ISO 27001 → PCI → HIPAA). The mature play is a **unified control framework (UCF)** that maps every requirement to one set of internal controls, then evidence is collected once and reused. This is the difference between 1× audit cost and 4× audit cost.

**2. Compliance is not security.** This is the single most important CISO message. The company can be fully SOC 2 compliant and still be breached the next day (e.g., a vendor compromise not covered by the in-scope system). Conversely, a company can be more secure than its compliance requires. CISOs must explain to the board that **audit opinion ≠ security posture** — and back the opinion with operational metrics from [[logging-monitoring-and-siem]] and [[vulnerability-management-lifecycle]].

**3. The "evidence on demand" discipline.** A control that cannot produce evidence within 24 hours is, for audit purposes, not a control. Build the evidence pipeline *with* the control, not as a separate workstream.

**4. The audit-relationship-as-asset view.** External auditors (Big-4 for SOC 2, certification bodies for ISO) become *advisors* if treated well. They flag common findings across many clients — that insight is a free strategic input to the program. **Rotate audit firms every 3–5 years** to preserve independence.

**Budget framing:** SOC 2 Type II audit: $50K–$300K depending on size; ISO 27001 cert: $30K–$150K + ongoing surveillance; PCI QSA: $40K–$200K. These are *fixed* costs; the variable cost is the *remediation* of findings. **Budget 2–3× the audit fee for remediation** to avoid surprise.

**Board reporting:** the CISO's quarterly report to the audit committee should include (a) audit findings open / closed, (b) control exceptions, (c) auditor relationship health, (d) regulatory developments that change the control set.

## Related Connections

### Sibling L2 deep-dives
- [[vulnerability-management-lifecycle]] (vuln mgmt is a key audited control)
- [[penetration-testing-methodology]] (pen test = mandatory SOC 2 / ISO / PCI evidence)
- [[logging-monitoring-and-siem]] (SIEM evidence is what makes SOC 2 CC7.1 pass)

### L3 specifics
- [[owasp-top-10-and-web-app-testing]] (web controls are common audit focus)
- [[mitre-att-and-ck]] (audits increasingly demand threat-informed control testing)
- [[red-team-vs-blue-team-vs-purple-team]] (continuous assurance beyond annual audit)

### Cross-domain
- [[Domain 1 — Security and Risk Management]] (audit is a primary risk-treatment verification)
- [[Domain 2 — Asset Security]] (asset inventory is the input to audit scope)
- [[Domain 7 — Security Operations]] (audit findings drive operational change)
- [[Domain 8 — Software Development Security]] (SDLC controls are audited in SOC 2 CC8.1)

## References

- AICPA SSAE-18 / SSAE-20 (2022 update), *Reporting on an Examination of Controls at a Service Organization*
- AICPA TSP Section 100, *Trust Services Criteria for Security, Availability, Processing Integrity, Confidentiality, and Privacy* (2017, updated 2022)
- ISO/IEC 27001:2022 — Information security, cybersecurity and privacy protection — ISMS — Requirements
- ISO/IEC 27002:2022 — Information security, cybersecurity and privacy protection — Controls
- ISO/IEC 19011:2018 — Guidelines for auditing management systems
- ISO/IEC 27008:2022 — Guidelines for the assessment of information security controls
- PCI DSS v4.0 (March 2022) and v4.0.1 (June 2024)
- PCI SSC — QSA and ASV qualification documents
- HIPAA Security Rule (45 CFR §164.302–318)
- FedRAMP Authorization Act / FedRAMP Rev. 5 (2023)
- NIST CSF 2.0 (Feb 2024)
- NIST SP 800-53 Rev. 5, *Security and Privacy Controls for Information Systems*
- NIST SP 800-53A Rev. 5, *Assessing Security and Privacy Controls*
- ISACA *Information Technology Assurance Framework (ITAF)* — auditing standards for IT
- IIA *International Standards for the Professional Practice of Internal Auditing*
- COSO *Internal Control — Integrated Framework* (2013)
- ISACA *CISA Review Manual* — sampling theory chapter
- EU GDPR Art. 32, NIS2 Directive (EU 2022/2555), DORA (EU 2022/2554)
- APRA CPS 234 (Australia)
- CMMC 2.0 (US DoD, 2024 final rule)
