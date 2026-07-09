---
title: Domain 8 — Software Development Security
layer: 1
domain: 8
tags: [cissp, software-security, sdlc, devsecops, application-security]
related_domains: [2, 3, 5, 6]
---

# Domain 8 — Software Development Security

## Feynman Explanation

Imagine building a house. You can choose where to put the locks — on the front door, on every window, on the safe, in the walls. If you wait until the drywall is up, you will spend ten times as much and still leave gaps. **Software Development Security is the discipline of deciding where the locks go while the blueprints are still on the table.** It covers the models used to build software (SDLC, agile, DevSecOps), the secure-coding practices that go into each line of code, the threat modeling that decides which attacks to design against, and the supply-chain and API controls that protect the system once it is running.

The CISSP framing of this domain is **process, not code** — the CISO does not review every function; the CISO designs the program that makes insecure code hard to merge and easy to find.

## Technical Details

### Scope of Domain 8 (CISSP CBK)

The 2024 CBK organizes Domain 8 into four concerns that span the entire software lifecycle:

| Concern | Question Answered | Layer Anchor |
|---|---|---|
| **Software development lifecycle (SDLC)** | How is software built and where do security gates fit? | [[sdlc-models-and-security-integration]] |
| **Secure coding & OWASP** | What patterns produce code that resists attack? | [[secure-coding-practices-owasp]] |
| **Threat modeling & design** | What attacks do we design against, and how? | [[threat-modeling-stride-pasta]] |
| **API & microservices security** | How do we protect the modern distributed attack surface? | [[api-security-and-microservices]] |

Two cross-cutting sub-disciplines reinforce all four: **DevSecOps** (security in CI/CD) and **software supply-chain risk** (provenance, SBOM, SLSA). Both are covered as Layer 3 deep-dives.

### Core Principles

1. **Security is a lifecycle concern, not a phase.** Bolt-on security at the end is 10–100× more expensive than design-time security. NIST estimates a defect fixed in production is $\approx 30\times$ the cost of fixing it at design.
2. **Default-deny and least-privilege apply to code too.** Parameterized queries, prepared statements, allow-lists, and capability-based access are the same pattern as firewall rules — refuse what is not explicitly permitted.
3. **Trust boundaries are where controls live.** Every input that crosses a boundary (browser → server, microservice → DB, third-party SDK → app) is a candidate injection point.
4. **The SDLC is a feedback loop, not a waterfall.** Modern programs are iterative: commit → scan → test → review → deploy → monitor → feedback. Every stage produces evidence the next stage consumes.
5. **Code, build, and runtime provenance must be verifiable.** SLSA, SBOM, signed images, and reproducible builds are the supply-chain equivalent of a tamper-evident seal on a pharmaceutical.

### Where Domain 8 Sits in the CISSP Map

| Domain | What It Owns | What It Hands to D8 |
|---|---|---|
| D1 Risk | Threat register, risk acceptance | Threat actors that threat models must address |
| D2 Asset | Data classification, handling rules | Data sensitivity that drives input validation |
| D3 Engineering | Crypto primitives, secure protocols | TLS, hashing, signing libraries used in code |
| D5 IAM | Identity, OAuth, JWT, sessions | AuthN/AuthZ that APIs must enforce |
| D6 Testing | SAST/DAST/IAST, pen-test reports | Defect data that feeds back into SDLC |
| D7 Operations | Patch management, monitoring | Runtime signals (RASP, WAF) that close the loop |

### Maturity Model — Application Security (BSIMM / SAMM preview)

| Level | Characteristic | Board-visible metric |
|---|---|---|
| **1 — Ad hoc** | Penthests after launch, no standards | # of high/critical findings per release |
| **2 — Repeatable** | SAST in CI, secure-coding training | % of code covered by SAST, time-to-remediate |
| **3 — Defined** | Threat models per feature, security gates in CI | % of releases blocked by gate, MTTR |
| **4 — Managed** | Metrics drive investment, SLSA L2+, SBOM in production | Mean defect-escape rate, supply-chain attestations |
| **5 — Optimized** | Continuous verification, RASP, chaos testing | Defect-escape rate $\leq 0.1\%$, automated rollback |

### Key Frameworks & Standards

| Framework | Owner | Use |
|---|---|---|
| **NIST SP 800-218 SSDF** (Secure Software Development Framework) | NIST | Four practice groups: PO, PS, PW, RV — required by US EO 14028 |
| **OWASP SAMM** | OWASP | AppSec maturity model, 5 business functions |
| **OWASP ASVS** | OWASP | Verifiable secure-coding requirements (3 levels) |
| **BSIMM** | Synopsys | Observed-maturity benchmark across 130+ firms |
| **MS SDL** | Microsoft | 17 practices across 7 phases |
| **SLSA** | OpenSSF | Build provenance framework (4 levels) |
| **OWASP Top 10 (Web & API)** | OWASP | Awareness; not a standard |
| **NIST SP 800-53 SI / SA families** | NIST | FedRAMP / government SDLC controls |

### The Economic Argument (Board-level Math)

Industry data (Veracode / Synopsys State of Software Security reports) consistently shows:

$$\text{Defect escape rate} = \frac{\text{Vulns found in prod}}{\text{Vulns found in dev + prod}}$$

A mature AppSec program drives escape rate from $\sim 30\%$ (maturity 1) to $< 1\%$ (maturity 4). The cost differential:

$$\text{Cost ratio} = \frac{\text{Fix in production}}{\text{Fix at design}} \approx 30 : 1$$

A 1,000-engineer organization producing 100 production defects/year at maturity 1 spends $\approx 100 \times C_{\text{prod}}$ in incident response and rework. At maturity 4, the same program spends $\approx 3 \times C_{\text{prod}}$ — a reduction of $\sim 97\%$ in defect-driven cost.

### Sample Defect-Reduction Curve (illustrative)

$$\text{Escape rate after } n \text{ SDLC gates} = e^{-\lambda n}, \quad \lambda \approx 0.7 \text{ per gate}$$

Adding SAST + DAST + IAST + threat-model review + dependency scan (5 gates) drops escape rate to $\sim 3\%$ of the no-gate baseline.

<!-- EXPANDED — Expert Knowledge Expansion (Phase 5) -->

## Expert Knowledge Expansion

### Foundations

**"Security is a quality attribute, not a feature."** This is the single most important sentence in Domain 8. Security cannot be added after the fact — it must be a property of the system, like reliability or performance. The **cost-of-fix curve** quantifies this intuition:

| Stage Found | Relative Cost | Real-World Anchor |
|---|---|---|
| Requirements | $1 \times$ | A one-line spec change |
| Design | $10 \times$ | Rewriting architecture docs |
| Implementation | $100 \times$ | Refactoring code + retesting |
| Testing / QA | $1{,}000 \times$ | Full regression, delayed release |
| Production | $10{,}000+ \times$ | Incident response, PR, legal, customer loss |

**The shift-left principle:** move security activities as early as possible — threat modeling at design, SAST at commit, SCA at PR, DAST in staging. Every gate shifted left reduces the escape rate exponentially (see the defect-reduction curve above).

**What a senior AppSec engineer knows:** every framework has an "unhappy path" — the escape hatches that bypass security defaults. React's `dangerouslySetInnerHTML`, Django's `mark_safe`, SQLAlchemy's `text()`, Nginx's `merge_slashes`. Attackers find these first. The **vulnerability lifecycle** is: introduced (dev writes escape hatch) → discovered (researcher/attacker finds it) → weaponized (exploit published) → patched (vendor ships fix) → exploited in the wild (mass scanning begins). The gap between "patch available" and "patch deployed" is where breaches happen — this is why MTTR is a board-level metric.

### Architecture

**Secure SDLC architecture (end-to-end):**
```
Requirements → Threat Modeling → Secure Design Review → Secure Coding
  → SAST → Dependency Scan (SCA) → DAST → Pentest
  → Production Monitoring (RASP/WAF) → Feedback Loop
```
Every stage produces **evidence** (SARIF, SBOM, pentest report, SLSA attestation) consumed by the next stage. A mature program automates this pipeline; a mature CISO measures defect-escape rate across it.

**DevSecOps pipeline architecture (canonical gate sequence):**
```
git commit → SAST (CodeQL/Semgrep) → SCA (Snyk/OSV) → Secrets Scan (TruffleHog)
  → Build → Container Scan (Trivy/Grype) → IaC Scan (Checkov/tfsec)
  → Deploy to Staging → DAST (ZAP/Burp) → Approval Gate → Deploy to Prod
  → SBOM Attestation → SLSA Provenance → Image Signing (cosign)
```
Each gate is a **blocking check** — a finding above an agreed severity threshold stops the pipeline. The engineering team owns the fix; the AppSec team owns the policy (severity thresholds, rule tuning, false-positive management).

**API security architecture:**
```
Client → API Gateway (AuthN, Rate Limiting, Schema Validation, WAF)
  → Backend Service (AuthZ: BOLA/BFLA check, Input Validation, Output Encoding)
  → Service Mesh (mTLS, Policy Enforcement)
  → Database (Parameterized Queries, Least Privilege, TLS)
```

**Microservices security mesh (SPIFFE/SPIRE pattern):**
Every service gets a cryptographic identity (SPIFFE ID) via SPIRE. The service mesh (Istio/Linkerd) enforces mTLS and authorization policies at the proxy layer. A policy might read: "Service `payment-processor` may call `fraud-detection` on path `/v1/score` — all other calls denied." This is **zero-trust east-west** — no service trusts another by network location.

### Execution

**Conducting threat modeling (STRIDE applied):**

| STRIDE Category | Maps to Security Property | Controls | DFD Element |
|---|---|---|---|
| **S**poofing | Authentication | MFA, certificates, OIDC | External entities, processes |
| **T**ampering | Integrity | HMAC, digital signatures, TLS, DB constraints | Processes, data stores, data flows |
| **R**epudiation | Non-repudiation | Signed audit logs, tamper-evident logging | External entities, processes, data stores |
| **I**nformation Disclosure | Confidentiality | Encryption, access control, classification (D2) | Processes, data stores, data flows |
| **D**enial of Service | Availability | Rate limiting, capacity planning, circuit breakers (D7) | Processes, data stores, data flows |
| **E**levation of Privilege | Authorization | Least privilege, input validation, server-side authZ (D5) | All elements |

**Secure code review checklist:**
1. **Input validation** — Are all trust-boundary inputs validated against an allow-list?
2. **Output encoding** — Is output encoded for the destination context (HTML, JS, URL, SQL)?
3. **Auth checks** — Does every endpoint re-check authorization server-side? (BOLA prevention)
4. **Error handling** — Do errors leak stack traces, SQL, file paths, or framework versions?
5. **Logging** — Are security events logged; are secrets/sessions/PII excluded from logs?
6. **Crypto usage** — Is `Math.random()`, `MD5`, `DES`, or `ECB` used anywhere?
7. **Session management** — Are tokens short-lived? Is `alg: none` rejected?
8. **Dependencies** — Are versions pinned? Are there known CVEs?

**Deploying SAST/DAST/SCA in CI/CD:**
- **SAST (Static):** False-positive management is the real job. A SAST tool generating 10,000 findings with no triage destroys the program. Tune rules aggressively: block on high/critical only; let medium/low findings feed a backlog. Use SARIF as the interchange format.
- **DAST (Dynamic):** Authenticated crawling matters — a DAST scan that cannot log in tests only the login page. Use ZAP's "AJAX spider" or Burp's "crawl" for SPAs. Run DAST in staging, not just pre-release.
- **SCA (Dependency):** The real problem is **transitive dependency explosion** — your 10 direct dependencies pull in 500 transitive ones. Use lockfiles, internal proxies, and reachability analysis (does your code actually call the vulnerable function?).

**Implementing SQLi defenses (defense-in-depth):**
1. **Parameterized queries with bound parameters** — the primary control. Prepared statements separate SQL structure from data.
2. **Input validation (allow-list)** — reject unexpected characters at the boundary.
3. **Least-privilege DB accounts** — app user has `SELECT/INSERT/UPDATE/DELETE`, never `DROP` or `GRANT`.
4. **WAF as defense-in-depth** — not a primary control. WAF bypass is routine; parameterization is the only guarantee.
5. **TLS to the database** — `verify-full`, never `disable` or `prefer`.
6. **No DDL in app queries** — schema changes are migrations, not runtime SQL.

```sql
-- NEVER: string concatenation
SELECT id FROM users WHERE name = '${user}' AND pwd = '${pwd}'

-- PRIMARY DEFENSE: parameterized
-- Python: db.execute("SELECT id FROM users WHERE name=? AND pwd=?", (user, pwd))
-- Java:   PreparedStatement ps = conn.prepareStatement("SELECT id FROM users WHERE name=? AND pwd=?")
-- Go:     db.Query("SELECT id FROM users WHERE name=$1 AND pwd=$2", user, pwd)
```

### Mastery

**SLSA (Supply-chain Levels for Software Artifacts) — levels 0 through 3:**

| Level | Guarantee | Build Requirement | Verifiability |
|---|---|---|---|
| **SLSA 0** | None | Any build process | No provenance |
| **SLSA 1** | Provenance exists | Documented build steps | Unsigned; trust the builder's word |
| **SLSA 2** | Signed provenance | Hosted build platform (GitHub Actions, GitLab, GCP Cloud Build) | Cryptographically verifiable to a platform identity |
| **SLSA 3** | Non-falsifiable provenance | Hardened build platform, two-person review, HSM-backed signing | Even the build platform cannot forge provenance |

Most enterprises should target SLSA 2 within 12 months (~5% build-pipeline budget). SLSA 3 is for highest-assurance systems (payment processing, aerospace, critical infrastructure).

**Sigstore/cosign for artifact signing:** Sigstore provides free, OIDC-based code signing. `cosign sign` attaches a signature to a container image; `cosign verify` checks it against the Rekor transparency log. The chain: developer identity (OIDC) → Fulcio (short-lived cert) → cosign (sign) → Rekor (public ledger). This is the open-source answer to "how do I know this image came from my build pipeline?"

**SBOM — what is in your software:**
- **SPDX** (ISO/IEC 5962:2024): Linux Foundation; license-focused; rich metadata.
- **CycloneDX**: OWASP; security-focused; supports VEX, SaaSBOM, ML-BOM.
- SBOM is not shelfware — operationalize it: pipe to vulnerability management, feed to customer transparency portal, query on CVE drop.

**Dependency confusion attacks:** An internal package name (`acme-internal-auth`) is published publicly with a higher version number. The package manager resolves to the public (attacker-controlled) version. Defenses: scope private packages to internal registry; namespace reservation on public registries; internal package proxy (Artifactory/Nexus) as the sole upstream.

**Fuzzing:** Coverage-guided fuzzing (libFuzzer, AFL++) feeds mutated inputs to a target and measures code coverage — new coverage paths are queued for further mutation. Structure-aware fuzzing for protocols (Protobuf, JSON, HTTP) uses the schema to generate syntactically valid inputs that still explore edge cases. Fuzzing finds bugs that code review and SAST miss — memory corruption, assertion failures, hangs.

**Memory safety:** The three-generation shift:
- **C/C++**: Manual memory management — buffer overflows, use-after-free, double-free. Mitigations: ASLR, stack canaries, Control Flow Guard. These are mitigations, not fixes.
- **Go**: Garbage collected; no manual `free()` — eliminates use-after-free. Still has race conditions and nil-pointer dereferences.
- **Rust**: Ownership model — borrow checker eliminates data races, use-after-free, and buffer overflows at compile time. Unsafe blocks are explicit and auditable.
- Why it matters: ~70% of Chrome/Microsoft/Android critical CVEs are memory-safety bugs. Memory-safe languages eliminate entire vulnerability classes.

**Formal verification:** Proving mathematically that a program satisfies its specification. Worth the cost for safety-critical systems (flight control, medical devices, cryptographic libraries). For typical SaaS, the cost-benefit currently favors fuzzing + SAST + code review.

### Resilience

**Top AppSec mistakes in production:**
1. Trusting user input (the root cause of injection, XSS, BOLA, path traversal).
2. Security as the last sprint ("we'll pentest before launch").
3. No threat modeling — design flaws discovered in production at 10,000× cost.
4. SQLi still the #1 injection vector — 20+ years after discovery.
5. Hardcoded secrets in repos — detected by automated scanners, still pervasive.
6. No dependency scanning — Log4Shell exposure was SBOM-gated.
7. Shipping debug mode — `/admin/debug`, stack traces in 500 responses, verbose errors.
8. No API rate limiting — enumeration and brute-force are trivial.

**When SAST produces 10,000 findings:** Triage is the real job. Categorize into: (a) true positive — remediate; (b) true positive but not exploitable (VEX) — document why; (c) false positive — suppress with rule tuning. If you can't triage to zero monthly, the program is noise, not signal.

**When "shift left" becomes "shift left and forget":** Shifting security into CI is necessary but insufficient. Production still needs shield-right: RASP (runtime self-protection), WAF, egress filtering, behavioral monitoring. The XZ Utils backdoor was not caught by SAST or SCA — it required behavioral detection (500ms sshd latency).

**The XZ Utils backdoor (CVE-2024-3094):** A maintainer spent 2 years building trust, then introduced a backdoor in `liblzma` that intercepted `sshd` authentication. It was caught by Andres Freund (Microsoft) noticing a 500ms latency anomaly — not by static analysis, not by code review. Nearly made it into Debian, Fedora, openSUSE, and Kali production. Lesson: trust is not a control; provenance + behavioral monitoring + anomaly detection are.

**When WAF gives false confidence:** WAFs are bypassable via HTTP parameter pollution, encoding tricks (double URL encoding, Unicode normalization), request smuggling, and JSON/XML body injection. WAF is defense-in-depth, not the primary control. Parameterized queries prevent SQLi regardless of WAF presence.

### Context

**Security in Agile:** Treat security stories as first-class backlog items with story points. Definition of Done includes: threat model attached, SAST clean, secrets scanned, dependency scan passed, code review done. Security Champions program: embed one AppSec-trained engineer per team — they triage findings, run threat models, and are the bridge to the central AppSec team.

**Security in Waterfall:** Security gates at each phase transition — requirements review → design review (threat model) → code review → test review → release review. Each gate is a formal sign-off. The waterfall model is compatible with high-assurance systems (FDA, DO-178C, IEC 62443) where each phase must be documented and audited.

**Open source security assessment:**
| Factor | Green Flag | Red Flag |
|---|---|---|
| Maintainer activity | Regular commits, multiple maintainers | Single maintainer, last commit 2+ years ago |
| Security posture | Security policy, SECURITY.md, vuln disclosure process | No security contact, issues ignored |
| Dependency health | Up-to-date, pinned, lockfile present | Outdated, no lockfile, known CVEs unfixed |
| Incident history | Transparent post-mortems, CVE published, fixed promptly | CVEs ignored or hidden, no disclosure policy |
| Build provenance | SLSA L2+, signed releases, SBOM available | Unsigned tarballs, no provenance |

**Commercial vs OSS — different risk models:**
- **Commercial (COTS):** You trust the vendor. Control via contract, SLA, attestation (SSDF, SOC 2, ISO 27001). Risk: vendor breach (SolarWinds, MOVEit).
- **OSS:** You own the risk. Control via SCA, internal mirror, code review. Risk: malicious maintainer (XZ), abandoned package, typosquatting.
- Neither is inherently safer — both require supply-chain controls appropriate to the trust model.

**Mobile app security specifics:**
- Certificate pinning: hardcode or pin the server's public key to prevent MITM via rogue CA.
- Jailbreak/root detection: detect compromised devices and restrict functionality or wipe data.
- Secure enclave key storage: iOS Secure Enclave, Android StrongBox — private keys never leave hardware.
- App attestation: Apple DeviceCheck, Google SafetyNet/Play Integrity — server verifies the app is unmodified and running on a genuine device.

### Nuance

**SAST (white-box) vs DAST (black-box) vs IAST (gray-box):**

| Tool | Visibility | Finds | Misses | Best Use |
|---|---|---|---|---|
| **SAST** | Source code | Injection, XSS, hardcoded secrets, weak crypto | Logic flaws, auth bypass, BOLA | Per-PR, blocking gate |
| **DAST** | Running application | Runtime misconfig, auth bypass, injection in context | Code-level patterns, dead code | Staging/per-release validation |
| **IAST** | Instrumented runtime (agent inside app) | Data-flow verification, runtime context | Requires test coverage | Complementary to SAST in staging |

**Penetration testing vs vulnerability scanning — why you need both:** A vulnerability scanner finds known patterns (CVEs, misconfigurations). A pentester finds logic flaws (BOLA, business logic abuse, workflow bypass) that scanners miss because they require understanding the application's intent. Scan continuously; pentest quarterly or per-major-release.

**Code review vs static analysis:** SAST finds patterns — SQL concatenation, weak crypto, missing encoding. Humans find logic flaws — "this endpoint checks authN but skips authZ," "this loop has a race condition." Tools scale to every PR; humans scale to critical-path code only. The combination catches what neither catches alone.

**Bug vs Flaw:**

| | Bug (Implementation Error) | Flaw (Design Error) |
|---|---|---|
| **Origin** | Developer wrote wrong code | Architect designed wrong system |
| **Example** | SQL string concatenation | No authZ check on `/admin` endpoints |
| **Detection** | SAST, fuzzing, code review | Threat modeling, design review, pentest |
| **Fix** | Rewrite the function | Redesign the architecture |
| **Cost multiplier** | 1× (code change) | 10×–100× (redesign, re-implement, re-test) |

Both are vulnerabilities, but flaws are far more expensive. Threat modeling catches flaws before implementation.

**WAF bypass techniques (why WAF is defense-in-depth, not a silver bullet):**
- HTTP parameter pollution: `?id=1&id=' OR 1=1-- ` — server picks one parameter, WAF sees the other.
- Encoding tricks: double URL encoding (`%2527` → `%27` → `'`), Unicode normalization.
- Request smuggling: Content-Length / Transfer-Encoding discrepancies at reverse proxy.
- JSON/XML bodies: some WAFs only inspect URL-encoded form data.
- Comment injection: `'/**/OR/**/1=1-- ` bypasses regex-based rules.
- **Conclusion:** Parameterized queries prevent SQLi regardless of WAF presence. WAF buys time, not safety.

### Resources

**SDLC security gate checklist:**

| Phase | Gate | Evidence | Owner |
|---|---|---|---|
| Requirements | Security requirements documented | Requirements doc | Product + AppSec |
| Design | Threat model attached | STRIDE/PASTA output | Engineering |
| Implementation | SAST clean (high/critical) | SARIF report | Engineering |
| Implementation | Secrets scan clean | Scan report | Engineering |
| Implementation | SCA: no known critical CVEs | SBOM + vuln report | Engineering |
| Verification | DAST clean (high/critical) | DAST report | QA + AppSec |
| Verification | Code review complete | PR approval | Engineering + AppSec |
| Release | SLSA provenance + signed image | Attestation + signature | DevOps |
| Post-release | Incident response plan current | IR doc | SecOps + Engineering |

**STRIDE-per-element mapping table (see Execution section above for full matrix).**

**Secure code review checklist (see Execution section above).**

**API security testing checklist:**
1. [ ] BOLA: attempt to access another user's object by ID
2. [ ] BFLA: attempt admin endpoints with user token
3. [ ] Rate limiting: brute-force login, enumeration, expensive queries
4. [ ] JWT validation: `alg: none`, algorithm confusion, expired token, wrong audience
5. [ ] Mass assignment: send extra fields (`role: "admin"`, `isAdmin: true`)
6. [ ] SSRF: inject internal URLs, metadata service IPs, loopback addresses
7. [ ] Schema validation: malformed JSON, excessive nesting, huge payloads
8. [ ] CORS: `Access-Control-Allow-Origin: *` with credentials

**SQLi defense priority stack:**
1. Parameterized queries (primary — eliminates the vulnerability class)
2. Input validation (defense-in-depth — allow-list over blocklist)
3. Least-privilege DB accounts (blast-radius reduction)
4. WAF (defense-in-depth — buys time, not safety)
5. TLS to DB (prevents wire-tap of valid traffic)
6. Query monitoring / anomaly detection (detects exploitation in progress)

**OWASP ASVS → CISSP Domain mapping:**

| ASVS Chapter | CISSP Domain | Primary Control |
|---|---|---|
| V1 Architecture | D8 Software Dev | Secure design |
| V2 Authentication | D5 IAM | AuthN hygiene |
| V3 Session Management | D5 IAM | Token lifecycle |
| V4 Access Control | D5 IAM | RBAC/ABAC |
| V5 Validation/Sanitization | D8 Software Dev | Rules 1–3 (input, output, param) |
| V6 Stored Cryptography | D3 Engineering | Key mgmt, algorithms |
| V7 Error Handling/Logging | D8 Software Dev | Rules 4–5 (errors, logs) |
| V8 Data Protection | D2 Asset | At rest/in transit |
| V9 Communication | D4 Network | TLS, cert pinning |
| V10 Malicious Code | D8 Software Dev | Supply chain, anti-malware |
| V11 Business Logic | D8 Software Dev | Abuse cases, workflow |
| V12 Files & Resources | D8 Software Dev | Path traversal, uploads |
| V13 API & Web Service | D8 Software Dev + D5 | CORS, rate limit, JWT |
| V14 Configuration | D7 Operations | Hardening, dependencies |

**Dependency risk assessment template:**
| Factor | Score (1–5) | Notes |
|---|---|---|
| Maintainer count & activity | | Single maintainer = high risk |
| CVE history & patch speed | | Recent unpatched CVEs = high risk |
| Bus factor (contributors) | | 1–2 core contributors = high risk |
| Upstream dependencies | | Nested supply-chain risk |
| License compatibility | | GPL in proprietary = blocker |
 | Build provenance (SLSA) | | Unsigned tarball = SLSA 0 |

### ⚠️ CONFLICT — OWASP Top 10 Version

**Original references OWASP Top 10 for Web (2021).** The current version as of 2025 is **OWASP Top 10:2025**. The key change: A10 was "SSRF (Server-Side Request Forgery)" in the 2021 edition; it is now **"Mishandling of Exceptional Conditions"** in 2025. The remaining 9 categories are largely stable, but the SSRF entry has been retired from the Top 10 and moved to a secondary list.

| Edition | A10 Category | Status |
|---|---|---|
| OWASP Top 10:2021 | A10 — SSRF | **Superseded** |
| OWASP Top 10:2025 | A10 — Mishandling of Exceptional Conditions | **Current** |

The sibling note [[secure-coding-practices-owasp]] also references the 2021 edition. Update both when refreshing for the 2025+ exam cycle. The primary structural defense against A10 (2025) — don't leak sensitive data in error messages, fail securely — is already covered by Rules 4 (error handling) and 5 (safe logging) in the secure-coding practices note.

<!-- /EXPANDED -->

## CISO / Risk Manager View

**Board framing.** Domain 8 answers the question *"How do we make sure the software we ship is not the next breach headline?"* Every major breach of the last decade (SolarWinds, 3CX, MOVEit, Log4Shell, XZ Utils) had a software-development root cause. The board does not need to understand STRIDE; the board needs to understand that **the AppSec program is the firewall around the codebase**, and that firewall is measured in defect-escape rate, mean-time-to-remediate, and SBOM coverage.

**Strategic priorities (12-month plan).**

1. **Adopt SSDF as the spine.** US Executive Order 14028 makes SSDF effectively mandatory for federal suppliers; map current practices to PO/PS/PW/RV and report the gap.
2. **Move from annual pentest to continuous verification.** SAST + DAST + SCA in CI, RASP in production, monthly attack-surface review.
3. **Publish a Software Bill of Materials (SBOM).** CycloneDX or SPDX, generated automatically per build, retained for the artifact's life.
4. **Reach SLSA Level 2+ on critical-path builds.** Provenance attestation, signed images, hermetic builds. Costs $\sim 5\%$ of build pipeline budget at Level 2.
5. **Mandate threat models for new features.** STRIDE or PASTA, attached to the design doc, reviewed by AppSec before code merge.
6. **Operationalize supply-chain risk.** Typosquatting, dependency confusion, malicious maintainers — every dependency is a potential initial-access vector. See [[software-supply-chain-attacks-slsa]].

**Metrics the board cares about.**

| Metric | Target (Year 1) | Why |
|---|---|---|
| High/critical findings in prod per release | $\leq 2$ | Direct risk indicator |
| % of code covered by SAST | 100% of trunk | Coverage, not just presence |
| Mean time to remediate critical | $\leq 15$ days | SLA for risk burn-down |
| SBOM coverage of production artifacts | 100% | Regulatory + supply-chain |
| SLSA level of critical builds | $\geq$ L2 | Provenance + integrity |
| Threat models per feature | 100% of new features | Design-time risk reduction |
| Developer secure-coding training | $\geq 90\%$ annually | Human firewall |

**The AppSec maturity conversation.** BSIMM/SAMM data shows that moving from Level 2 to Level 3 typically costs $2–5$M/year for a 1,000-engineer org but reduces incident-related loss by an order of magnitude. The ROI is realized in 18–24 months.

## Related Connections

### Layer 2 — Deep Dives
- [[sdlc-models-and-security-integration]] — waterfall, iterative, agile, DevSecOps, MS SDL, NIST SSDF
- [[secure-coding-practices-owasp]] — input validation, output encoding, parameterized queries, ASVS
- [[threat-modeling-stride-pasta]] — STRIDE categories, PASTA stages, DFDs, attack trees
- [[api-security-and-microservices]] — OWASP API Top 10, OAuth 2.0, JWT, service mesh

### Layer 3 — Edge Cases & Frameworks
- [[devsecops-and-cicd-security]] — pipeline gates, secrets, IaC scanning, SBOM, SLSA preview
- [[database-security-and-sql-injection]] — SQLi types, parameterized queries, ORM safety, NoSQLi
- [[software-supply-chain-attacks-slsa]] — SolarWinds, 3CX, XZ utils, dependency confusion, SLSA levels

### Cross-Domain
- [[../Domain-01-Security-and-Risk-Management/risk-management-framework]] — threat modeling consumes the threat register
- [[../Domain-02-Asset-Security/data-classification-and-handling]] — input validation strength is set by data tier
- [[../Domain-03-Security-Architecture-and-Engineering/Domain-03-Index]] — secure protocols and crypto primitives used in code
- [[../Domain-05-Identity-and-Access-Management/Domain-05-Index]] — OAuth, JWT, sessions all live in the application layer
- [[../Domain-06-Security-Assessment-and-Testing/Domain-06-Index]] — SAST/DAST/IAST/pen-test feed defects back into the SDLC
- [[../Domain-07-Security-Operations/Domain-07-Index]] — runtime signals (RASP, WAF, EDR) close the loop with dev

## References

- (ISC)² CISSP CBK — Domain 8: Software Development Security (2024)
- NIST SP 800-218 Rev. 1 — Secure Software Development Framework (SSDF) v1.1
- NIST SP 800-53 Rev. 5 — SA / SI families
- NIST IR 8397 — Guidelines on Minimum Standards for Developer Verification of Software
- OWASP SAMM v2.0; OWASP ASVS v4.0.3
- OWASP Top 10 for Web (2021) and API Security (2023)  <!-- ⚠️ CONFLICT: OWASP Top 10:2025 is current (A10 changed from SSRF → Mishandling of Exceptional Conditions); see EXPANDED section above -->
- Microsoft Security Development Lifecycle (SDL)
- SLSA v1.0 — Supply-chain Levels for Software Artifacts
- Executive Order 14028 — Improving the Nation's Cybersecurity (May 2021)
- NIST SP 800-161r2 — Cybersecurity Supply Chain Risk Management Practices
- BSIMM 13 — Building Security In Maturity Model
