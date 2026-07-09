---
title: "Risk Management Frameworks"
layer: 2
domain: 1
tags:
  - cissp
  - domain-01
  - risk-management
  - nist-rmf
  - iso-27005
  - cobit
  - octave
  - fair
related_domains:
  - "[[domain-03-security-architecture-and-engineering]]"
  - "[[domain-06-security-assessment-and-testing]]"
  - "[[domain-07-security-operations]]"
  - "[[risk-formulas-and-quantitative-analysis]]"
  - "[[gdpr-deep-dive]]"
  - "[[hipaa-security-rule]]"
---

# Risk Management Frameworks

## Feynman Explanation

A risk management framework is just a recipe book for figuring out "what bad things could happen, how likely they are, and what we'll do about it." Different industries use different recipe books because they care about different things — banks care about money, hospitals care about patient safety, governments care about espionage. A CISO doesn't pick one and ignore the rest; they blend them.

## Technical Details

### Why a Framework?

Without a documented framework, "risk management" becomes a personality test — whoever yelled loudest in the meeting wins. Frameworks give us:

1. **Repeatable process** — same steps every quarter, every audit, every new system.
2. **Defensibility** — when the regulator asks "why?", we point to the framework.
3. **Common language** — "likelihood" means the same thing to the CFO, the CIO, and the CISO.
4. **Auditability** — evidence trail for SOC 2, ISO 27001, FedRAMP, etc.

### NIST Risk Management Framework (RMF) - SP 800-37 Rev. 2

The most cited framework in U.S. federal space and increasingly in the private sector. 7 steps, designed to integrate into the SDLC:

| Step | Name | Output |
|---|---|---|
| 1 | **Prepare** | Organization-wide risk framing, roles, tolerance |
| 2 | **Categorize** | System impact level (Low/Moderate/High) per FIPS 199 |
| 3 | **Select** | Choose baseline controls from SP 800-53 |
| 4 | **Implement** | Deploy and document the controls |
| 5 | **Assess** | Test whether controls work as designed |
| 6 | **Authorize** | Authorizing Official (AO) signs the ATO (Authority to Operate) |
| 7 | **Monitor** | Continuous monitoring, change control, re-assess |

**Key concept:** The ATO is a *business* decision, not a technical one. The AO is typically a business executive, not the CISO. If the AO signs an ATO without understanding residual risk, they have personal liability.

### NIST SP 800-39 - Risk Management Hierarchy

| Tier | Scope | Who |
|---|---|---|
| Tier 1 | Organization | C-suite, Board, CISO |
| Tier 2 | Mission / Business Process | VP-level process owners |
| Tier 3 | Information System | System owners, ISSOs, ops teams |

Risk flows **down** (Tier 1 sets appetite → Tier 3 implements controls) and risk decisions **bubble up** (Tier 3 reports residual risk → Tier 1 adjusts appetite).

### ISO/IEC 27005:2022

International standard, harmonized with ISO 31000 (overarching risk management). Process:

1. **Context establishment** — scope, criteria, appetite, stakeholders
2. **Risk identification** — assets, threats, vulnerabilities, consequences
3. **Risk analysis** — qualitative or quantitative
4. **Risk evaluation** — compare to criteria
5. **Risk treatment** — modify, retain, avoid, share
6. **Risk acceptance** — formal sign-off
7. **Risk communication & consultation** — continuous dialogue
8. **Risk monitoring & review** — continuous oversight

**ISO 27001 vs 27005:**
- 27001 = the certifiable management system (ISMS)
- 27005 = the risk assessment *methodology* that feeds the ISMS

### COBIT 2019

ISACA's framework for **governance and management of enterprise IT**. 40 governance/management objectives organized into 5 domains:

| Domain | Purpose |
|---|---|
| EDM (Evaluate, Direct, Monitor) | Governance — board-level |
| APO (Align, Plan, Organize) | Strategic planning |
| BAI (Build, Acquire, Implement) | Delivery |
| DSS (Deliver, Service, Support) | Operations |
| MEA (Monitor, Evaluate, Assess) | Oversight |

COBIT is **not** a security-specific framework — it's a governance framework that *includes* security. Useful for "how do we make sure IT supports the business?" — less useful for "how do I patch this box tonight?"

### OCTAVE (Operationally Critical Threat, Asset, and Vulnerability Evaluation)

Carnegie Mellon SEI method. Three phases:

1. **Build asset-based threat profiles** — what matters to the business?
2. **Identify infrastructure vulnerabilities** — what tech weaknesses exist?
3. **Develop security strategy and plans** — what do we do about it?

OCTAVE is **self-directed** — meant to be run by internal staff, not consultants. Popular in mid-sized organizations that want a "do it ourselves" approach.

### FAIR (Factor Analysis of Information Risk)

A quantitative framework that decomposes risk into measurable factors. The basic equation:

$$Risk = Loss\ Event\ Frequency \times Loss\ Magnitude$$

Where:

- **Loss Event Frequency (LEF)** = Threat Event Frequency × Vulnerability (probability that threat acts on vulnerability)
- **Loss Magnitude (LM)** = Primary Loss + Secondary Loss (response, reputational, legal, competitive)

FAIR is the bridge between qualitative "red/yellow/green" and the dollar-denominated ALE world. See [[risk-formulas-and-quantitative-analysis]] for the math.

### Framework Comparison

| Framework | Type | Best For | Output |
|---|---|---|---|
| NIST CSF 2.0 | Enterprise program | All orgs, all sectors, board-level | 6-function maturity (Govern/ID/PR/DE/RS/RC) |
| NIST RMF | Process / lifecycle | U.S. federal, regulated industries | ATO, continuous monitoring |
| ISO 27005 | Process | International, ISO 27001 certifiable orgs | Statement of Applicability |
| COBIT 2019 | Governance | Enterprise IT alignment | Goals cascade metrics |
| OCTAVE | Self-assessment | Mid-size, self-directed | Risk profiles + plans |
| FAIR | Quantitative | Cyber insurance, board reporting | Dollar-denominated risk |

### NIST Cybersecurity Framework (CSF) 2.0

Published February 2024 (DOI 10.6028/NIST.CSWP.29), CSF 2.0 is the **current version** (CSF 1.1 is archived). Designed for **all organizations** regardless of sector, size, or cybersecurity maturity. While RMF = system-level ATO, CSF = enterprise-level cybersecurity program.

CSF 2.0 added the **Govern** function — now **6 functions total**:

| Function | Purpose |
|---|---|
| **Govern** (GV) | Establish and monitor cybersecurity risk governance, supply chain, and workforce |
| **Identify** (ID) | Asset management, risk assessment, business environment |
| **Protect** (PR) | Access control, awareness, data security, maintenance |
| **Detect** (DE) | Anomalies and events, continuous monitoring |
| **Respond** (RS) | Response planning, communications, analysis, mitigation |
| **Recover** (RC) | Recovery planning, improvements, communications |

**Key CSF 2.0 enhancements over 1.1:**
- **Govern** function makes cybersecurity risk a board-level concern (maps to COBIT EDM).
- **Supply chain risk management** now a cross-cutting category.
- **Workforce management** Quick-Start Guide (NIST SP 1308).
- **Profiles** for sector-specific tailoring (ransomware, PNT, etc.).
- **Informative References** link CSF outcomes to SP 800-53 controls, ISO 27001, CIS Controls.

**CSF 2.0 vs RMF:** CSF tells you *what* to do at the program level; RMF tells you *how* to authorize a specific system. Most CISOs run CSF enterprise-wide and RMF per-system.

### Threat Modeling Frameworks (related)

| Framework | Acronym | Best For |
|---|---|---|
| STRIDE | Microsoft's six threat categories | Per-design threat modeling |
| PASTA | Process for Attack Simulation and Threat Analysis | App-centric, 7-stage |
| VAST | Visual, Agile, Simple Threat | Scaling threat modeling in agile |
| DREAD | Damage, Reproducibility, Exploitability, Affected Users, Discoverability | Legacy, deprecated by Microsoft |
| ATT&CK | MITRE adversary tactics/techniques | Threat intel, detection gap analysis |

## CISO / Risk Manager View

Frameworks are the **defense** and the **attack** in board conversations. They are:

- **Defense** — "We follow NIST RMF, here's our ATO" — auditor / regulator can't argue.
- **Attack** — "NIST RMF requires this control, and we don't have it" — leverage for budget ask.

**Strategic moves for a CISO:**

1. **Adopt one primary framework** (NIST RMF or ISO 27005) and map others to it. Don't run two parallel risk programs.
2. **Use FAIR for board communication** — translating "high/medium/low" into dollar ranges is the difference between "we're secure" and "our top 5 risks total $42M of exposure."
3. **Tie cyber insurance to FAIR outputs** — insurers now demand probabilistic loss estimates, not control checklists.
4. **Make the ATO a business ritual** — quarterly AO briefings, signed acceptance forms, retained for 7+ years.

A common pitfall: treating the framework as a compliance chore. If the framework is not driving real investment decisions, it is waste.

## Related Connections

### Sibling L2
- [[risk-formulas-and-quantitative-analysis]] - The math behind the frameworks
- [[security-governance-policies-standards]] - Governance outputs of risk decisions
- [[legal-regulatory-compliance-landscape]] - Compliance drivers for framework choice
- [[business-continuity-and-disaster-recovery]] - One of the major risk outcomes

### L3
- [[gdpr-deep-dive]] - Article 32 ties directly to ISO 27002 controls
- [[hipaa-security-rule]] - HIPAA Security Rule maps to NIST SP 800-66

### Cross-Domain
- [[domain-03-security-architecture-and-engineering]] - Control catalog (SP 800-53) lives there
- [[domain-06-security-assessment-and-testing]] - Step 5 "Assess" is the test plan
- [[domain-07-security-operations]] - Step 7 "Monitor" lives in operations

## Sources / References

- NIST SP 800-37 Rev. 2 - Risk Management Framework for Information Systems and Organizations
- NIST SP 800-39 - Managing Information Security Risk: Organization, Mission, and Information System View
- NIST SP 800-30 Rev. 1 - Guide for Conducting Risk Assessments
- ISO/IEC 27005:2022 - Information security, cybersecurity and privacy protection — Guidance on managing information security risks
- ISO/IEC 31000:2018 - Risk management — Guidelines
- COBIT 2019 Framework: Governance and Management Objectives - ISACA
- OCTAVE Allegro - Carnegie Mellon SEI
- Open Group FAIR Standard
- MITRE ATT&CK - https://attack.mitre.org
