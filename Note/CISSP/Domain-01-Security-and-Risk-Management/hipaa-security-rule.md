---
title: "HIPAA Security Rule"
layer: 3
domain: 1
tags:
  - cissp
  - domain-01
  - hipaa
  - healthcare
  - phi
  - safeguards
  - breach-notification
  - hitech
related_domains:
  - "[[legal-regulatory-compliance-landscape]]"
  - "[[security-governance-policies-standards]]"
  - "[[business-continuity-and-disaster-recovery]]"
  - "[[gdpr-deep-dive]]"
  - "[[pci-dss-4-0]]"
---

# HIPAA Security Rule

## Feynman Explanation

HIPAA is the US federal law that protects your medical records from being read by anyone who is not treating you. The HIPAA Security Rule is the technical chapter that says "if you are a doctor, hospital, or anyone who touches that data, you must put specific safeguards in place — administrative rules, physical locks, and technical controls." The breach notification rule is the part that says "and if it leaks anyway, you have 60 days to tell the government and the patients."

## Technical Details

### Statutory Framework

| Law | Year | Purpose |
|---|---|---|
| **HIPAA** (Public Law 104-191) | 1996 | Healthcare portability, fraud, and data protection |
| **HIPAA Privacy Rule** (45 CFR §164.500-534) | 2000/2003 | Governs use/disclosure of PHI |
| **HIPAA Security Rule** (45 CFR §164.302-318) | 2003/2013 | Safeguards for **ePHI** (electronic PHI) |
| **HITECH Act** | 2009 | Strengthened enforcement, breach notification, business associates |
| **Omnibus Rule** | 2013 | Implemented HITECH; expanded BAA requirements |
| **HIPAA Modernization** (proposed) | 2024+ | Updates to Security Rule (proposed) |

### Who Must Comply

| Entity Type | Examples |
|---|---|
| **Covered Entity (CE)** | Healthcare providers, health plans, healthcare clearinghouses |
| **Business Associate (BA)** | Vendor that creates, receives, maintains, or transmits PHI on behalf of a CE (e.g., cloud provider, billing service, IT managed services) |
| **Subcontractor** | Vendor of a Business Associate that also handles PHI |

**If you store PHI for a hospital, you are a Business Associate and must sign a Business Associate Agreement (BAA).**

### Protected Health Information (PHI)

PHI is **individually identifiable health information** held or transmitted by a CE or BA, in any form, where the individual can be identified. 18 HIPAA identifiers (must be removed for "de-identified" data):

| # | Identifier | # | Identifier |
|---|---|---|---|
| 1 | Names | 10 | Certificate/license numbers |
| 2 | Geographic subdivisions smaller than state | 11 | Vehicle identifiers |
| 3 | Dates (except year) related to individual | 12 | Device identifiers |
| 4 | Telephone numbers | 13 | URLs |
| 5 | Fax numbers | 14 | IP addresses |
| 6 | Email addresses | 15 | Biometric identifiers |
| 7 | Social security numbers | 16 | Full-face photographs |
| 8 | Medical record numbers | 17 | Any other unique identifier |
| 9 | Health plan beneficiary numbers | 18 | Demographic info (if identity could be revealed) |

De-identified data per the Safe Harbor or Expert Determination method is **not PHI** and is not subject to HIPAA.

### The Three Safeguard Categories (Security Rule)

| Category | Number | Type | Examples |
|---|---|---|---|
| **Administrative** | 9 standards | Required (mostly) | Risk analysis, sanction policy, training, incident procedures, contingency plan, evaluation |
| **Physical** | 4 standards | Required | Facility access controls, workstation use, workstation security, device and media controls |
| **Technical** | 5 standards | Required | Access control, audit controls, integrity, person/entity authentication, transmission security |
| **Organizational** | (subset) | Required (docs) | BAA, workforce clearance, workforce training, breach notification |
| **Policies/Procedures/Documentation** | 1 | Required | Written policies, retention (6 years), updates |

### Administrative Safeguards (§164.308) — 9 Standards

| # | Standard | Required / Addressable |
|---|---|---|
| 1 | Security Management Process (risk analysis, risk management) | **Required** |
| 2 | Assigned Security Responsibility | **Required** (e.g., Security Officer) |
| 3 | Workforce Security (authorization/supervision, termination, clearance) | Addressable |
| 4 | Information Access Management (BAA, access authorization, access establishment/modification) | **Required** |
| 5 | Security Awareness and Training (login monitoring, password mgmt, awareness, protection from malicious software, log-in monitoring) | **Required** |
| 6 | Security Incident Procedures (response and reporting) | **Required** |
| 7 | Contingency Plan (data backup, disaster recovery, emergency mode, testing/revision, applications/data criticality analysis) | **Required** |
| 8 | Evaluation (periodic technical and non-technical evaluation) | **Required** |
| 9 | Business Associate Contracts (BAA) | **Required** |

**Note on "Addressable":** Addressable does not mean optional. It means "either implement it, or document why an alternative is reasonable and appropriate." Many CISSP exam questions trip on this.

### Physical Safeguards (§164.310) — 4 Standards

| # | Standard | Required / Addressable |
|---|---|---|
| 1 | Facility Access Controls (contingency operations, facility security plan, access control & validation, maintenance records) | **Required** |
| 2 | Workstation Use | **Required** |
| 3 | Workstation Security | **Required** |
| 4 | Device and Media Controls (disposal, media re-use, accountability, data backup/storage) | **Required** |

### Technical Safeguards (§164.312) — 5 Standards

| # | Standard | Required / Addressable |
|---|---|---|
| 1 | Access Control (unique user ID, emergency access, automatic logoff, encryption/decryption) | **Required** for unique user ID, emergency access; Addressable for logoff, encryption |
| 2 | Audit Controls (hardware, software, procedural mechanisms) | **Required** |
| 3 | Integrity (PHI not altered or destroyed in unauthorized manner) | **Required** |
| 4 | Person or Entity Authentication (verify the person is who they claim) | **Required** |
| 5 | Transmission Security (integrity controls, encryption) | Addressable |

**Note:** "Addressable" for encryption means the CISO can document a business reason *not* to encrypt at rest, but it is on them to defend. Most BAs just encrypt everything because the alternative is a hard sell to regulators.

### Breach Notification Rule (45 CFR §164.400-414)

A breach is an **unauthorized acquisition, access, use, or disclosure** of unsecured PHI that compromises PHI's privacy or security.

**Unsecured PHI** = PHI that is not encrypted or destroyed (per NIST guidance).

| Recipient | Notice Required | Deadline |
|---|---|---|
| **Affected Individuals** | Yes | **60 days** from discovery; sooner if possible |
| **Media** (if ≥ 500 individuals affected) | Yes | Prominently on website within 60 days |
| **HHS Secretary** | Yes | **60 days** (≥ 500 affected) or **annually** (< 500 affected) |
| **State Attorneys General** | Yes (varies) | Per state law |

Required content of individual notice:
- Description of what happened
- Types of information involved
- Steps individuals should take
- Steps the entity is taking
- Contact procedures

**Safe harbor:** if PHI was encrypted (NIST-approved algorithm) at the time of breach, no notification is required. This is why encryption is *the* single most useful HIPAA control.

### HITECH Act Effects

- **Direct liability for Business Associates** (post-Omnibus, 2013) — BAs are directly liable for many Security Rule provisions.
- **State attorneys general** can sue on behalf of residents.
- **Increased penalties** — tiered system (up to $1.5M per violation per year, per identical provision).
- **Breach notification** formalized (BAs must notify the CE; CEs notify the individuals).
- **Meaningful Use / Promoting Interoperability** — encourages EHR adoption, drives more PHI in digital form.

### Penalty Tiers (per violation, per year, per provision)

| Tier | Culpability | Min | Max |
|---|---|---|---|
| 1 | Unknowing | $137/violation | $34,464/violation; $2,067,813 annual cap for all violations of identical provision |
| 2 | Reasonable cause | $1,379/violation | $68,928/violation; $2,067,813 annual cap |
| 3 | Willful neglect — corrected | $13,785/violation | $206,798/violation; $2,067,813 annual cap |
| 4 | Willful neglect — not corrected | $68,928/violation | $2,067,813/violation; **$8,271,814 annual cap** |

Adjusted annually for inflation. **Criminal penalties** (separate from HHS civil): up to $250K fine and 10 years prison for violations involving intent to sell/transfer/use PHI for personal gain or commercial advantage.

## CISO / Risk Manager View

**CISO's HIPAA program priorities:**

1. **Encrypt everything.** Storage, transit, backup media, mobile devices. The single biggest liability shield is encryption — breaches of encrypted PHI don't trigger notification.
2. **Sign BAAs before you touch PHI.** No BAA, no PHI. Period. This includes cloud providers, transcription services, and even IT support firms.
3. **Run the required risk analysis (annual).** The Office for Civil Rights (OCR) opens investigations based on complaints, breach reports, and missing risk analyses. The number-one "finding" is "no documented risk analysis." It is the first thing they ask for.
4. **Train the workforce (annual minimum).** Document it. Sanction policy must be in place *and* used.
5. **Test the contingency plan (annually).** HIPAA-required. Many organizations fail at this.
6. **Document, document, document.** If a breach occurs, the regulator's first question is "show me your last risk analysis, your last training records, your last contingency plan test." If those are missing, the penalty is automatically tier 3 or 4.
7. **Minimize PHI in systems.** "Minimum necessary" rule — access is restricted to the minimum necessary to accomplish the purpose.

**Board-level framing:** HIPAA compliance is not a security program, it is a *legal* program with security implications. The covered entity (e.g., the hospital) carries the ultimate responsibility; the CISO is the operational implementer, but the CEO is the named "Security Officer" in many organizations. The CEO goes to prison, not the CISO, if the program is fraudulent.

**Real-world consequence examples:**

- Anthem (2015): 78.8M records, $16M settlement (pre-tiers), $115M total.
- Excellus (2015): 10M records, $5.1M settlement.
- Premera (2015): 11M records, $6.85M settlement.
- Cignet Health (2011): refusal to provide records, $4.3M.
- L.A. County DHS (2016): accessed PHI for celebrity patients, $865K settlement.

The regulators don't have to catch the breach. They can also catch a missing risk analysis or a missing BAA.

## Related Connections

### Parent L2
- [[legal-regulatory-compliance-landscape]] - HIPAA is one of the major US healthcare laws
- [[security-governance-policies-standards]] - The three safeguards map to the policy pyramid
- [[business-continuity-and-disaster-recovery]] - Contingency Plan is a HIPAA Administrative Safeguard

### Sibling L3
- [[gdpr-deep-dive]] - GDPR is broader (all personal data), HIPAA is narrower (PHI only)
- [[pci-dss-4-0]] - PCI-DSS is card data; if a hospital takes card payments, both apply
- [[isc2-code-of-ethics]] - Canon 1 "protect society" includes protecting patient safety

### Cross-Domain
- [[domain-05-identity-and-access-management]] - Minimum necessary / role-based access is core
- [[domain-06-security-assessment-and-testing]] - Periodic technical evaluation is required
- [[domain-07-security-operations]] - Audit controls, incident procedures, contingency
- [[domain-08-software-development-security]] - Application security for clinical/EHR systems

## Sources / References

- 45 CFR Parts 160, 162, 164 - HIPAA Privacy and Security Rules
- HITECH Act (Title XIII of Division A and Title IV of Division B of the American Recovery and Reinvestment Act of 2009)
- HHS Office for Civil Rights (OCR) - enforcement and guidance
- NIST SP 800-66 Rev. 2 - HIPAA Security Rule implementation guide
- NIST SP 800-111 - Guide to Storage Encryption Technologies
- HIPAA Modernization NPRM (proposed, 2024)
- (ISC)² CISSP CBK, 2024
- HICP (Health Industry Cybersecurity Practices) - HHS/HSCC publication
- AHIMA (American Health Information Management Association)
- HIMSS (Healthcare Information and Management Systems Society)
