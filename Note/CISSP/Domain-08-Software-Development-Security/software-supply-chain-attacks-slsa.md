---
title: Software Supply Chain Attacks and SLSA
layer: 3
domain: 8
tags: [cissp, supply-chain, slsa, sbom, solarwinds, xz-utils, dependency-confusion, typosquatting, sigstore]
related_domains: [1, 3, 6]
parent: devsecops-and-cicd-security
---

# Software Supply Chain Attacks and SLSA

## Feynman Explanation

A restaurant that buys flour from a single supplier is vulnerable if the supplier is poisoned. **Software supply-chain attacks are the same: a single compromised dependency — a library, a build tool, an update server — can inject malicious code into thousands of downstream products in one stroke.** SolarWinds (18,000 customers), 3CX, MOVEit, and the XZ Utils backdoor were all supply-chain compromises. SLSA (Supply-chain Levels for Software Artifacts) is the framework that rates a build pipeline's resistance to this class of attack, the way a hygiene rating rates a restaurant.

## Technical Details

### The Major Incidents — What Happened

#### SolarWinds Orion (2020)

| Field | Detail |
|---|---|
| Vector | Compromised build pipeline (Sunspot malware inserted into Orion build) |
| Payload | SUNBURST backdoor in Orion DLL |
| Blast radius | ~18,000 SolarWinds customers, ~100 US federal agencies, Fortune 500 |
| Detection lag | ~9 months (FireEye discovered it; Mandiant had been hit) |
| Root cause | Build server compromised; unsigned updates; weak build environment hygiene |
| SLSA level at time | SLSA 0 |

**Lesson:** The attacker did not break into 18,000 organizations. They broke into *one* build server. The victim's customers trusted the vendor's update channel — a single point of failure.

#### 3CX Desktop Client (2023)

| Field | Detail |
|---|---|
| Vector | Trojanized X_TRADER financial-trading app (legacy, never used) infected a 3CX developer |
| Payload | DLL side-loaded from a legitimate X_TRADER DLL; beaconed to attacker infrastructure |
| Blast radius | 3CX customers (~12M daily users) |
| Root cause | Developer's personal machine compromised; no separation of work vs. personal development |
| Lesson | Endpoint security on the developer is supply-chain security |

#### XZ Utils Backdoor (CVE-2024-3094, 2024)

| Field | Detail |
|---|---|
| Vector | A *maintainer* of `xz-utils` (Jia Tan) spent **2 years** gaining trust and committing to the project, then introduced a backdoor in `liblzma` |
| Payload | `ifunc` resolver that intercepts `sshd`'s OpenSSL `RSA_public_decrypt` — pre-auth RCE on millions of Linux servers |
| Detection | Andres Freund (Microsoft) noticed a 500ms lag in `sshd` logins |
| Blast radius | Caught just before being merged into major distros (Debian, Fedora, openSUSE, Kali) |
| Lesson | Even mature, well-reviewed open-source projects can be subverted over years of patient social engineering |

#### MOVEit Transfer (2023, CL0P)

| Field | Detail |
|---|---|
| Vector | Zero-day SQL injection in MOVEit Transfer (CVE-2023-34362) |
| Payload | Web shell → data exfiltration of customer files |
| Blast radius | 2,700+ organizations, 95M+ individuals |
| Root cause | SQLi in a file-transfer product used by HR/payroll teams — supply chain for *data*, not for *software* |
| Lesson | Vendor compromise = customer compromise. SBOM + incident response plan matter even when you trust your vendor |

### Attack Categories (Attack Tree)

```
Goal: Compromise downstream users via a trusted dependency
├── Compromise a popular open-source library
│   ├── Social-engineer the maintainer (XZ Utils)
│   ├── Add a new malicious maintainer
│   ├── Submit a malicious "typosquatted" package
│   ├── Hijack a maintainer's account
│   └── Buy/abandon a popular package (left-pad class)
├── Compromise a build / CI system
│   ├── SolarWinds-style: modify build to inject payload
│   ├── CircleCI / Travis token theft → repo access
│   └── GitHub Actions: malicious third-party action
├── Compromise a vendor with privileged access
│   ├── SaaS vendor → OAuth token to customer cloud
│   ├── File-transfer vendor (MOVEit) → customer data
│   └── Update server → malicious updates
├── Compromise the developer
│   ├── Personal laptop (3CX)
│   ├── Phish / session hijack
│   └── Insider recruitment
└── Compromise the package distribution channel
    ├── npm / PyPI / RubyGems account takeover
    ├── Dependency confusion (private name hijacked by public)
    └── Malicious update pushed from official channel
```

### Dependency Confusion

A subtle but devastating attack:

```
1. Company "Acme" has an internal package: acme-internal-auth (in private registry)
2. Acme's build runs: npm install acme-internal-auth
3. Attacker publishes a *public* package named acme-internal-auth on npm
4. npm client resolves to highest version → public package wins
5. Attacker's code runs in Acme's build
```

**Defenses:**

```ini
# .npmrc — pin scope to internal registry
@acme:registry=https://npm.acme.internal
# or — set registry, and add to package.json:
"acme-internal-auth": "https://npm.acme.internal/acme-internal-auth/-/acme-internal-auth-1.2.3.tgz"
```

```python
# pip.conf — scoped index
[install]
index-url = https://pypi.acme.internal/simple/
# Critical: never set both an internal index AND allow pypi.org fallback
```

**Best practice:** Run an internal package proxy (Sonatype Nexus, JFrog Artifactory, Cloudsmith) that mirrors approved public packages and serves private packages — all build traffic goes through it.

### Typosquatting

Adversaries register packages with names similar to popular ones, hoping for a typo or autocomplete:

| Real package | Typosquatted variants |
|---|---|
| `requests` | `requets`, `request`, `reqests`, `reqeusts` |
| `python-dateutil` | `python_dateutil`, `python-dateutil2` |
| `boto3` | `boto`, `botto`, `boto4` |
| `lodash` | `lodesh`, `loadash` |

**Defenses:**
- Internal package proxy that blocks unapproved public packages
- Mandatory lockfiles (`package-lock.json`, `Pipfile.lock`, `go.sum`) — installs from hashes, not names
- SCA tools flag typosquats and unknown packages
- `npm install --ignore-scripts` to prevent post-install payloads

### SLSA (Supply-chain Levels for Software Artifacts) v1.0

SLSA defines **four levels**, each with progressively stronger guarantees about how an artifact was built.

| Level | Guarantees | Practical Meaning |
|---|---|---|
| **SLSA 0** | None | No provenance, no integrity |
| **SLSA 1** | Provenance exists, but is **unsigned**; build process documented | "We can show you how it was built, but you can't verify the builder was us" |
| **SLSA 2** | Provenance is **signed** by a hosted build platform; build is hardened against runtime tampering | "Signed by GitHub Actions / GitLab / GCP Cloud Build — verifiable provenance" |
| **SLSA 3** | Provenance is **non-falsifiable** (two-party review, hardened build platform); signing key is in a HSM; source verified | "Even the build platform cannot forge a provenance for a build that did not happen" |

### SLSA Build Track Requirements (Detailed)

| Requirement | L1 | L2 | L3 |
|---|---|---|---|
| Provenance generated | ✓ | ✓ | ✓ |
| Provenance signed | | ✓ | ✓ |
| Provenance non-falsifiable | | | ✓ |
| Build runs on hosted build platform | | ✓ | ✓ |
| Build platform hardened | | | ✓ |
| Source verified (VCS) | ✓ | ✓ | ✓ |
| Provenance includes all build steps | | | ✓ |
| Provenance available to consumer | | ✓ | ✓ |

```yaml
# GitHub Actions — SLSA L3 provenance (using slsa-github-generator)
- uses: slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml@v1.9.0
  with:
    image: myapp
    registry: ghcr.io/myorg
    digest: ${{ steps.build.outputs.digest }}
    # Generates in-toto provenance, signed by Sigstore Fulcio
```

### SBOM and Sigstore — The Two Pillars of Modern Defense

| Pillar | Tool | Purpose |
|---|---|---|
| **SBOM** (Software Bill of Materials) | CycloneDX, SPDX | "What is in this artifact?" — every component, every version |
| **Sigstore** (cosign + Fulcio + Rekor) | cosign, fulcio, rekor | "Who built it, and can I verify that?" — signing + transparency log |

```bash
# Sign an image with cosign
$ cosign sign --yes ghcr.io/myorg/myapp@sha256:abc123...

# Verify an image signature
$ cosign verify ghcr.io/myorg/myapp@sha256:abc123... \
    --certificate-identity-regexp 'https://github.com/myorg/myapp' \
    --certificate-oidc-issuer 'https://token.actions.githubusercontent.com'

# Generate SBOM in CI
$ syft packages ghcr.io/myorg/myapp:latest -o cyclonedx-json > sbom.json
# Attest SBOM to image
$ cosign attest --yes --predicate sbom.json --type cyclonedx \
    ghcr.io/myorg/myapp@sha256:abc123...
```

### In-Toto Attestation (Beyond SLSA)

In-toto is a framework for **verifiable claims** about software artifacts. Attestations can include:

| Attestation | Answers |
|---|---|
| SLSA provenance | "How was this built?" |
| SBOM (CycloneDX / SPDX) | "What is in it?" |
| VEX (Vulnerability Exploitability eXchange) | "Is this CVE actually exploitable in our build?" |
| Code review (Pritunl-style) | "Who reviewed this change?" |
| Test results | "Did it pass our test suite?" |

### Defense-in-Depth — The 8-Layer Supply-Chain Model

| Layer | Defense |
|---|---|
| 1. **Source** | Branch protection, required reviewers, signed commits |
| 2. **Dependencies** | SCA, lockfiles, internal proxy, allow-list, typosquat detection |
| 3. **Build** | SLSA L2+, ephemeral runners, no internet access from build |
| 4. **Artifacts** | Signed images, hash-pinned deploys, content-addressable storage |
| 5. **Distribution** | TLS-only, integrity-verified package mirrors |
| 6. **Deploy** | Admission control (OPA, Kyverno), signed manifests |
| 7. **Runtime** | RASP, allow-listed syscalls, eBPF, behavior monitoring |
| 8. **Response** | SBOM-driven CVE matching, VEX, automated rollback |

### Software Composition Analysis (SCA) Tools

| Tool | Strengths |
|---|---|
| **Snyk Open Source** | Largest vuln DB; auto-PR fixes |
| **GitHub Dependabot** | Native to GitHub; auto-PR |
| **GitLab Dependency Scanning** | Native to GitLab; container scanning included |
| **OWASP Dependency-Check** | Free; multiple DBs (NVD, OSS Index) |
| **Snyk / Mend (formerly WhiteSource)** | License + vuln + reachability analysis |
| **Endor Labs / Anchore** | Reachability analysis — does my code actually call the vulnerable function? |
| **OSV-Scanner** | Google; OSV.dev DB; fast |

> **Reachability is the new SCA frontier.** Knowing *you have `log4j`* is step 1; knowing *your code calls the vulnerable `JndiLookup.lookup()` method* is what matters. Tools like Endor Labs, Anchore, and Snyk Reachability reduce false positives by 5–10×.

### Common Anti-Patterns

1. **"We use `npm install` against public registry"** — direct exposure to typosquats, dependency confusion, and malicious updates.
2. **No lockfile** — `package-lock.json` / `Pipfile.lock` / `go.sum` are missing; every install resolves freshly.
3. **Long-lived tokens in CI** — stolen CI token = push to your repo.
4. **SBOM generated, never used** — an SBOM in a PDF is shelfware. Operationalize: pipe to vuln-mgmt and customer transparency feeds.
5. **Trusting unsigned packages** — no Sigstore / no cosign verification at deploy.
6. **No separation of build and runtime identities** — same cloud creds used in dev, build, and prod.

## CISO / Risk Manager View

**Board framing.** Supply-chain risk is **the dominant software-security threat of the decade**. SolarWinds, MOVEit, and XZ Utils are not anomalies — they are the leading edge. The board does not need to know what SLSA L3 is; the board needs to know **what percentage of our production builds are signed, what percentage have an SBOM, and what our incident response plan is when a major upstream dependency is compromised**. Boards increasingly ask these questions of every CIO and CISO.

**Strategic priorities (12-month).**

1. **Adopt SLSA L2 as the floor for all production builds.** $\sim 5\%$ build-pipeline cost; non-negotiable for high-risk systems.
2. **Generate and retain SBOMs for every release.** CycloneDX or SPDX; auto-generated in CI; retained for the artifact's lifetime.
3. **Pin and verify all dependencies.** Lockfiles mandatory in CI; internal proxy as the only path to public packages.
4. **Sign all production artifacts.** cosign / Sigstore; reject unsigned at the deployment gate.
5. **Subscribe to vuln intel and VEX.** When Log4Shell hit, organizations with SBOM answered "are we affected?" in minutes; those without took days.
6. **Tabletop exercise: major dependency compromised.** What is the blast radius? How do you patch 500 services in 24 hours? Practice annually.
7. **Mandate MFA + hardware keys for all maintainers of internal critical packages.** XZ Utils was a 2-year social engineering campaign against a single person.

**Metrics the board cares about.**

- % of production builds with SLSA ≥ L2 (target: 100% of critical path)
- % of production artifacts with SBOM (target: 100%)
- % of production artifacts with signed provenance (target: 100%)
- Mean time to identify "are we affected?" when a major CVE drops (target: $\leq 4$ hours)
- # of public-package downloads bypassing internal proxy (target: 0)
- Tabletop exercise on supply-chain compromise (target: $\geq 1$/year)

### The Bottom Line for Boards

The supply chain is a **chain of trust**. Every link — source, dependency, build, artifact, distribution, deploy, runtime — must be verified. The cost of doing this right is $\sim 5–10\%$ of build-pipeline engineering effort. The cost of getting it wrong is the next SolarWinds.

## Related Connections

### Parent L2 (via L3)
- [[devsecops-and-cicd-security]] — supply-chain defenses are implemented as pipeline gates

### Parent L1
- [[domain-08-software-development-security]] — D8 hub — supply chain is cross-cutting

### Sibling L2 / L3
- [[sdlc-models-and-security-integration]] — SSDF PS group is supply-chain focused
- [[secure-coding-practices-owasp]] — the code itself must be trustworthy
- [[api-security-and-microservices]] — API #10 (unsafe consumption of APIs) is supply-chain
- [[database-security-and-sql-injection]] — ORM package is a supply-chain dependency

### Cross-Domain
- [[../Domain-01-Security-and-Risk-Management/risk-management-framework]] — supply-chain risk enters the risk register
- [[../Domain-03-Security-Architecture-and-Engineering/Domain-03-Index]] — HSM-backed signing keys, PKI for code signing
- [[../Domain-06-Security-Assessment-and-Testing/Domain-06-Index]] — SCA + pentest validate supply-chain defenses

## References

- OpenSSF — SLSA v1.0 Specification (slsa.dev)
- NIST SP 800-218 Rev. 1 — SSDF v1.1
- NIST SP 800-161r2 — Cybersecurity Supply Chain Risk Management Practices
- NIST IR 8397 — Guidelines on Minimum Standards for Developer Verification of Software
- Executive Order 14028 — Improving the Nation's Cybersecurity
- OMB M-22-18 / M-24-04 — Federal software procurement attestation
- CISA — Secure Software Development Attestation Form
- OWASP — Top 10 CI/CD Security Risks (2023)
- CycloneDX v1.6 specification
- SPDX v2.3 specification (ISO/IEC 5962:2024)
- Sigstore — cosign, Fulcio, Rekor documentation
- In-toto attestation framework
- ENISA — Threat Landscape for Supply Chain Attacks (2023)
- (ISC)² CISSP CBK — Domain 8: Software Development Security
- CWE-1357 (Reliance on Unverified Software Component), CWE-829 (Inclusion of Functionality from Untrusted Control Sphere)
- SolarWinds / SUNBURST — CISA Advisory AA20-352A
- XZ Utils Backdoor — CVE-2024-3094
- MOVEit — CVE-2023-34362, CISA Advisory AA23-158A
