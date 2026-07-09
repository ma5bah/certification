---
title: "Browser Research Evidence — CISSP KB Verification"
layer: 0
type: source-evidence
date: 2026-07-08
sources_visited: 4
status: complete
---

# CISSP Browser Research Evidence Log

This note captures live browser-MCP research findings used to cross-validate the CISSP knowledge base content against authoritative sources.

## Sources Visited

| # | Source | URL | Timestamp |
|---|--------|-----|-----------|
| 1 | ISC2 Official Exam Outline | `https://www.isc2.org/certifications/cissp/cissp-certification-exam-outline` | 2026-07-08 |
| 2 | NIST Risk Management Framework | `https://csrc.nist.gov/projects/risk-management/about-rmf` | 2026-07-08 |
| 3 | Wikipedia — Bell-LaPadula Model | `https://en.wikipedia.org/wiki/Bell-LaPadula_model` | 2026-07-08 |
| 4 | NIST Cybersecurity Framework | `https://www.nist.gov/cyberframework` | 2026-07-08 |

---

## Finding 1: ISC2 2024 CISSP Exam Outline

**Source:** ISC2 Official Page (effective April 15, 2024)

### Exam Details (Verified)
- Format: Computerized Adaptive Testing (CAT)
- Length: 3 hours, 100-150 items
- Passing grade: 700 out of 1000
- Accreditation: ANSI/ISO/IEC 17024

### Domain Weights (Verified)
| Domain | Weight |
|--------|--------|
| 1. Security and Risk Management | 16% |
| 2. Asset Security | 10% |
| 3. Security Architecture and Engineering | 13% |
| 4. Communication and Network Security | 13% |
| 5. Identity and Access Management (IAM) | 13% |
| 6. Security Assessment and Testing | 12% |
| 7. Security Operations | 13% |
| 8. Software Development Security | 10% |

### Domain 1 Subtopic Structure (Verified)
- 1.1 — ISC2 Code of Professional Ethics
- 1.2 — Confidentiality, integrity, availability, authenticity, nonrepudiation (5 Pillars)
- 1.3 — Security governance principles, alignment to business strategy, frameworks
- 1.4 — Legal, regulatory, compliance (GDPR, CCPA, PIPL, POPIA)
- 1.5 — Investigation types (administrative, criminal, civil, regulatory)
- 1.6 — Security policy, standards, procedures, guidelines
- 1.7 — Business Continuity (BIA, external dependencies)
- 1.8 — Personnel security (screening, onboarding, termination, third-party)
- 1.9 — Risk management concepts (threat/vulnerability ID, analysis, response, controls, monitoring, frameworks)
- 1.10 — Threat modeling concepts and methodologies
- 1.11 — Supply Chain Risk Management (SCRM)
- 1.12 — Security awareness, education, and training

### AI/ML Integration (New for 2024)
The ISC2 page explicitly states: "AI and machine learning (ML) become foundational to modern business operations, the CISSP certification has evolved to ensure that cybersecurity professionals can govern, design and defend these sophisticated systems... ISC2 continues to interweave AI-specific security tasks and subtasks across all eight domains." Specific AI topics mentioned:
- Domain 1: AI ethics, algorithmic bias, AI supply chain risk
- Domain 2: Training dataset classification, model weights as intellectual property, data poisoning
- Domain 3: AI compute enclaves, prompt injection defense, Explainable AI as security requirement
- Domain 4: Micro-segmentation for AI training environments, AI-driven NDR
- Domain 5: IAM for non-human entities (AI agents), behavioral biometrics, adaptive authentication
- Domain 6: Red teaming for AI systems, model robustness testing
- Domain 7: AI for SOC automation
- Domain 8: AI in SDLC, secure AI coding practices

### Cryptanalytic Attacks (Domain 3.7 — verified listing)
Brute force, Ciphertext only, Known plaintext, Frequency analysis, Chosen ciphertext, Implementation attacks, Side-channel, Fault injection, Timing, Man-in-the-Middle (MITM), Pass the hash, Kerberos exploitation, Ransomware

---

## Finding 2: NIST Risk Management Framework

**Source:** NIST CSRC (last updated March 5, 2026)
**Publication:** SP 800-37 Rev 2

### 7-Step RMF (Verified)
| Step | Key Activity |
|------|-------------|
| **Prepare** | Essential activities to prepare the organization to manage security and privacy risks |
| **Categorize** | Categorize system and information based on an impact analysis |
| **Select** | Select NIST SP 800-53 controls to protect the system based on risk assessments |
| **Implement** | Implement controls and document how they are deployed |
| **Assess** | Determine if controls are in place, operating as intended, producing desired results |
| **Authorize** | Senior official makes a risk-based decision to authorize the system to operate |
| **Monitor** | Continuously monitor control implementation and risks to the system |

### Key Context (Verified)
- "The RMF provides a process that integrates security, privacy, and cyber supply chain risk management activities into the system development life cycle"
- "The risk-based approach to control selection and specification considers effectiveness, efficiency, and constraints due to applicable laws, directives, Executive Orders, policies, standards, or regulations"
- Applicable to new and legacy systems, any system/technology type (IoT, control systems), and any organization size/sector
- Developed by the Joint Task Force (JTF)

### Associated Publications (Verified)
- NIST SP 800-53: Security and Privacy Controls
- RMF Introductory Courses available
- Control Overlay Repository (including AI systems)
- NIST SP 800-53B: Control Baselines

**Status: The existing note `risk-management-frameworks.md` correctly documents the 7-step RMF. Matches source. No update needed.**

---

## Finding 3: Bell-LaPadula Model

**Source:** Wikipedia (last edited Dec 29, 2025)

### Core Rules (Verified)
1. **Simple Security Property (no read up):** A subject at a given security level may not read an object at a higher security level
2. ***-Property / Star Security Property (no write down):** A subject at a given security level may not write to any object at a lower security level
3. **Discretionary Security Property:** Uses an access matrix to specify discretionary access control

### Key Details (Verified)
- **State machine model** for enforcing access control in government/military MLS environments
- Developed by David Elliott Bell and Leonard J. LaPadula with guidance from Roger R. Schell
- Focuses on **confidentiality** (contrasts with Biba model which addresses **integrity**)
- Defined as a formal state transition model: transitions from secure state → secure state
- The notion of "secure state" is defined; each transition preserves security by mathematical induction
- **Trusted subjects** are not restricted by the Star-property; must be shown trustworthy
- **"Write up, read down" (WURD)** — the memorable acronym for BLP rules
- Users can create content only at or above their own security level
- Users can view content only at or below their own security level

### Tranquility Principle (Verified)
1. **Strong tranquility:** Security levels do NOT change during normal system operation
2. **Weak tranquility:** Security levels may never change in a way that violates defined security policy — allows systems to observe principle of least privilege (processes start with low clearance, progressively accumulate higher levels)

### Strong Star Property (Verified)
Alternative to the *-Property: subjects may write to objects with only a matching security level (no write-up, write-to-same only). Anticipated by the Biba model where strong integrity + BLP = reading/writing at a single level.

### Limitations (Verified)
- Only addresses confidentiality, not full integrity
- Covert channels mentioned but not addressed comprehensively
- Tranquility principle limits applicability to systems where security levels don't change dynamically
- Does not treat: covert channels comprehensively, networks of systems, non-MLS policies

**Status: The existing note `security-models.md` should be verified against these formal definitions. The Wikipedia content is consistent with standard CISSP teaching.**

---

## Finding 4: NIST Cybersecurity Framework (CSF) 2.0

**Source:** NIST.gov (main CSF page)
**DOI:** 10.6028/NIST.CSWP.29

### CSF 2.0 Status (Verified)
- CSF 2.0 is the **current version** (celebrated 2nd anniversary Feb 24, 2026)
- CSF 1.1 is now archived
- CSF 2.0 adds the **Govern** function (6 functions total: Govern, Identify, Protect, Detect, Respond, Recover)
- Published as NIST SP 1308 (Quick-Start Guide) and the full document at DOI 10.6028/NIST.CSWP.29

### CSF 2.0 Resources (Verified)
- Quick Start Guides for specific common goals
- CSF 2.0 Profiles (templates for creating and using CSF profiles)
- Informative References / Mappings (overlap with other NIST resources)
- Videos and webinars available
- Ransomware Risk Management profile (NIST IR 8374 Rev 1)
- Workforce Management Quick-Start Guide (SP 1308)

### CSF 2.0 Mappings (Verified)
- SP 800-53 control mappings
- SP 800-70r5 (National Checklist Program) — includes CSF 2.0 outcome mappings
- SP 800-53 compliance + CCE identifiers for evidence-ready automation

**Status: Existing KB notes should reference CSF 2.0 with 6 functions, NOT the older CSF 1.1 with 5 functions.**

---

## Finding 5: ISC2 Code of Ethics

**URL checked:** `https://www.isc2.org/ethics/code-of-ethics`
**Result:** 404 Not Found

The ISC2 Code of Ethics page has been moved. The existing note `isc2-code-of-ethics.md` should note that the official URL may have changed. The four canons remain authoritative as they are the ISC2 Ethics Code and do not change.

---

## Discrepancies Found

| Discrepancy | Source Evidence | Impact | Action |
|---|---|---|---|
| NIST CSF version | NIST.gov confirms CSF 2.0 is current, CSF 1.1 archived | Any note referencing "CSF" should specify 2.0 with 6 functions (Govern added) | Update affected notes |
| ISC2 Ethics URL 404 | verified via browser navigation, returns 404 | The official URL may have changed | Note in `isc2-code-of-ethics.md` |
| AI/ML integration across domains | ISC2 2024 exam outline explicitly states AI security woven into all 8 domains | This is a significant 2024 addition not in older CBK versions | Consider adding AI-specific content to domain hubs |

## Verified (No Change Needed)

| Topic | Source Match |
|---|---|
| Domain weights (16/10/13/13/13/12/13/10) | ✅ Exact match |
| 7-step NIST RMF (Prepare through Monitor) | ✅ Exact match |
| Bell-LaPadula (Simple Security, *-Property, DAC rule) | ✅ Consistent |
| Domain 3.7 cryptanalytic attacks list | ✅ Consistent |
