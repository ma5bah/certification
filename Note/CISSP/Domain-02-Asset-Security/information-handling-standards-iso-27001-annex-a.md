---
title: Information Handling Standards — ISO 27001 Annex A
layer: 3
domain: 2
tags: [cissp, iso-27001, iso-27002, annex-a, isms, controls, governance, framework]
related_domains: [1, 3, 5, 7, 8]
parent: data-classification-and-handling
---

# Information Handling Standards — ISO 27001 Annex A

## Feynman Explanation

Imagine you want to run a chain of banks. Each country has its own rules, so instead of inventing your own, you adopt a **single standard** that auditors worldwide already trust — the ISO 27001 standard. Annex A is the **list of 93 specific checks** that any auditor will run on you: "do you have a firewall policy? a backup procedure? a classification scheme? an incident response plan?" Each check is a **control**; together they form a **management system** that you operate, measure, and improve. ISO 27001 is the world's most-recognized information security framework, and Annex A is its punch list.

## Technical Details

### The 2022 Restructure

The 2022 revision collapsed the 2013 14-domain structure (A.5–A.18) into **4 themes** with **93 controls** (down from 114 in 2013, via merging and de-duplication).

| Theme | Range | Focus | Number |
|---|---|---|---|
| **A.5 — Organizational** | A.5.1 – A.5.37 | Policies, roles, supplier relationships, incident management | 37 |
| **A.6 — People** | A.6.1 – A.6.8 | HR security, awareness, remote work, disciplinary process | 8 |
| **A.7 — Physical** | A.7.1 – A.7.14 | Perimeter, entry, secure areas, equipment lifecycle, clear desk | 14 |
| **A.8 — Technological** | A.8.1 – A.8.34 | Endpoint, network, crypto, data, development, monitoring | 34 |

> *Total: 93 controls. Each control has an **objective**, a set of **statements**, and (in 27002) **implementation guidance**.*

### The Asset-Security-Relevant Controls (Domain 2 focus)

The CISSP Asset Security domain maps most closely to a subset of Annex A. The following table shows the **most directly relevant** controls:

| # | Control | Theme | CISSP / Asset-Security Link |
|---|---|---|---|
| A.5.9 | Inventory of information and other associated assets | Organizational | The [[asset-inventory-and-categorization]] spine |
| A.5.10 | Acceptable use of information and other associated assets | Organizational | The handling rules from [[data-classification-and-handling]] |
| A.5.11 | Return of assets | Organizational | Termination / offboarding obligation |
| A.5.12 | Classification of information | Organizational | The sensitivity-tier taxonomy |
| A.5.13 | Labelling of information | Organizational | Tags, banners, metadata |
| A.5.14 | Information transfer | Organizational | Sharing, transit, exchange |
| A.5.15 | Access control *(policy)* | Organizational | Cross-links to Domain 5 |
| A.5.16 | Identity management | Organizational | Cross-links to Domain 5 |
| A.5.19 | Information security in supplier relationships | Organizational | Third-party risk |
| A.5.20 | Addressing information security in supplier agreements | Organizational | Contractual clauses |
| A.5.21 | Managing information security in the ICT supply chain | Organizational | Software / hardware integrity |
| A.5.22 | Monitoring, review and change management of supplier services | Organizational | Continuous supplier assessment |
| A.5.23 | Information security for use of cloud services | Organizational | Cloud-specific ([[cloud-asset-security-shared-responsibility]]) |
| A.5.24 | Information security incident management planning | Organizational | IR, escalation |
| A.5.28 | Collection of evidence | Organizational | Forensics readiness |
| A.5.30 | ICT readiness for business continuity | Organizational | BCP / DR |
| A.5.34 | Privacy and protection of PII | Organizational | [[data-privacy-and-protection]] |
| A.5.36 | Compliance with policies, standards, and regulations | Organizational | Audit posture |
| A.6.1 | Screening | People | HR background checks |
| A.6.2 | Terms and conditions of employment | People | Confidentiality, IP, code of conduct |
| A.6.3 | Information security awareness, education, training | People | The human firewall |
| A.6.5 | Responsibilities after termination or change of employment | People | Access removal, asset return |
| A.6.6 | Confidentiality or non-disclosure agreements | People | Legal cover for trade secrets |
| A.6.7 | Remote working | People | BYOD, home networks |
| A.7.1 | Physical security perimeters | Physical | Data center walls, mantraps |
| A.7.2 | Physical entry | Physical | Badges, biometrics, mantraps, visitor escort |
| A.7.3 | Securing offices, rooms and facilities | Physical | Clean desk, screen locks |
| A.7.4 | Physical security monitoring | Physical | CCTV, guards, motion sensors |
| A.7.9 | Security of assets off-premises | Physical | Laptops, mobile, travel |
| A.7.10 | Storage media | Physical | The bridge to [[data-retention-and-disposal-nist-800-88]] |
| A.7.11 | Supporting utilities | Physical | Power, HVAC, fire suppression |
| A.7.13 | Equipment maintenance | Physical | Maintenance windows, escort |
| A.7.14 | Secure disposal or re-use of equipment | Physical | The bridge to [[data-retention-and-disposal-nist-800-88]] |
| A.8.1 | User endpoint devices | Technological | The endpoint control plane |
| A.8.2 | Privileged access rights | Technological | Domain 5 + Domain 7 |
| A.8.3 | Information access restriction | Technological | RBAC, need-to-know |
| A.8.5 | Secure authentication | Technological | MFA, password policy |
| A.8.8 | Management of technical vulnerabilities | Technological | Patch mgmt, vuln scan |
| A.8.9 | Configuration management | Technological | Baselines, drift detection |
| A.8.10 | Information deletion | Technological | The [[data-retention-and-disposal-nist-800-88]] bridge |
| A.8.11 | Data masking | Technological | [[data-privacy-and-protection]] |
| A.8.12 | Data leakage prevention | Technological | [[data-loss-prevention-dlp]] |
| A.8.13 | Information backup | Technological | 3-2-1, immutability |
| A.8.15 | Logging | Technological | SIEM ingestion |
| A.8.16 | Monitoring activities | Technological | SOC, UEBA |
| A.8.17 | Clock synchronization | Technological | Log correlation |
| A.8.20 | Network security | Technological | Segmentation, firewalls |
| A.8.21 | Security of network services | Technological | Provider SLAs |
| A.8.22 | Segregation of networks | Technological | DMZ, OT/IT, prod/dev |
| A.8.24 | Use of cryptography | Technological | [[../Domain-03-Security-Architecture-and-Engineering/Domain-03-Index]] |
| A.8.25 | Secure development life cycle | Technological | [[../Domain-08-Software-Development-Security/Domain-08-Index]] |
| A.8.28 | Secure coding | Technological | OWASP, SAST/DAST |
| A.8.31 | Separation of development, test and production environments | Technological | Environment hygiene |
| A.8.32 | Change management | Technological | Change control board |

> The 2022 revision explicitly added: **A.5.30 ICT readiness**, **A.5.34 PII**, **A.7.4 Physical security monitoring**, **A.8.11 Data masking**, **A.8.12 Data leakage prevention**, **A.8.16 Monitoring activities**, **A.8.23 Web filtering**, **A.8.29 Security testing in development**. These directly align with CISSP Domain 2 controls.

### Statement-of-Applicability (SoA)

The **SoA** is the single most important document in an ISMS — the auditable map of:

| Column | Example |
|---|---|
| **Control ID** | A.8.12 |
| **Control name** | Data leakage prevention |
| **Applicable?** | Yes / No / Partially |
| **Justification** | "Yes — financial and PII data are in scope; DLP deployed across endpoint, network, cloud" |
| **Implementation status** | Implemented / In progress / Planned |
| **Owner** | CISO / Data Protection Officer |
| **Linked risks** | RISK-0017 (insider data theft), RISK-0021 (regulatory non-compliance) |
| **Linked policies** | POL-DLP-001 |
| **Linked evidence** | DLP dashboard, audit logs, incident tickets |

The SoA is the **document the auditor opens first** — it tells them where to look.

### The Plan-Do-Check-Act (PDCA) Loop

ISO 27001 ISMS operates on Deming's PDCA cycle:

```
   ┌──────────────┐
   │    PLAN      │ ← Context, leadership, scope, risk assessment, SoA, risk treatment
   └──────┬───────┘
          ↓
   ┌──────────────┐
   │     DO       │ ← Implement controls, train, operate, document
   └──────┬───────┘
          ↓
   ┌──────────────┐
   │    CHECK     │ ← Monitor, measure, audit, review, KPI tracking
   └──────┬───────┘
          ↓
   ┌──────────────┐
   │     ACT      │ ← Corrective action, continual improvement, management review
   └──────┬───────┘
          └─→ back to PLAN
```

### Risk Treatment Options (ISO 27001 §6.1.3)

For each identified risk, the organization must choose:

| Option | Description | CISSP Analog |
|---|---|---|
| **Risk modification** | Apply controls to reduce likelihood/impact | The default — implement a control |
| **Risk acceptance** | Formally accept residual risk | Risk register sign-off by owner |
| **Risk avoidance** | Remove the activity that creates risk | Decommission system / drop data |
| **Risk sharing** | Transfer to a third party (insurer, outsourcer) | Cyber insurance, contract clauses |
| **Risk exploitation** *(opportunity)* | Take the risk for upside | Strategic business decision |

> ISO 27001's distinguishing feature vs. NIST CSF: it is **prescriptive** (you must implement the controls or justify their exclusion in the SoA) and **certifiable** (an accredited body audits and certifies).

### Certification Process

| Stage | Description | Typical Duration |
|---|---|---|
| **Stage 1 audit (Documentation)** | Auditor reviews ISMS documentation, SoA, policies, scope | 1–3 days |
| **Stage 2 audit (On-site / remote evidence)** | Auditor tests control effectiveness by sampling evidence | 5–15 days |
| **Non-conformities** | Major (control missing or ineffective) or Minor (partial) | Resolved within 60–90 days |
| **Certification issued** | 3-year validity, annual surveillance audits | Year 0 |
| **Surveillance audit** | Annual partial re-audit | Year 1, Year 2 |
| **Recertification** | Full re-audit | Year 3 |

### Related Standards (the ISO 27000 family)

| Standard | Scope | CISSP Use |
|---|---|---|
| **ISO/IEC 27000** | Vocabulary (definitions) | Glossary |
| **ISO/IEC 27001** | ISMS requirements — **the certifiable standard** | The framework |
| **ISO/IEC 27002** | Implementation guidance for the 93 controls | How to implement |
| **ISO/IEC 27003** | ISMS implementation guidance | Project playbook |
| **ISO/IEC 27004** | Monitoring, measurement, analysis, evaluation | KPI definitions |
| **ISO/IEC 27005** | Information security risk management | Risk methodology |
| **ISO/IEC 27017** | Cloud-specific controls | Cloud ([[cloud-asset-security-shared-responsibility]]) |
| **ISO/IEC 27018** | PII protection in public clouds | Privacy in cloud |
| **ISO/IEC 27035-1/2/3** | Incident management | IR |
| **ISO/IEC 27701** | Privacy Information Management System (PIMS) extension | Privacy |
| **ISO/IEC 27033** | Network security | Network |
| **ISO/IEC 27042** | Digital evidence / forensics | IR |
| **ISO/IEC 27043** | Incident investigation principles | IR |
| **ISO/IEC 15408 (Common Criteria)** | Product evaluation | Procurement |

### Mapping CISSP → ISO 27001 (Asset Security focus)

| CISSP Topic | ISO 27001 Control(s) |
|---|---|
| Data classification | A.5.12, A.5.13 |
| Asset inventory | A.5.9 |
| Data lifecycle | A.5.10, A.5.11, A.5.14, A.8.10, A.8.13 |
| DLP | A.8.12 |
| Cloud security | A.5.19–A.5.23 |
| Sanitization | A.7.10, A.7.14, A.8.10 |
| Privacy | A.5.34 + ISO 27701 |
| Records / audit | A.5.28, A.5.36 |
| Asset valuation / risk | A.5.7 (Threat intel), ISO 27005 |
| Ownership | A.5.9, A.5.24, A.5.2 (Roles) |
| Physical security | A.7.1–A.7.14 |

## CISO / Risk Manager View

**Why this matters at the board level.** **ISO 27001 is the most-recognized ISMS certification in the world** — over 70,000 certificates issued globally (ISO Survey 2023). It is a **board-level signal of maturity**: "we operate an independently-audited information security management system." In enterprise B2B sales, the absence of ISO 27001 is a **deal-breaker** for many procurement processes; the presence of it shortens vendor onboarding.

**Strategic value beyond compliance.**

- **Customer trust.** ISO 27001 + SOC 2 Type II is the de-facto B2B trust combination in North America; ISO 27001 is the same in EMEA/APAC.
- **Cyber insurance.** Many underwriters reduce premiums for ISO 27001-certified organizations (typically 5–15%).
- **Regulatory leverage.** ISO 27001 is recognized by GDPR, NIS2, DORA, PCI-DSS, HIPAA, and most national regulators as a *presumption of conformity*.
- **Operational efficiency.** A documented ISMS reduces audit fatigue (one internal ISMS audit cycle satisfies multiple framework requests).

**Economic lens.**

- Initial certification cost (consulting + audit): **$150k–$1M** for mid-to-large enterprise.
- Annual maintenance (surveillance + continual improvement): **$75k–$400k**.
- **Multi-framework leverage:** ISO 27001 + NIST CSF + SOC 2 share ~70% of their controls. Many enterprises adopt a "control once, map to many" strategy that halves the cost of any single framework.

**Strategic actions.**

1. **Scope narrowly first.** A "whole-company" ISMS is a 2-year program; a "single business unit" ISMS is a 6-month program and grows from there.
2. **Adopt "control once, map to many"** — build the ISMS control library once, then derive NIST CSF, CIS v8, PCI-DSS, HIPAA mappings as views.
3. **Train internal auditors.** External audit once a year is necessary; internal audits quarterly keep the ISMS honest.
4. **Tie the ISMS to the risk register.** Without risk-based prioritization, the SoA becomes a 93-row checkbox exercise with no value.
5. **Use ISO 27002 for implementation guidance, ISO 27005 for risk management** — they are the operational companions to 27001.
6. **Plan for re-certification early.** The 3-year cycle requires continuous evidence; it is not a "year-3 cram."
7. **For cloud and privacy** — pair ISO 27001 with **ISO 27017 (cloud)**, **ISO 27018 (cloud PII)**, and **ISO 27701 (PIMS)** for comprehensive coverage.

## Related Connections

### Parent L2
- [[data-classification-and-handling]] — A.5.12, A.5.13
- [[data-lifecycle-and-remnants]] — A.5.10, A.5.11, A.5.14, A.8.10, A.8.13
- [[data-privacy-and-protection]] — A.5.34 + ISO 27701
- [[asset-inventory-and-categorization]] — A.5.9

### Sibling L3
- [[data-loss-prevention-dlp]] — A.8.12
- [[cloud-asset-security-shared-responsibility]] — A.5.19–A.5.23, ISO 27017, ISO 27018
- [[data-retention-and-disposal-nist-800-88]] — A.7.10, A.7.14, A.8.10

### Cross-Domain
- [[../Domain-01-Security-and-Risk-Management/security-governance-principles]] — the ISMS is a governance system
- [[../Domain-01-Security-and-Risk-Management/risk-management-framework]] — ISO 27005 vs. NIST RMF
- [[../Domain-01-Security-and-Risk-Management/legal-regulatory-and-compliance-issues]] — ISO 27001 as presumption of conformity
- [[../Domain-03-Security-Architecture-and-Engineering/Domain-03-Index]] — A.8.24 Use of cryptography
- [[../Domain-05-Identity-and-Access-Management/Domain-05-Index]] — A.5.15, A.5.16, A.8.2, A.8.3, A.8.5
- [[../Domain-07-Security-Operations/Domain-07-Index]] — A.5.24–A.5.28, A.8.13–A.8.17
- [[../Domain-08-Software-Development-Security/Domain-08-Index]] — A.8.25, A.8.28, A.8.31, A.8.32

## References

- **ISO/IEC 27001:2022** — Information security, cybersecurity and privacy protection — Information security management systems — Requirements
- **ISO/IEC 27002:2022** — Information security, cybersecurity and privacy protection — Information security controls
- ISO/IEC 27000:2018 — Information security management systems — Overview and vocabulary
- ISO/IEC 27003:2017 — Guidance on ISMS implementation
- ISO/IEC 27004:2016 — Monitoring, measurement, analysis and evaluation
- ISO/IEC 27005:2022 — Information security risk management
- ISO/IEC 27017:2015 — Cloud services (Code of Practice)
- ISO/IEC 27018:2019 — PII in public clouds (Code of Practice)
- ISO/IEC 27701:2019 — Privacy Information Management Systems (PIMS)
- ISO/IEC 27035-1/2:2016/2023 — Incident management
- ISO/IEC 15408 (Common Criteria) — Product evaluation
- ISO Survey of Certifications to Management System Standards 2023
- (ISC)² CISSP CBK — Domain 2: Asset Security (the ISO 27001 mapping is part of the exam)
