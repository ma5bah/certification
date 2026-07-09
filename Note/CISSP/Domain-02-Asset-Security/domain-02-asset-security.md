---
title: Domain 2 — Asset Security
layer: 1
domain: 2
tags: [cissp, asset-security, data-protection, governance]
related_domains: [1, 3, 5, 7, 8]
---

# Domain 2 — Asset Security

## Feynman Explanation

Imagine you own a chain of jewelry stores. Before you can protect the diamonds, you must **know exactly what diamonds you own, where they are, who is allowed to touch them, and what to do when a customer returns one**. Asset Security is exactly that — the discipline of identifying, valuing, classifying, handling, and disposing of information and the systems that store it. Every other CISSP domain (cryptography, operations, software security) is a **control** that protects assets; this domain is the **inventory and the labeling system** those controls wrap around.

## Technical Details

### Scope of Asset Security (CISSP CBK)

Asset Security covers the **confidentiality, integrity, and availability of information assets** across their entire lifecycle. The domain is organized into three concerns:

| Concern | Question Answered | Primary Output |
|---|---|---|
| **Identification** | What do we have? | Asset inventory + CMDB |
| **Classification** | How sensitive is it? | Sensitivity label + handling rules |
| **Handling** | How do we protect it through life? | Policies, DLP, sanitization |

### The Asset Universe

| Asset Class | Examples | Primary Risk |
|---|---|---|
| **Data / Information** | PII, PHI, PCI, IP, source code | Confidentiality breach |
| **Hardware** | Servers, endpoints, mobile, IoT, media | Availability + data remanence |
| **Software** | OS, middleware, SaaS licenses, APIs | Integrity + license compliance |
| **People** | Employees, contractors, third parties | Insider threat + social engineering |
| **Intangible** | Reputation, brand, trade secrets, goodwill | Reputational loss |
| **Services** | Cloud workloads, DNS, certificate authorities | Dependency failure |

### Core Principles

1. **Know your data** — you cannot protect what you cannot name. Asset Security is the prerequisite for [[data-classification-and-handling|data classification]].
2. **Ownership drives accountability** — every asset has a named **owner** (a manager, not a system); custodians operate it under the owner's policy.
3. **Lifecycle ends at destruction** — data that is no longer needed is a **liability**, not an asset. See [[data-lifecycle-and-remnants]].
4. **Classification drives control selection** — the sensitivity label determines the controls (crypto strength in [[../Domain-03-Security-Architecture-and-Engineering/Domain-03-Index|Domain 3]], access rigor in [[../Domain-05-Identity-and-Access-Management/Domain-05-Index|Domain 5]], retention rules per [[data-retention-and-disposal-nist-800-88|NIST 800-88]]).
5. **Residuals must be quantified** — Single Loss Expectancy (SLE) and Annualized Loss Expectancy (ALE) frame every asset decision (see [[../Domain-01-Security-and-Risk-Management/risk-management-framework|Risk Management]]).

### The Information Lifecycle (preview)

$$\text{Collection} \rightarrow \text{Storage} \rightarrow \text{Use} \rightarrow \text{Sharing} \rightarrow \text{Archival} \rightarrow \text{Destruction}$$

Each transition is a **trust boundary** where controls can fail. Full detail in [[data-lifecycle-and-remnants]].

### Classification Tiers (preview)

| Tier | Examples | Default Control Set |
|---|---|---|
| **Public** | Marketing site, press releases | Integrity controls only |
| **Internal** | Org charts, internal wiki | Authentication + RBAC |
| **Confidential** | Customer lists, source code | Encryption at rest/in transit, DLP, audit |
| **Restricted** | PII, PHI, PCI, trade secrets | Strongest crypto, need-to-know, tokenization, regulatory controls |

Full taxonomy in [[data-classification-and-handling]].

<!-- EXPANDED — Phase 5 Expert Transformation -->

## Foundations

### The Core Mental Model: Data's Lifecycle

Every asset-security decision is a **lifecycle decision**. The six-phase pipeline — $$\text{Collection} \rightarrow \text{Storage} \rightarrow \text{Use} \rightarrow \text{Sharing} \rightarrow \text{Archival} \rightarrow \text{Destruction}$$ — is not a diagram; it is the **control plane**. Each transition is a trust boundary where classification, ownership, and handling rules must be enforced. When a control fails at a boundary, data changes state without authorization. The entire domain is built to answer one question at every boundary: *"Is this data still in a state the owner authorized?"*

### "Who Owns This Data?" — The Question That Scales

At 10 employees, the CEO answers this from memory. At 10,000, no one can. The **three-role model** — Owner (accountable), Custodian (responsible), User (consumes) — is the scaffolding that survives organizational chaos. The distinction is legal, not semantic: the **Data Owner** carries **accountability** (signs off on classification, accepts residual risk, answers to regulators). The **Data Custodian** carries **responsibility** (implements controls, manages backups, provisions access). The owner cannot delegate accountability; the custodian cannot assume it. This separation is the single most-tested concept in the CISSP Domain 2 exam. See [[data-classification-and-handling]] for the full roles matrix.

### The CISO's Framework: Inventory → Classify → Control → Audit

The asset-security program is a **four-step cycle**, not a one-time project:

1. **Inventory** — discover every asset (60–80% of enterprise data is "dark" — unknown to security). See [[asset-inventory-and-categorization]].
2. **Classify** — assign a sensitivity tier (Public / Internal / Confidential / Restricted) per the taxonomy in [[data-classification-and-handling]]. Classification must be **automation-driven** because manual labeling fails at scale.
3. **Control** — apply the handling rules mapped to each tier: encryption, DLP, RBAC, retention, audit. Classification drives dozens of downstream controls automatically.
4. **Audit** — verify control effectiveness. The metrics that matter: % of data classified, % of sensitive data encrypted at rest, mean time to classify new repositories.

> **The gap.** Most organizations never finish step 1 — they are perpetually starting from a partial inventory. The organizations that complete all four steps reduce breach costs by **~40%** (Ponemon 2023) and pass audits with no qualified opinions.

## Architecture

### Classification Tiers → Control Tiers: The Mapping

Each classification tier maps to a **control tier** — a pre-configured bundle of protections that activate automatically. This is the architecture that turns a label into security:

| Classification Tier | Control Tier | Default Protections | Breach Notification |
|---|---|---|---|
| **Public** | Baseline | Integrity controls only (signing, versioning, CDN protection) | None |
| **Internal** | Standard | SSO + RBAC + audit logging + DLP in audit mode + no external sharing | Discretionary |
| **Confidential** | Elevated | Encryption at rest/transit + AES-256 + DLP soft-block + JIT access + watermarking + audit trail + per-asset keys | Regulatory if PII/PHI |
| **Restricted** | Maximum | Strongest crypto (CMK/HSM) + tokenization + need-to-know + split-knowledge + DLP hard-block + no-copy + crypto-shred on dispose + FIPS 140-2 L3 | Mandatory (72 h GDPR) |

Each tier inherits all protections from lower tiers. A control gap at any tier becomes the attack vector for the tier above it.

### DLP Architecture: The Four-Stack Model

DLP is not a single tool; it is a **four-stack** defense against exfiltration. Each stack covers different egress channels and failure modes:

| Stack | Coverage | Strength | Blind Spot |
|---|---|---|---|
| **Endpoint DLP** (EDLP) | Clipboard, USB, print, screens, offline files | Stops the removable-media path | OS subversion; crypto-rancher bypass |
| **Network DLP** (NDLP) | Email, HTTP/S, FTP, API calls | Stops the network path (60% of exfil) | Encrypted traffic without TLS MITM |
| **Cloud DLP / CASB** | SaaS apps, IaaS buckets, sharing permissions | Stops the cloud/SaaS shadow path | API lag (min–hrs); personal instances |
| **Storage DLP / DSPM** | File shares, DBs, S3, SharePoint at rest | Discovers dark data; retroactively labels | No real-time blocking |

Full detail in [[data-loss-prevention-dlp]]. The mature architecture deploys all four stacks with a unified policy engine and **classification-driven rule enforcement**.

### CMDB: The Single Source of Truth

The Configuration Management Database (CMDB) is the **spine** of the ITIL-based security program. A CMDB without **relationships** (depends-on, runs-on, connects-to) is just an asset list. A CMDB with relationships answers: *"If this server dies, what breaks?"* — the foundation of business impact analysis (BIA). The architecture: Configuration Items (CIs) → Attributes → Relationships → Baselines → Change log. See [[asset-inventory-and-categorization]] for the full schema.

## Execution

### Running a Data Discovery Project

Data discovery is the **prerequisite** for every other control. The playbook:

1. **Scope** — Tier 3+ data first (PII, PHI, PCI, trade secrets). Don't try to find everything at once.
2. **Deploy 4+ discovery methods** — agent-based (Tanium) + agentless scan (Qualys) + passive network + cloud-native (AWS Config) + data discovery (Purview/BigID). No single method exceeds 60% coverage.
3. **Normalize** — map every discovered data store to the standard classification taxonomy and handling rules matrix. Use automated labeling (MIP, Purview, Google DLP labels) as the primary mechanism.
4. **Classify** — run content inspection (regex + EDM + ML + OCR) against each store. Flag unclassified data as "high risk" by default.
5. **Remediate** — encrypt, tokenize, or delete based on classification. A Tier 4 data store with no encryption is a critical finding.
6. **Continuous** — discovery is not a project; it is a pipeline. New S3 buckets, SaaS apps, and shadow IT appear daily.

### Building a Classification Taxonomy

The **four-tier model** is the practical upper limit. Organizations that adopt 7+ tiers end up classifying nothing because the matrix is too complex. The taxonomy requires:

1. **Named owners** per data domain (HR owns employee PII, CFO owns financial data, CTO owns source code)
2. **Per-activity handling rules** — a matrix of what each tier permits: view, print, email, USB, cloud upload, share, archive, dispose
3. **Defense-in-depth labeling** — banner + metadata + ACL + container tag so no single mechanism can be silently stripped

See [[data-classification-and-handling]] for the full handling rules matrix.

### Labeling Mechanisms (Strength-Ordered)

| Mechanism | Survivability | Use Case |
|---|---|---|
| Header/footer banners | Advisory only; stripped on copy | Email, printed documents |
| Metadata tags (MIP, Google DLP labels) | Survives file copy; travels with document | Office 365, Google Workspace |
| File-system ACLs | Enforced by OS; survives rename | Network shares, OneDrive |
| Container/object tags | Survives cloud movement | S3 object tags, Azure Purview |
| Watermarking / DRM | Deters screenshots; forensic trace | IP, source code, board materials |
| Database column/row-level security | Enforced at data tier; strongest | Customer PII in RDBMS |

### Asset Management Lifecycle: The Operationalization

```
Register → Acquire → Deploy → Operate → Maintain → Retire → Dispose
   │         │         │         │          │         │         │
   │         │         │         │          │         │         └─ NIST 800-88 (Clear/Purge/Destroy)
   │         │         │         │          │         └─ De-register, retire in CMDB
   │         │         │         │          └─ Patch, EOL tracking, re-verify classification
   │         │         └─ Monitor, audit, SOC ingestion, DLP coverage
   │         └─ CMDB registration, classification label, owner assigned
   └─ Budget, procurement, risk acceptance
```

**Rule.** Every asset must be **registered before deployment** and **de-registered at retirement**. Unregistered assets are **unmanaged risk** — auditors treat them as a control failure. The retirement SLA: 30 days to sanitized for non-Restricted, 7 days for Restricted.

## Mastery

### Cloud Shared Responsibility: IaaS/PaaS/SaaS with Specific Control Ownership

The shared responsibility model answers one question: *Where does the CSP's job end and mine begin?* The **invariant**: you always own the **data** and the **access**. Everything below is negotiable.

| Layer | On-Prem | IaaS (EC2) | PaaS (RDS) | SaaS (M365) |
|---|---|---|---|---|
| **Data** | You | **You** | **You** | **You** |
| **Applications** | You | **You** | Shared | CSP |
| **OS** | You | **You** | CSP | CSP |
| **Virtualization** | You | CSP | CSP | CSP |
| **Physical** | You | CSP | CSP | CSP |

The **8 most-missed customer controls**: IAM hygiene, public S3 buckets, open security groups, CSP-managed keys (instead of CMK), logging disabled, guest OS unpatched, classification labels not traveling with data, shadow data in personal SaaS. Full matrix in [[cloud-asset-security-shared-responsibility]].

### Data Residency and Sovereignty at Global Scale

Data residency (where data is stored) and data sovereignty (which laws govern it) are distinct concepts with overlapping enforcement:

| Mechanism | Region/Scope | Key Constraint |
|---|---|---|
| **GDPR Adequacy Decision** | EU ↔ approved countries (JP, UK, NZ, CH, KR) | Full data flow; reviewed every 4 yr |
| **Standard Contractual Clauses (SCCs)** | EU → anywhere | Requires Transfer Impact Assessment (TIA) post-Schrems II |
| **Binding Corporate Rules (BCRs)** | Intra-group, EU → non-EU | 12–24 month DPA approval |
| **Data Privacy Framework (DPF)** | EU ↔ US (2023) | US self-certification required |
| **Regional mandates** | CN, RU, IN, ID | **Local storage + local processing** required — no transfer |
| **Data localization laws** | 30+ countries | Varied; often sector-specific (health, finance, telecom) |

The enterprise architecture response: **sovereign cloud** (AWS GovCloud, Azure Sovereign, OVHcloud) + **crypto-shred as a data-residency tool** (destroy the key in a specific region = destroy the data in that region).

### NIST SP 800-88: Clear vs Purge vs Destroy

| Decision | Definition | Recovery | Method | Use Case |
|---|---|---|---|---|
| **Clear** | Logical overwrite; keyboard recovery infeasible | Lab recovery possible | Single-pass overwrite (0s/1s/random); ATA Secure Erase | Internal reuse, same classification |
| **Purge** | Render lab recovery infeasible | State-of-the-art lab infeasible | Degauss (HDD/tape); crypto-erase (ATA, NVMe, file-level); firmware reset | Media leaving org |
| **Destroy** | Physical destruction | Physically impossible | Shred ≤2mm, incinerate, disintegrate, pulverize, melt | Restricted / classified end-of-life |

Full decision tree in [[data-retention-and-disposal-nist-800-88]].

### Data Remanence: SSDs vs HDDs

The **fundamental asymmetry** that drives sanitization decisions:

| Property | HDD (Magnetic) | SSD (Flash/NAND) |
|---|---|---|
| **Why delete ≠ gone** | Magnetic domains persist; MFM microscopy can recover overwritten tracks | Wear-leveling + over-provisioning: logical write ≠ physical block overwrite; data remains in spare area |
| **NIST-accepted sanitize** | Single-pass overwrite (Clear) or degauss (Purge) | **Crypto-erase only** for Purge; overwrite is **insufficient** |
| **Degauss effectiveness** | Effective if coercivity matched | **No effect** — SSD is non-magnetic |
| **Preferred destruction method** | Degauss + shred | Shred (≤2mm particle) or crypto-shred |

> **CISSP exam trap.** "Formatting" or "factory reset" is never sanitization. Quick-format rewrites only the file table; data is recoverable with off-the-shelf tools. On SSD, even a full overwrite may not touch wear-leveled blocks.

### Crypto-Shred Dominance

The **modern default** for any media that supports encryption. Encrypt at rest with a **per-asset** DEK wrapped by a KEK in HSM/KMS. Disposal = destroy the key. The audit log entry is the certificate of destruction. NIST 800-88 Rev. 1 treats Cryptographic Erase (CE) as equivalent to Purge. Works for: SSDs, tapes, cloud volumes, virtual machines, containers. See [[data-retention-and-disposal-nist-800-88]] for the full pattern.

## Resilience

### The #1 Mistake: Over-Classification

Classifying everything as "Confidential" is **the most common security-program self-inflicted wound**. When everything is labeled the same, the label is meaningless — users ignore it, DLP rules become noise, and the board sees 100% classification coverage on a meaningless number. The consequence: real Restricted data (PII, PHI, trade secrets) receives the same handling as a cafeteria menu labeled "Confidential." **Pick 4 tiers; force owners to justify anything above Internal; audit the distribution quarterly.**

### When DRM Breaks Workflows

DRM (Digital Rights Management) and watermarking work well for **static, read-only documents viewed by trusted internal audiences**. They break rapidly for:
- Collaborative editing (Google Docs, Office 365 co-authoring)
- External sharing (partners, auditors, regulators who need native format)
- Export/transform pipelines (ETL, reporting, ML training)
- Screenshots (watermarks are forensic, not preventive)

The resilience strategy: **DRM for board packs and IP documents; DLP for everything else.** DRM is a deterrence layer, not a perimeter control.

### Shadow IT Discovery Failures

Shadow IT — SaaS apps, personal cloud drives, rogue IaaS accounts — represents **30–50% of enterprise data** that the security program cannot see. The failure chain: no network visibility → no discovery → no classification → no DLP → data exfiltrated. The detection architecture: **CASB log ingestion + firewall/DNS log analysis + browser extension telemetry**. Without all three, shadow IT is invisible.

### Misconfigured S3 Buckets: The Recurring Pattern

Public S3 buckets are the **#1 cloud misconfiguration** (CSA Top Threats 2024) and the root cause of 200M+ exposed records annually. The pattern: a developer creates a bucket for testing, copies production data into it, sets it to public-read, never audits it. The defense: **S3 Block Public Access (account-wide, Deny)** + **AWS Config auto-remediation** + **CSPM in block mode** + **DSPM to detect PII in open buckets**. A public bucket is not a technology failure; it is a **process failure** — no pipeline between asset discovery and classification.

## Context

### Building a GDPR-Compliant Asset Inventory

GDPR Art. 30 requires a **Record of Processing Activities (RoPA)** — essentially a privacy-aware asset inventory. The minimum fields: data subject category, data categories (PII, special categories), purpose of processing, legal basis (Art. 6), recipients (internal + third parties), transfers to third countries (with safeguard mechanism documented), retention period, and a general description of technical/organizational security measures. The RoPA must be **available on demand** to the supervisory authority. Linking the RoPA to the CMDB and data-flow diagrams is the standard for audit-readiness.

### Responding to a Data Subject Access Request (DSAR)

A DSAR (GDPR Art. 15, CCPA/CPRA) requires the organization to produce **all personal data held about a specific individual**, typically within **30 days (GDPR)** or **45 days (CCPA)**. Without a classified asset inventory and data discovery pipeline, the DSAR response is manual, error-prone, and legally dangerous. The architecture: **data catalog (Collibra/Purview) + DSAR workflow engine (OneTrust/TrustArc) + automated search across structured + unstructured data stores + redaction pipeline**. A DSAR that misses a data store is a **regulatory violation**, even if unintentional.

### When to Tokenize vs Encrypt vs Pseudonymize

| Technique | Reversible? | Data Utility | PCI Scope Reduction | Best For |
|---|---|---|---|---|
| **Tokenization** | Yes (via vault) | Medium (FPE = high) | **Maximum** | PANs, SSNs, bank accounts — specific high-value fields |
| **Encryption** | Yes (with key) | Full (decryptable) | None (data is still in-scope) | Data at rest, data in transit — bulk protection |
| **Pseudonymization** | Yes (with key) | High | Partial | Research datasets, analytics, SSO identifiers |

The decision rule: **tokenize at the point of capture** for PAN/SSN/bank account → downstream systems see only the token, reducing PCI/regulatory scope by 60–90%. **Encrypt at rest** for everything else. **Pseudonymize** for analytic workloads that need join capability without re-identification. Full comparison in [[data-privacy-and-protection]].

## Nuance

### Data Owner vs Data Custodian: The Accountability Gap

The **Data Owner** is a named senior manager with **legal accountability** for the asset's protection, classification, and disposition. The **Data Custodian** is the IT/system role that implements the owner's policy. The gap: most organizations name custodians but cannot name owners. A security program without named data owners is a **policy without accountability** — auditors and regulators treat it as a control failure. See [[data-classification-and-handling]] for the full three-role model.

### Why "Data at Rest / In Transit / In Use" Is an Oversimplification

The CISSP triad of data states masks critical nuance:

| State | Traditional View | What It Misses |
|---|---|---|
| **At rest** | Storage encryption (AES-256) | Backups, replicas, snapshots (same data, different locations, often unencrypted); swap files; hibernation files |
| **In transit** | TLS 1.3 / mTLS | Core-network taps, SPAN ports, API gateways that terminate TLS before forwarding; internal east-west traffic often unencrypted |
| **In use** | Enclave / TEE / confidential compute | Spectre/Meltdown side-channels; cold-boot attacks on DRAM; clipboard; GPU memory; browser DOM |

Modern architectures add **"in archive"** (WORM, immutable storage, key-preservation) and **"in copy"** (every replica, snapshot, CI/CD clone) as explicit states requiring their own controls.

### FIPS 199 Impact Levels and Classification Mapping

FIPS 199 defines three impact levels per security objective (C, I, A):

| Impact Level | Confidentiality | Integrity | Availability | Maps To |
|---|---|---|---|---|
| **Low** | Limited adverse effect | Limited adverse effect | Limited adverse effect | Public / Internal |
| **Moderate** | Serious adverse effect | Serious adverse effect | Serious adverse effect | Confidential |
| **High** | Severe / catastrophic | Severe / catastrophic | Severe / catastrophic | Restricted |

$$\text{Security Category}_{\text{info}} = \{(C, I, A) \mid C, I, A \in \{\text{L, M, H}\}\}$$

The **waterline** is the highest of the three. A dataset that is Low-C, Moderate-I, High-A requires High-tier availability controls even though it's only Low for confidentiality. This is a **CISSP exam favorite**: the system inherits the highest watermark across all three objectives.

## Resources

### Data Classification Policy — Minimum Sections

1. **Purpose and scope** — which data domains, which jurisdictions
2. **Classification taxonomy** — 4 tiers with definitions, examples, and default handling rules per tier
3. **Roles** — Owner (accountable, executive), Custodian (responsible, IT), User (adheres)
4. **Labeling standard** — mechanism (banner, metadata, ACL, container tag), format, placement
5. **Handling rules matrix** — per tier × per activity (view, print, email, USB, cloud, share, archive, dispose), as defined in [[data-classification-and-handling]]
6. **Reclassification / declassification procedure** — owner approval required; aggregation risk addressed
7. **Violations and enforcement** — escalation path, HR involvement
8. **Annual review and audit cadence**

### Asset Inventory — Minimum Fields Checklist

| Field | Required | Purpose |
|---|---|---|
| Unique ID | Primary key | Audit trail |
| Asset class | Data / Hardware / Software / People / Intangible / Service | Policy scoping |
| Owner (named) | Senior accountable individual | Auditor's first check |
| Custodian | Operational team | Day-to-day contact |
| Location | Physical + logical (datacenter, region, CSP) | Data residency |
| Classification tier | Public / Internal / Confidential / Restricted | Control selection |
| Criticality tier | Tier 1–3 | Recovery priority |
| Dependencies | Upstream + downstream CIs | BIA |
| Regulatory scope | GDPR / HIPAA / PCI / SOX / None | Compliance tracking |
| Last verified | Date | Audit freshness (≤90 days target) |

Full schema in [[asset-inventory-and-categorization]].

### DLP Rule Priority Matrix

Deploy DLP rules in this order — depth before breadth:

| Priority | Data Type | Technique | Action Mode |
|---|---|---|---|
| 1 | SSN / government ID | Regex + EDM | Hard-block on egress |
| 2 | PCI PAN | Luhn-valid regex + EDM + vault tokenization | Tokenize at capture; hard-block clear PAN egress |
| 3 | PHI / medical record numbers | EDM + keyword (patient, diagnosis, prescription) | Soft-block → hard-block after tuning |
| 4 | Source code / IP | Fingerprinting + ML similarity | Audit → soft-block (threshold: >500 lines of code matching) |
| 5 | Financial records / M&A | Keyword + classification label ("Confidential — Finance") | Hard-block on external domains; soft-block internal |
| 6 | PII (general) | ML + regex patterns | Audit → notify → soft-block |
| 7 | Unclassified but containing PII-like patterns | ML anomaly detection | Notify DLP admin; flag for classification |

Full architecture in [[data-loss-prevention-dlp]].

<!-- /EXPANDED -->

## CISO / Risk Manager View

**Board framing.** Asset Security is where cyber risk becomes **financial risk**. The board does not care about "encryption" — it cares about *"what is the worst-case loss if our customer database leaks?"* The answer is:

$$\text{SLE} = \text{Asset Value (AV)} \times \text{Exposure Factor (EF)}$$
$$\text{ALE} = \text{SLE} \times \text{Annualized Rate of Occurrence (ARO)}$$

A CISSP-aligned Asset Security program produces a defensible **asset register** that the board, auditors, and regulators (GDPR Art. 30, SOX, HIPAA Security Rule) all require. Without it, the CISO cannot answer: *"What data do we have, where is it, who owns it, and how is it protected?"*

**Strategic priorities.**

1. **Data discovery before data protection.** 60–80% of enterprise data is "dark" — unknown to security. Closing that gap is the single highest-ROI program.
2. **Consolidate, then classify.** Cloud sprawl and SaaS proliferation mean the same data lives in 5–10 places. Reduce copies before labeling.
3. **Tie classification to automation.** Manual labeling fails at scale; classification should drive [[data-loss-prevention-dlp|DLP]], encryption, retention automatically.
4. **Quantify residual risk.** Use ALE × % of asset covered to show the board a 3-year program roadmap with measurable risk reduction.
5. **Defensible disposal.** A disk that leaves the data center unaccounted for can shift the regulatory conversation from "negligence" to "willful" — see [[data-retention-and-disposal-nist-800-88]].

**Metrics the board cares about.**

- % of data assets classified (target: 95% in 24 months)
- % of sensitive data encrypted at rest
- Mean time to discover and classify new data repositories
- Records affected by incidents (regulator-mandated disclosure threshold)
- $ALE reduction attributable to the program

## Related Connections

### Layer 2 — Deep Dives
- [[data-classification-and-handling]] — sensitivity tiers, ownership, custodianship, labeling
- [[data-lifecycle-and-remnants]] — collection→destruction, data remanence, sanitization preview
- [[data-privacy-and-protection]] — PII/PHI/PCI, anonymization, pseudonymization, tokenization
- [[asset-inventory-and-categorization]] — hardware, software, data, people, intangible; CMDB

### Layer 3 — Standards & Edge Cases
- [[data-loss-prevention-dlp]] — endpoint / network / cloud DLP, content inspection
- [[cloud-asset-security-shared-responsibility]] — IaaS/PaaS/SaaS responsibility matrix
- [[data-retention-and-disposal-nist-800-88]] — Clear / Purge / Destroy decisions by media type
- [[information-handling-standards-iso-27001-annex-a]] — ISO 27001:2022 Annex A control families

### Cross-Domain
- [[../Domain-01-Security-and-Risk-Management/risk-management-framework]] — risk framing, SLE/ALE
- [[../Domain-01-Security-and-Risk-Management/security-governance-principles]] — classification is a governance control
- [[../Domain-03-Security-Architecture-and-Engineering/Domain-03-Index]] — crypto protects classified data
- [[../Domain-05-Identity-and-Access-Management/Domain-05-Index]] — RBAC + need-to-know enforce classification
- [[../Domain-07-Security-Operations/Domain-07-Index]] — operations handles the lifecycle day-to-day
- [[../Domain-08-Software-Development-Security/Domain-08-Index]] — DevSecOps embeds classification into data flows

## References

- (ISC)² CISSP CBK — Domain 2: Asset Security
- ISO/IEC 27001:2022 — Information Security Management Systems
- ISO/IEC 27002:2022 — Information Security Controls
- NIST SP 800-88 Rev. 1 — Guidelines for Media Sanitization
- NIST SP 800-60 Vol. 1/2 — Guide for Mapping Types of Information and Information Systems to Security Categories
- GDPR Art. 30, 32, 17 — Records of Processing, Security of Processing, Right to Erasure
- HIPAA Security Rule §164.308, §164.310 — Administrative & Physical Safeguards
- PCI-DSS v4.0 — Requirement 3 (Protect Stored Account Data)
