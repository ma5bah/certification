---
title: "Legal, Regulatory, and Compliance Landscape"
layer: 2
domain: 1
tags:
  - cissp
  - domain-01
  - legal
  - compliance
  - gdpr
  - hipaa
  - sox
  - glba
  - pci-dss
  - fedramp
  - fisma
related_domains:
  - "[[security-governance-policies-standards]]"
  - "[[risk-management-frameworks]]"
  - "[[gdpr-deep-dive]]"
  - "[[hipaa-security-rule]]"
  - "[[pci-dss-4-0]]"
---

# Legal, Regulatory, and Compliance Landscape

## Feynman Explanation

Different industries have different rules because they handle different kinds of sensitive data — banks handle money, hospitals handle bodies, governments handle secrets, retailers handle credit cards. A CISO must know which laws apply to *their* company's data, where the data lives, and where the customers are, because a single piece of personal data can be regulated by 5 different laws at once. Missing one is not "we'll fix it next quarter" — it's a fine, a lawsuit, or jail time.

## Technical Details

### Categories of Law (CISSP categorization)

| Category | Purpose | Examples |
|---|---|---|
| Criminal | Punish wrongdoing against society | Computer Fraud and Abuse Act (US), Computer Misuse Act (UK) |
| Civil | Resolve disputes between parties | Contract law, tort law (negligence) |
| Administrative / Regulatory | Government oversight of industry | HIPAA, SOX, GLBA |
| Intellectual Property | Protect creations of the mind | Copyright, Patent, Trademark, Trade Secret |
| Privacy | Govern personal data | GDPR, CCPA, PIPEDA |
| Industry-specific | Sector rules | PCI-DSS, NERC CIP, FERPA, COPPA |

### Major US Federal Laws (Quick Map)

| Law | Year | Scope | Penalty |
|---|---|---|---|
| **CFAA** - Computer Fraud and Abuse Act | 1986 | Federal computer crime, "unauthorized access" | Up to 20 years prison |
| **ECPA** - Electronic Communications Privacy Act | 1986 | Wiretaps, email interception | Civil + criminal |
| **HIPAA** - Health Insurance Portability and Accountability Act | 1996 | PHI, healthcare | Up to $1.5M/yr per violation, criminal |
| **GLBA** - Gramm-Leach-Bliley Act | 1999 | Financial PII, financial institutions | Up to $100K/violation/day |
| **SOX** - Sarbanes-Oxley Act | 2002 | Public company financial reporting | Up to $5M prison, $25M fine |
| **FISMA** - Federal Information Security Management Act | 2002 | Federal agencies | Loss of budget authority |
| **FERPA** - Family Educational Rights and Privacy Act | 1974 | Student education records | Loss of federal funding |
| **COPPA** - Children's Online Privacy Protection Act | 1998 | Children under 13 data | Up to $50K/violation |
| **CAN-SPAM** | 2003 | Commercial email | Up to $51K/email |
| **Patriot Act** | 2001 | National security, surveillance | Civil liberty controversies |
| **DMCA** - Digital Millennium Copyright Act | 1998 | Copyright, anti-circumvention | Civil + criminal |

### International / Regional Laws

| Law | Region | Scope | Key Feature |
|---|---|---|---|
| **GDPR** | EU/EEA + extraterritorial | Personal data of EU residents | 4% global revenue or €20M |
| **UK GDPR + DPA 2018** | UK | Mirrors GDPR post-Brexit | Similar fines |
| **PIPEDA** | Canada | Personal info in commercial activity | Consent + accountability |
| **LGPD** | Brazil | Personal data | 2% revenue, up to R$50M per violation |
| **PIPL** | China | Personal data, China-based | Cross-border transfer restrictions |
| **PDPA** | Singapore / Thailand | Personal data | Consent + notification |
| **Privacy Act** | Australia | Personal data, 13 Australian Privacy Principles | Penalties up to A$50M |
| **CCPA / CPRA** | California (US state) | Consumer personal info | $7,500 per intentional violation |
| **NYDFS 23 NYCRR 500** | NY State | Financial services | $250K/yr, 60-day breach notice |

### Industry Standards (Not Laws, But Mandatory By Contract)

| Standard | Sector | Mandatory Because |
|---|---|---|
| **PCI-DSS 4.0** | Card payments | Contract with card brands |
| **FedRAMP** | US federal cloud | Government contract |
| **SOC 2** | Service organizations | Customer contract |
| **ISO 27001** | International | Customer contract, global |
| **HITRUST CSF** | Healthcare US | Often required by hospitals |
| **NIST CSF** | Cross-industry | Voluntary but de-facto standard |
| **CIS Controls (v8)** | Cross-industry | Voluntary but widely required |

### Intellectual Property

| Type | Protects | Duration | Example |
|---|---|---|---|
| **Copyright** | Original creative works (code, art, music) | Life + 70 years | Software source code |
| **Patent** | Inventions, processes | 20 years from filing | A new encryption algorithm |
| **Trademark** | Brand identifiers | Renewable, indefinite | Nike swoosh, logos |
| **Trade Secret** | Confidential business info | Indefinite if kept secret | KFC recipe, algorithm details |

**CISSP gotcha:** Software is *automatically* copyrighted the moment it is fixed in a tangible medium. You do NOT need to register a copyright in the US, but registration gives you statutory damages.

**Software licensing models:**

- **Proprietary** — single vendor owns source (e.g., Microsoft Office)
- **Open Source** — varies (GPL, MIT, Apache, BSD)
- **Freeware / Shareware** — model-based distribution
- **SaaS** — subscription, no local license

### Privacy Concepts

| Concept | Definition |
|---|---|
| **PII** (Personally Identifiable Information) | Any data that identifies a person |
| **PHI** (Protected Health Information) | PII + health data (HIPAA scope) |
| **Cardholder Data (CHD)** | Primary account number, cardholder name, expiry, service code |
| **Sensitive PII** | PII that alone identifies (SSN, biometric, medical, financial account) |
| **Data Subject** | The person the data is about (GDPR term) |
| **Data Controller** | Determines purpose and means of processing (GDPR) |
| **Data Processor** | Processes on behalf of the controller (GDPR) |

### Contract Law Essentials

| Element | Meaning |
|---|---|
| **Offer** | One party proposes terms |
| **Acceptance** | Other party agrees to those exact terms |
| **Consideration** | Something of value exchanged |
| **Capacity** | Both parties legally able to contract |
| **Legal purpose** | Contract is for a legal activity |

**Without all 4, the contract is not enforceable.**

### Types of Investigation

| Type | Goal | Authority |
|---|---|---|
| **Criminal** | Punish, prosecute | Law enforcement, prosecutor |
| **Civil** | Resolve dispute, damages | Attorneys, plaintiff |
| **Regulatory** | Enforce regulation | Regulator (e.g., HHS for HIPAA) |
| **Operational** | Internal review, root cause | Internal security/HR |
| **Electronic Discovery (eDiscovery)** | Evidence for litigation | Litigation hold, attorneys |

**Order of volatility (collect evidence here, in this order):** CPU registers → cache → RAM → disk → logs → archival → offsite.

### Software Licensing Pitfalls

- **Audit clauses:** most enterprise contracts let the vendor audit your usage; running unlicensed copies = breach.
- **Open source license contamination:** GPL code in a proprietary product can force the entire product to be open-sourced.
- **SaaS subscription terms:** data ownership on termination must be explicit.

## CISO / Risk Manager View

**The CISO's "Compliance Matrix" question:**

> "Of all the data we hold, which laws apply, and what is our exposure?"

The answer is rarely one law. A typical mid-size US company might face:

| Data Type | Law | Penalty Exposure |
|---|---|---|
| Employee data (US) | State breach laws, GLBA, HIPAA if health plan | $100K/violation |
| Customer credit cards | PCI-DSS + state laws | Fines + loss of merchant status |
| EU customer data | GDPR | 4% of global revenue |
| Financial filings | SOX | Prison for executives |
| Healthcare data (US) | HIPAA | $1.5M/yr/category, criminal |
| Children under 13 | COPPA + state | $50K/violation |

**Strategic moves:**

1. **Map data → laws first.** Do not start with the law catalog; start with the actual data inventory and tag each class to applicable law.
2. **Outsource the legal opinion, own the operational compliance.** CISO ≠ General Counsel. The GC interprets, the CISO implements.
3. **Document the security program against the most stringent law, and the others usually fall in line.** A HIPAA-grade program typically satisfies GLBA; a PCI-DSS-grade program typically satisfies state breach laws.
4. **Train the board on personal liability.** SOX section 802, HIPAA criminal penalties, GDPR DPO responsibility — these can land executives in prison, not just the company in fines.

## Related Connections

### Sibling L2
- [[security-governance-policies-standards]] - Compliance is enforced via the policy pyramid
- [[risk-management-frameworks]] - Many frameworks are derived from laws (NIST RMF from FISMA)
- [[business-continuity-and-disaster-recovery]] - Some laws mandate BC/DR (SOX, HIPAA)
- [[risk-formulas-and-quantitative-analysis]] - Fines are a secondary loss input

### L3
- [[gdpr-deep-dive]] - Most consequential privacy law
- [[hipaa-security-rule]] - Most-tested healthcare regulation
- [[pci-dss-4-0]] - Most-tested retail/payment regulation
- [[isc2-code-of-ethics]] - Canons (1) "protect society" is grounded in legal compliance

### Cross-Domain
- [[domain-02-asset-security]] - Data classification drives law applicability
- [[domain-07-security-operations]] - Investigations and evidence handling live in ops
- [[domain-08-software-development-security]] - Software licensing and IP live at the SDLC level

## Sources / References

- (ISC)² CISSP CBK, 2024 - Legal, Regulations, Investigations, and Compliance domain
- US Code Title 18 §1030 - Computer Fraud and Abuse Act
- 45 CFR Parts 160, 162, 164 - HIPAA Privacy and Security Rules
- Regulation (EU) 2016/679 - General Data Protection Regulation
- PCI Security Standards Council - PCI-DSS v4.0
- NIST SP 800-66 Rev. 2 - HIPAA Security Rule implementation
- 17 USC - Copyright law
- 35 USC - Patent law
- NYDFS 23 NYCRR 500 (2017, amended 2023)
- Health Insurance Portability and Accountability Act of 1996 (HIPAA)
- Sarbanes-Oxley Act of 2002 (SOX), Section 404
- Gramm-Leach-Bliley Act of 1999 (GLBA)
