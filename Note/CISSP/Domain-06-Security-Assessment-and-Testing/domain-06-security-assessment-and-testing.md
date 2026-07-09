---
title: Domain 6 — Security Assessment and Testing (Hub)
layer: 1
domain: 6
tags: [cissp, domain-6, assessment, testing, audit, vulnerability-management, pentest, siem, hub]
related_domains: [1, 2, 4, 5, 7, 8]
---

# Domain 6 — Security Assessment and Testing

## Feynman Explanation

Imagine you spent a million dollars installing locks, cameras, and alarms on a house — Domain 6 is the part where you actually *check* that the locks can't be picked, the cameras actually see what they should, and the alarms go off when they're supposed to. Security controls are theories until you test them. This domain is the science of *proving* — through scans, penetration tests, audits, and log analysis — that what you designed in D3 and operated in D7 actually works against a real adversary. Without Domain 6, security is a feeling, not a discipline.

## Technical Details

### Scope of Domain 6 (CISSP CBK 2024 outline)

| Subtopic | Weight | L2 / L3 children |
|---|---|---|
| Assessment, test, and audit strategies | Core | [[security-audits-and-compliance-testing]] |
| Vulnerability management lifecycle | Core | [[vulnerability-management-lifecycle]] |
| Penetration testing methodology | Core | [[penetration-testing-methodology]] |
| Logging, monitoring, and SIEM | Core | [[logging-monitoring-and-siem]] |
| Web application security testing (OWASP) | High | [[owasp-top-10-and-web-app-testing]] |
| Threat-informed defense (ATT&CK) | High | [[mitre-att-and-ck]] |
| Red / Blue / Purple team operations | High | [[red-team-vs-blue-team-vs-purple-team]] |
| Test coverage, sampling, evidence handling | Foundational | [[security-audits-and-compliance-testing]] |
| KPI / metrics (MTTD, MTTR, coverage) | Foundational | [[logging-monitoring-and-siem]] |

### The three testing strategies (one-page reference)

| Strategy | Question it answers | Cadence | Who executes | Output |
|---|---|---|---|---|
| **Vulnerability scan** | "What known weaknesses exist on my assets?" | Weekly–monthly | Internal SOC / MSSP | CVE list with CVSS + EPSS |
| **Penetration test** | "Can an attacker actually chain weaknesses to reach a crown jewel?" | Annual + post-major change | Independent red team | Exploited paths + risk rating |
| **Security audit** | "Do my controls meet the policy / regulation I claimed?" | Annual (SOX, SOC 2) + ad-hoc | Internal audit / external CPA | Opinion + management letter |
| **Continuous monitoring** | "Is anything breaking *right now*?" | 24×7 | SOC + SIEM/SOAR | Alerts, dashboards, MTTD/MTTR |

These are **not interchangeable**: a clean scan does not mean the system is safe; a clean pen test does not mean controls are compliant; a clean audit does not mean attackers can't get in. CISSP expects the candidate to know which test answers which question.

### The feedback loop (one mental model)

```
    ┌─────────────┐    discover     ┌──────────────┐
    │  D3 design  │ ──────────────▶ │  D6 testing  │
    │  controls   │ ◀────────────── │  scans/pentest│
    └─────────────┘     findings    └──────┬───────┘
                                            │ incidents
                                            ▼
                                     ┌──────────────┐
                                     │  D7 respond  │
                                     │  (SOC, IR)   │
                                     └──────┬───────┘
                                            │ lessons
                                            ▼
                                     ┌──────────────┐
                                     │  D1 risk     │
                                     │  (treatment) │
                                     └──────────────┘
```

The same finding (e.g., a vulnerable public-facing web server) flows from **discovery** (D6 scan) → **exploitation validation** (D6 pentest) → **incident response** (D7) → **risk treatment decision** (D1) → **control redesign** (D3) → **re-test** (D6). Domain 6 is the **verification step** in that loop.

### Test independence (a CISSP exam favorite)

| Independence level | Definition | Trust signal |
|---|---|---|
| Self-assessment | Same team that built it tests it | Lowest — bias guaranteed |
| Internal independent | Different team, same org (internal audit) | Medium — good for control health |
| External independent | Third-party firm (Big-4 CPA, certified pen-test firm) | Highest for regulators |
| Regulatory mandated | Required by law/regulation (PCI ASV, FedRAMP 3PAO) | Highest for compliance |

Independence is a control itself: auditors cannot audit their own work; testers cannot release code they tested. The same separation-of-duties principle from D5 applies to the testing function.

### Mapping to other CISSP domains

| Cross-domain link | Where it lives | Why it matters here |
|---|---|---|
| Risk treatment prioritization → [[Domain 1 — Security and Risk Management]] | D1 risk register, treatment options | Testing results *are* the empirical evidence behind residual-risk claims |
| Vulnerability in code → [[Domain 8 — Software Development Security]] | D8 SDL, SAST/DAST/SCA | Dev-time scanning + prod-time scanning are complementary |
| Web app attack surface → [[Domain 4 — Communication and Network Security]] | D4 WAF, TLS, segmentation | Web findings need D4 controls to remediate at scale |
| Identity weaknesses → [[Domain 5 — Identity and Access Management]] | D5 IAM, MFA, privileged access | Weak IAM is the #1 pen-test finding |
| Alerting & response → [[Domain 7 — Security Operations]] | D7 SIEM, IR, BCP | Findings become SIEM use cases; incidents trigger ad-hoc tests |
| Asset criticality (what to test first) → [[Domain 2 — Asset Security]] | D2 data classification, asset inventory | You test the things that matter, not the things that are easy to scan |

### The "verification pyramid" (mental model)

```
                    ┌──────────────┐
                    │   Red Team   │   Adversary emulation, purple-team BAS
                    ├──────────────┤
                    │  Pen Test    │   Annual, scoped, rules-of-engagement
                    ├──────────────┤
                    │  Vuln Scan   │   Weekly / monthly, authenticated
                    ├──────────────┤
                    │  Config /    │   CIS Benchmarks, hardening baselines
                    │  Compliance  │   (PCI ASV, DISA STIG, CIS)
                    ├──────────────┤
                    │  SIEM        │   Continuous, log-based detection
                    │  Monitoring  │   coverage
                    └──────────────┘
        Most frequent ◀──────────────────▶ Most realistic
        Cheapest                              Most expensive
```

Each layer catches a different class of problem. Skipping a layer creates blind spots a determined adversary will exploit.

<!-- EXPANDED -->

## Foundations

Security testing is the **feedback loop that validates whether D1–D5 controls actually work**. The assessment pyramid orders activity by breadth vs. depth: **audits** (broad, shallow — "do you have process?"), **vulnerability scans** (medium breadth/depth — "what known CVEs exist?"), **penetration tests** (narrow, deep — "can an attacker chain exploits?"), and **red team exercises** (adversary simulation — "can they achieve the objective?"). The guiding axiom: *"You can't protect what you don't test."*

The fundamental distinction CISSP demands: a **vulnerability assessment** finds *known issues* against a signature database; a **penetration test** demonstrates *exploit chains* against a live target. Test output is only as good as the **scope and rules of engagement** — a senior assessor knows that what's out of scope is the most valuable part of the attack surface. [[penetration-testing-methodology]] [[vulnerability-management-lifecycle]]

## Architecture

**Assessment program architecture** follows an annual cycle: audits → vuln scanning → pen tests → red team. The cadence stacks from frequent/cheap (weekly authenticated scans) to annual/expensive (external pen test, ISCM per [[logging-monitoring-and-siem]]).

**Security testing in CI/CD** — the shift-left pipeline:

| Stage | Tool class | What it catches |
|---|---|---|
| Commit | SAST, secrets scanning (gitleaks, truffleHog), pre-commit hooks | Hardcoded keys, dangerous functions |
| Build | SCA (SBOM, Dependabot, Renovate), container scanning (Trivy, Grype) | Known-vuln libs, base-image CVEs |
| Staging | DAST (Burp, ZAP), IaC scanning (tfsec, Checkov, kics) | Runtime vulns, misconfigs, open ports |
| QA | IAST (Contrast, Seeker) — agent-instrumented runtime analysis | Context-aware code flaws |
| Deploy | WAF rule deploy, RASP, canary deployment validation | Prod-time detections |

**SIEM architecture** follows the pipeline: **log collection** (agents, syslog, API) → **parsing/normalization** (CEF, LEEF, ECS, OTel) → **correlation engine** (rules, ML, UEBA) → **alerting** (tier-1 queue) → **SOAR playbooks** (auto-containment, enrichment) → **long-term storage** (hot/warm/cold tiers). See [[logging-monitoring-and-siem]] for the full logical architecture diagram and KPI math.

**Detection engineering architecture**: threat intel ([[mitre-att-and-ck]]) → Sigma rules (detection-as-code) → SIEM queries → alert triage → incident — a continuous loop measured by MTTD and MTTR.

## Execution

**Running a vulnerability management program** proceeds through five stages (NIST SP 800-40r4): discovery scanning → authenticated scanning → prioritization (using the intersection of CVSS + EPSS + SSVC + asset criticality + CISA KEV) → ticketing with SLA enforcement → verification rescan. The prioritization triangle:

$$
\text{Patch first} = \{\text{CVSS High}\} \cap \{\text{EPSS High}\} \cap \{\text{Critical asset}\} \cap \{\text{KEV membership}\}
$$

CVSS tells you *how bad it could be*; EPSS tells you *how likely it is to be exploited in the next 30 days*; SSVC tells you *what action to take* (Track / Track* / Attend / Act). See [[vulnerability-management-lifecycle]] for the full formula and SLA matrix.

**Conducting a penetration test** per PTES: scoping and RoE → recon (OSINT + active) → vulnerability identification → exploitation → post-exploitation → lateral movement → reporting with executive summary + technical findings + remediation guidance. The RoE is the *contract* — without it, the test is legally an attack. [[penetration-testing-methodology]]

**Writing effective SIEM detection rules** uses three correlation types: **threshold rules** (e.g., >10 failed logins in 5 min — medium FP risk), **behavioral/anomaly baselines** (UEBA — higher FP risk, needs tuning), and **cross-source correlation** (e.g., firewall block + EDR process + DNS query within 60s → low FP, high fidelity). Top SOCs target precision ≥ 0.7 and detection coverage ≥ 80% of relevant ATT&CK techniques.

**Statistical sampling for audits**: given confidence level $Z$ and margin of error $e$:

$$
n = \frac{Z^2 \cdot p \cdot (1 - p)}{e^2}
$$

For 95% confidence, $p=0.5$, $e=0.05$: $n \approx 384$ records. Finite population correction ($n_{adj} = n / (1 + (n-1)/N)$) applies when $n/N > 5\%$. Sampling types: **attribute** (pass/fail per item — most common in IT audits), **variable** (measuring a continuous characteristic), **discovery** (finding at least one deviation — used for fraud detection). See [[security-audits-and-compliance-testing]] for full sampling math and worked examples.

## Mastery

**Purple teaming** is collaborative, not competitive — blue team shows detection coverage gaps in *real-time* during red team exercises, closing the loop in days rather than post-engagement reports that rot in Jira backlogs. The purple-team lead (often dual-hat with Detection Engineering or AppSec) is the facilitator, not a third team. [[red-team-vs-blue-team-vs-purple-team]]

**Breach and Attack Simulation (BAS)** automates adversary emulation at scale. Platforms (MITRE Caldera, Atomic Red Team, AttackIQ, Cymulate, Picus, SCYTHE) execute single techniques or attack chains daily/weekly and report what was **blocked**, **detected**, or **missed**. BAS provides *continuous* detection-coverage validation — complementary to annual pen tests. [[red-team-vs-blue-team-vs-purple-team]]

**Threat-informed defense** uses [[mitre-att-and-ck]] to map controls to adversary TTPs, identifying *detection coverage gaps*. The CISO's strategic artifact is an ATT&CK Navigator heatmap updated quarterly — green (detected), yellow (partial), red (gap). This shifts the conversation from "we have a SIEM" to "we detect 87% of techniques adversaries use against our industry, with named gaps." Each gap becomes a prioritized detection engineering task.

**EPSS vs. CVSS**: EPSS (Exploit Prediction Scoring System, FIRST.org) is a machine-learning model that outputs the *probability* a CVE will be exploited in the wild within 30 days. CVSS is worst-case technical severity; EPSS is *realized* likelihood. A program patching CVSS 9+ indiscriminately wastes effort on bugs no one is exploiting; a program patching CVSS 7 + EPSS ≥ 0.5 first is threat-informed.

**Sigma rules** are detection-as-code — platform-agnostic detection rules (e.g., "detect Mimikatz LSASS access") that compile to SIEM-specific queries (Splunk SPL, Elastic KQL, Sentinel KQL, QRadar AQL). Sigma enables sharing, versioning, and testing of detection content across heterogeneous SOC stacks. They are the operational bridge between ATT&CK technique and SIEM alert.

## Resilience

**Top assessment mistakes**: scope creep (pen tests that drag on for 6 months produce unreadable reports); running vuln scans without *written* authorization (the scan traffic is indistinguishable from an attack); falsifying pentest reports (a compliance crime, not just a security failure); treating compliance as security (SOC 2 Type II ≠ unbreachable); checkbox auditing (signing off controls without testing them); ignoring false positives until alert fatigue kills the SOC (analyst burnout → real attacks hide in noise → breach).

**When SIEM becomes log ingestion hell**: a SIEM collecting 10,000 EPS with 20 unmaintained rules, no SOAR, and 4,000 daily un-triaged alerts is a *liability*, not a capability. Analysts burn out; real attacks hide in noise. Investment in detection engineering (people, process, use-case library) must match or exceed the SIEM license cost.

**When pen tests miss everything**: the three most common causes: (1) scope too narrow — testing only the public web tier while the real asset is an internal database; (2) no credentials provided — unauthenticated scans miss 40–60% of vulnerabilities; (3) no social engineering allowed — the #1 breach vector excluded from the test.

**The "patched but not mitigated" problem**: a vulnerability is patched on the asset, but the *root cause* (e.g., no input validation in a framework used by 40 apps) persists across the estate. The patch closes one instance; the systemic weakness remains unaddressed until the next scan finds it on a different host.

## Context

**SOC 2 Type I vs Type II**: Type I = point-in-time ("controls were suitably designed as of date X"); Type II = period-of-time ("controls operated effectively over the period," typically 6–12 months). Type II is the higher-assurance report enterprises and regulators require. See [[security-audits-and-compliance-testing]].

**ISO 27001 certification audit cycle**: Stage 1 (documentation review — is the ISMS properly designed?) → Stage 2 (implementation audit — are the Annex A controls operating effectively?) → surveillance audits years 2–3 → recertification year 3. Companion standard ISO 27002:2022 provides implementation guidance for Annex A controls.

**PCI DSS assessment**: **SAQ** (Self-Assessment Questionnaire) applies to lower-volume merchants (Level 2–4); **ROC** (Report on Compliance) requires an on-site audit by a QSA (Qualified Security Assessor) for Level 1 merchants (>6M transactions/year). All levels require quarterly ASV (Approved Scanning Vendor) external vulnerability scans. PCI DSS v4.0 §11 mandates penetration testing and vulnerability scanning cadences.

**FedRAMP 3PAO assessment**: US federal cloud services require assessment by an accredited Third-Party Assessment Organization (3PAO) against the FedRAMP baseline (NIST SP 800-53 Rev. 5 controls). The 3PAO produces the Security Assessment Report (SAR) that feeds the Authority to Operate (ATO) decision.

**HIPAA Security Risk Analysis**: *not optional* — 45 CFR §164.308(a)(1)(ii)(A) requires covered entities and business associates to "conduct an accurate and thorough assessment of the potential risks and vulnerabilities to the confidentiality, integrity, and availability of electronic protected health information." It is the single most-cited HIPAA enforcement finding.

## Nuance

**Vulnerability vs. exploit**: a vulnerability is a *flaw* (the unlocked window); an exploit is the *act of using it* (climbing through). Risk = $Threat \times Vulnerability \times Impact$. A critical CVE on an air-gapped test server has zero risk; a medium CVE on the internet-facing payment gateway has high risk. Asset context determines severity.

**False positive vs. false negative in detection**: false positive = alert on benign activity → alert fatigue, analyst desensitization, burnout. False negative = missed real attack → undetected breach. Both are bad, but the *organizational failure mode* differs: high FP kills the SOC (people quit); high FN kills the business (data exfiltrated without detection). Mature programs optimize for **both** using precision/recall tuning ($F_1 = 2 \cdot \frac{P \cdot R}{P + R}$).

**Internal vs. external testing**: different threat models, different value. External testing simulates the internet-based attacker; internal testing simulates the insider, phished employee, or attacker who breached the perimeter. Both are mandatory for a complete assessment — a PCI DSS Level 1 assessment requires both.

**Authenticated vs. unauthenticated scanning**: authenticated scans (with domain/user-level credentials) find 40–60% more vulnerabilities than unauthenticated because they can check file versions, registry keys, and patch levels directly. Unauthenticated scans only see what's exposed to the network. CISSP expects you to know this delta.

**Testing vs. exercising**: testing validates a specific control (e.g., "does the WAF block SQLi?"). Exercising validates the *process* (e.g., "can the IR team contain a ransomware event within 4 hours?"). Testing is technical; exercising is organizational. Both live in Domain 6's assessment mandate.

## Resources

| Resource | What it provides |
|---|---|
| **Vuln mgmt program maturity model** | CREST, Gartner — 5-level maturity from ad-hoc scanning to continuous, threat-informed VM |
| **Pen test scoping questionnaire** | PTES pre-engagement template — in-scope IPs, RoE, authorization, stop conditions, data handling |
| **SIEM use-case prioritization matrix** | Rank detection use cases by ATT&CK technique prevalence × asset criticality × industry threat actors |
| **Audit evidence collection checklist** | Policy docs, config screenshots, SIEM query results, ticketing trails, attestation signatures — per control |
| **Common false-positive patterns by scanner type** | Nessus: stale credentials, shared libraries; Qualys: end-of-life OS detection; DAST: session timeout, CSRF tokens; NDR: NAT/load-balancer IP multiplexing |

> **⚠️ CONFLICT NOTE — outdated references in child notes:**
> - [[owasp-top-10-and-web-app-testing]] references **OWASP Top 10:2021** — the current version as of 2025 is **OWASP Top 10:2025** (with updated category rankings and new entries). The testing methodology (WSTG) and ASVS remain valid; the Top 10 categorization has shifted.
> - [[mitre-att-and-ck]] describes **14 Enterprise tactics** — the current ATT&CK Enterprise matrix (v16+) includes **15 tactics**, adding TA0040 **Impact** which was previously a sub-tactic and is now a top-level tactic.
> - The parent hub's reference "MITRE ATT&CK Enterprise Matrix v15+" remains broadly correct but should be updated to v16+ for current exams.

<!-- /EXPANDED -->

## CISO / Risk Manager View

For the board, Domain 6 is the answer to the question: **"How do we *know* we are secure?"** Every CISO eventually has to defend, in front of a regulator or a board, the statement *"our security program is effective."* Domain 6 is what makes that statement defensible — or exposes it as a slogan.

Three board-level questions this domain forces the CISO to answer:

1. **What is tested, how often, and by whom?** A maturity model: Level 1 = annual vulnerability scan; Level 2 = monthly authenticated scans + annual pen test; Level 3 = continuous BAS + purple-team exercises + adversarial emulation aligned to [[mitre-att-and-ck]]. Most regulators expect Level 2 as a *floor*.
2. **What is our MTTD and MTTR, and what is the trend?** The two KPIs from [[logging-monitoring-and-siem]] are the single best summary of operational security. Industry median MTTD ≈ 200 days; top-quartile programs are <24 hours. Trend matters more than absolute value.
3. **Are we testing the right things?** Coverage bias is the silent killer: a company that scans 100% of internet-facing hosts but 0% of internal AD has *worse* security than a peer that scans 30% of both. Asset inventory from D2 is the input to test scope.

**Budget framing for the CFO:** a continuous-testing program (vuln scanning + SIEM + annual pen test) is typically **3–8% of IT security spend**. Reactive spending after a breach averages **$4.88M per incident** (IBM Cost of a Data Breach 2024) — the math favors prevention. **Risk-based testing** is the 2024–2025 board theme: spend the testing dollar on the assets whose compromise would cause a material-loss event, not on the assets that are easiest to scan.

**Regulator posture:** PCI DSS, SOC 2, ISO 27001, HIPAA, FedRAMP, NIS2, and DORA all *mandate* specific test types and cadences. Domain 6 is the only domain that the regulator will directly inspect — you cannot pass an audit by having policies; you must produce evidence of execution.

## Related Connections

### Layer 2 deep-dives (children)
- [[vulnerability-management-lifecycle]]
- [[penetration-testing-methodology]]
- [[security-audits-and-compliance-testing]]
- [[logging-monitoring-and-siem]]

### Layer 3 specifics (children)
- [[owasp-top-10-and-web-app-testing]]
- [[mitre-att-and-ck]]
- [[red-team-vs-blue-team-vs-purple-team]]

### Cross-domain
- [[Domain 1 — Security and Risk Management]] (risk treatment, residual risk)
- [[Domain 2 — Asset Security]] (asset inventory drives test scope)
- [[Domain 4 — Communication and Network Security]] (network testing, WAF, segmentation verification)
- [[Domain 5 — Identity and Access Management]] (identity is the #1 tested control)
- [[Domain 7 — Security Operations]] (SIEM, IR, BCP)
- [[Domain 8 — Software Development Security]] (SAST/DAST/SCA, dev-time testing)

## References

- ISC² CISSP CBK, 5th ed. — Domain 6
- NIST SP 800-40r4, *Guide to Enterprise Patch Management Planning* (2022)
- NIST SP 800-115, *Technical Guide to Information Security Testing and Assessment* (2008, still authoritative)
- NIST SP 800-137A, *Information Security Continuous Monitoring (ISCM) for Federal Information Systems*
- NIST SP 800-53A, *Assessing Security and Privacy Controls*
- PTES — Penetration Testing Execution Standard (www.pentest-standard.org)
- OSSTMM 3 — Open Source Security Testing Methodology Manual (ISECOM)
- ISACA *Information Technology Assurance Framework (ITAF)*
- AICPA SSAE-18 / SOC 2 (2017, with 2022 SSAE-20 update)
- ISO/IEC 27001:2022 Annex A (control testing clauses)
- ISO/IEC 19011:2018, *Guidelines for Auditing Management Systems*
- ISO/IEC 27008, *Technical Cybersecurity Assessment*
- OWASP Testing Guide (WSTG) v4.2
- MITRE ATT&CK Enterprise Matrix v15+
- FIRST.org — CVSS v3.1, EPSS, SSVC
- PCI DSS v4.0 §11 (Test security of systems and networks regularly)
- IBM *Cost of a Data Breach Report 2024*
- Mandiant *M-Trends 2024* (industry MTTD benchmarks)
