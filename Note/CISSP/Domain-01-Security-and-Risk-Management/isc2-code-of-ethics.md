---
title: "ISC² Code of Ethics"
layer: 3
domain: 1
tags:
  - cissp
  - domain-01
  - isc2
  - code-of-ethics
  - canons
  - professionalism
  - ethics
related_domains:
  - "[[security-governance-policies-standards]]"
  - "[[risk-management-frameworks]]"
  - "[[legal-regulatory-compliance-landscape]]"
  - "[[gdpr-deep-dive]]"
  - "[[hipaa-security-rule]]"
  - "[[pci-dss-4-0]]"
---

# ISC² Code of Ethics

## Feynman Explanation

The (ISC)² Code of Ethics is the promise a CISSP makes when they get the certification. It is a four-part oath: protect the public first, act honestly, do good work, and help the security profession grow. The first canon is non-negotiable — if the other three conflict with it, the first one wins. Breaking the code can get your CISSP revoked, and for most CISSPs that means their career is over.

## Technical Details

### The Preamble (must be agreed before any canon)

> "The safety and welfare of society and the common good, duty to our principals, and to each other, requires that we adhere, and be seen to adhere, to the highest ethical standards of behavior."

> "Therefore, strict adherence to this Code is a condition of certification."

Every (ISC)² member (CISSP, CCSP, CGRC, CC, SSCP, etc.) must:

- Agree to the Code as a condition of certification.
- Sign an attestation of truth on the application.
- Report violations to (ISC)² Ethics Committee.
- Maintain the Code throughout the certification lifecycle.

### The Four Canons (in mandatory priority order)

| # | Canon | Plain English |
|---|---|---|
| **1** | **Protect society, the common good, necessary public trust and confidence, and the infrastructure.** | The public comes first. Always. |
| **2** | **Act honorably, honestly, justly, responsibly, and legally.** | Don't lie, don't cheat, don't steal. |
| **3** | **Provide diligent and competent service to principals.** | Do good work, not sloppy work. |
| **4** | **Advance and protect the profession.** | Help the next generation of security pros. |

**Order matters.** If Canon 1 (protect society) conflicts with Canon 2 (act legally), Canon 1 wins **only if** breaking the law is the only way to protect society — for example, whistle-blowing on a felony. In the normal case, Canon 2 (act legally) is just doing the right thing.

**CISSP exam trap:** the exam will sometimes present an "ethical dilemma" where all four canons pull in different directions. The answer is **always the one that protects society first**, then honors the principal, then protects the profession.

### Detailed Interpretation of Each Canon

#### Canon 1 — Protect Society

- Don't build systems that hurt people.
- Don't stay silent about known vulnerabilities that put the public at risk.
- Don't help adversaries — even for a "really good paycheck."
- Report illegal activity to the appropriate authorities.
- Disclose conflicts of interest.

**Example:** A CISSP discovers a colleague is selling zero-day exploits to a hostile nation-state. Canon 1 obliges them to report this to (ISC)² and likely to law enforcement.

#### Canon 2 — Act Honorably, Honestly, Justly, Responsibly, and Legally

- Don't falsify reports, audit results, or risk assessments.
- Don't take bribes.
- Don't misrepresent your qualifications or your company's capabilities.
- Honor contracts and NDAs.
- Disclose relevant information — don't hide a known breach to protect a client.
- If a law conflicts with another canon, follow the law **unless** it would clearly cause great harm to society.

**Example:** A CISO is asked to backdate a security assessment report to cover a period before a known breach. Canon 2 forbids this — it is dishonest.

#### Canon 3 — Provide Diligent and Competent Service

- Maintain your competence — keep learning, get CPEs.
- Don't claim expertise you don't have.
- Don't take work you cannot do well.
- Render opinions only when you have reasonable basis.
- Inform principals of conflicts.
- Respect the privacy and confidentiality of principal's information.
- Don't compete unfairly.
- Don't use principal's resources for personal gain.

**Example:** A CISSP is asked to design a SCADA-ICS security architecture but has no ICS experience. Canon 3 obliges them to either get qualified, partner with an expert, or decline the work.

#### Canon 4 — Advance and Protect the Profession

- Sponsor and mentor new professionals.
- Support education and certification in the field.
- Don't commit acts that injure the reputation of other professionals or the profession.
- Don't use the CISSP mark in ways that disparage the profession.
- Don't violate the (ISC)² bylaws.

**Example:** A CISSP publicly denigrates another professional in a dispute on social media. Canon 4 suggests restraint — address the technical disagreement, not the person.

### Common Exam Scenarios

| Scenario | Correct Canon(s) | Reasoning |
|---|---|---|
| Colleague selling exploits to hostile state | **Canon 1** | Public safety supersedes loyalty to colleague |
| Asked to falsify an audit report | **Canon 2** | Dishonest, illegal |
| Asked to do work outside your competence | **Canon 3** | Incompetence harms the principal |
| Witnessed another CISSP lying on a resume | **Canon 2 + 4** | Report to (ISC)² (1, 2, 4) |
| Vendor offers a "thank you" gift worth $5K to influence a contract | **Canon 2** | Bribery |
| Asked to keep a breach secret from regulators | **Canon 1 + 2** | Society + illegal |
| New CISO with no SIEM experience but is told to "figure it out" | **Canon 3** | Get help, don't pretend |

### (ISC)² Ethics Committee Process

1. Complaint filed by member, employer, or third party.
2. (ISC)² Ethics Committee reviews.
3. Member has right to respond.
4. Outcomes:
   - Dismissal (no violation)
   - Letter of censure (private or public)
   - Suspension of certification
   - Permanent revocation of certification
   - Lifetime ban from (ISC)²

Revocation is **public** — your name is removed from the (ISC)² member directory and the certification is recorded as "revoked." This is effectively career-ending for most security professionals.

### Continuing Professional Education (CPE) Requirements (Ethics-Adjacent)

To stay certified, CISSPs must earn:

- **120 CPE credits every 3-year cycle** (40 per year average).
- Of which **at least 20 must be in Group A** (domain-specific, e.g., CISSP CBK domains).
- **Annual maintenance fee** ($135 for CISSP in good standing, with $35 AMF for regional adjustments).
- Failure to meet = certification suspension → revocation if not cured.

The CPE requirement itself is an expression of Canon 3 — keep your competence current.

### Confidentiality of Principal's Information

Canon 3 specifically calls out "the privacy and protect the confidentiality of information of one's principals." This applies even after the engagement ends, and even when the principal becomes hostile or litigious.

**Exception:** if the principal's information is about illegal activity that threatens public safety, Canon 1 trumps Canon 3 — disclose to authorities.

### (ISC)² Code vs Other Professional Codes

| Code | Organization | Emphasizes |
|---|---|---|
| **(ISC)² Code of Ethics** | (ISC)² | Society first, then principal |
| **ISACA Code of Professional Ethics** | ISACA (CISA, CISM, CGEIT, CRISC) | Due care, due diligence, confidentiality, professionalism |
| **EC-Council Code of Ethics** | EC-Council (CEH, CHFI) | Confidentiality, lawful conduct |
| **SANS Code of Ethics** | GIAC certifications | Confidentiality, lawful conduct, public interest |
| **PMI Code of Ethics** | PMI (PMP) | Responsibility, respect, fairness, honesty |
| **IEEE Code of Ethics** | IEEE engineers | Public welfare, competence, honesty |

All share a "society first" core, but the wording and priority differ. CISSP exam: always answer with (ISC)² Code of Ethics, in its 4-canon order.

## CISO / Risk Manager View

**Why the Code of Ethics matters to the CISO, not just the certificate:**

1. **The Code is the only thing between a CISSP and the door.** A revoked CISSP is a fired CISO in most companies. Knowing the Code is not optional.
2. **The Code is a personal shield.** When the CEO tells you to "just sign the SOC 2 report and ship it," the Code is your protection. "Per (ISC)² Code of Ethics Canon 2, I cannot sign an attestation I do not believe is accurate." The Code outranks the org chart.
3. **The Code is a fiduciary anchor.** Many CISSPs are now signing personal attestations for regulatory filings (SOX 404, SEC cyber disclosures). The Code is the foundation of that personal liability framework.
4. **The Code supports the whistleblower path.** A CISSP who reports a covered-up breach is protected by Canon 1 — the (ISC)² Ethics Committee and the courts are increasingly willing to back whistleblowers acting under professional codes.
5. **The Code requires you to maintain competence.** This is the CISO's defense when budget is being cut for training: "Canon 3 requires that I maintain competence. Cutting CPE budget is a Canon 3 violation."
6. **The Code shapes vendor relationships.** A CISSP who takes a $100K "consulting fee" from a vendor is in Canon 2 territory. Many companies have explicit CISSP Code-aligned ethics policies for vendor interactions.

**Practical governance moves:**

- Put the Code in the security team's onboarding deck.
- Reference it in your security policy ("All security personnel shall comply with the (ISC)² Code of Ethics").
- Use it as the language for escalation: "I'm escalating per Canon 1 obligations."
- Include Code attestation in annual reviews.

## Related Connections

### Parent L2
- [[security-governance-policies-standards]] - Code of Ethics is the "soft" governance layer above the policy pyramid
- [[legal-regulatory-compliance-landscape]] - Code principles ground legal compliance (Canon 2)
- [[risk-management-frameworks]] - The Code informs the security professional's risk-decision ethics

### Sibling L3
- [[gdpr-deep-dive]] - Canon 1 supports the GDPR public-interest mission
- [[hipaa-security-rule]] - Canon 1 supports patient safety as public interest
- [[pci-dss-4-0]] - Canon 2 supports honest attestation of PCI compliance

### Cross-Domain
- All other domains inherit the ethical framing of Domain 1.
- [[domain-08-software-development-security]] - Canon 3 applies to AppSec professional competence.
- [[domain-07-security-operations]] - Canon 3 applies to SOC analyst competence and honesty in incident reporting.

## Sources / References

- (ISC)² Code of Ethics - https://www.isc2.org/Ethics
- (ISC)² Bylaws - Article on Ethics
- (ISC)² Ethics Committee Procedures
- CISSP CPE Handbook
- (ISC)² Member Agreement
- CISSP Common Body of Knowledge (CBK), 2024 - Domain 1.1 Professional Ethics
- Harold F. Tipton & Micki Krause - "Information Security Management Handbook"
- (ISC)² white papers on ethical decision-making
