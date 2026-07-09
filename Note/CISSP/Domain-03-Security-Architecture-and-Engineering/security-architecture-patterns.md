---
title: Security Architecture Patterns and Design Principles
layer: 2
domain: 3
tags: [cissp, security-architecture, defense-in-depth, zero-trust, togaf, sabsa, reference-monitor, saltzer-schroeder, cpted, secure-design, least-privilege, fail-secure]
related_domains: [1, 4, 5, 6, 7, 8]
---

# Security Architecture Patterns and Design Principles

## Feynman Explanation
A security architecture is a *layered defense*: don't trust one wall, one guard, or one lock — assume each will fail and have the next layer ready. The original 1975 Saltzer–Schroeder paper distilled every secure-design lesson into eight one-line principles (least privilege, fail-safe defaults, economy of mechanism, complete mediation, open design, separation of privilege, least common mechanism, psychological acceptability) that still drive every audit checklist today. Modern enterprises layer *defense in depth* on top, then increasingly *zero trust* — the realization that inside-the-firewall is not safe, every request must prove itself, and the network is treated as if it were the public internet.

## Technical Details

### 1. The Saltzer–Schroeder Design Principles (1975)

Jerry Saltzer and Michael Schroeder, in *The Protection of Information in Computer Systems* (Proc. IEEE), gave the canonical eight principles of secure design. Every CISSP architect cites these by number.

#### 1.1 Least Privilege
A subject should have *only* the privileges it needs to complete its authorized task — and not for a moment longer.

- **Mechanism**: capability tokens (POSIX capabilities on Linux, AWS IAM roles, Kubernetes service accounts).
- **Failure mode**: superuser accounts, "admin" service principals, "root" containers.
- **Measure**: time-of-check-to-time-of-use (TOCTOU) window; minimize standing privilege; *just-in-time* elevation.

#### 1.2 Fail-Safe Defaults (Fail Secure)
The default state of the system is the *deny* state; access is granted only by explicit permission.

- **Example**: a firewall's default rule is `deny any any`.
- **Contrast with fail-open**: a fail-open door is dangerous in a vault, fail-secure is dangerous on a fire exit. Context matters; the *principle* says "default deny."
- **In the cloud**: IAM default-deny + explicit policy; S3 buckets default-private.

#### 1.3 Economy of Mechanism (Keep It Simple)
The protection mechanism should be as small and simple as possible — both to verify and to find bugs in.

- The *reference monitor* is the classic example: small enough to analyze formally.
- **Anti-pattern**: an "AI-driven" anomaly-detection WAF whose decision path is unobservable. KISS applies.

#### 1.4 Complete Mediation
Every access to every object by every subject is checked — *every time*. Caching breaks this.

- **Concrete rule**: permissions checks happen at the *data layer* (database row), not at the UI. UI checks are hints; data-layer checks are the policy.
- **TOCTOU**: check the file exists and you have access (at $T_0$), then open it (at $T_1$). If the file is replaced in between, the check is invalid. Use `open(O_RDONLY | O_NOFOLLOW)` and re-check at $T_1$.

#### 1.5 Open Design
Security should *not* depend on the secrecy of the mechanism. Kerckhoffs's principle: *the algorithm is public; only the key is secret*.

- **Anti-pattern**: "security by obscurity" (custom cipher, hidden URL, hard-coded secret).
- **Modern example**: AES is open; Cloudflare's CAPTCHA logic is not secret, just well-instrumented.

#### 1.6 Separation of Privilege
A single subject should not be able to complete a sensitive action alone. Two (or more) identities must cooperate.

- **Implementations**: dual-control key release, four-eyes principle, M-of-N Shamir, two-person rule.
- **Clark-Wilson** formulation: a single subject cannot perform both halves of a CDI-modifying TP.
- **Modern**: Maker-Checker in banking; pull-request approval in CI/CD; second-factor for production deploys.

#### 1.7 Least Common Mechanism
Mechanisms shared by multiple subjects (e.g., a global variable, a shared cache, a single connection pool) should be minimized because they create covert channels.

- **Modern**: avoid shared service accounts; avoid shared credentials; prefer per-tenant resources.

#### 1.8 Psychological Acceptability
A security mechanism must be *usable*; if users cannot use it correctly, they will route around it.

- **Example**: an MFA system that requires copy-paste of a 6-digit TOTP *and* a hardware key on the same screen fails this. Choose one.
- **Anti-pattern**: passwords that expire every 30 days with 16-character minimums; users will write them on monitors.

### 2. The Reference Monitor (Anderson 1972, TCSEC 1983)

The *reference monitor* is the abstract machine that mediates every subject–object access. It must satisfy three properties:

1. **Complete mediation** — every access is checked.
2. **Tamper-proof** — the monitor cannot be modified by untrusted subjects.
3. **Verifiable** — small enough to be analyzed and proven correct.

The reference monitor is *abstract*; the concrete implementation is the **security kernel** of the TCB. The TCB includes the kernel, the trusted OS, the trusted hardware, and the chain of trust that boots it. See [[secure-boot-and-trusted-platform-module-tpm]] for the hardware layer.

**Common Criteria EAL4–EAL7** all require a reference monitor as part of the Security Target (ST) of the TOE (Target of Evaluation).

### 3. Defense in Depth

Defense in depth is the *layered* application of security controls. The classic Cast-Iron Stack:

```
                        (1) Policies / Standards
                    (2) Physical (perimeter, locks, CCTV)
                (3) Network (firewall, NIDS, segmentation)
            (4) Host (anti-malware, EDR, hardening, secure boot)
        (5) Application (authZ, input validation, output encoding)
    (6) Data (encryption at rest, tokenization, DLP, classification)
```

A single layer's failure is *not* total failure. A perimeter breach is contained by network segmentation. A network breach is contained by host hardening. A host breach is contained by application-layer authentication. An application bug is contained by data encryption (the stolen DB is ciphertext without the HSM-held key).

**Castledine–Cromwell–Salerno (2012)**: defense in depth is not just technical; it is *capability-rich* (defense across policy/people/process/technology). The OWASP Defense-in-Depth framework formalizes this.

### 4. Zero Trust Architecture (NIST SP 800-207, 2020)

#### 4.1 Premise
> "No implicit trust is granted to assets or user accounts based solely on their physical or network location."

The original perimeter is dissolved. Inside the firewall is not trusted; the cloud is not trusted; the device is not trusted; the user is not trusted until proven per request.

#### 4.2 Core Components
- **Policy Engine (PE)** — the decision point.
- **Policy Administrator (PA)** — the enforcement point.
- **Policy Enforcement Point (PEP)** — at every network/device boundary.
- **CDMs** (Continuous Diagnostics and Mitigation) — feeds.
- **Industry compliance** — feeds.

```
                    [ Subject ] (user, device, app)
                          |
                       (request)
                          v
+-------------------+      Trust Algorithm      +-------------------+
| Identity Provider |--------->[PE + PA]<--------| Threat Intelligence|
+-------------------+                            +-------------------+
                          |                       |  CDM / SIEM       |
                       (decision: allow/deny)     +-------------------+
                          v
                   [ PEP / Resource ]
```

#### 4.3 Tenets (NIST SP 800-207)
1. All data sources and computing services are considered resources.
2. All communication is secured regardless of network location.
3. Access to individual enterprise resources is granted on a per-session basis.
4. Access is governed by dynamic policy — including observable client state.
5. The enterprise monitors and measures the integrity and security posture of all owned and associated assets.
6. All resource authentication and authorization are dynamic and strictly enforced before access is granted.
7. The enterprise collects as much information as possible about the current state of assets, network infrastructure, and communications and uses it to improve its security posture.

#### 4.4 Microsegmentation
Per-workload, per-service policy — typically implemented as a service mesh (Istio, Linkerd) or per-pod NetworkPolicy. *East-west* traffic inside the cluster is subject to the same allow-listing as north-south.

#### 4.5 Practical Zero Trust: Beyond the Buzzword
A common anti-pattern is "ZTA = MFA on the VPN." Real ZTA requires:
- Strong identity (FIDO2/WebAuthn) — *not* SMS.
- Device posture (TPM attestation, MDM compliance).
- Per-app authorization (OAuth 2.0 + scopes, not VPN tunnels).
- Continuous evaluation (risk signals, session binding).
- Encrypted transport everywhere (mTLS, TLS 1.3).
- Telemetry-driven policy.

### 5. TOGAF (The Open Group Architecture Framework)

#### 5.1 Architecture Domains
- **Business Architecture** — strategy, governance, organization.
- **Data Architecture** — data assets, models, governance.
- **Application Architecture** — application portfolio, interactions.
- **Technology Architecture** — hardware, software, infrastructure.

#### 5.2 Architecture Development Method (ADM) Cycle
```
Preliminary Phase → Architecture Vision
                 → Business Architecture
                 → Information Systems Architectures (Data + Application)
                 → Technology Architecture
                 → Opportunities and Solutions
                 → Migration Planning
                 → Implementation Governance
                 → Architecture Change Management
                 → Requirements Management (continuous)
```

Security is a *cross-cutting concern* that touches every ADM phase; it is rarely a single ADM phase (a common criticism). In modern practice, *security* is treated as a parallel architecture stream.

### 6. SABSA (Sherwood Applied Business Security Architecture)

SABSA is a *security-domain-specific* architecture framework, the security complement to TOGAF/Zachman.

#### 6.1 The Six Layers (the SABSA Matrix)

| Layer | What (Assets) | Why (Motivation) | How (Process) | With (People) | Where (Location) | When (Time) |
|-------|---------------|------------------|---------------|---------------|-------------------|--------------|
| Contextual | Business goals | Risk appetite | Business processes | Org | Geography | Time horizon |
| Conceptual | Business attributes | Risk objectives | Business process risk | Roles | Sites | Times of risk |
| Logical | Info assets | Security policies | Security services | Credentials | Domains | Intervals |
| Physical | Data assets | Security standards | Security mechanisms | Identity | Platforms | Cycles |
| Component | IT assets | Security products | Tools | Users | Nodes | Events |
| Service | Service-level metrics | Operational risk | Service management | Operators | Facilities | Service hours |

SABSA is *traceable*: a control at the Component layer must trace up to a business motivation at the Contextual layer. The matrix is the tool.

### 7. Zachman Framework (1987)

A 6 × 6 classification of architectural artifacts:

| | What (Data) | How (Function) | Where (Network) | Who (People) | When (Time) | Why (Motivation) |
|--|-------------|----------------|-----------------|---------------|--------------|------------------|
| **Scope (Planner)** | List of things | List of processes | List of places | List of orgs | List of events | List of goals |
| **Business (Owner)** | Semantic | Business | Business | Business | Business | Business |
| **System (Designer)** | Logical | Logical | Logical | Logical | Logical | Logical |
| **Technology (Builder)** | Physical | Physical | Physical | Physical | Physical | Physical |
| **Component (Subcontractor)** | Detailed | Detailed | Detailed | Detailed | Detailed | Detailed |
| **Operation (User)** | Functioning | Functioning | Functioning | Functioning | Functioning | Functioning |

Security is best planned at the System (Designer) row — the *logical* model — and implemented at the Technology (Builder) row.

### 8. Common Criteria (ISO/IEC 15408) — Evaluation Assurance Levels

| EAL | Description | Common use |
|-----|-------------|------------|
| 1 | Functionally tested | Self-asserted |
| 2 | Structurally tested | Low-risk commercial |
| 3 | Methodically tested and checked | OS, firewalls |
| 4 | Methodically designed, tested, reviewed | OS, firewalls (high-grade) |
| 5 | Semiformally designed and tested | High-grade OS, smart cards |
| 6 | Semiformally verified design and tested | High-grade secure OS |
| 7 | Formally verified design and tested | Highest assurance (rare) |

- **EAL4+** is the practical ceiling for commercial-off-the-shelf products.
- **EAL5–7** is generally reserved for government/military (e.g., VxWorks MILS, Integrity DO-178B).
- A **Protection Profile (PP)** is a reusable security requirements document for a class of products (e.g., "Software-based Disk Encryption").
- A **Security Target (ST)** is a vendor's claim of conformance to one or more PPs.

### 9. Secure Boot, Attestation, Hardware Roots of Trust

See [[secure-boot-and-trusted-platform-module-tpm]] for full detail. Architectural summary:

- **Measured boot** — the boot loader's hash is extended into a TPM PCR; OS kernels and drivers are measured; the chain produces an *attestation quote* the remote verifier can validate against an *expected* PCR policy.
- **Trusted boot** — refuses to load an unmeasured component.
- **Secure boot** (UEFI) — refuses to load a component whose signature is not in the OEM's DB.
- **FIDO2 / WebAuthn** — uses the TPM (or secure element) to produce a *cryptographic* assertion of user presence, replacing passwords.

### 10. Physical Security Architecture

#### 10.1 CPTED (Crime Prevention Through Environmental Design)
Three principles:
- **Natural surveillance** — sight lines, lighting, low landscaping.
- **Natural access control** — fences, gates, paths that funnel.
- **Natural territorial reinforcement** — distinct boundaries that signal ownership.

#### 10.2 Site / Facility Layers
1. **Perimeter** — fence, gate, vehicle barrier (Bollard K-rating: K4, K12).
2. **Building exterior** — doors, mantraps, vehicle sally ports.
3. **Building interior** — zoned; reception, public, staff, restricted, secure (data center).
4. **Data center / SCIF** — 4-zone cage, biometric + mantrap, Faraday-cage wall, dual-utility feeds.
5. **Server / equipment** — locked cabinet, tamper-evident seals.

#### 10.3 Media Storage and Destruction
- NIST SP 800-88 Rev. 1 — *Guidelines for Media Sanitization*:
  - **Clear**: overwrite with a known pattern (remanence risk on modern drives).
  - **Purge**: cryptographic erase (encrypt the disk, lose the key) or block erase.
  - **Physical destruction**: shredding, degaussing, incineration, disintegration.
- For SSDs, *cryptographic erase* is the only effective software-level sanitization; HDDs require physical destruction.

### 11. Concurrency, Race Conditions, TOCTOU

- **Race condition**: two processes access a shared resource; the result depends on the order of operations.
- **TOCTOU** (time-of-check to time-of-use):

$$
T_0: \text{check} \rightarrow T_1: \text{use}, \quad \text{where } T_1 - T_0 > 0
$$

Defense: re-check at $T_1$; use *atomic* operations (compare-and-swap, file locks, advisory locks); use *capabilities* (file descriptors are capabilities — the kernel performs the check at use time).

### 12. Cloud and Microservices Patterns

| Pattern | Purpose |
|---------|---------|
| Service mesh (Istio, Linkerd) | mTLS between pods, per-route authZ |
| Sidecar proxy | PEP in zero trust |
| API gateway | Centralized authZ, rate limit, WAF |
| Identity-aware proxy (Google IAP, Cloudflare Access) | ZTA in front of legacy HTTP |
| Secret management (HashiCorp Vault, AWS Secrets Manager) | Centralized, auditable, short-lived secrets |
| Immutable infrastructure (containers, AMIs) | No drift; rebuild, not patch |
| Policy-as-code (OPA, Cedar) | Versioned, testable authZ |
| SLSA / Sigstore | Software supply chain integrity |

### 13. Anderson's Report Findings (1972 — the *original* reference)

James P. Anderson's report to the US Air Force identified the three requirements that became the *reference monitor* — and added a fourth:

1. Complete mediation
2. Tamper-proof
3. Verifiable
4. (Anderson's addition) **Least surprise** — the system's behavior should be predictable from the user's mental model.

The fourth principle, often overlooked, is the basis of *psychological acceptability* (Saltzer–Schroeder) and the *principle of least surprise* in UX/HCI.

## CISO / Risk Manager View

**Board framing**: "What is the *one* architectural control that, if implemented correctly, cuts the most risk?" The answer is the *reference monitor*: complete, tamper-proof, verifiable. A working reference monitor turns every other control into a check on a known-good baseline.

**Architectural program essentials**:
1. **Named framework** — pick one (TOGAF, SABSA, Zachman) and be able to defend the choice to the auditor. SABSA is the security-domain-specific choice; TOGAF is the enterprise choice.
2. **Documented principles** — the eight Saltzer–Schroeder principles, written down, signed by the CISO, in the design review checklist.
3. **Reference monitor identified** — every system has a named reference monitor with properties 1/2/3 documented.
4. **Defense in depth, named layers** — at least 5 distinct control layers, each with a designated owner.
5. **Zero-trust roadmap** — per-app authorization, identity-aware proxy, mTLS, FIDO2.
6. **CTEM** (Continuous Threat Exposure Management) — continuous evaluation of controls, not annual.
7. **Crypto-agility** — algorithms behind versioned APIs, no hard-coded ciphers.

**Common board questions** (and architectural answers):
- *How do you know a breach hasn't already happened?* Reference monitor + continuous monitoring + CTEM.
- *What is your insider-threat posture?* Zero trust + least privilege + separation of duty + just-in-time elevation.
- *How do you keep up with quantum?* Crypto-agility + PQC roadmap + hybrid KEM.
- *What about supply chain?* SLSA Level 3+ for build provenance + Sigstore-signed artifacts + SBOMs.

**Common architecture anti-patterns** to call out in audit:
- *Castle-and-moat* — implicit trust inside the firewall. Replace with ZTA.
- *Snowflake* — every app has its own auth, no SSO. Replace with central IdP.
- *Hard-coded secrets in code*. Replace with secret manager + short-lived tokens.
- *Admin-by-default*. Replace with standing least privilege + JIT.
- *Auditor = developer*. Replace with separate teams, separate credentials (Clark-Wilson separation of duty).

**Vendor selection heuristic**: ask the vendor for their Security Target (ST) and Protection Profile (PP) under CC. If they are evaluated at EAL4+ and the ST names the threat model, the architecture is documented. If not, the architecture is folklore.

## Related Connections
- Security models (the formal layer): [[security-models]]
- Cryptography fundamentals: [[cryptography-fundamentals-and-math]]
- Secure boot / TPM: [[secure-boot-and-trusted-platform-module-tpm]]
- TLS 1.3: [[tls-1-3-deep-dive]]
- FIPS 140-3: [[fips-140-3]]
- Domain 5 (IAM) — implements the reference monitor in practice
- Domain 7 (operations) — runs the controls
- Domain 8 (secure dev) — builds the architecture into the code
- Domain hub: [[domain-03-security-architecture-and-engineering]]

## References
- Saltzer, J. H.; Schroeder, M. D. (1975). *The Protection of Information in Computer Systems*. Proc. IEEE 63(9).
- Anderson, J. P. (1972). *Computer Security Technology Planning Study*. USAF ESD-TR-73-51.
- DoD 5200.28-STD (1985). *Trusted Computer System Evaluation Criteria* (TCSEC / Orange Book).
- NIST SP 800-207 (2020). *Zero Trust Architecture*.
- NIST SP 800-204A (2020). *Building Secure Microservices-based Applications Using Service-Mesh Architecture*.
- NIST SP 800-88 Rev. 1 (2014). *Guidelines for Media Sanitization*.
- ISO/IEC 15408 (2022). *Common Criteria for Information Technology Security Evaluation*.
- The Open Group (2022). *TOGAF Standard, Version 10*.
- Sherwood, J. (2009). *SABSA White Paper*.
- Zachman, J. A. (1987). *A Framework for Information Systems Architecture*. IBM Sys. J.
- OWASP Foundation — *Defense in Depth* and *Cheat Sheet Series*.
- NIST SP 800-160 Vol. 1 Rev. 1 (2018). *Systems Security Engineering*.
- MITRE ATT&CK and MITRE D3FEND — operational architecture.
- Crowe, J. et al. (2017). *Security Engineering: A Guide to Building Dependable Distributed Systems*.
