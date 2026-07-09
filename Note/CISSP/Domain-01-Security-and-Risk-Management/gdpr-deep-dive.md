---
title: "GDPR Deep Dive"
layer: 3
domain: 1
tags:
  - cissp
  - domain-01
  - gdpr
  - privacy
  - dpo
  - data-protection
  - eu-regulation
related_domains:
  - "[[legal-regulatory-compliance-landscape]]"
  - "[[security-governance-policies-standards]]"
  - "[[risk-management-frameworks]]"
  - "[[risk-formulas-and-quantitative-analysis]]"
  - "[[hipaa-security-rule]]"
  - "[[pci-dss-4-0]]"
---

# GDPR Deep Dive

## Feynman Explanation

GDPR is the European Union's rulebook for how a company anywhere in the world must handle the personal data of anyone living in the EU. It applies to a US startup with 5 customers in Berlin. The fines are massive — up to 4% of *global* revenue or €20 million, whichever is higher — and they are not theoretical. The point of GDPR is to shift control of personal data back to the individual, and to make the company that touches that data fully responsible for proving it is doing the right thing.

## Technical Details

### Regulation Scope

- **Full name:** Regulation (EU) 2016/679
- **Effective:** 25 May 2018
- **Extraterritorial:** applies to any organization that processes the personal data of EU residents, *or* that offers goods/services to EU residents, *or* that monitors the behavior of EU residents.
- **Territorial:** applies to organizations established in the EU AND to organizations outside the EU that meet the above criteria.

This is the **extraterritoriality** principle — and it is why a US-only SaaS company still needs a GDPR program if it has EU customers.

### Key Definitions

| Term | Definition |
|---|---|
| **Personal Data** | Any information relating to an identified or identifiable natural person |
| **Data Subject** | The identified or identifiable natural person (the individual) |
| **Data Controller** | Entity that determines the purposes and means of processing |
| **Data Processor** | Entity that processes data on behalf of the controller |
| **Processing** | Any operation performed on personal data (collection, storage, use, disclosure, erasure) |
| **Special Categories** | Sensitive data: racial, ethnic, political, religious, genetic, biometric, health, sex life/orientation |
| **Profiling** | Automated processing to evaluate certain personal aspects |
| **Pseudonymization** | Replacing identifiers with tokens, reversible with separate key |
| **Anonymization** | Irreversibly removing identifiers (anonymized data is NOT personal data) |

### Article 5 — Principles of Processing (the "7 principles")

Personal data must be:

1. **Lawful, fair, and transparent** — processed lawfully, fairly, and with clear notice.
2. **Purpose limitation** — collected for specified, explicit, legitimate purposes; no further use incompatible with those purposes.
3. **Data minimization** — adequate, relevant, and limited to what is necessary.
4. **Accuracy** — accurate and, where necessary, kept up to date.
5. **Storage limitation** — kept in a form that permits identification for no longer than is necessary.
6. **Integrity and confidentiality** — processed in a manner that ensures appropriate security.
7. **Accountability** — the controller is **responsible for and able to demonstrate** compliance.

**Exam tip:** "Accountability" is the most-tested GDPR principle. It reverses the burden: the controller must *prove* it is compliant, not the regulator prove it is not.

### Article 6 — Lawful Bases for Processing

| Basis | Example |
|---|---|
| (a) Consent | Marketing opt-in |
| (b) Contract | Fulfilling a purchase order |
| (c) Legal obligation | Tax reporting |
| (d) Vital interests | Life-threatening emergency |
| (e) Public interest | Public health, government |
| (f) Legitimate interests | Fraud prevention (subject to balancing test) |

Consent must be **freely given, specific, informed, and unambiguous**. Pre-ticked boxes are not consent. Silence is not consent. The data subject can withdraw consent at any time, as easily as they gave it.

### Article 17 — Right to Erasure ("Right to be Forgotten")

The data subject can demand erasure when:

- Data is no longer necessary for the purpose it was collected
- Consent is withdrawn
- Processing is unlawful
- Erasure is required by EU or member state law
- The data was collected in relation to information society services offered to a child

**Exceptions:** freedom of expression, legal obligations, public interest in public health, archival/research purposes, establishment/exercise/defense of legal claims.

### Article 25 — Data Protection by Design and by Default

- **By Design:** privacy and data protection must be embedded into processing activities and business practices from the earliest design stage.
- **By Default:** the strictest privacy settings must apply by default — only data necessary for each specific purpose is processed by default.

**CISSP mapping:** this is the legal mandate for "Privacy by Design" and directly drives the controls in [[security-governance-policies-standards]] and [[risk-management-frameworks]].

### Article 30 — Records of Processing Activities (ROPA)

Controllers and processors must maintain a written record of processing activities including:

- Purposes of processing
- Categories of data subjects and personal data
- Categories of recipients
- Transfers to third countries
- Time limits for erasure
- Description of security measures

### Article 32 — Security of Processing

Controllers and processors must implement **appropriate technical and organizational measures (TOMs)** to ensure a level of security appropriate to the risk, including:

- **Pseudonymization and encryption** of personal data
- Ability to ensure **ongoing confidentiality, integrity, availability, and resilience**
- Ability to **restore availability** after an incident (**timely backup + DR**)
- Regular **testing and evaluation** of effectiveness

This is the article that maps directly to ISO 27001 controls and is what the security team actually implements.

### Article 33 — Breach Notification to the Supervisory Authority

- Notify the **supervisory authority within 72 hours** of becoming aware of a breach.
- Notification must describe: nature of the breach, categories and approximate number of data subjects affected, likely consequences, measures taken or proposed.
- If the breach is **unlikely to result in a risk** to the rights and freedoms of data subjects, no notification is required (but must be documented).

### Article 34 — Breach Notification to Data Subjects

If the breach is **likely to result in a high risk** to the rights and freedoms of data subjects, the controller must communicate the breach to the data subject **without undue delay**.

### Article 35 — Data Protection Impact Assessment (DPIA)

Required when processing is **likely to result in a high risk** to data subjects — e.g., large-scale processing of special categories, systematic monitoring of public areas, profiling with legal effects.

The DPIA must:

- Describe the processing and its purposes
- Assess necessity and proportionality
- Assess risks to data subjects
- Describe measures to address the risks

### Article 37 — Data Protection Officer (DPO)

Mandatory DPO appointment when:

- Public authority or body
- Core activities require **regular, systematic, large-scale monitoring** of data subjects
- Core activities consist of **large-scale processing of special categories** of data

The DPO must:

- Be **independent** (no instructions from controller)
- Report directly to the **highest management level**
- Not be dismissed or penalized for performing the role
- Operate without conflicts of interest
- Be reachable by data subjects and the supervisory authority

### Cross-Border Data Transfers

Allowed only to countries with **adequate protection** or via:

- **Standard Contractual Clauses (SCCs)** — pre-approved contract terms
- **Binding Corporate Rules (BCRs)** — for intra-group transfers
- **Codes of Conduct / Certification**
- **Derogations** (consent, contract, public interest) — narrow exceptions

The 2020 **Schrems II** ruling invalidated the EU-US Privacy Shield; transfers to the US now rely on SCCs plus supplementary measures. The EU-US Data Privacy Framework (DPF, 2023) restored an adequacy decision, but legal challenges continue.

### Fines (Article 83)

| Tier | Maximum | Applies to |
|---|---|---|
| **Lower** | €10M or 2% of global annual turnover (whichever higher) | Violations of controller/processor obligations (e.g., Article 25, 32) |
| **Higher** | €20M or 4% of global annual turnover (whichever higher) | Violations of data processing principles, data subject rights, cross-border transfers |

"Fines are effective, proportionate, and dissuasive." Real examples: Amazon €746M (2021), Meta €1.2B (2023), Uber €290M (2024).

### Data Subject Rights (8 rights)

1. **Right to be informed** (transparency)
2. **Right of access** (Article 15)
3. **Right to rectification** (Article 16)
4. **Right to erasure** (Article 17) - "right to be forgotten"
5. **Right to restriction of processing** (Article 18)
6. **Right to data portability** (Article 20) - in machine-readable format
7. **Right to object** (Article 21) - including for direct marketing
8. **Right not to be subject to automated decision-making** (Article 22) - including profiling

## CISO / Risk Manager View

**The CISO's GDPR checklist:**

1. **Data Inventory** — do you know every system that holds personal data? If not, you cannot answer "what data did the breach affect?"
2. **Records of Processing Activities (ROPA)** — the legal team usually owns this, but the CISO must provide the security measures section.
3. **Lawful basis for every processing activity** — if marketing runs on "implied consent" today, that is a fine waiting to happen.
4. **Privacy Impact Assessment (DPIA) process** — embed it in the project intake; if it does not get done, you cannot launch a high-risk project.
5. **72-hour breach response plan** — the 72-hour clock starts when you become "aware," which is usually the moment the SOC confirms a breach, not when it is fully scoped. Practice this.
6. **Cross-border transfer mechanism** — every data flow leaving the EU needs a documented mechanism. Standard Contractual Clauses + Transfer Impact Assessment (TIA) is the new norm post-Schrems II.
7. **Vendor management** — Article 28 requires a written contract with every processor. No vendor stores EU personal data without a signed Data Processing Addendum (DPA).
8. **DPO appointment** — if required, the DPO must be a real, empowered person, not a title on someone's business card.

**Board framing:** GDPR fines are a **secondary loss** in the FAIR model. A €20M fine is the regulator's cost. The *real* cost is the brand damage, the customer churn, and the loss of EU business — which is often 5-10x the fine. Quantify all three.

**Strategic moves:**

- Adopt a **"GDPR-grade" baseline** for *all* customer data. The marginal cost is small, the marginal risk reduction is large.
- Bake privacy into the SDLC ([[domain-08-software-development-security]]) — Article 25 by Design is not a paper exercise, it is a code review checkpoint.
- Train sales and marketing — most GDPR fines start with a complaint about consent or marketing practices, not a hack.

## Related Connections

### Parent L2
- [[legal-regulatory-compliance-landscape]] - GDPR is one of the major privacy laws in the landscape

### Sibling L3
- [[hipaa-security-rule]] - Comparison: HIPAA is US healthcare, GDPR is EU personal data
- [[pci-dss-4-0]] - PCI-DSS handles card data, GDPR handles personal data; both can apply simultaneously
- [[isc2-code-of-ethics]] - Canon 1 "protect society" includes protecting privacy

### Cross-Domain
- [[domain-02-asset-security]] - Data inventory and classification feed Article 30
- [[domain-05-identity-and-access-management]] - "Need to know" enforcement supports Article 32
- [[domain-08-software-development-security]] - Privacy by Design (Article 25) is an SDLC practice
- [[domain-07-security-operations]] - Breach response and 72-hour notification (Articles 33, 34)

## Sources / References

- Regulation (EU) 2016/679 (GDPR) - full text
- EDPB Guidelines (European Data Protection Board)
- ICO (UK Information Commissioner's Office) guidance
- Article 29 Working Party opinions (now EDPB)
- Schrems II - Case C-311/18, Court of Justice of the EU, 16 July 2020
- EU-US Data Privacy Framework (adequacy decision, 10 July 2023)
- (ISC)² CISSP CBK, 2024
- ENISA (European Union Agency for Cybersecurity) - technical guidance
- IAPP (International Association of Privacy Professionals) - CIPP/E certification
