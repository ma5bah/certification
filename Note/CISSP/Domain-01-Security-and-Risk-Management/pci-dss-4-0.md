---
title: "PCI-DSS 4.0"
layer: 3
domain: 1
tags:
  - cissp
  - domain-01
  - pci-dss
  - payment-cards
  - cardholder-data
  - saq
  - compliance
related_domains:
  - "[[legal-regulatory-compliance-landscape]]"
  - "[[security-governance-policies-standards]]"
  - "[[risk-management-frameworks]]"
  - "[[gdpr-deep-dive]]"
  - "[[hipaa-security-rule]]"
---

# PCI-DSS 4.0

## Feynman Explanation

PCI-DSS is the rulebook written by the credit card companies (Visa, MasterCard, Amex, etc.) for anyone who stores, processes, or transmits a credit card number. It is not a law — it is a contract. Sign the merchant agreement, agree to the rules, get to take card payments. Break the rules, lose the right to take card payments (and probably pay a fine). v4.0 is the 2022 refresh that is forcing companies to actually script and test their security controls, not just write them down.

## Technical Details

### What is PCI-DSS?

- **Full name:** Payment Card Industry Data Security Standard
- **Owner:** PCI Security Standards Council (PCI SSC) — founded by AmEx, Discover, JCB, MasterCard, Visa
- **Current version:** **v4.0** (March 2022)
- **Previous:** v3.2.1 (2018)
- **Effective date for v4.0:** March 31, 2024 (v3.2.1 retired March 31, 2024)
- **Future-dated requirements:** v4.0 "best practices" become mandatory **March 31, 2025**

### Scope

Applies to **all entities** that store, process, or transmit **cardholder data (CHD)** or **sensitive authentication data (SAD)**, and to all system components included in or connected to the cardholder data environment (CDE).

The CDE = the people, processes, and technology that store, process, or transmit CHD/SAD, plus any system component connected to or affecting the security of that data.

### Cardholder Data vs Sensitive Authentication Data

| Type | Examples | May Store? | May Encrypt and Store? |
|---|---|---|---|
| **Primary Account Number (PAN)** | 16-digit card number | Yes | Yes (must be unreadable) |
| **Cardholder Name** | John Smith | Yes | Yes |
| **Service Code** | 3-digit | Yes | Yes |
| **Expiration Date** | MM/YY | Yes | Yes |
| **Full Track Data** | Magnetic stripe | **NO** (post-authorization) | No |
| **CVV / CVC / CAV / CID** | 3- or 4-digit code | **NO** (post-authorization) | No |
| **PIN / PIN Block** | Encrypted PIN | **NO** | No |

**The single most-tested PCI rule:** you **cannot store CVV/CVC** even encrypted, even briefly, after authorization. Anything that does, fails.

### The 12 Requirements (v4.0)

Grouped into 6 "Goals":

#### Goal 1: Build and Maintain a Secure Network and Systems

| # | Requirement | What's New in 4.0 |
|---|---|---|
| 1 | Install and maintain network security controls | Replaces "firewall"; broader scope (NSC) |
| 2 | Apply secure configurations to all system components | — |

#### Goal 2: Protect Account Data

| # | Requirement | What's New in 4.0 |
|---|---|---|
| 3 | Protect stored account data | Inventory of trusted keys/certificates; PAN truncation per ISO 7812; hashed PANs |
| 4 | Protect cardholder data with strong cryptography during transmission | Cryptographic suites to be inventoried; TLS 1.2+ required for new implementations (TLS 1.0/1.1 disallowed) |

#### Goal 3: Maintain a Vulnerability Management Program

| # | Requirement | What's New in 4.0 |
|---|---|---|
| 5 | Protect all systems and networks from malicious software | Anti-malware on all systems (not just "commonly affected") |
| 6 | Develop and maintain secure systems and software | Internal vulnerability scans via authenticated scanning; reviewed at least once every 12 months; "bespoke and custom" software requires additional controls (6.4.3) |

#### Goal 4: Implement Strong Access Control Measures

| # | Requirement | What's New in 4.0 |
|---|---|---|
| 7 | Restrict access to system components and cardholder data by business need to know | Role-based; documented access needs |
| 8 | Identify users and authenticate access to system components | MFA for **all access into the CDE** (8.4.2) — big change; strong passwords (12 chars if used, 8 if not; max 90 days); service accounts managed |
| 9 | Restrict physical access to cardholder data | — |

#### Goal 5: Regularly Monitor and Test Networks

| # | Requirement | What's New in 4.0 |
|---|---|---|
| 10 | Log and monitor all access to system components | 12 months of logs; 3 months immediately available |
| 11 | Test security of systems and networks regularly | Internal vulnerability scans (authenticated); external by ASV quarterly; **internal penetration testing annually** (11.4); segmentation tested annually (11.4.5) |

#### Goal 6: Maintain an Information Security Policy

| # | Requirement | What's New in 4.0 |
|---|---|---|
| 12 | Support information security with organizational policies and programs | Targeted risk analysis for each requirement that allows it; service provider responsibility matrix; quarterly security awareness |

### Customized Approach (v4.0's biggest change)

PCI-DSS v4.0 introduces a **"customized approach"** alongside the traditional **"defined approach"** (each requirement met exactly as written).

- **Defined Approach** — implement the requirement exactly as stated (same as v3.2.1).
- **Compensating Control** (legacy) — only for a *missed* requirement.
- **Customized Approach** — meet the **stated objective** of the requirement, but with a different control implementation, documented in a **Targeted Risk Analysis (TRA)**.

This is the SSC's concession that one-size-fits-all doesn't work for cloud and modern architectures.

### Self-Assessment Questionnaire (SAQ) Types

The SAQ is a self-completion tool for smaller merchants; large merchants (Level 1) require an on-site QSA (Qualified Security Assessor) Report on Compliance (ROC).

| SAQ | Use Case | # of Requirements |
|---|---|---|
| **A** | Card-not-present merchants that **fully outsource** all cardholder data functions to a PCI-DSS-compliant third party (no electronic storage, processing, or transmission of CHD). E.g., redirect / iframe / tokenization-only payment page. | ~22 |
| **A-EP** | E-commerce merchants using a third-party processor but who **control the page that redirects** (e.g., the merchant's server has JavaScript that sends PAN to the processor). | ~139 |
| **B** | Merchants with **standalone, dial-out terminal** only (no electronic CHD storage). | ~32 |
| **B-IP** | Merchants using **standalone, PTS-approved payment terminals** with IP connectivity. | ~46 |
| **C-VT** | Merchants using **virtual payment terminal on an isolated computer**. | ~74 |
| **C** | Merchants with **payment application systems connected to the Internet**, no electronic CHD storage. | ~139 |
| **D** | All other merchants and **all service providers**. Full ~300+ requirement assessment. | ~300+ |
| **P2PE-HW** | Merchants using **hardware P2PE (point-to-point encryption) validated solutions**. | ~33 |

**Exam tip:** SAQ A is the smallest. E-commerce with full redirect / iframe qualifies. If the merchant's web server *touches* a real PAN in any way, they are at minimum SAQ A-EP.

### Merchant and Service Provider Levels

| Level | Volume (per brand) | Validation |
|---|---|---|
| **1** | > 6M Visa transactions/year (or any level for compromised entities) | Annual on-site QSA audit, ROC, quarterly ASV scan |
| **2** | 1M - 6M | Annual SAQ, quarterly ASV scan |
| **3** | 20K - 1M e-commerce OR up to 1M other | Annual SAQ, quarterly ASV scan |
| **4** | < 20K e-commerce OR < 1M other | Annual SAQ, quarterly ASV scan (recommended) |

### Network Segmentation

- **Out-of-scope** = systems that do NOT store, process, or transmit CHD AND do NOT connect to any system that does AND do NOT affect the security of the CDE.
- Segmentation **must be tested annually** per v4.0 (11.4.5) — penetration test from out-of-scope to in-scope.
- Common segmentation methods: VLANs, firewalls, separate physical networks, container/micro-segmentation, cloud-native network policies.

### Encryption and Key Management

- **PAN must be unreadable** wherever stored (one-way hash, truncation, strong encryption, index tokens).
- Cryptographic keys must be **inventoried** (3.5.1), **protected** against disclosure/misuse, **rotated** at end of cryptoperiod, and **retired/replaced** when compromised.
- Key custodians should be split — **dual control** and **split knowledge** of keys.
- HSMs (Hardware Security Modules) for top-tier.

### The 6.4.3 "Bespoke and Custom Software" Cluster (v4.0's Most Debated)

For all custom and bespoke software (including internal and third-party-developed), merchants must:

| Sub-req | Requirement |
|---|---|
| 6.4.1 | Address common coding vulnerabilities (OWASP Top 10, SANS CWE Top 25) via training or experience |
| 6.4.2 | **Software development life cycle** that includes security |
| 6.4.3 | **Security testing** (static, dynamic, manual review, IAST) in pre-production **and** post-production |

This is widely seen as the **AppSec** requirement that brings PCI-DSS into the DevSecOps age.

## CISO / Risk Manager View

**CISO's PCI program priorities (for any company that touches cards):**

1. **Minimize CDE scope.** The less you touch, the less you audit. Outsource to a PCI-compliant payment processor with tokenization (e.g., Stripe Elements, Adyen, Checkout.com). Use SAQ A or A-EP. Avoid SAQ D if you can possibly avoid it.
2. **Network segmentation is leverage.** Test it annually. Document it. Without working segmentation, your "office Wi-Fi" is in scope, which is a multi-multiplier on cost.
3. **No storage of SAD (CVV, full track, PIN).** Just say no. Build validation into the application layer. This is the rule auditors check first.
4. **MFA everywhere into the CDE.** v4.0 makes this a hard requirement. Plan for it. If you have not, you have a gap.
5. **Quarterly external ASV scan, internal scans, annual penetration test, annual training, annual risk assessment.** All v4.0. Put them on a calendar. Miss one quarter and you have a finding.
6. **Encryption everywhere — at rest and in transit.** TLS 1.2+ minimum for new. For data at rest, AES-256 with documented key management.
7. **Logging and monitoring — 12 months of audit trail, 3 months immediately available.** This is operational and ongoing; most of the cost of compliance is in maintaining this.
8. **Targeted Risk Analysis (TRA) for customized approach.** If you go custom, document the rationale. The TRA is the evidence.

**Board framing:** PCI non-compliance is one of the few compliance failures with **immediate business consequences**:

- **Forced termination of merchant account** — you cannot take card payments. For e-commerce, this is an existential event.
- **Fines** — $5,000 - $100,000/month from the card brands.
- **Reimbursement** for fraud losses from the breach.
- **Mandatory forensic investigation** (PFI, PCI Forensic Investigator) at your cost ($50K - $200K+).
- **Public listing** on the card brand's list of non-compliant or compromised merchants (Visa's CISP List, MasterCard's MATCH list — once on MATCH, you may not be able to get a new merchant account at all).

The **MATCH** list (MasterCard Alert to Control High-Risk Merchants) is the death penalty. Most acquirers will not onboard a merchant on MATCH.

**Strategic moves:**

- If you are starting a new product, **architect for compliance** from day one. PCI retrofit on a working product is brutal.
- Cloud is your friend — most major cloud payment offerings are PCI-DSS Level 1, and they will share the AOC (Attestation of Compliance) so you can leverage it.
- Treat the QSA as an advisor, not an adversary. The cheaper the assessor, the worse the audit.

## Related Connections

### Parent L2
- [[legal-regulatory-compliance-landscape]] - PCI-DSS is a contractual standard, not a law
- [[security-governance-policies-standards]] - Requirement 12 mandates the policy pyramid
- [[risk-management-frameworks]] - Targeted Risk Analysis is FAIR-adjacent

### Sibling L3
- [[gdpr-deep-dive]] - PAN is personal data, so GDPR + PCI-DSS can both apply
- [[hipaa-security-rule]] - Healthcare + payments = both apply; covered entity + business associate
- [[isc2-code-of-ethics]] - Canon 1 includes protecting financial safety of customers

### Cross-Domain
- [[domain-03-security-architecture-and-engineering]] - Encryption, HSM, key management live here
- [[domain-05-identity-and-access-management]] - MFA, role-based access are core
- [[domain-06-security-assessment-and-testing]] - Scans and pentests are required
- [[domain-08-software-development-security]] - 6.4.3 is essentially a DevSecOps mandate

## Sources / References

- PCI Security Standards Council - https://www.pcisecuritystandards.org
- PCI-DSS v4.0 document (March 2022)
- PCI-DSS v4.0 Summary of Changes
- PCI-DSS Quick Reference Guide
- PCI SSC SAQ Instructions and Guidelines
- ROC Reporting Template
- MasterCard Site Data Protection (SDP) Program
- Visa Cardholder Information Security Program (CISP)
- (ISC)² CISSP CBK, 2024
- OWASP Top 10 (referenced in 6.4.1)
- SANS CWE Top 25 Most Dangerous Software Errors
