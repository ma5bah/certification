---
title: DevSecOps and CI/CD Security
layer: 3
domain: 8
tags: [cissp, devsecops, cicd, secrets-management, iac, sbom, slsa, security-gates]
related_domains: [3, 6, 7]
parent: sdlc-models-and-security-integration
---

# DevSecOps and CI/CD Security

## Feynman Explanation

A factory that makes cars has inspectors at every station: incoming steel, the welding arm, the paint booth, final assembly. **DevSecOps puts security inspectors in the software factory — every commit triggers a battery of automated checks (secrets scan, dependency scan, static analysis, image scan, IaC scan) that must all pass before code can leave the building.** The cultural shift is "security is everyone's job, but the machine enforces it." The economic shift is that finding a defect at the weld station costs 1×; finding it at the dealership costs 100×.

## Technical Details

### The CI/CD Security Gate Inventory

A mature pipeline has **10–14 gates** organized into phases. The model below maps gates to NIST SSDF practice groups.

| Phase | Gate | Tools (Examples) | Blocks Release On |
|---|---|---|---|
| **Pre-commit** | Secret scan (local) | `gitleaks`, `trufflehog` | High-confidence secret |
| **PR open** | SCA (dependency vuln) | `snyk`, `dependabot`, `osv-scanner` | Known CVE with fix; license violation |
| | SAST | `codeql`, `semgrep`, `sonarqube` | High/critical finding |
| | Secret scan (CI) | `gitleaks`, `trufflehog` | Any secret in diff |
| | IaC scan | `checkov`, `tfsec`, `kics` | Misconfig with severity ≥ high |
| | Container scan | `trivy`, `grype`, `snyk-container` | Critical CVE in base image |
| | License compliance | `syft`, `scancode-toolkit` | Banned license (GPL in proprietary) |
| | SBOM generation | `cyclonedx-bom`, `syft` | SBOM must be produced |
| | Codeowner review | GitHub `CODEOWNERS` | Required reviewers must approve |
| **Build** | Reproducible build | `repro-build`, Bazel | Build hash mismatch |
| | SLSA provenance | `slsa-github-generator` | Provenance missing or invalid |
| **Release** | Image signing | `cosign` (Sigstore) | Image not signed |
| | Artifact attestation | `in-toto`, `sigstore` | Missing attestation |
| | DAST (staging) | `zap`, `burp enterprise` | High/critical finding |
| **Deploy** | Admission control | OPA/Gatekeeper, Kyverno | Policy violation |
| | Drift detection | Terraform Cloud, Spacelift | Unmanaged resource |

### Secrets Management

#### The Anti-Patterns

```python
# NEVER — hard-coded
api_key = "sk-prod-1a2b3c4d5e6f"

# NEVER — in env file checked into repo
DATABASE_URL=postgres://user:pass@db.prod

# NEVER — passed in URL/CLI arg (visible in process list)
psql "postgres://user:pass@db.prod"
```

Each of these is detected by automated secret scanners and lands in the **secret leak** OWASP category. The blast radius is "rotate this credential across every system that has ever seen it."

#### The Pattern — Centralized Vault + Short-Lived Tokens

```python
# Pattern: dynamic secrets from Vault
import hvac
client = hvac.Client(url="https://vault.prod:8200",
                     token=open("/var/run/secrets/vault.token").read())
db_creds = client.secrets.database.generate_credentials(
    name="app-db", ttl="15m"
)
conn = psycopg2.connect(
    host="db.prod",
    user=db_creds["username"],
    password=db_creds["password"],
    dbname="orders",
)
# credential expires in 15 min
```

| Tool | Use Case | Strengths |
|---|---|---|
| **HashiCorp Vault** | Dynamic secrets, PKI, transit encryption | Open-source, large ecosystem |
| **AWS Secrets Manager / SSM Parameter Store** | AWS-native apps | Tight IAM integration |
| **Azure Key Vault** | Azure-native apps | HSM-backed, managed HSM tier |
| **Google Secret Manager** | GCP-native apps | Per-version IAM |
| **Sealed Secrets (Bitnami)** | GitOps for K8s secrets | Encrypted-at-rest in git |

> **CISSP / exam tip.** "Never store secrets in source code" is the *rule*; "use a secrets manager with dynamic, short-lived credentials" is the *implementation*.

### Infrastructure-as-Code (IaC) Scanning

IaC scanning catches misconfigurations **before** the cloud resource is created. Two layers:

| Layer | Tools | Examples |
|---|---|---|
| **Static** (in CI, pre-apply) | `checkov`, `tfsec`, `kics`, `terrascan` | S3 bucket not encrypted, SG `0.0.0.0/0:22` |
| **Runtime** (drift / config) | AWS Config, Azure Policy, GCP SCC | S3 bucket public *after* deploy, encryption disabled |

```hcl
# Terraform — flagged by checkov: CKV_AWS_18  (S3 access logging)
resource "aws_s3_bucket" "logs" {
  bucket = "company-logs"
  # missing: logging { target_bucket = ... }
}

# Fix
resource "aws_s3_bucket" "logs" {
  bucket = "company-logs"
}
resource "aws_s3_bucket_logging" "logs" {
  bucket        = aws_s3_bucket.logs.id
  target_bucket = aws_s3_bucket.audit.id
  target_prefix = "log/"
}
```

### SBOM (Software Bill of Materials)

An SBOM is a machine-readable list of every component in a build. Two dominant formats:

| Format | Owner | Strengths |
|---|---|---|
| **SPDX** (ISO/IEC 5962:2024) | Linux Foundation | Standardized; license-focused |
| **CycloneDX** | OWASP | Lightweight; security-focused; supports VEX, SaaSBOM, ML-BOM |

```bash
# Generate CycloneDX SBOM for a Node app
$ npx @cyclonedx/cyclonedx-npm --output-format JSON --output-file sbom.json

# Generate SPDX SBOM for a container
$ syft packages docker:myapp:latest -o spdx-json > sbom.spdx.json
```

SBOM enables:
- **Vulnerability impact analysis** — when Log4Shell hit, NIST NVD + vendor advisories → grep SBOM → "are we affected?"
- **License compliance** — automatically detect GPL/AGPL in proprietary code
- **Procurement & supply-chain transparency** — required for US federal procurement (EO 14028)
- **VEX (Vulnerability Exploitability eXchange)** — vendor statement "we use this, but we don't load the vulnerable code path"

### SLSA (Supply-chain Levels for Software Artifacts) — Preview

Full framework detail in [[software-supply-chain-attacks-slsa]]. Quick reference:

| Level | Requirement | Implementation Cost |
|---|---|---|
| **SLSA 0** | (Baseline, no guarantees) | None |
| **SLSA 1** | Build provenance exists (unsigned) | Low |
| **SLSA 2** | Signed provenance, hosted build service | Medium |
| **SLSA 3** | Hardened build, two-party review, non-falsifiable provenance | High |

> **Practical target.** Most enterprises should reach SLSA 2 on critical-path builds (payment, auth, customer-data) within 12 months. SLSA 3 is required for highest-assurance systems.

### Supply-Chain Defenses in CI/CD

| Risk | Defense |
|---|---|
| **Compromised upstream package** | Pin versions; verify hashes; `npm install --ignore-scripts`; use lockfile-aware installs |
| **Typosquatting** (`requets` vs `requests`) | Adopt internal package mirror (e.g., Artifactory, Cloudsmith, Sonatype Nexus) with allow-list |
| **Dependency confusion** (private `internal-auth` overwritten by public `internal-auth`) | Scope-check on `npm config get registry`; `npm install` from internal registry only; cookie-based auth |
| **Build pipeline compromise** | SLSA L2+; ephemeral build runners; signed provenance |
| **Stolen CI secrets** | OIDC federation (no long-lived cloud creds in CI); short-lived tokens; environment-scoped secrets |

### The OIDC Federation Pattern (No Long-Lived Cloud Keys)

```yaml
# GitHub Actions — assume AWS role via OIDC, no static key
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123:role/github-actions-deploy
    aws-region: us-east-1
    # No access-key/secret-key needed
```

The CI system asserts its identity via OIDC to the cloud IAM, which issues a short-lived (≤ 1 hr) token. Stolen CI token = useless after 1 hour and only valid for the IAM role's allow-listed actions.

### Observability & Feedback Loops

DevSecOps is incomplete without feedback. Every gate produces an artifact that must flow back to engineering:

| Artifact | Sink | Use |
|---|---|---|
| SAST SARIF | Code-scanning dashboard | Per-PR annotation |
| SCA findings | Vulnerability mgmt (Snyk, Dependabot) | Auto-PR fix |
| SBOM | Artifact registry + VEX feed | Customer transparency |
| DAST findings | Issue tracker | Triage queue |
| Pipeline failures | ChatOps / incident mgmt | On-call rotation |
| Provenance | Sigstore Rekor / transparency log | Public verification |

### The "Paved Road" Concept

The most effective DevSecOps programs offer a **paved road** — self-service, secure-by-default templates that teams adopt because they are the path of least resistance:

- Hardened Terraform modules (VPC, EKS, RDS with logging + encryption on by default)
- Hardened container base images (distroless, minimal CVE surface)
- Service templates with OAuth, mTLS, observability pre-wired
- CI/CD templates with all security gates enabled

> **Why it works.** Teams adopt paved road because it is *faster* than building from scratch. The CISO's leverage is making the secure path also the easy path.

### Common DevSecOps Anti-Patterns

1. **Tooling without ownership** — SAST produces 10,000 findings, no triage, alerts ignored.
2. **Bypass-able gates** — "I can push with `--no-verify`" → not a real gate.
3. **Long-lived cloud creds in CI** — high blast-radius breach.
4. **Static secrets in repos** — the leak is in the git history even after deletion.
5. **No SBOM in production** — cannot answer "are we affected by CVE-2024-XXXX?" in time.
6. **Paved road exists but is not advertised** — teams reinvent insecure patterns.

## CISO / Risk Manager View

**Board framing.** DevSecOps is the **automation of the AppSec program**. The board does not need to know what SAST is; the board needs to know that **every release passes through 10+ automated security checks, that no commit can ship with a known critical vulnerability, and that we have a verified software inventory (SBOM) for every artifact in production.** DevSecOps converts security from a manual review function into a continuous, measurable, automated control — the only model that scales to daily deployments.

**Strategic priorities (12-month).**

1. **Adopt a "shift-left and shield-right" stance.** Shift-left = SCA/SAST/secret-scan on every PR. Shield-right = RASP / WAF / egress filtering in production. Both are needed.
2. **Reach SLSA L2 on critical builds.** $5\%$ build-pipeline budget; reduces supply-chain risk by an order of magnitude.
3. **Standardize on dynamic secrets.** Vault or cloud-native; eliminate static cloud creds in CI.
4. **Generate and retain SBOMs.** CycloneDX for new code; backfill for legacy.
5. **Build the paved road and measure adoption.** If $<80\%$ of new services use the templates, the paved road is failing.
6. **Close the loop with production.** Runtime signals (RASP, WAF, EDR) feed back into SAST rules so the same class of finding never recurs.

**Metrics the board cares about.**

- % of builds with all gates passing (target: $\geq 95\%$)
- Mean time to remediate pipeline finding (target: critical $\leq 7$ days)
- # of static cloud creds in CI (target: 0)
- % of production artifacts with SBOM (target: 100%)
- % of new services using paved-road templates (target: $\geq 80\%$)
- Mean age of unpatched high/critical CVE in production (target: $\leq 30$ days)

## Related Connections

### Parent L2
- [[sdlc-models-and-security-integration]] — DevSecOps is the modern SDLC overlay

### Sibling L3
- [[database-security-and-sql-injection]] — runtime SQLi defense (WAF) closes the loop with dev-time parameterized queries
- [[software-supply-chain-attacks-slsa]] — supply-chain defenses (provenance, SBOM, SLSA) live in CI

### Cross-Domain
- [[../Domain-03-Security-Architecture-and-Engineering/Domain-03-Index]] — PKI for code signing; HSM-backed key management
- [[../Domain-06-Security-Assessment-and-Testing/Domain-06-Index]] — DAST and IAST are gates in the pipeline
- [[../Domain-07-Security-Operations/Domain-07-Index]] — runtime signals feed back into the SDLC loop

## References

- NIST SP 800-218 Rev. 1 — Secure Software Development Framework (SSDF)
- NIST IR 8397 — Guidelines on Minimum Standards for Developer Verification of Software
- OWASP Top 10 CI/CD Security Risks (2023) — CWE-798 hard-coded creds, CWE-829 untrusted functionality, etc.
- SLSA v1.0 — Supply-chain Levels for Software Artifacts
- CycloneDX v1.6 / SPDX v2.3 specifications
- Sigstore — cosign, fulcio, rekor
- CIS Benchmarks — Docker, Kubernetes, AWS, Azure, GCP
- (ISC)² CISSP CBK — Domain 8: Software Development Security
- BSIMM 13 — DevSecOps practice maturity data
