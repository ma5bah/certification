---
title: OWASP Top 10 and Web Application Testing
layer: 3
domain: 6
tags: [cissp, domain-6, owasp, top-10, wstg, asvs, web-security, sast, dast, sca]
related_domains: [2, 3, 4, 7, 8]
parent: domain-06-security-assessment-and-testing
---

# OWASP Top 10 and Web Application Testing

## Feynman Explanation

OWASP is the industry consensus on what *actually* breaks web applications. The **Top 10** is a regularly-updated list of the most common, highest-impact categories of web flaws — not a checklist, but a *vocabulary* you must speak to design, test, and audit web apps. The **ASVS** is the deeper checklist with hundreds of testable requirements. The **Testing Guide (WSTG)** is the *how* — for each category, what to test and how. If you build or test web apps, these are the documents that anchor your work and the documents auditors will ask for.

## Technical Details

### OWASP Top 10:2021 (the canonical list)

| # | Category | What it is (one-line) | Common root cause | D8 mitigation |
|---|---|---|---|---|
| **A01:2021** | **Broken Access Control** | Users can act on data/functionality they shouldn't | Missing / misapplied authorization checks | RBAC/ABAC, deny-by-default, server-side enforcement |
| **A02:2021** | **Cryptographic Failures** | Weak / missing / misapplied crypto | HTTP, weak TLS, weak hashes, hardcoded keys | TLS 1.2+, AEAD (AES-GCM, ChaCha20), proper key mgmt |
| **A03:2021** | **Injection** | Untrusted input executed as code/query/command | String concatenation, no parameterization, no validation | Parameterized queries, ORMs, allow-list input validation, output encoding |
| **A04:2021** | **Insecure Design** | Architectural flaw (can't be fixed by code) | Missing threat model, missing security requirements, "we'll secure later" | Threat modeling, secure design patterns, security requirements in user stories |
| **A05:2021** | **Security Misconfiguration** | Default creds, open admin, debug enabled, missing headers | Hardening checklist not followed, IaC drift | Hardened baselines, IaC scanning, security headers (CSP, HSTS, X-Frame-Options) |
| **A06:2021** | **Vulnerable & Outdated Components** | Known-vuln libs / runtimes | No SCA, no SBOM, no patch process | SBOM, SCA in CI, automated dependency updates (Dependabot, Renovate) |
| **A07:2021** | **Identification & Authentication Failures** | Weak / broken auth | Weak passwords, no MFA, predictable sessions, credential stuffing | MFA everywhere, password managers, strong session handling, account lockout with throttling |
| **A08:2021** | **Software & Data Integrity Failures** | Untrusted update / pipeline / deserialization | Insecure deserialization, unsigned updates, CI/CD compromise | Signed artifacts, SLSA framework, integrity checks, deserialization guards |
| **A09:2021** | **Security Logging & Monitoring Failures** | Attacks not detected / not responded to | Logs missing or unactionable, no alerting | Audit-grade logging, SIEM use cases, alerting, MTTD/MTTR — see [[logging-monitoring-and-siem]] |
| **A10:2021** | **Server-Side Request Forgery (SSRF)** | Server fetches attacker-controlled URL | Insufficient URL validation, internal services reachable | Network segmentation, allow-list egress, IMDSv2 on cloud |

> **A01 (Broken Access Control) has been the #1 web flaw in every Top 10 since 2017.** Memorize the patterns: IDOR (Insecure Direct Object Reference), missing function-level access control, CORS misconfiguration, force browsing, parameter tampering.

### Testing techniques matrix

| Technique | When | What it finds | Example tools | Layer |
|---|---|---|---|---|
| **SAST** (static) | At commit / in CI | Source-code flaws, hardcoded secrets, dangerous APIs | Semgrep, SonarQube, Checkmarx, CodeQL | White-box |
| **DAST** (dynamic) | Against running app | Runtime vulns, misconfigs, injection, XSS | Burp Suite Pro, OWASP ZAP, Invicti (Netsparker) | Black-box |
| **IAST** (interactive) | In app via agent | Combines SAST+DAST with runtime context | Contrast Security, Seeker | Grey-box |
| **SCA** (composition) | At build | Known-vuln 3rd-party libs | Snyk, Dependabot, OWASP Dependency-Check, JFrog Xray | Component |
| **RASP** (runtime app self-protection) | In prod | Detect/block attacks from inside app | Sqreen (now Datadog), Imperva RASP | In-app |
| **Manual pen test** | Pre-release / annual | Logic, business-flow, chained vulns | Human + Burp Pro / custom | All |

> **CISSP trap:** SAST and DAST find *different* bug classes. You need both. SAST finds unsafe deserialization in code; DAST finds the XSS that only manifests in a specific parameter.

### OWASP ASVS (Application Security Verification Standard) v4.0

ASVS is a deeper, verifiable checklist organized in 14 chapters (V1–V14). It defines **three verification levels**:

| Level | Target | Examples |
|---|---|---|
| **Level 1** | All apps | Opportunistic checks; automated |
| **Level 2** | Apps handling sensitive data | Standard: required for SOC 2 / ISO 27001 web apps |
| **Level 3** | Critical apps (financial, health, military) | Advanced: threat-modeled, defense-in-depth |

ASVS chapters (v4.0):

| # | Chapter |
|---|---|
| V1 | Architecture, Design and Threat Modeling |
| V2 | Authentication |
| V3 | Session Management |
| V4 | Access Control |
| V5 | Validation, Sanitization and Encoding |
| V6 | Stored Cryptography |
| V7 | Error Handling and Logging |
| V8 | Data Protection |
| V9 | Communication |
| V10 | Malicious Code |
| V11 | Business Logic |
| V12 | Files and Resources |
| V13 | API and Web Service |
| V14 | Configuration |

> ASVS is the de facto standard for "show me you tested the app" — auditors and procurement teams increasingly demand a current ASVS Level 2/3 attestation.

### OWASP Testing Guide (WSTG) v4.2 — the "how"

WSTG is the playbook. Each section follows the same pattern:

```
[Technique name]
  Summary
  Test objectives
  How to test
    Black-box / Grey-box / White-box
  Tools
  References
  Remediation
```

Example sections (illustrative, not exhaustive):

| WSTG ID | Topic | Maps to Top 10 |
|---|---|---|
| WSTG-INPV-05 | Testing for SQL Injection | A03 |
| WSTG-INPV-02 | Testing for Stored XSS | A03 |
| WSTG-ATHZ-01 | Testing Directory Traversal | A01 |
| WSTG-CRYP-04 | Testing for Weak Encryption | A02 |
| WSTG-CONF-05 | Testing for Missing Security Headers | A05 |
| WSTG-SESS-04 | Testing for Session Fixation | A07 |
| WSTG-BUSL-02 | Testing for Business Logic | A04 |
| WSTG-INPV-19 | Testing for Server-Side Request Forgery | A10 |

### Security headers (the A05 hardener's checklist)

| Header | Value (example) | What it does |
|---|---|---|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains; preload` | Force HTTPS |
| `Content-Security-Policy` | `default-src 'self'; script-src 'self' 'nonce-...'` | Mitigate XSS, clickjacking |
| `X-Content-Type-Options` | `nosniff` | Prevent MIME sniffing |
| `X-Frame-Options` | `DENY` (or `SAMEORIGIN`) | Clickjacking (legacy; CSP `frame-ancestors` preferred) |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Limit referrer leakage |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=()` | Limit browser features |
| `Cache-Control` | `no-store` | For authenticated pages |

### Other OWASP projects worth knowing

| Project | Purpose |
|---|---|
| **OWASP Top 10** | Awareness document — common categories |
| **OWASP ASVS** | Verifiable requirements — testing checklist |
| **OWASP WSTG** | Testing methodology — how to test |
| **OWASP MASVS / MASTG** | Mobile app security |
| **OWASP SAMM v2** | Software assurance maturity model — process |
| **OWASP ModSecurity CRS** | Open-source WAF ruleset |
| **OWASP Juice Shop** | Vulnerable training app |
| **OWASP Web Security Testing Guide (WSTG)** | This guide |
| **OWASP API Security Top 10** | API-specific |
| **OWASP CycloneDX** | SBOM standard |
| **OWASP Cheat Sheet Series** | Quick-reference for devs |

### How the OWASP stack fits in Domain 6

| Layer | Tool | Cadence | Owner |
|---|---|---|---|
| Design (D8 SDL) | SAMM, threat modeling | Per feature | Architect |
| Code (D8) | SAST, secrets scanning, pre-commit hooks | Per commit | Dev |
| Build (D8) | SCA, SBOM, container scan | Per build | DevOps |
| Test (D6 / D8) | DAST, manual pen test, ASVS | Pre-release | AppSec |
| Deploy (D6) | WAF rules from ModSecurity CRS | Per release | NetSec |
| Operate (D7) | RASP, WAF, bot mgmt | 24/7 | SOC |
| Audit (D6) | ASVS Level 2/3 attestation | Annual | Audit / customer |

> Web app testing is the **only** testing type that touches every layer of the org: design, dev, build, deploy, ops, audit. CISSP expects the candidate to know that *no one* tool catches all classes — defense in depth applies to the testing pipeline too.

## CISO / Risk Manager View

**Web applications are the dominant breach vector in every Verizon DBIR 2020–2024.** The strategic CISO positions OWASP as the *common language* that lets engineering, security, audit, and procurement all talk about the same risks. The two artifacts the CISO should demand:

1. **A current ASVS Level 2/3 attestation** for any customer-facing application, renewable annually. The attestation is the *evidence* that the app was actually tested, not just scanned.
2. **A published application security policy** that maps every OWASP Top 10 category to a named control and an owner. This is what auditors and regulators want to see.

**The Top 10 is awareness, not testing.** CISOs who treat the Top 10 as the *test plan* are testing too little. ASVS Level 2 has ~150 requirements — Top 10 is the elevator pitch. The CISO's message: "We test against ASVS Level 2, not Top 10, and we attest annually."

**Budget framing:**
- SAST (commercial): $30K–$300K/year
- DAST (commercial): $20K–$150K/year
- SCA (commercial): $20K–$100K/year, often per-developer seat
- WAF (commercial): $20K–$200K/year
- External web pen test: $15K–$80K per app
- RASP: typically bundled with EDR or WAF

These are *per-application* recurring costs — they scale with portfolio size.

**Procurement angle:** the CISO's security review of new SaaS vendors should *require* an OWASP ASVS Level 2 attestation or an independent pen test report. This is the single most leveraged security contract clause a CISO can write.

## Related Connections

### Parent
- [[domain-06-security-assessment-and-testing]] (D6 hub)

### L2 siblings (testing context)
- [[penetration-testing-methodology]] (WSTG is the pen-tester's playbook)
- [[vulnerability-management-lifecycle]] (SCA findings flow to vuln queue)
- [[security-audits-and-compliance-testing]] (ASVS = SOC 2 web-app evidence)
- [[logging-monitoring-and-siem]] (A09 — security logging is a Top 10 category)

### Cross-domain
- [[Domain 2 — Asset Security]] (data classification drives which ASVS level applies)
- [[Domain 3 — Security Architecture and Engineering]] (secure design patterns, threat modeling)
- [[Domain 4 — Communication and Network Security]] (WAF, TLS, network controls)
- [[Domain 7 — Security Operations]] (WAF/SIEM rules operationalize Top 10)
- [[Domain 8 — Software Development Security]] (SAST/SCA/DAST live in SDLC)

## References

- OWASP Top 10:2021 (owasp.org/Top10)
- OWASP ASVS 4.0.3 (Application Security Verification Standard)
- OWASP Web Security Testing Guide (WSTG) v4.2
- OWASP Mobile Application Security Verification Standard (MASVS) v2
- OWASP Mobile Security Testing Guide (MASTG) v1
- OWASP Software Assurance Maturity Model (SAMM) v2
- OWASP API Security Top 10:2023
- OWASP Cheat Sheet Series (cheatsheetseries.owasp.org)
- OWASP ModSecurity Core Rule Set (CRS) v4
- OWASP Dependency-Check / Dependency-Track (SCA tools)
- OWASP Juice Shop (training)
- OWASP Amass, ZAP, CycloneDX (open-source projects)
- NIST SP 800-53A — application security control assessment
- PCI DSS v4.0 §6 (Develop and maintain secure systems and software)
- CWE — Common Weakness Enumeration (MITRE) — categorization
- CVE / NVD — vulnerability catalog
- CVE → CWE → OWASP Top 10 mapping (CWE catalog)
- Verizon *DBIR 2024* — web applications as #1 breach vector
- Synopsys *BSIMM* (Building Security In Maturity Model) — peer-benchmark
- CSA STAR (Cloud Security Alliance) — SaaS security attestation
