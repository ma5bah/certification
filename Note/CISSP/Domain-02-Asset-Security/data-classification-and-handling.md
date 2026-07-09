---
title: Data Classification and Handling
layer: 2
domain: 2
tags: [cissp, data-classification, labeling, ownership, custodianship, data-handling]
related_domains: [1, 3, 5, 7, 8]
parent: domain-02-asset-security
---

# Data Classification and Handling

## Feynman Explanation

Think of a hospital. Patient files are stamped "Restricted" — only the named doctor and nurse on the case may read them, and they cannot photocopy them or take them home. The hospital's internal newsletter is "Internal" — every employee may read it, but not the public. A press release is "Public" — anyone may have it. **Classification is the stamp; handling rules are the policies that tell each person what they may do with each stamp.** Without the stamp, no one knows which file deserves which protection; without the rules, the stamp is decoration.

## Technical Details

### The Four-Tier Sensitivity Model

CISSP uses (and most enterprises adopt) a four-level model. The exact labels vary by organization, but the *semantic distance* is universal: each tier requires strictly more protection than the one below.

| Tier | Common Labels | Who Can Access | Default Controls | Breach Notification |
|---|---|---|---|---|
| **Tier 1 — Public** | Public, Unclassified, Open | Everyone (incl. public internet) | Integrity + availability | None required |
| **Tier 2 — Internal** | Internal Use, Proprietary, Sensitive | Employees + authorized contractors | Authentication, RBAC, no external sharing | Discretionary |
| **Tier 3 — Confidential** | Confidential, Private, Highly Sensitive | Need-to-know workforce + named third parties | Encryption at rest/transit, DLP, audit logging | Regulatory if PII/PHI/PCI |
| **Tier 4 — Restricted** | Restricted, Secret, Top Secret, Regulated | Named individuals only, often with MFA + JIT access | Strongest crypto, tokenization, split-knowledge, no-copy | Mandatory (GDPR 72h, HIPAA, state laws) |

> **Gov vs. Commercial divergence.** Government classifications (Unclassified / Controlled Unclassified Information (CUI) / Confidential / Secret / Top Secret) are *legally defined* and use the **Need-to-Know principle** strictly. Commercial classifications are *policy-defined* and use **Need-to-Know** as a default for Tier 3–4 and **Least Privilege** for Tier 2.

### Government Classification (US) — Reference

| Level | Damage Threshold | Examples |
|---|---|---|
| **Top Secret** | "Exceptionally grave damage" to national security | SCI, SAP, compartmented intelligence |
| **Secret** | "Serious damage" | Classified military plans |
| **Confidential** | "Damage" | Personnel files, operational details |
| **Controlled Unclassified Information (CUI)** | Not classified but sensitive | CUI//SP-PRIV, CUI//SP-PROP, ITAR, EAR |
| **Unclassified** | No damage | Public-facing material |

> *CISSP exam focus: the commercial 4-tier model + the principle that **classification authority is delegated, not assumed**.*

### Roles and Responsibilities

The **Three-Role Model** is the heart of ownership. Conflating these roles is the #1 governance failure.

| Role | Responsibility | Example |
|---|---|---|
| **Owner (Data Owner)** | A **named senior manager** accountable for the asset's protection, classification, and disposition decisions. Sets the policy. | VP of HR owns employee PII |
| **Custodian / Steward** | Implements the owner's policy day-to-day: storage, backup, encryption, access provisioning. Does **not** decide classification. | Database admin, IT operations |
| **User / Subject** | Consumes the data per the handling rules. Reports misuse. | Sales rep reading a customer record |
| **Data Controller** *(privacy term)* | The entity that determines **purposes and means** of processing (GDPR Art. 4(7)) | The company itself |
| **Data Processor** *(privacy term)* | Processes on behalf of the controller (GDPR Art. 4(8)) | SaaS payroll vendor |

> **Critical distinction.** A *Data Owner* is **accountable** (signs off on classification and risk acceptance). A *Data Custodian* is **responsible** (does the work). The owner cannot delegate accountability.

### Classification Criteria

A consistent set of criteria removes ambiguity. NIST SP 800-60 and most enterprise programs use a **confidentiality × integrity × availability** matrix mapped to FIPS 199 categories:

| FIPS 199 Category | Confidentiality | Integrity | Availability |
|---|---|---|---|
| **Low** | Limited adverse effect | Limited adverse effect | Limited adverse effect |
| **Moderate** | Serious adverse effect | Serious adverse effect | Serious adverse effect |
| **High** | Severe / catastrophic | Severe / catastrophic | Severe / catastrophic |

$$\text{Security Category} = \{(C, I, A) \mid C, I, A \in \{\text{Low, Moderate, High}\}\}$$

The **waterline** is the highest of the three. A dataset that is Low confidentiality but High integrity (e.g., audit logs) still requires High-tier controls for integrity.

### Handling Rules Matrix

| Activity | Public | Internal | Confidential | Restricted |
|---|---|---|---|---|
| View on corporate device | ✓ | ✓ | ✓ (need-to-know) | ✓ (named) + MFA |
| Print | ✓ | ✓ | Watermark + audit | Prohibited or vault-print |
| Email to external | ✓ | ✗ | Encrypted, DLP-scanned | Blocked by DLP |
| Copy to USB | ✓ | ✓ | Blocked by DLP | Blocked at OS level |
| Cloud upload (personal) | ✓ | ✗ | Blocked | Blocked + alert |
| Share via collaboration tool | ✓ | ✓ (org) | Link expiry + audit | Blocked or DLP-redacted |
| Archive | Optional | 3–7 yr | Per regulation | Per regulation + crypto-shred on dispose |
| Disposal | Recycle bin | Secure shred | NIST 800-88 Purge | NIST 800-88 Destroy |

See [[data-retention-and-disposal-nist-800-88]] for the disposal half of this table.

### Labeling Mechanisms

| Mechanism | Strength | Use Case |
|---|---|---|
| **Footer / header banners** ("CONFIDENTIAL — DO NOT DISTRIBUTE") | Weakest — advisory | Email, documents |
| **Metadata tags** (Office sensitivity labels, MIP labels) | Medium — survives copy | Office 365, Google Workspace |
| **File-system ACLs** | Strong — enforced by OS | Network shares, OneDrive |
| **Database column / row-level security** | Strongest — enforced at data tier | Customer PII in RDBMS |
| **Containerized data** (tagged blobs in object stores) | Strong — survives movement | S3 with object tags, Azure Purview |
| **Watermarking / DRM** | Strong — deters screenshots | IP, source code, board materials |

Best practice: **defense in depth** — banner + metadata + ACL + container tag, so the label cannot be silently stripped.

### Declassification and Reclassification

- **Downgrade (declassification)** requires the **owner's explicit approval** and an audit trail. A field worker cannot unilaterally downgrade.
- **Reclassification upward** is automatic when context changes (e.g., an "Internal" draft is submitted to the SEC → becomes "Restricted" the moment the embargo is set).
- **Aggregation risk.** Combining two "Internal" datasets can produce a "Confidential" insight (e.g., a department roster + salary band reveals executive comp). The owner must classify the *aggregated view* at the higher tier.

## CISO / Risk Manager View

**Why this matters at the board level.** Classification is the *single most leveraged control* in the entire security program. Once a piece of data is correctly labeled, **dozens of downstream controls activate automatically** — DLP, encryption, retention, access reviews, audit, breach-notification clocks. Without labels, every downstream control is a guess. The CISSP exam and every major framework (NIST CSF Identify, ISO 27001 A.5.12–A.5.13, PCI-DSS 4.0 Req. 9) treat classification as a *governance* control with audit and board visibility.

**Asset valuation lens.** Different tiers carry radically different loss profiles. Approximate *public* benchmarks for per-record loss (IBM Cost of a Data Breach 2023):

| Data Type | Tier | Avg. Cost / Record |
|---|---|---|
| Public marketing | 1 | $0 |
| Internal corporate data | 2 | $50–150 |
| Customer PII | 3 | $165 |
| Healthcare PHI | 4 | $400+ |
| Payment card (PCI) | 4 | $200–300 |
| Intellectual property (trade secret) | 4 | $5,000–50,000+ per record (litigation + lost margin) |

**Strategic actions.**

1. **Pick 4 tiers, not 7.** Most enterprises that over-classify (Public/Internal/Restricted/Confidential/Private/Secret/Critical) end up labeling nothing because the matrix is too complex.
2. **Name the owners in writing.** A classification program without named data owners in the org chart is a *policy* without *accountability* — auditors and regulators will treat it as a control failure.
3. **Automate enforcement.** Sensitivity labels in M365 / Google Workspace / Purview should drive encryption, DLP, retention, and access reviews. Manual enforcement does not scale.
4. **Train once, audit forever.** Annual awareness is insufficient; spot-audit handling by department quarterly.
5. **Tie to cyber insurance.** Most underwriters now ask for the **% of data classified** and **% of sensitive data encrypted**. Programs that can answer these questions get better premiums.

## Related Connections

### Sibling L2
- [[data-lifecycle-and-remnants]] — classification drives lifecycle decisions
- [[data-privacy-and-protection]] — PII/PHI/PCI are Tier 4 by default
- [[asset-inventory-and-categorization]] — classification requires an inventory to label

### L3 Standards
- [[data-loss-prevention-dlp]] — enforces the handling matrix
- [[data-retention-and-disposal-nist-800-88]] — defines the disposal column
- [[information-handling-standards-iso-27001-annex-a]] — A.5.12–A.5.13 are the formal controls
- [[cloud-asset-security-shared-responsibility]] — labeling must travel with data into the cloud

### Cross-Domain
- [[../Domain-01-Security-and-Risk-Management/security-governance-principles]] — classification is a governance control
- [[../Domain-01-Security-and-Risk-Management/risk-management-framework]] — tier × threat = risk
- [[../Domain-03-Security-Architecture-and-Engineering/Domain-03-Index]] — crypto strength chosen by tier
- [[../Domain-05-Identity-and-Access-Management/Domain-05-Index]] — RBAC enforces the access column
- [[../Domain-08-Software-Development-Security/Domain-08-Index]] — DevSecOps data-flow tagging

## References

- NIST SP 800-60 Vol. 1 Rev. 1 — Guide for Mapping Types of Information Systems to Security Categories
- NIST FIPS 199 — Standards for Security Categorization of Federal Information and Information Systems
- NIST FIPS 199 — Standards for Security Categorization of Federal Information and Information Systems
- ISO/IEC 27001:2022 — A.5.12 Classification of Information, A.5.13 Labelling of Information
- ISO/IEC 27002:2022 — §5.12, §5.13
- Executive Order 13526 — Classified National Security Information
- 32 CFR Part 2002 — Controlled Unclassified Information (CUI)
- (ISC)² CISSP CBK — Domain 2: Asset Security
