---
title: Data Privacy and Protection
layer: 2
domain: 2
tags: [cissp, privacy, pii, phi, pci, gdpr, anonymization, pseudonymization, tokenization]
related_domains: [1, 3, 5, 7, 8]
parent: domain-02-asset-security
---

# Data Privacy and Protection

## Feynman Explanation

A piece of paper with your name, address, and Social Security Number on it is a **key to your bank account, your credit, your medical file, and your identity**. Whoever holds that paper can become you. Privacy is the discipline of **knowing which pieces of data are dangerous, minimizing how many dangerous pieces you hold, and making the ones you must hold unreadable to anyone but their owner.** CISSP separates three techniques — anonymization (no way back), pseudonymization (fake name, real person underneath), and tokenization (swap the number for a look-alike that means nothing outside the vault) — because they have radically different legal and security consequences.

## Technical Details

### The Regulated Data Types

| Type | Definition | Key Regulation | Per-Record Breach Cost (2023) |
|---|---|---|---|
| **PII** — Personally Identifiable Information | Any data that identifies a natural person | GDPR, CCPA/CPRA, US state laws (all 50) | ~$165 |
| **PHI** — Protected Health Information | Health info linked to an individual | HIPAA (US), GDPR Art. 9 (special category) | $400+ |
| **PCI** — Payment Card Industry data | Primary Account Number (PAN), CVV, magstripe | PCI-DSS v4.0 | $200–300 |
| **SPI** — Sensitive Personal Information (CA) | SSN, login, geolocation, racial, religious | CCPA/CPRA | varies |
| **NPI** — Nonpublic Personal Information | US financial services consumer data | GLBA | varies |
| **Children's data** | Under 13 (COPPA) / 16 (GDPR) | COPPA, GDPR-K, AADC | regulatory action |
| **Biometric** | Fingerprint, face, iris, voice | BIPA (IL), GDPR Art. 9 | high — irrevocable |
| **Government identifiers** | SSN, passport, driver's license | multiple | high |

### Identification, De-identification, Re-identification

Privacy is a **gradient**, not a binary. The same dataset can be identifiable or not, depending on context.

| Concept | Identifying? | Reversible? | Under GDPR? |
|---|---|---|---|
| **Identified data** | Direct identifiers present | n/a | Personal data — full GDPR |
| **Pseudonymized data** | Direct identifiers replaced with token; **key held separately** | Yes, with the key | **Still personal data** (Art. 4(5)) |
| **Anonymized data** | Cannot reasonably re-identify by any means | **No** | **Out of GDPR scope** |
| **Aggregated data** | Only statistical summaries | No | Out of scope if k-anonymous |
| **Synthetic data** | Generated from model; no real records | No | Out of scope if truly synthetic |

> **CISSP-critical distinction.** Anonymization is **irreversible**; pseudonymization is **reversible with effort**. Regulators reward irreversible anonymization with freedom from most obligations; pseudonymization is treated as a security control but **not** as a get-out-of-GDPR card.

### Anonymization Techniques

| Technique | Mechanism | Strength | Weakness |
|---|---|---|---|
| **Suppression** | Drop identifying columns | Strong for the dropped fields | Loses utility |
| **Masking** | Replace values (e.g., `***-**-1234`) | Easy to display | Predictable; weak against inference |
| **Generalization** | Replace exact value with range (`age: 27 → 20–30`) | Preserves utility | Still re-identifiable with aux data |
| **Perturbation** | Add noise to numeric values | Preserves statistical properties | Vulnerable to averaging attacks |
| **Swapping / shuffling** | Mix values across records | Preserves column distributions | Re-identification still possible with unique combinations |
| **k-anonymity** | Ensure every record is indistinguishable from at least k−1 others | Strong at k≥5 | Vulnerable to homogeneity / background knowledge |
| **l-diversity** | Each equivalence class has ≥l distinct sensitive values | Stronger than k-anonymity | Vulnerable to skewness attack |
| **t-closeness** | Distribution of sensitive values within each class is close to global | Even stronger | Computational cost |
| **Differential privacy** | Add calibrated noise; provable privacy budget ε | Mathematically provable | ε is a budget; cumulative queries erode privacy |

> **The de-anonymization war is real.** AOL (2006), Netflix Prize (2007), and the Massachusetts Governor (1997) re-identifications proved that "anonymized" is a marketing term, not a guarantee. Always use a **threat model**.

### Pseudonymization

Replace direct identifiers with a token; the mapping is held in a **separate vault** with strict access control.

```
Original:  John Smith, SSN 123-45-6789, DOB 1980-04-12
Pseudonym: USER-7F3A, SSN <token-A>, DOB <token-B>
Mapping:   USER-7F3A → {John Smith, 123-45-6789, 1980-04-12} (in HSM)
```

| Property | Detail |
|---|---|
| Reversibility | Yes, with the key — by **authorized** parties only |
| Data utility | High — joins across datasets still work via the pseudonym |
| GDPR status | **Still personal data**; pseudonymization is a **security measure** (Art. 32), not anonymization |
| Use cases | Research datasets, internal analytics, single sign-on identifiers |

### Tokenization

A token is a **non-sensitive substitute** that **has no mathematical relationship** to the original. Tokenization systems are typically **vault-based** (the actual PAN lives in a hardened token vault; the merchant never sees it).

| Property | Detail |
|---|---|
| Reversibility | Only via the token vault — not by algorithm |
| Data utility | Limited — token does not preserve format unless designed to (format-preserving tokenization, FPE) |
| PCI scope reduction | **The single largest PCI-DSS scope-reduction lever** — a tokenized PAN is **out of scope** for PCI requirements 3, 4, and most of 6 |
| Use cases | Payments, loyalty IDs, account numbers, anything going to non-PCI environments |

> **CISSP exam tip.** Tokenization is for **specific high-value fields** (card numbers, SSNs). Pseudonymization is for **whole records** in research/analytics. Anonymization is for **datasets you want to share broadly** with no re-identification risk.

### The Three Side-by-Side

| Property | Anonymization | Pseudonymization | Tokenization |
|---|---|---|---|
| Reversible | No | Yes (with key) | Yes (via vault) |
| Preserves data utility | Varies | High | Medium (FPE = high) |
| GDPR status | Out of scope | Personal data | Personal data |
| Reduces PCI scope | n/a | Partially | Yes (significant) |
| Implementation | Algorithmic + noise | Algorithm + key vault | Vault + lookup |
| Best for | Open data, research | Internal analytics | Payments, identifiers |

### Cross-Border Data Transfer

| Mechanism | Region | Notes |
|---|---|---|
| **Adequacy decision** | EU ↔ approved countries (e.g., Japan, UK, NZ, Switzerland, South Korea) | Full data flow allowed |
| **Standard Contractual Clauses (SCCs)** | EU → anywhere | Contract + Transfer Impact Assessment (TIA) post-Schrems II |
| **Binding Corporate Rules (BCRs)** | Intra-group, EU → non-EU | Approved by lead DPA; takes 12–24 months |
| **Derogations (Art. 49)** | Consent, contract performance, public interest | **Narrowly interpreted** after Schrems II |
| **Data Privacy Framework (DPF)** | EU ↔ US (2023) | Successor to Privacy Shield; requires US self-certification |
| **Regional data residency** | China, Russia, India, Indonesia | Local storage and processing required |

### Privacy-Enhancing Technologies (PETs)

| PET | What It Does | Use Case |
|---|---|---|
| **Homomorphic encryption** | Compute on encrypted data | Outsourced analytics on encrypted data |
| **Secure Multi-Party Computation (SMPC)** | Multiple parties compute a function without revealing inputs | Joint fraud detection between banks |
| **Zero-Knowledge Proofs (ZKP)** | Prove a statement without revealing the data | Passwordless auth, age verification |
| **Federated learning** | Train ML models without centralizing data | Healthcare AI |
| **Trusted Execution Environments (TEE / SGX)** | Hardware-isolated compute | Confidential computing in cloud |
| **Differential privacy** | Provable privacy budget for aggregates | Census, telemetry |

## CISO / Risk Manager View

**Why this matters at the board level.** Privacy has shifted from a compliance checkbox to a **strategic business issue**. GDPR fines have exceeded **€5.88 billion cumulative through 2024** (top 10 fines alone). CCPA/CPRA carries **$2,500 per violation, $7,500 per intentional violation or violation involving a minor**. Class-action exposure for biometric (BIPA) breaches is **$1,000–$5,000 per violation**, and several settlements have crossed $100M. The board now treats privacy the same way it treats financial controls: a material risk with quarterly reporting.

**Privacy as a competitive advantage.** Apple, Microsoft, and Salesforce use privacy posture (differential privacy, on-device processing) as a *marketing differentiator*. A documented privacy-by-design program reduces marketing-friction and procurement friction (every enterprise RFP now asks for SOC 2 + ISO 27701 + DPF + DPIA template).

**Privacy engineering ROI.** Tokenization alone has saved merchants **billions in PCI scope reduction** — the average enterprise reduces its PCI audit scope by 60–90% by tokenizing PANs at the point of capture.

**Strategic actions.**

1. **Appoint a Data Protection Officer (DPO)** if you process EU resident data at scale, or if HIPAA / GLBA / BIPA applies. The DPO reports to the board, not the CIO.
2. **Maintain a Record of Processing Activities (RoPA)** — GDPR Art. 30 mandate. This is the inventory that ties privacy to [[asset-inventory-and-categorization]].
3. **Default to data minimization + pseudonymization.** If you don't have the data, it can't leak, and you don't have to defend it.
4. **Tokenize the high-risk fields.** PAN, SSN, bank account, biometric — tokenize at the point of capture; downstream systems never see the clear value.
5. **Conduct Data Protection Impact Assessments (DPIAs)** for every new processing activity involving special categories. Build the template into the change-management workflow.
6. **Train the workforce on de-identification vs. re-identification.** Marketing and analytics teams are the highest-risk consumers of "anonymized" data.

## Related Connections

### Sibling L2
- [[data-classification-and-handling]] — privacy data defaults to Tier 3–4
- [[data-lifecycle-and-remnants]] — retention, erasure, and crypto-shred are privacy obligations
- [[asset-inventory-and-categorization]] — the RoPA is the privacy inventory

### L3 Standards
- [[data-loss-prevention-dlp]] — enforces the privacy boundary at egress
- [[cloud-asset-security-shared-responsibility]] — cloud vendor's privacy posture is your posture
- [[data-retention-and-disposal-nist-800-88]] — disposal is also an erasure obligation
- [[information-handling-standards-iso-27001-annex-a]] — ISO 27701 is the privacy extension; A.5.34, A.8.10–A.8.12

### Cross-Domain
- [[../Domain-01-Security-and-Risk-Management/security-governance-principles]] — privacy is a governance principle
- [[../Domain-01-Security-and-Risk-Management/legal-regulatory-and-compliance-issues]] — GDPR, HIPAA, PCI, GLBA
- [[../Domain-03-Security-Architecture-and-Engineering/Domain-03-Index]] — homomorphic encryption, ZKP, TEE
- [[../Domain-05-Identity-and-Access-Management/Domain-05-Index]] — consent, RBAC, identity-bound access
- [[../Domain-08-Software-Development-Security/Domain-08-Index]] — privacy by design in code

## References

- GDPR — Regulation (EU) 2016/679, esp. Art. 4, 5, 6, 9, 17, 25, 30, 32, 44–49
- CCPA / CPRA — California Civil Code § 1798.100 et seq.
- HIPAA — 45 CFR § 164.514 (De-identification, Safe Harbor, Expert Determination)
- PCI-DSS v4.0 — Req. 3 (Protect Stored Account Data), Req. 4 (Protect with Strong Cryptography During Transmission)
- ISO/IEC 27701:2019 — Privacy Information Management Systems (PIMS)
- NIST SP 800-188 — Trusted Geolocation in the Cloud
- NIST IR 8062 — Introduction to Privacy Engineering and Risk Management
- ENISA Pseudonymisation techniques and best practices (2024)
- Apple Differential Privacy overview (2017, ongoing)
- Sweeney, "k-anonymity: a model for protecting privacy" (2002)
- Dwork, "Differential Privacy" (2006)
- Schrems II — Case C-311/18, CJEU (2020)
