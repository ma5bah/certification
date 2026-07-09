---
title: Cloud Asset Security and Shared Responsibility
layer: 3
domain: 2
tags: [cissp, cloud-security, shared-responsibility, iaas, paas, saas, csp, casb, cspm]
related_domains: [1, 3, 5, 7, 8]
parent: asset-inventory-and-categorization
---

# Cloud Asset Security and Shared Responsibility

## Feynman Explanation

When you rent an apartment, the landlord fixes the roof and the building's structure, but **you** lock your door, choose your own insurance, and decide who gets a key. The cloud is the same: Amazon, Microsoft, or Google secure the **data center, the network, the hypervisor, and the physical server**; **you** secure your **data, your access keys, your OS patches, your firewall rules, and your encryption**. Where the landlord's job ends and yours begins is the **shared responsibility model**, and confusing that boundary is the #1 cause of cloud breaches.

## Technical Details

### The Shared Responsibility Matrix

| Layer | On-Prem (you) | IaaS (e.g., EC2, Azure VM) | PaaS (e.g., RDS, App Service, BigQuery) | SaaS (e.g., M365, Salesforce, Slack) |
|---|---|---|---|---|
| **Data** | You | **You** | **You** | **You** |
| **Applications** | You | **You** | You (or vendor app code) | CSP |
| **Runtime / middleware** | You | **You** | CSP | CSP |
| **OS** | You | **You** | CSP | CSP |
| **Virtualization** | You | CSP | CSP | CSP |
| **Servers** | You | CSP | CSP | CSP |
| **Storage** | You | CSP | CSP | CSP |
| **Network (physical)** | You | CSP | CSP | CSP |
| **Physical data center** | You | CSP | CSP | CSP |

> **Mnemonic.** "**You** always own the data and the access." The further up the stack you go (IaaS → PaaS → SaaS), the more the CSP takes on — **but never the data**.

### AWS Shared Responsibility — Concrete Mapping

| Component | AWS Responsibility | Customer Responsibility |
|---|---|---|
| **Global infrastructure** (Regions, AZs, Edge locations) | AWS | — |
| **Compute** (EC2 hypervisor, host OS) | AWS | Guest OS patches, app security, IAM |
| **Storage** (S3, EBS) | AWS | Bucket policies, ACLs, encryption keys, object ACLs |
| **Database** (RDS — managed) | AWS: underlying instance, patching | Database-level access, query patterns, encryption keys (with KMS) |
| **IAM** (AWS IAM) | Service availability | **Policy correctness, least privilege, MFA, key rotation** |
| **Encryption in transit** | TLS termination infrastructure | TLS configuration, certificate management |
| **Encryption at rest** | Optional service | **Choice of CMK vs. AWS-managed; key rotation policy** |
| **Network** (VPC) | Provides the abstraction | **Subnet design, NACLs, Security Groups, route tables, internet gateway policy** |
| **Logging** (CloudTrail, Config) | Service availability | **Enablement, log retention, monitoring** |

> **Famous AWS breach example.** The **2017 Capital One breach** was *not* an AWS failure — it was the customer (Capital One) misconfiguring a WAF rule that allowed an SSRF to reach IMDS, retrieve temporary credentials, and access an S3 bucket. **The shared responsibility held; the customer's side failed.**

### Azure Shared Responsibility

| Azure Service | Microsoft | Customer |
|---|---|---|
| **Azure VMs (IaaS)** | Hypervisor, host, fabric | OS, patches, apps, data, network config |
| **Azure SQL / PaaS DB** | Service runtime, OS, backups | Data, access, query-level controls, encryption keys (TDE) |
| **App Service (PaaS)** | Runtime, scaling | App code, deployment slots, connection strings |
| **Microsoft 365 (SaaS)** | Service availability, infrastructure | **Data, identity, access, sharing policy, label application** |
| **Azure Active Directory** | Service availability | Identity lifecycle, conditional access, MFA, hybrid config |

### GCP Shared Responsibility

| Layer | Google | Customer |
|---|---|---|
| **Google Kubernetes Engine (GKE)** | Control plane | Node pools, workloads, RBAC, network policies |
| **BigQuery** | Service runtime, storage | Data, query IAM, column/row-level security |
| **Cloud Storage** | Service | Bucket IAM, retention, CMEK, object holds |

### Customer Responsibilities — The 8 Most-Missed

These are the controls CISOs consistently find gaps in, even in mature cloud programs:

1. **IAM hygiene** — over-privileged service accounts, long-lived access keys, no MFA on root.
2. **Storage exposure** — public S3 buckets / Azure Blob / GCP Storage. (Trend Micro reported 230M+ exposed records in 2023 from misconfigured buckets alone.)
3. **Network exposure** — security groups / NACLs open to `0.0.0.0/0` on management ports.
4. **Key management** — using CSP-managed keys when the threat model requires customer-managed (CMK / CMEK / Customer-Managed HSM).
5. **Logging enablement** — CloudTrail, Azure Activity Log, GCP Audit Logs disabled or retained 7 days.
6. **Patch cadence** — guest OS unpatched in IaaS; container images unpatched in PaaS.
7. **Data classification not traveling with data** — labels set on-prem vanish in the cloud.
8. **Shadow data in SaaS** — sensitive data uploaded to personal Google Drive, then shared externally.

### Cloud-Native Security Stack

| Domain | Tool Category | Examples |
|---|---|---|
| **CSPM** (Cloud Security Posture Management) | Continuous config audit; misconfiguration detection; compliance packs | Wiz, Prisma Cloud, Microsoft Defender for Cloud, AWS Security Hub, Lacework, Orca |
| **CIEM** (Cloud Infrastructure Entitlement Management) | Detect over-privileged identities, least-privilege analysis | CloudKnox, Ermetic, Sonrai |
| **CWPP** (Cloud Workload Protection Platform) | Workload runtime, vulnerability scan, runtime protection | Prisma Cloud, Trend Micro, Aqua, Wiz |
| **CASB** (Cloud Access Security Broker) | SaaS visibility, DLP for cloud, shadow IT | Netskope, Zscaler, Microsoft Defender for Cloud Apps |
| **SSPM** (SaaS Security Posture Management) | M365, Google Workspace, Salesforce config audit | Adaptive Shield, Reco, AppOmni |
| **DSPM** (Data Security Posture Management) | Discover and classify data in cloud stores; track data flow | Dig Security, Cyera, Sentra, Wiz |
| **CNAPP** (Cloud-Native Application Protection Platform) | Unifies CSPM + CIEM + CWPP + DSPM | Palo Alto Prisma Cloud, Wiz, Microsoft Defender for Cloud |
| **CIEM + DSPM + CSPM** convergence | "Identity-aware data security" | Wiz DSPM, Sentra, Cyera |

> **Trend.** The 2023–2024 market has consolidated toward **CNAPP** (single pane) and **DSPM** (data-first). The 2017-era "best-of-breed per silo" approach is now considered a liability for talent and TCO.

### Cloud Asset Inventory — Specific Techniques

| Technique | Tool | Captures |
|---|---|---|
| **Cloud Asset Inventory** | AWS Config, Azure Resource Graph, GCP Cloud Asset Inventory | All cloud resources; tags; relationships |
| **Cloud security graph** | Wiz, Lacework, Orca | Resources + identities + data + network paths in a single graph |
| **Tagging** | Resource tags (`Owner`, `DataClass`, `Environment`) | Logical grouping; cost allocation; policy scope |
| **Data discovery** | Macie (S3), Purview (Azure), BigQuery DLP, Sentra, Cyera | Sensitive data fields in cloud stores |
| **SBOM** | Anchore, Snyk, GitHub Dependency Graph | Software components in container images |
| **CloudTrail / Activity Log** | Per-CSP log | API calls — the audit trail |
| **CASB log ingestion** | Netskope, Defender for Cloud Apps | Shadow IT, unsanctioned SaaS |

### CSP-Specific Edge Cases

| Edge Case | AWS | Azure | GCP |
|---|---|---|---|
| **Customer-managed HSM** | CloudHSM (FIPS 140-2 L3) | Dedicated HSM (L3) | Cloud HSM (L3) |
| **Confidential compute** | Nitro Enclaves, Graviton (always-encrypt memory) | Confidential VMs (AMD SEV-SNP, Intel TDX) | Confidential VMs (AMD SEV, SEV-SNP) |
| **BYOK** (Bring Your Own Key) | KMS Import + External Key Store | Key Vault BYOK, HSM-backed | Cloud KMS with External Key Manager (EKM) |
| **HYOK** (Hold Your Own Key) | External Key Store + revocation API | Key Vault + revocation | EKM with on-prem HSM |
| **Object lock / WORM** | S3 Object Lock | Immutable Blob Storage | Bucket Lock (preview) |
| **Egress cost attacks** | Data transfer fees; enable S3 Block Public Access | ExpressRoute costs; private endpoints | Egress costs; Private Service Connect |

### Cloud Contracts — What to Negotiate

| Clause | Why It Matters |
|---|---|
| **Data ownership** | Customer owns all data; CSP has no rights except to process per instructions |
| **Data location** | Specific region commitment; no movement without notice |
| **Subprocessor list and notification** | Right to object to new subprocessors |
| **Egress / deletion** | Return of all data in industry-standard format; deletion certification within 30 days of termination |
| **Audit rights** | SOC 2, ISO 27001, ISO 27017, ISO 27018, PCI-DSS, FedRAMP, C5, ENS, IRAP reports |
| **Encryption** | Customer-managed keys available; encryption at rest and in transit by default |
| **Incident notification** | CSP notifies customer within 24–72 h of a confirmed breach affecting customer data |
| **Liability cap** | Negotiate carve-outs for data breaches, IP infringement, confidentiality |
| **Insider threat protection** | CSP background checks, separation of duties, monitoring of CSP personnel |
| **Right to terminate for cause** | Customer can exit if CSP fails security commitments |

### Common Cloud Misconfigurations (Top 10, per CSA & Wiz)

1. Public S3 / Blob / GCS buckets
2. Overly permissive IAM (especially `*:*` on `*:*`)
3. Long-lived access keys (no rotation)
4. Security group / NACL open to `0.0.0.0/0` on RDP/SSH (22, 3389)
5. Default service account with broad scopes
6. Unencrypted databases (no TDE / at-rest encryption disabled)
7. Logging disabled (CloudTrail off, Azure Activity Log off)
8. No MFA on root / break-glass account
9. Secrets in code / public GitHub
10. Shadow data — public-facing test buckets with production PII

## CISO / Risk Manager View

**Why this matters at the board level.** Cloud is now the **default deployment model** for new systems; "on-prem" is the legacy. Yet the IBM Cost of a Data Breach Report 2023 found that **breaches in hybrid cloud took the longest to identify and contain** (108 days longer than on-prem). The board asks: *"Are we using cloud more safely than our competitors?"* The answer is the maturity of your shared-responsibility program.

**Economic lens.**

- Average breach cost: **$4.45M** (2023, all-industry); cloud-mature organizations saved **$1.5M per breach** vs. cloud-immature.
- Cloud spend is now **30%+ of total IT spend** in most enterprises. Security must be < 3% of that — but 3% of a large cloud bill is a large security program.
- Multi-cloud fragmentation: 89% of enterprises are multi-cloud (Flexera 2023), and **each CSP is a separate security stack to integrate**.

**Strategic actions.**

1. **Codify the shared responsibility matrix per service.** A one-page matrix per CSP service type, signed by the CISO, the cloud architect, and the service owner. Most enterprises do not have this.
2. **Default to CNAPP + DSPM**, not best-of-breed. Talent cost and alert fatigue both favor consolidation.
3. **Mandate customer-managed keys** for all Tier 3+ data. CSP-managed keys are operationally easier but offer no protection against CSP insider abuse or subpoena (CSP may be compelled to decrypt; with CMK, you control the key).
4. **Continuous posture management, not annual audit.** A misconfigured S3 bucket is a 5-minute problem; an annual scan is 360-day exposure. Deploy CSPM in *block* mode where possible (e.g., S3 Block Public Access account-wide, Azure Defender for Storage).
5. **Train cloud architects on security.** The cloud architect is the new sysadmin — and the source of 80% of cloud misconfigurations.
6. **Insist on contractual clarity** — data location, subprocessor notice, breach notification, deletion attestation, right to terminate.
7. **Crypto-shred the cloud.** Encrypt everything at rest with per-asset keys; destruction is a key-deletion event (see [[data-lifecycle-and-remnants]]).

## Related Connections

### Parent L2
- [[asset-inventory-and-categorization]] — cloud resources are assets; the inventory must include them
- [[data-classification-and-handling]] — labels must travel with data into the cloud
- [[data-lifecycle-and-remnants]] — cloud introduces new lifecycle stages (snapshots, replicas, region copies)
- [[data-privacy-and-protection]] — data residency, subprocessor risk, GDPR transfer mechanisms

### Sibling L3
- [[data-loss-prevention-dlp]] — Cloud DLP / CASB is the DLP architecture for SaaS
- [[data-retention-and-disposal-nist-800-88]] — cloud disposal = crypto-shred + provider attestation
- [[information-handling-standards-iso-27001-annex-a]] — ISO 27017 (cloud) and ISO 27018 (PII in public cloud) extend Annex A

### Cross-Domain
- [[../Domain-01-Security-and-Risk-Management/risk-management-framework]] — third-party risk + cloud risk treatment
- [[../Domain-01-Security-and-Risk-Management/legal-regulatory-and-compliance-issues]] — contract negotiation, jurisdiction
- [[../Domain-03-Security-Architecture-and-Engineering/Domain-03-Index]] — BYOK, HSM, confidential compute
- [[../Domain-05-Identity-and-Access-Management/Domain-05-Index]] — cloud IAM, federation, CIEM
- [[../Domain-07-Security-Operations/Domain-07-Index]] — SOC, CloudTrail ingestion, IR in cloud
- [[../Domain-08-Software-Development-Security/Domain-08-Index]] — DevSecOps in CI/CD, IaC scanning, SBOM

## References

- AWS Shared Responsibility Model — aws.amazon.com/compliance/shared-responsibility-model
- Microsoft Azure Shared Responsibility — learn.microsoft.com/azure/security/fundamentals/shared-responsibility
- Google Cloud Shared Responsibility — cloud.google.com/privacy/resources
- ISO/IEC 27017:2015 — Code of Practice for Information Security Controls Applicable to the Provision and Use of Cloud Services
- ISO/IEC 27018:2019 — Code of Practice for Protection of Personally Identifiable Information in Public Clouds
- Cloud Security Alliance (CSA) — Cloud Controls Matrix v4, Security Guidance for Critical Areas of Cloud Computing v4
- NIST SP 800-144 — Guidelines on Security and Privacy in Public Cloud Computing
- NIST SP 800-145 — The NIST Definition of Cloud Computing
- NIST SP 800-210 — General Access Control Guidance for Cloud Systems
- CIS Benchmarks — AWS, Azure, GCP (foundational + prescriptive)
- CSA Top Threats to Cloud Computing (2024) — "The Egregious 11"
- ENISA Cloud Computing Risk Assessment
- FedRAMP, C5 (Germany), ENS (Spain), IRAP (Australia) — regional assurance frameworks
