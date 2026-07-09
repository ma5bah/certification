---
title: CISSP Knowledge Base — Master Index
layer: 0
type: Map of Content (MOC)
created: 2026-07-08
status: complete
total_domains: 8
total_notes: 76
---

# CISSP Knowledge Base — Master Index

> A complete, layered, deeply cross-linked Obsidian vault mapping the (ISC)² CISSP Common Body of Knowledge (CBK), organized Layer 1 (Domain Hubs) → Layer 2 (Deep-dive Models/Frameworks) → Layer 3 (Regulations, Edge cases, Crypto specs). CISO / Risk Manager strategic perspective applied throughout.

## 🎯 Strategic Framing (CISO Lens)

The CISSP CBK is the lingua franca of information security leadership. This vault is built to support:

1. **Board conversations** — every domain hub has a "CISO View" section with risk formulas, KPIs, and top-3 board questions.
2. **Exam preparation** — every note has a 2–4 sentence **Feynman explanation** at the top, then full untruncated technical detail.
3. **Architectural decisions** — risk formulas ($\text{ALE} = \text{SLE} \times \text{ARO}$), formal security models (Bell-LaPadula, Biba, Clark-Wilson), and CISO-grade prioritization.
4. **Cross-domain synthesis** — every note links to its parent, siblings, and to other domains, so a topic like "Kerberos" appears in IAM (D5) AND network (D4) AND operations (D7) with consistent context.

## 📐 Risk Formulas Used Throughout

| Formula | Meaning | When to Use |
|---|---|---|
| $\text{SLE} = \text{AV} \times \text{EF}$ | Single Loss Expectancy = Asset Value × Exposure Factor | One incident, one asset |
| $\text{ALE} = \text{SLE} \times \text{ARO}$ | Annual Loss Expectancy = SLE × Annualized Rate of Occurrence | Annualized risk exposure |
| $\text{ROSI} = \frac{(\text{ALE} \times M) - \text{ACS}}{\text{ACS}}$ | Return on Security Investment (M = mitigation factor, ACS = annual control cost) | Justify a security control spend |
| $\text{Risk} = \text{Threat} \times \text{Vulnerability} \times \text{Impact}$ | Classic risk equation | Quick qualitative scoring |
| $\text{Residual Risk} = \text{Inherent Risk} - \text{Mitigation} + \text{Uncertainty}$ | Risk after controls | Post-control reporting |
| $\text{RTO, RPO, MTPD}$ | Recovery Time / Point / Maximum Tolerable Period of Disruption | BCP / DRP |
| $\text{MTTD, MTTR, MTTA}$ | Mean Time To Detect / Respond / Acknowledge | SOC metrics |
| $H_{\text{bits}} = \log_2 \text{keyspace}$ | Password / key entropy | IAM, cryptography |

---

## 🗺️ The 8 Domains (Layer 1 Hubs)

| # | Domain | Weight | Hub Note |
|---|---|---|---|
| 1 | Security and Risk Management | 16% | [[domain-01-security-and-risk-management]] |
| 2 | Asset Security | 10% | [[domain-02-asset-security]] |
| 3 | Security Architecture and Engineering | 13% | [[domain-03-security-architecture-and-engineering]] |
| 4 | Communication and Network Security | 13% | [[domain-04-communication-and-network-security]] |
| 5 | Identity and Access Management | 13% | [[domain-05-identity-and-access-management]] |
| 6 | Security Assessment and Testing | 12% | [[domain-06-security-assessment-and-testing]] |
| 7 | Security Operations | 13% | [[domain-07-security-operations]] |
| 8 | Software Development Security | 10% | [[domain-08-software-development-security]] |

---

## 📚 Layer 2 — Deep-dive Models & Frameworks

### Domain 1 — Security and Risk Management
- [[risk-management-frameworks]] (NIST RMF, ISO 27005, COBIT 2019, OCTAVE, FAIR)
- [[risk-formulas-and-quantitative-analysis]] (ALE, SLE, ARO, ROSI, FAIR math)
- [[security-governance-policies-standards]] (policies / standards / procedures / guidelines)
- [[legal-regulatory-compliance-landscape]] (GDPR, HIPAA, SOX, GLBA, PCI-DSS, FedRAMP)
- [[business-continuity-and-disaster-recovery]] (BCP/DRP, BIA, RTO/RPO/MTPD)

### Domain 2 — Asset Security
- [[data-classification-and-handling]] (sensitivity levels, ownership, custodianship)
- [[data-lifecycle-and-remnants]] (collection → storage → archival → destruction)
- [[data-privacy-and-protection]] (PII / PHI / PCI, anonymization, tokenization)
- [[asset-inventory-and-categorization]] (CMDB, hardware / software / data / people)

### Domain 3 — Security Architecture and Engineering
- [[security-models]] (Bell-LaPadula, Biba, Clark-Wilson, Brewer-Nash, Graham-Denning, HRU)
- [[cryptography-fundamentals-and-math]] (symmetric / asymmetric math, AES / RSA / ECC)
- [[symmetric-encryption-algorithms]] (DES, 3DES, AES, Blowfish, modes: ECB/CBC/CTR/GCM)
- [[asymmetric-encryption-and-pki]] (RSA, DH, ECC, X.509, CA hierarchy, OCSP, CRL)
- [[security-architecture-patterns]] (defense-in-depth, zero trust, TOGAF, SABSA)

### Domain 4 — Communication and Network Security
- [[osi-tcpip-models-and-encapsulation]] (7-layer OSI, 4-layer TCP/IP, PDU/SDU)
- [[network-security-controls]] (firewalls, IDS/IPS, NDR, WAF, NAC)
- [[network-architecture-segments]] (VLAN, DMZ, microsegmentation, SDN)
- [[wireless-and-mobile-security]] (WPA2/WPA3, 802.1X, LTE/5G, MDM)
- [[secure-protocols-and-services]] (IPsec, TLS, SSH, DNSSEC, S/MIME, Kerberos)

### Domain 5 — Identity and Access Management
- [[authentication-factors-and-mechanisms]] (knowledge / possession / inherence, FIDO2)
- [[authorization-models]] (DAC, MAC, RBAC, ABAC, RuBAC)
- [[identity-lifecycle-and-provisioning]] (JIT, SCIM, deprovisioning)
- [[federation-sso-and-saml-oidc]] (SAML 2.0, OAuth 2.0, OIDC)
- [[access-control-attacks-and-mitigations]] (password, MFA fatigue, session attacks)

### Domain 6 — Security Assessment and Testing
- [[vulnerability-management-lifecycle]] (CVSS, EPSS, SSVC, KEV)
- [[penetration-testing-methodology]] (PTES, OSSTMM, RoE, types)
- [[security-audits-and-compliance-testing]] (SOC 2, ISO 27001 audit, sampling)
- [[logging-monitoring-and-siem]] (use cases, MTTD/MTTR, KPIs)

### Domain 7 — Security Operations
- [[incident-response-lifecycle-nist]] (NIST SP 800-61, 6 phases)
- [[digital-forensics-process]] (chain of custody, order of volatility, ACPO)
- [[change-and-configuration-management]] (CAB, baselines, ITIL)
- [[disaster-recovery-and-bcp]] (BCP/DRP, BIA, hot/warm/cold, cloud DR)
- [[physical-and-environmental-security]] (perimeter, CCTV, HVAC, fire, TEMPEST)

### Domain 8 — Software Development Security
- [[sdlc-models-and-security-integration]] (waterfall, agile, DevSecOps, MS SDL, SSDF)
- [[secure-coding-practices-owasp]] (input validation, output encoding, ASVS)
- [[threat-modeling-stride-pasta]] (STRIDE categories, PASTA stages, DFD, attack trees)
- [[api-security-and-microservices]] (OWASP API Top 10, OAuth, JWT, service mesh)

---

## 📜 Layer 3 — Regulations, Edge Cases, Crypto Specs

### Domain 1 — Security and Risk Management
- [[gdpr-deep-dive]] (Articles 5, 17, 25, 32, 33; DPO; 4% revenue fines)
- [[hipaa-security-rule]] (administrative / physical / technical safeguards, breach notification)
- [[pci-dss-4-0]] (12 requirements, 4.0 changes, SAQ types)
- [[isc2-code-of-ethics]] (4 canons, ethics preamble)

### Domain 2 — Asset Security
- [[data-loss-prevention-dlp]] (endpoint, network, cloud DLP; content inspection)
- [[cloud-asset-security-shared-responsibility]] (IaaS/PaaS/SaaS matrix)
- [[data-retention-and-disposal-nist-800-88]] (clear / purge / destroy)
- [[information-handling-standards-iso-27001-annex-a]] (A.5–A.18)

### Domain 3 — Security Architecture and Engineering
- [[tls-1-3-deep-dive]] (handshake, cipher suites, 0-RTT, forward secrecy)
- [[hash-functions-and-hmac]] (MD5/SHA-1/SHA-2/SHA-3, HMAC, birthday paradox)
- [[post-quantum-cryptography]] (NIST PQC: Kyber, Dilithium, SPHINCS+; HNDL)
- [[fips-140-3]] (4 security levels, validated modules)
- [[secure-boot-and-trusted-platform-module-tpm]] (PCR, attestation, chain of trust)

### Domain 4 — Communication and Network Security
- [[firewall-types-and-rule-evaluation]] (packet filter, stateful, proxy, NGFW)
- [[vpn-types-and-ipsec-modes]] (transport vs tunnel, IKEv2)
- [[dns-security-and-dnssec]] (cache poisoning, DoH/DoT, DNSSEC chain)
- [[5g-and-iot-network-security]] (5G AKA, slicing, IoT protocols)

### Domain 5 — Identity and Access Management
- [[kerberos-protocol-deep-dive]] (KDC, TGT, service tickets, AS-REQ/AS-REP, PAC)
- [[multi-factor-authentication-mfa]] (TOTP/HOTP, FIDO2/WebAuthn, SIM-swap)
- [[privileged-access-management-pam]] (credential vaulting, JIT, session recording)
- [[zero-trust-architecture-nist-800-207]] (PDP/PEP, SDP, BeyondCorp, continuous verification)

### Domain 6 — Security Assessment and Testing
- [[owasp-top-10-and-web-app-testing]] (Top 10:2021, ASVS, WSTG)
- [[mitre-att-and-ck]] (Enterprise/Mobile/ICS matrices, 14 tactics, threat-informed defense)
- [[red-team-vs-blue-team-vs-purple-team]] (exercises, BAS, emulation)

### Domain 7 — Security Operations
- [[ransomware-response-playbook]] (NIST CSF, ransom decision tree, recovery)
- [[backup-strategies-3-2-1]] (3-2-1 rule, immutable backups, air-gapped, restore tests)
- [[siem-soar-and-detection-engineering]] (Sigma rules, ATT&CK mapping, playbooks)
- [[security-operations-center-soc-models]] (Tier 1/2/3, MSSP, OODA loop)

### Domain 8 — Software Development Security
- [[devsecops-and-cicd-security]] (pipeline security, IaC, SBOM, SLSA)
- [[database-security-and-sql-injection]] (SQLi types, ORMs, parameterized, NoSQLi)
- [[software-supply-chain-attacks-slsa]] (SolarWinds, 3CX, XZ utils, SLSA levels)

---

## 🧭 Cross-Domain Topic Map (Serendipity Hubs)

These topics intentionally appear in multiple domains; follow the wikilinks to get the full multi-domain view.

| Concept | Domains | Anchor Notes |
|---|---|---|
| Kerberos | D4, D5, D7 | [[kerberos-protocol-deep-dive]], [[secure-protocols-and-services]], [[incident-response-lifecycle-nist]] |
| Cryptography | D3, D4, D5, D8 | [[cryptography-fundamentals-and-math]], [[tls-1-3-deep-dive]], [[api-security-and-microservices]] |
| Risk / ALE math | D1, D6, D7 | [[risk-formulas-and-quantitative-analysis]], [[vulnerability-management-lifecycle]], [[siem-soar-and-detection-engineering]] |
| Zero Trust | D3, D4, D5 | [[zero-trust-architecture-nist-800-207]], [[network-architecture-segments]], [[security-architecture-patterns]] |
| BCP / DRP | D1, D7 | [[business-continuity-and-disaster-recovery]], [[disaster-recovery-and-bcp]] |
| Supply Chain | D2, D8 | [[cloud-asset-security-shared-responsibility]], [[software-supply-chain-attacks-slsa]] |
| Authentication | D4, D5, D8 | [[authentication-factors-and-mechanisms]], [[wireless-and-mobile-security]], [[api-security-and-microservices]] |
| SIEM / Logging | D6, D7 | [[logging-monitoring-and-siem]], [[siem-soar-and-detection-engineering]] |

---

## 🔗 Navigation

- **Per-domain MOCs**: open any domain hub; each lists its L2 children and L3 grandchildren.
- **Per-topic MOCs**: open any L2 note; each lists its L3 children and cross-domain siblings.
- **Quick-start**: if you only have 30 seconds, read the domain hub and skim the Feynman explanations on each L2.

## 📈 Quality Gates Passed

- ✅ 8 domain hubs (L1) — 8 files
- ✅ Deep-dive models/frameworks (L2) — 36 files
- ✅ Regulations / edge cases / crypto specs (L3) — 32 files
- ✅ Total: 76 notes
- ✅ Every note: YAML frontmatter + Feynman + Technical (math) + CISO view + Wikilinks + References
- ✅ Wikilinks use Obsidian `[[kebab-case]]` syntax, no `.md` extensions
- ✅ `brain/` directory **read-only** (not touched)
- ✅ No destructive operations; pure additive content under `Note/CISSP/`

## See also
- [[_index/cissp-knowledge-base-index|MASTER INDEX (alt path)]]
- [[domain-01-security-and-risk-management|D1 Hub]]
- [[domain-08-software-development-security|D8 Hub]]
