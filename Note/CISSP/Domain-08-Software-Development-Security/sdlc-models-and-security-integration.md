---
title: SDLC Models and Security Integration
layer: 2
domain: 8
tags: [cissp, sdlc, devsecops, ssf, ms-sdl, waterfall, agile, secure-by-design]
related_domains: [1, 6, 7]
parent: domain-08-software-development-security
---

# SDLC Models and Security Integration

## Feynman Explanation

Building software is like building a bridge. You can hand the architect a sketch and a deadline and say "no inspections until the paint is dry" (waterfall) — cheap to start, ruinous to fix. Or you can build it lane by lane, inspecting each segment before pouring the next (iterative / agile). **The SDLC you choose determines where your security inspectors stand, how often they show up, and what they can do when they find a crack.** NIST SSDF and MS SDL are not new SDLCs — they are *security overlays* that say which inspector must sign off at which stage of any model you pick.

## Technical Details

### The Five Common SDLC Models

| Model | Cadence | Security Strength | Security Weakness | Best Fit |
|---|---|---|---|---|
| **Waterfall** | One big drop | Heavy up-front requirements, formal gates | Late discovery; security review at end | Regulated hardware/medical/aerospace |
| **Iterative / Incremental** | 3–6 cycles | Security review per iteration | May treat security as a *per-iteration* bolt-on | Mature monoliths, ERP rollouts |
| **Agile (Scrum/Kanban)** | 1–4 week sprints | Continuous; "Definition of Done" includes security | Pressure to skip security for velocity | SaaS, web, mobile apps |
| **DevSecOps** | Continuous delivery | Security as code; gates in CI/CD; "paved road" | Toolchain sprawl, alert fatigue | Cloud-native, microservices |
| **Spiral** | Risk-driven cycles | Explicit risk analysis at every loop | Heavy process; analyst-dependent | High-assurance, defense, medical devices |

### NIST SSDF (SP 800-218) — The Modern Spine

SSDF defines **four practice groups** and 19 practices. They are *model-agnostic* — overlay them on any of the five above.

| Group | Practices | Stage |
|---|---|---|
| **PO — Prepare the Organization** | Define security requirements, roles, tools, training | Org-wide, before the project |
| **PS — Protect the Software** | SBOM, provenance, signing, hardening of build env | During design + build |
| **PW — Produce Well-Secured Software** | Secure coding standards, reviews, SAST, threat modeling | During code + test |
| **RV — Respond to Vulnerabilities** | Disclosure policy, CVE handling, patch SLAs | Post-release, continuous |

> **Why SSDF dominates US policy.** Executive Order 14028 (May 2021) directs NIST to define "critical software" standards and obliges federal suppliers to attest to SSDF practice use. The 2024 OMB memo M-24-04 makes this attestation mandatory for federal software procurement.

### Microsoft Security Development Lifecycle (SDL) — 17 Practices in 7 Phases

| Phase | Security Activities |
|---|---|
| **1. Training** | Annual secure-coding training; role-based (crypto, web, cloud) |
| **2. Requirements** | Security requirements, quality gates, bug bars |
| **3. Design** | Threat modeling (STRIDE), attack surface analysis |
| **4. Implementation** | Approved tools, banned APIs, static analysis |
| **5. Verification** | DAST, fuzzing, third-party assessment |
| **6. Release** | Final security review (FSR), incident response plan |
| **7. Response** | Post-release monitoring, MSRC response, push patches |

> **Modern usage.** Microsoft itself no longer enforces SDL as a linear process; it is a checklist overlaid on Azure DevOps / GitHub pipelines. The 17 practices survive as gate requirements.

### OWASP SAMM — Five Business Functions

| Function | Maturity Question |
|---|---|
| **Governance** | Can leadership set and enforce security strategy? |
| **Design** | Is security designed in, not bolted on? |
| **Implementation** | Do we build secure code by default? |
| **Verification** | Do we prove it works before release? |
| **Operations** | Do we detect and respond in production? |

Each function is rated Levels 1–3. The total score is the program's SAMM level.

### Where Security Gates Fit (DevSecOps View)

A modern CI/CD pipeline typically contains **8–12 security gates**. Each gate is an automated check that blocks a release on failure:

```yaml
# .github/workflows/build.yml — illustrative security gate sequence
name: build-with-gates
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: SCA — dependency vulnerabilities
        uses: snyk/actions/node@master
        with:
          args: --severity-threshold=high
      - name: SAST — static analysis
        uses: github/codeql-action/analyze@v3
        with:
          languages: javascript,python
      - name: Secret scan
        uses: trufflesecurity/trufflehog@main
      - name: IaC scan (Terraform/CloudFormation)
        uses: bridgecrewio/checkov-action@master
      - name: Container image scan
        uses: aquasecurity/trivy-action@master
      - name: SBOM generation (CycloneDX)
        run: cyclonedx-bom -o sbom.json
      - name: SLSA provenance attestation
        uses: slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml@v1.9.0
      - name: Sign image (cosign / Sigstore)
        run: cosign sign --yes image@sha256:...
      - name: Deploy only if all gates pass
        if: success()
        run: kubectl apply -f deploy/
```

### The Shift-Left Imperative

The economic case is not academic. NIST SP 800-218 cites industry data:

| Stage Detected | Relative Cost to Fix |
|---|---|
| Requirements / design | $1 \times$ baseline |
| Implementation | $5 \times$ |
| Integration test | $10 \times$ |
| Beta / staging | $15 \times$ |
| Production | $30 \times$ |
| Post-breach (incident response) | $100 \times$ |

Shift-left = move security activities as early in the cycle as possible.

### Security Activity Map Across SDLCs

| Activity | Waterfall | Iterative | Agile | DevSecOps |
|---|---|---|---|---|
| Security requirements | Requirements phase | Cycle 1 | Backlog grooming | Paved road templates |
| Threat modeling | Design phase | Each cycle | Sprint 0 + per feature | Per-PR lightweight model |
| Secure coding standards | Coding phase | Coding phase | "Done" criteria | Linting + SAST in IDE |
| Code review | Code-complete | End of cycle | Per-PR | Per-PR, automated |
| SAST | Test phase | Each cycle | Per-PR | Per-PR, blocking |
| DAST | Test phase | Each cycle | Per-release | Per-build, blocking |
| SCA / SBOM | Test phase | Each cycle | Per-PR | Per-build |
| Pentest | Pre-release | Pre-release | Quarterly | Quarterly + continuous |
| Incident response plan | Maintenance | Maintenance | Maintenance | Continuous (runbooks in repo) |

### Selecting an SDLC (Decision Lens)

| Factor | Prefer Waterfall | Prefer Iterative | Prefer Agile | Prefer DevSecOps |
|---|---|---|---|---|
| Regulatory regime | Heavy (FDA, DO-178) | Moderate | Light | Light + cloud-native |
| Requirement stability | Fixed at start | Mostly fixed | Evolving | Rapidly evolving |
| Release cadence | Annual | Quarterly | Monthly | Daily/hourly |
| Team size | 5–50 | 20–100 | 20–200 | 50–5,000+ |
| Architecture | Monolith | Modular monolith | Service-oriented | Microservices / cloud |

### Common SDLC Anti-Patterns

1. **"Big-bang integration"** — only security-test at the end. Defects escape at the maximum rate.
2. **Security as a phase, not a practice** — assigning a "security sprint" defeats the model.
3. **Paved road that nobody walks** — secure templates exist but are not the default; teams bypass them.
4. **Tooling without ownership** — SAST produces 10,000 findings, no one triages. False-positive fatigue kills the program.
5. **Compliance theatre** — SSDF attestation with no actual practice use; auditors increasingly demand evidence.

## CISO / Risk Manager View

**Board framing.** The SDLC choice is a *risk posture decision*, not a methodology preference. A regulated medical-device firm that adopts pure DevSecOps without proper gates will fail an FDA audit; a consumer-internet firm that runs waterfall will ship vulnerabilities 10× faster than the market patches. The board does not need to choose — the CISO must show that **whichever model is selected, the security gates (SSDF practices) are mapped to it and produce measurable evidence**.

**Strategic priorities.**

1. **Pick a model; overlay SSDF.** Most enterprises are "agile with waterfall governance" — embrace it. Overlay SSDF rather than fighting the model.
2. **Codify the "Definition of Done"** to include security. A PR is not done until SAST passes, secrets are clean, threat model is attached, SBOM is generated. The gate is the policy.
3. **Invest in the paved road.** Self-service secure templates (Terraform modules, hardened base images, OAuth starters) cut secure-feature delivery time by 50%+ and prevent teams from going off-road.
4. **Make security evidence continuous.** Every gate produces an artifact (SARIF, SBOM, attestation). Aggregate them in a single dashboard; that is your program report.
5. **Attest annually to SSDF for federal work.** OMB M-24-04 is now mandatory; non-compliance disqualifies you from federal supply.

**Metrics the board cares about.**

- % of new projects with threat model attached
- % of merges blocked by security gates
- Mean time from PR open to "all gates green"
- # of production escapes per quarter (should be $\leq 1$/team/quarter)
- % of developers completing annual secure-coding training

## Related Connections

### Sibling L2
- [[secure-coding-practices-owasp]] — what the "secure code" in PW actually looks like
- [[threat-modeling-stride-pasta]] — the activity that lives in the design phase
- [[api-security-and-microservices]] — API-specific SDL design considerations

### L3 Standards
- [[devsecops-and-cicd-security]] — full DevSecOps pipeline gate inventory
- [[software-supply-chain-attacks-slsa]] — PS-group practices (provenance, SBOM, signing)

### Cross-Domain
- [[../Domain-01-Security-and-Risk-Management/risk-management-framework]] — SDLC consumes the threat register
- [[../Domain-06-Security-Assessment-and-Testing/Domain-06-Index]] — SAST/DAST/IAST are the verification family of D6
- [[../Domain-07-Security-Operations/Domain-07-Index]] — RV practices bridge to incident response

## References

- NIST SP 800-218 Rev. 1 — Secure Software Development Framework (SSDF) v1.1 (Feb 2022)
- NIST IR 8397 — Guidelines on Minimum Standards for Developer Verification of Software
- NIST SP 800-161r2 — Cybersecurity Supply Chain Risk Management Practices
- Microsoft Security Development Lifecycle (SDL) — 2022 update
- OWASP SAMM v2.0
- OWASP ASVS v4.0.3
- (ISC)² CISSP CBK — Domain 8: Software Development Security
- OMB Memorandum M-24-04 — Software Attestation for Federal Procurement (2024)
- Executive Order 14028 — Improving the Nation's Cybersecurity (May 2021)
- BSIMM 13 — Building Security In Maturity Model
