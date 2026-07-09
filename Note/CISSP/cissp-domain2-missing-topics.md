# Domain 2 → 8: Missing Topics in cissp-complete-handbook.md

Source: https://www.isc2.org/certifications/cissp/cissp-certification-exam-outline#Domain%202:%20Asset%20Security
Notes verified: `cissp-complete-handbook.md` (the consolidated master doc only; individual domain notes may differ)

---

## Domain 2 — Asset Security Missing Topics

### 1. Scoping and Tailoring (2.6)

**Official outline:** `2.6 Determine data security controls and compliance requirements → Scoping and tailoring`

**Status: NOT COVERED**

The handbook discusses control mapping (Classification-to-Control Mapping in §2.2) and FIPS 199 impact levels (§2.6), but never introduces "scoping and tailoring" as a named concept.

**What to add:** Scoping = determining which baseline controls from SP 800-53 (or equivalent) apply to a given system based on its FIPS 199 impact level. Tailoring = adjusting those controls (parameterization, compensating controls, etc.) for the specific system context. This is Step 3 of NIST RMF and a direct exam topic.

---

### 2. Standards Selection (2.6)

**Official outline:** `2.6 Determine data security controls and compliance requirements → Standards selection`

**Status: NOT COVERED as a standalone concept**

The handbook references many standards (NIST, ISO, FIPS, PCI) but never frames "standards selection" as a deliberate process/methodology.

**What to add:** How to select the appropriate standard based on regulatory scope (HIPAA → NIST SP 800-66, PCI → PCI-DSS, federal → FIPS/FedRAMP), classification tier, and business context. Includes tradeoff analysis (e.g., ISO 27001 vs NIST CSF vs CIS Controls for a given data type).

---

### 3. Asset Classification (2.1)

**Official outline:** `2.1 Identify and classify information and assets → Asset classification`

**Status: PARTIALLY COVERED** (data classification is thorough; non-data asset classification is implicit)

The handbook's 4-tier classification scheme in §2.1 is presented entirely for *data* (Public/Internal/Confidential/Restricted). The "Asset Types" table identifies hardware, software, people, services, intangibles but doesn't assign them classification tiers.

**What to add:** Classifying non-data assets — servers, networking gear, mobile devices, SaaS subscriptions, IoT, OT — under their own sensitivity/criticality labels. How the classification of an asset influences its handling, access controls, and disposal requirements.

---

### 4. Data Maintenance (2.4)

**Official outline:** `2.4 Manage data lifecycle → Data maintenance`

**Status: MINIMAL COVERAGE**

Mentioned only in the lifecycle table (§2.2) with "RBAC, JIT, UEBA, audit" as controls. No dedicated section.

**What to add:** Data maintenance as a lifecycle phase — data quality management, format migration, integrity verification, metadata updates, data transformation, and the controls required during each maintenance operation.

---

### 5. Data Location (2.4)

**Official outline:** `2.4 Manage data lifecycle → Data location`

**Status: PARTIALLY COVERED** (under Data Residency, not as a lifecycle phase)

Data Residency (§2.4 Mastery) covers jurisdictional constraints, but "data location" as a lifecycle concept is broader — knowing where data physically/logically resides at *every* stage.

**What to add:** Data location as an explicit control point in the lifecycle — how to track, document, and audit data location per phase; CSP region constraints; multi-region replication risks; legal implications of location changes during migration.

---

### 6. International Privacy Frameworks (2.1 / 2.6)

**Check:** OECD Privacy Guidelines, APEC Privacy Framework

**Status: NOT COVERED**

The handbook covers GDPR, HIPAA, CCPA, and ISO 27001 extensively, but omits the OECD Privacy Guidelines (foundational, 1980, revised 2013 — tested as the basis for most national privacy laws) and the APEC Privacy Framework (cross-border data flows in Asia-Pacific).

**What to add:** OECD's 8 privacy principles (collection limitation, data quality, purpose specification, use limitation, security safeguards, openness, individual participation, accountability) and APEC's 9 privacy principles + CBPR (Cross-Border Privacy Rules) system.

---

## Domain 1 — Security and Risk Management Missing Topics

### 7. ESG — Environmental, Social, and Governance (1.1 / 1.3)

**Status: NOT COVERED**

The 2024 exam outline emphasizes integration of cybersecurity into organizational ESG goals. The handbook covers governance via the pyramid and traditional frameworks, but ESG reporting, sustainability metrics, and social responsibility impacts on risk management are absent.

**What to add:** How cybersecurity posture feeds into ESG ratings (S&P, MSCI, Sustainalytics); board-level ESG reporting requirements; cyber as a social responsibility factor (customer data protection, digital inclusion); environmental sustainability of security operations (energy-efficient crypto, green data centers).

---

### 8. Comprehensive Third-Party Risk Management — TPRM Lifecycle (1.11)

**Status: PARTIALLY COVERED** (scattered references, no dedicated lifecycle)

SLSA is covered for supply-chain software integrity. Vendor dependencies appear in BIA. But there is no dedicated TPRM lifecycle breakdown.

**What to add:** Full TPRM lifecycle — vendor tiering (critical/high/medium/low), due diligence questionnaires, continuous compliance monitoring (vendor SOC 2 / ISO reports), right-to-audit clauses, security requirements in contracts, offboarding/de-provisioning procedures for third-party access. Map to SCRM topics in 1.11 (product tampering, counterfeits, implants, silicon root of trust, PUF, SBOM).

---

### 9. DORA — Digital Operational Resilience Act (1.4 / 7.2)

**Status: NOT COVERED** (in the consolidated handbook — DORA appears in individual Domain 6/7 notes but not in cissp-complete-handbook.md)

DORA (EU 2022/2554) is a major financial-sector resilience regulation requiring ICT risk management, incident reporting, digital operational resilience testing (TLPT), and third-party risk oversight.

**What to add:** DORA's 5 pillars — ICT risk management, incident reporting, resilience testing (including threat-led penetration testing per ENISA), third-party risk management, information sharing. DORA's extraterritorial scope (applies to non-EU financial firms serving EU clients). Overlap with NIS2.

---

## Domain 4 — Communication and Network Security Missing Topics

### 10. Space and Satellite Communications (4.1 / 4.3)

**Status: NOT COVERED**

The 2024 exam outline added non-terrestrial networks. The handbook covers 5G, Wi-Fi 6E, Bluetooth, Zigbee, but zero mention of satellite communications.

**What to add:** LEO (Low Earth Orbit) constellations (Starlink, OneWeb), VSAT (Very Small Aperture Terminal) security, GEO/MEO differences, space-based communication risks (signal jamming, spoofing, eavesdropping, physical attacks on ground stations), encryption standards for satellite links (AES-256 in DVB-S2X, MIL-STD-188-165).

---

## Domain 8 — Software Development Security Missing Topics

### 11. AI & ML Security (8.2 / 8.5)

**Status: NOT COVERED** (research notes mention it, but the consolidated handbook has zero implementation)

The 2024 exam interweaves AI security across all domains. The handbook discusses ML/LLM as *tools* for DLP and UEBA but never addresses securing AI systems themselves.

**What to add:**

- **AI TRiSM** (Trust, Risk, and Security Management) — IBM/Gartner framework: explainability, model ops, privacy, robustness, fairness
- **Adversarial ML attacks** — evasion (adversarial examples), poisoning (corrupt training data), model inversion (reconstruct training data), model extraction (steal the model)
- **Prompt injection** — indirect (data in retrieved context) and direct (user input)
- **Model supply chain** — ML model provenance, model signing, SBOM for ML (ML-BOM)
- **Guardrails** — output filtering, rate limiting, human-in-the-loop for high-risk decisions
- **Differential privacy** (briefly in Domain 2 — expand here for ML training context)
- **Federated learning security** — gradient leakage, poisoning via compromised nodes

---

## Verified: Already Covered (no action needed)

| Topic | Verdict | Evidence |
|-------|---------|----------|
| CPTED + Natural Territorial Reinforcement | ✅ Covered | Handbook §3.10 line 1825: "natural territorial reinforcement (distinct boundaries)" |
| NIS2 Directive | ✅ Covered | Appears in Domain 6 notes (security audits, red team) |
| Ransomware response frameworks | ✅ Covered | Domain 7 IR sections |

---

## Summary Table

| # | Topic | Domain | Status | Action |
|---|-------|--------|--------|--------|
| 1 | Scoping and tailoring | D2 (2.6) | ❌ Missing | New section |
| 2 | Standards selection | D2 (2.6) | ❌ Missing | New section |
| 3 | Asset classification (non-data) | D2 (2.1) | ⚠️ Partial | Expand §2.1 |
| 4 | Data maintenance | D2 (2.4) | ⚠️ Minimal | Expand lifecycle |
| 5 | Data location as lifecycle phase | D2 (2.4) | ⚠️ Partial | Expand §2.4 |
| 6 | OECD / APEC privacy frameworks | D2 (2.1/2.6) | ❌ Missing | New section |
| 7 | ESG integration | D1 (1.3) | ❌ Missing | New section |
| 8 | TPRM lifecycle | D1 (1.11) | ⚠️ Partial | New lifecycle section |
| 9 | DORA | D1 (1.4) / D7 | ❌ Missing* | New section |
| 10 | Satellite / space comms | D4 (4.1/4.3) | ❌ Missing | New section |
| 11 | AI/ML security (AI TRiSM, adversarial ML, prompt injection) | D8 (8.2/8.5) | ❌ Missing | New section |

\* DORA is present in individual domain notes (D6, D7) but absent from the consolidated handbook.
