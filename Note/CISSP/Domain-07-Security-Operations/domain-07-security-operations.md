---
title: Domain 7 — Security Operations (Hub)
layer: 1
domain: 7
tags: [cissp, domain-7, security-operations, hub, soc, ir, dr, forensics, change-management, physical]
related_domains: [1, 2, 4, 5, 6]
---

# Domain 7 — Security Operations

## Feynman Explanation

Security Operations is the "factory floor" of cybersecurity — every other domain produces a plan, a design, a control, or a policy, and Domain 7 is the team that actually runs those things 24×7. It is where detection, response, recovery, forensics, change, and physical safety all become measurable daily work. If a firewall rule is never reviewed, a backup is never tested, or a camera feed is never watched, then the design on paper is worthless — Domain 7 is the proof.

## Technical Details

### Scope of Domain 7 (CISSP CBK 2024 outline)

| Subtopic | Weight guidance | Children |
|---|---|---|
| Security operations concepts (principle of least privilege, need-to-know, separation of duties, defense in depth) | Foundational | — |
| Investigations, digital forensics, e-evidence | High | [[digital-forensics-process]] |
| Logging, monitoring, SIEM, SOAR, detection engineering | High | [[siem-soar-and-detection-engineering]] |
| Incident management & response (NIST SP 800-61) | High | [[incident-response-lifecycle-nist]], [[ransomware-response-playbook]] |
| Change & configuration management (CAB, baselines, ITIL) | Medium | [[change-and-configuration-management]] |
| Disaster Recovery & BCP continuity (RTO/RPO/MTPD) | High | [[disaster-recovery-and-bcp]], [[backup-strategies-3-2-1]] |
| Physical & environmental security (perimeter, HVAC, fire, TEMPEST) | Medium | [[physical-and-environmental-security]] |
| SOC models, in-house vs MSSP, metrics, OODA | High | [[security-operations-center-soc-models]] |

### The three operations time-constants

| Metric | Definition | Typical target | Owner |
|---|---|---|---|
| **MTTA** (Mean Time To Acknowledge) | Time from alert fired to analyst engaged | <15 min for Sev-1 | Tier 1 SOC |
| **MTTD** (Mean Time To Detect) | Time from adversary action to detection | Hours to days | Detection engineering |
| **MTTR** (Mean Time To Respond/Recover) | Time from detection to containment + eradication + recovery | Hours | IR team + ops |

A useful identity for the board:

$$MTTR_{total} = MTTD + MTTA + MTTI_{contain} + MTTI_{eradicate} + MTTI_{recover}$$

where $MTTI_x$ is the mean time for each IR phase. Reducing any one term lowers dwell time and therefore blast radius.

### Core operational capabilities (the "ops plane")

```
                    ┌──────────────────────────┐
                    │   Governance & Risk (D1)  │
                    └─────────────┬────────────┘
                                  │
        ┌────────────────┬────────┴────────┬─────────────────┐
        │                │                 │                 │
   ┌────▼─────┐    ┌─────▼──────┐    ┌─────▼──────┐    ┌─────▼──────┐
   │  People  │    │  Process   │    │ Technology │    │  Physical  │
   │ SOC, IR, │    │ IR plan,   │    │ SIEM, EDR, │    │ CCTV, HVAC,│
   │ GRC, IAM │    │ CAB, BCP   │    │ NDR, SOAR  │    │ fire sup.  │
   └────┬─────┘    └─────┬──────┘    └─────┬──────┘    └─────┬──────┘
        │                │                 │                 │
        └────────────────┴────────┬────────┴─────────────────┘
                                  │
                       ┌──────────▼──────────┐
                       │   Detect → Respond  │
                       │   → Recover → Learn │
                       └─────────────────────┘
```

Every box in the diagram produces a stream of signals (logs, alerts, change tickets, physical events) and consumes a stream of decisions (policies, baselines, runbooks). Domain 7 is the glue.

### The four operational sub-domains (one-line summaries)

| Sub-domain | Question it answers | L2/L3 note |
|---|---|---|
| **Incident Response (IR)** | "We have a breach — what do we do, in what order, by whom, with what evidence?" | [[incident-response-lifecycle-nist]] |
| **Digital Forensics** | "What happened, on which system, at what time, who did it, and how do we prove it in court?" | [[digital-forensics-process]] |
| **Change & Config Mgmt** | "Who can change what, when, and how do we know the change did not break security or stability?" | [[change-and-configuration-management]] |
| **DR / BCP** | "When the building, region, or vendor goes away, how do we keep operating and get back to normal?" | [[disaster-recovery-and-bcp]], [[backup-strategies-3-2-1]] |
| **Physical Security** | "How do we stop someone from walking in, breaking in, or causing fire/water/EM damage?" | [[physical-and-environmental-security]] |
| **SOC & Detection Eng.** | "How do we run the watch floor, tune the SIEM, and engineer high-fidelity alerts at scale?" | [[siem-soar-and-detection-engineering]], [[security-operations-center-soc-models]] |
| **Ransomware Playbook** | "Encrypted by a threat actor — do we pay, restore, notify, isolate, rebuild?" | [[ransomware-response-playbook]] |

### Foundational principles (the "ops rules of the road")

| Principle | Operational meaning |
|---|---|
| **Least privilege** | Users, services, and processes get the minimum rights needed for the current task; reviewed quarterly. |
| **Need-to-know** | Information is shared only with those who need it for their role. |
| **Separation of duties (SoD)** | No single person can both request and approve a critical action (e.g., change + deploy). |
| **Defense in depth** | Multiple overlapping controls: perimeter → network → identity → application → data → audit. |
| **Privileged Access Management (PAM)** | Just-in-time elevation, session recording, vault for secrets; tied to D5 ([[domain-05-identity-and-access-management\|D5]]) and enforced in ops. |
| **Two-person integrity (TPI)** | Two authorized individuals required for the most sensitive actions (e.g., key ceremony, root CA). |
| **Record retention** | Logs, evidence, and chain-of-custody records kept per legal hold and regulatory minimum. |

### Resource protection (the CIA triad in operations)

| Domain | Operations task | Example |
|---|---|---|
| **Confidentiality** | DLP, encryption, access logs, PAM | Block USB exfil, encrypt at rest, alert on mass download |
| **Integrity** | File integrity monitoring, baselines, code signing, change control | Tripwire on system binaries, signed OS images |
| **Availability** | HA, DR, capacity, rate-limiting, BCP/DRP | Multi-AZ, hot standby, scrubbing center |

### The ops ↔ other domains map

| Operations activity | Tied to domain | Why |
|---|---|---|
| Asset discovery (CMDB) | [[domain-02-asset-security\|D2]] | Inventory is the input to every ops decision |
| Vulnerability scan → patch cycle | [[domain-06-security-assessment-and-testing\|D6]] | Ops consumes vuln findings as a patch pipeline |
| PAM/JIT, service account rotation | [[domain-05-identity-and-access-management\|D5]] | Identity is enforced by ops runbooks |
| Network monitoring, NDR, NGFW logs | [[domain-04-communication-and-network-security\|D4]] | Network telemetry is the largest SIEM source |
| RTO/RPO/MTPD targets | [[domain-01-security-and-risk-management\|D1]] | Business risk sets ops targets |
| Secure SDLC for in-house tools | [[domain-08-software-development-security\|D8]] | Ops engineers write SOAR playbooks and parsers |

<!-- EXPANDED -->

## Foundations

### Security operations as the immune system

Security operations is the immune system of the organization — not a cost center, but the body's built-in capability to detect intrusions, mount a response, clear the infection, recover to a healthy state, and learn (immunity) so the same pathogen cannot succeed twice.

| Immune concept | Ops equivalent |
|---|---|
| Innate immunity (skin, macrophages) | Perimeter controls, FW, EDR, baseline rules |
| Adaptive immunity (T-cells, B-cells) | Hunt teams, custom Sigma rules, threat intel |
| Inflammation → fever | SIEM alert → Sev-1 page → war room |
| Antibody memory | Post-incident rule, IOCs in blocklist |
| Autoimmune disease | Over-tuned rules (too many FPs), SOAR run amok |
| Immunodeficiency | Unfunded SOC, alert fatigue, no after-hours |

### The OODA loop — the core mental model for SOC operations

Colonel John Boyd's OODA loop (Observe → Orient → Decide → Act) is the irreducible engine of every SOC decision, from triage to hunt:

$$Loop\ Time = t_{observe} + t_{orient} + t_{decide} + t_{act}$$

| OODA | SOC stage | Time target (Sev-1) |
|---|---|---|
| Observe | Ingest log, alert fires, SIEM surfaces | < 1 sec (automated) |
| Orient | SOAR enrichment, analyst context, threat intel lookup | < 5 min |
| Decide | Triage: FP / TP / escalate? Containment strategy? | < 10 min |
| Act | Isolate, disable, block, page, escalate | < 15 min (containment) |

The adversary runs their own OODA loop. The SOC's job is to make its loop tighter than the adversary's. When the SOC's loop is slower, dwell time grows and blast radius expands. See [[security-operations-center-soc-models]] for tier-to-OODA mapping.

### Operations — the domain that makes D1–D6 decisions real

| If this domain... | ...produces this output | Domain 7 makes it real by... |
|---|---|---|
| [[domain-01-security-and-risk-management\|D1]] | RTO/RPO targets, risk appetite | Running DR, measuring MTTR, reporting risk exposure |
| [[domain-02-asset-security\|D2]] | Asset inventory, classification | Feeding CMDB into SIEM, scoping IR, sizing backup |
| [[domain-03-security-architecture-and-engineering\|D3]] | Security architecture, SCIF design | Operating and tuning the controls the architect designed |
| [[domain-04-communication-and-network-security\|D4]] | NGFW, NDR, DNS, proxy telemetry | Ingesting into SIEM, building detection on netflow/DNS |
| [[domain-05-identity-and-access-management\|D5]] | PAM, JIT, access reviews | Enforcing SoD in change mgmt, disabling accounts during IR |
| [[domain-06-security-assessment-and-testing\|D6]] | Vuln findings, pen test reports | Patching (the ops pipeline), feeding detection gaps |
| [[domain-08-software-development-security\|D8]] | Secure SDLC, code signing | Writing SOAR playbooks (code), deploying infra-as-code |

If assessment (D6) finds a gap, operations (D7) closes it. This is the feedback loop of a mature program.

### What a 10-year SOC director knows

> 90% of alerts are noise, and tuning is the actual job.

The most common failure mode is building a detection factory without a quality-control line. A SOC that fires 5,000 alerts/day with 3 analysts is not defending — it is performing theater. The director's actual job:

1. **Tune relentlessly** — retire stale rules, suppress known-good patterns, ship daily FP reductions.
2. **Automate Tier 0** — SOAR should handle 60%+ of L1 triage (automated enrichment, known-FP closure).
3. **Invest in retention** — a 3-year-tenured analyst sees patterns a 6-month analyst cannot. Burnout is the #1 security risk.
4. **Measure what matters** — MTTD, MTTR, dwell time, detection coverage (ATT&CK heatmap), not "alerts closed."
5. **Tabletop exercises** — the best IR plan in a binder is worth zero; the one rehearsed quarterly is worth millions.

## Architecture

### SOC architecture — the tier pipeline

```
Tier 0 (Automated) — SOAR auto-contain, auto-enrich, auto-close
        ↓  (~60% resolved)
Tier 1 (Triage)    — Alert acknowledgement, basic enrichment, FP triage
        ↓  (~5% escalate)
Tier 2 (Investigation) — Deep analysis, scoping, containment, forensics
        ↓  (~1% escalate)
Tier 3 (Threat Hunting / Engineering) — Hypothesis-driven hunt, custom rules, malware RE
        ↓  (findings feed back into detection engineering pipeline)
Tier 4 (SOC Manager) — Watch operations, escalations, vendor mgmt, board reporting
```

See [[security-operations-center-soc-models]] for staffing math (~6 analysts per 24×7 seat), operating models (in-house/MSSP/hybrid), and per-tier KPIs.

### Detection engineering pipeline

```
Threat Intel → Use Case → Sigma Rule → CI Test → SIEM Deploy → Alert → Triage → Tune → Metrics
    ↑                                                                         ↓
    └─────────────────────── Post-incident feedback ──────────────────────────┘
```

| Stage | Owner | Artifact |
|---|---|---|
| Threat model | T3 / Threat intel | ATT&CK heatmap, priority techniques |
| Use case development | Detection engineer | Written hypothesis, required log sources |
| Detection rule | Detection engineer | Sigma rule (version-controlled in Git) |
| CI/CD test | Detection engineer + red team | Atomic Red / Caldera replay, FP/TP rate |
| SIEM deployment | SOC engineer | Rule promoted to production |
| Alert validation | T1/T2 | Does the alert fire? Is the signal useful? |
| Tuning | Detection engineer | Threshold adjustment, exclusion list update |

The pipeline treats detection rules like software: version-controlled, tested, reviewed, deployed, and retired. See [[siem-soar-and-detection-engineering]] for the full Sigma-to-SIEM workflow and ATT&CK mapping.

### Log collection architecture

```
Agents (Beats/Fluentd/Sysmon) → Forwarders (Kafka/Logstash/Cribl) → Indexers (Elastic/Splunk/Chronicle) → Search Heads/Dashboards
```

| Layer | Purpose | Failure mode |
|---|---|---|
| Agents | Collect from endpoint/network/cloud, parse, tag | Silent failure, CPU spike, missing sourcetype |
| Forwarders | Buffer, route, transform (redact PII, enrich with CMDB) | Backpressure, clock skew, event drop |
| Indexers | Store, index, compact | Disk full, ingestion lag, hot/warm imbalance |
| Search heads | Query, correlate, dashboard | Slow queries, expired data, missing fields |

**Golden rule:** a detection rule cannot fire on data that is not ingested. The #1 detection gap is not missing rules — it is missing log sources. See [[siem-soar-and-detection-engineering]] for log source taxonomy and EPS sizing.

### Backup architecture — the 3-2-1-1-0 stack

```
Production Data
      ├──→ On-prem backup (disk, NAS) — Copy #2, Media #1
      │         └──→ Immutable snapshot (object lock, WORM) — Immutable copy
      └──→ Offsite backup (cloud, tape vault) — Copy #3, Media #2
                └──→ Air-gapped (separate account, MFA delete gate)
                         └──→ RESTORE TEST ← Quarterly, zero errors required
```

| Component | Implementation | Adversary resistance |
|---|---|---|
| 3 copies | Prod + on-prem + offsite | Single site loss survivable |
| 2 media | Disk + cloud/tape | Single media attack surface |
| 1 offsite | ≥ 100 km, different power grid | Regional disaster survivable |
| 1 immutable | Object lock compliance mode, WORM tape | Ransomware cannot delete |
| 0 errors | Quarterly restore drill, hash validation | Proof of recoverability |

See [[backup-strategies-3-2-1]] for per-component implementation details and cost math.

### DR architecture — site types and RTO/RPO trade-offs

| Site type | RTO | RPO | Cost multiplier | When |
|---|---|---|---|---|
| Active/active multi-region | Seconds | Near-zero | 4× | Mission-critical, zero-downtime |
| Hot site (active/passive) | Minutes | Seconds | 3× | Tier-1, regulated |
| Warm site (pilot light) | Hours | Minutes | 2× | Tier-2, async acceptable |
| Cold site (empty rack) | Days | Hours | 1× | Tier-3, backup-restore |
| Cloud DR (DRaaS) | Minutes–hours | Minutes | 1.5× | Modern default for most orgs |
| Reciprocal agreement | Variable | Variable | 0× (free) | Almost never works under real stress |

The architecture decision is driven by $RTO + RPO \le MTPD$. If RTO=4h and RPO=1h, MTPD must be ≥ 5h. See [[disaster-recovery-and-bcp]] for activation criteria, failover/failback sequence, and cloud DR patterns (backup & restore / pilot light / warm standby / multi-site active-active).

## Execution

### Running an incident response (NIST SP 800-61 Rev 3, CSF 2.0 Community Profile)

> ⚠️ **CONFLICT**: NIST SP 800-61 Rev 2 (2012) was **withdrawn April 2025** and superseded by **Rev 3**. Rev 3 is structured as a **CSF 2.0 Community Profile**, not a standalone guide. The lifecycle is now aligned with the six CSF 2.0 Functions (Govern, Identify, Protect, Detect, Respond, Recover), not the original 4-phase model. References in this vault that cite "Rev 2" or the 4-phase lifecycle (e.g., in [[incident-response-lifecycle-nist]], [[ransomware-response-playbook]], [[security-operations-center-soc-models]]) should be updated accordingly. The old 4-phase model still appears in CISSP exam content and may be used for exam purposes.

**CSF 2.0-aligned IR lifecycle (Rev 3):**

| CSF Function | IR phase (operational) | Key activities |
|---|---|---|
| Govern | Governance & Preparation | Policy, roles, risk appetite, legal pre-brief, IR plan, training, tabletops |
| Identify | Asset & Business Impact | Asset inventory, BIA, classification, third-party mapping |
| Protect | Prevention | EDR, MFA, segmentation, immutable backup, patch, least privilege |
| Detect | Detection & Analysis | SIEM triage, IOC correlation, scope, severity declaration, war room activation |
| Respond | Containment → Eradication | Isolate → kill C2 → image forensics → rotate creds → reimage → patch |
| Recover | Recovery & Post-Incident | Restore from backup → validate integrity → enhanced monitoring → lessons learned → CAPA |

| Old (Rev 2, 4-phase) | New (Rev 3 / CSF 2.0) |
|---|---|
| Preparation | Govern + Identify + Protect |
| Detection & Analysis | Detect |
| Containment, Eradication, Recovery | Respond + Recover |
| Post-Incident Activity | Recover (lessons learned, CAPA) |

For the full IR framework — roles (CSIRT), severity matrix, containment strategies, eradication checklist, and post-incident deliverables — see [[incident-response-lifecycle-nist]].

### Running a forensic investigation — the order of volatility

The single most important rule: **capture the most volatile evidence first.** If you capture disk before RAM, you lose encryption keys, running malware, network connections, and process state.

| # | Source | Lifetime | Tool | Why first |
|---|---|---|---|---|
| 1. | **CPU registers, cache** | Nanoseconds | Hardware debugger, JTAG | Current instruction, decoded data |
| 2. | **RAM** (network state, process table, kernel) | Seconds to minutes | LiME, winpmem, volatility, memprocfs | Encryption keys, malware in-memory, active connections |
| 3. | **Network state** (routing, ARP, flows) | Seconds to minutes | `netstat`, Zeek, netflow dump | Active C2, lateral movement paths |
| 4. | **Running processes, mounted volumes** | Minutes | EDR process tree, `/proc` | Malware still running, encrypted volumes |
| 5. | **Temporary filesystems** (tmp, swap, pagefile) | Minutes to hours | `tar`, custom scripts | Deleted-but-recoverable artifacts |
| 6. | **Disk / persistent storage** | Years | `dd`, FTK Imager, EnCase, X-Ways | Full forensic image |
| 7. | **Backup media / archives** | Persistent | Mount + image | Historical state, prior-compromise timeline |
| 8. | **Remote logs** (SIEM, cloud audit) | Per retention | Vendor API export | Corroborating evidence, timestamps |

**CISSP trap:** in a modern encrypted environment, you cannot image the disk without the decryption key, which lives in RAM. RAM becomes priority #1 in practice. Plan for live-system forensics as the default — see [[digital-forensics-process]] for live vs dead-system trade-offs, ACPO principles, chain-of-custody form, and tool validation.

### Running a DR test — the hierarchy

| # | Test type | Disruption | What it proves | Frequency |
|---|---|---|---|---|
| 1 | Checklist / document review | None | Plan is current and complete | Quarterly |
| 2 | Walkthrough / tabletop | None | People understand their roles | Bi-annual |
| 3 | Simulation / functional exercise | Low | Specific system recovers in isolation | Annual |
| 4 | Parallel test | Low (DR runs alongside prod) | DR site operational, data integrity holds | Annual |
| 5 | Cutover test | Medium (brief switch to DR) | Full failover works, RTO achievable | Annual |
| 6 | Full interruption test | High (production off) | End-to-end under real conditions | Rare (exec sign-off) |

A program that has never tested at level 4+ has Schrödinger's DR: simultaneously functional and broken, and you won't know which until the real disaster. See [[disaster-recovery-and-bcp]] for full activation sequence.

### Change management execution — the RFC lifecycle

```
RFC → Assess → CAB Review → Approval → Build → Test → Implementation Window → Deploy → Post-Change Validation → Close (PIR)
  │        │         │                                                                                      │
  └── Risk assessment ── Back-out plan required ──────────────────── Post-implementation review (PIR) ──────┘
```

| Gate | Decision | Output |
|---|---|---|
| RFC submitted | Change type (standard/normal/emergency/major) | Ticket with business case, risk class, scope |
| CAB review | Approve / Reject / Defer | CAB minutes with sign-off |
| Implementation window | Calendar slot, blackout dates avoided | Maintenance window confirmed |
| Back-out plan | Tested and approved rollback | Rollback artifact, procedure, time budget |
| Post-change validation | Smoke test, KPI comparison to pre-change baseline | Pass/fail → close or rollback |

**Emergency changes** fast path: IC declares → ECAB convened (<15 min) → risk-accept signed → deploy (<30 min) → retroactive PIR (<24h). A change with no RFC is, by definition, unauthorized. See [[change-and-configuration-management]] for ITIL types, CAB roles, SoD enforcement, and drift detection.

**Change correlation during IR:** when a Sev-1 fires, the first question is "what changed?" A SOC with a well-maintained CMDB answers this in seconds via `alert → deploy timestamp → change → RFC → approver`. Without it, the SOC hunts through Slack and memory.

## Mastery

### Threat hunting — three hypothesis types

| Type | Hypothesis source | Example | Output |
|---|---|---|---|
| **Intelligence-based** | Threat intel, ISAC, vendor bulletins | "APT29 uses T1003.001 (LSASS dump) — are we catching it?" | Detection gap → new rule |
| **Anomaly-based** | UEBA, statistical outlier, peer-group deviation | "Why is this helpdesk account accessing the finance DB?" | Investigation → insider alert |
| **TTP-based** | ATT&CK technique, red team findings, purple team | "If an attacker uses T1021.002 (SMB admin shares), do we see it?" | Coverage validation → detection content |

Hunt cycle: hypothesis → data validation → query → findings → detection rule → feedback to threat intel. A mature SOC runs weekly hunts; world-class has dedicated T3 hunters. See [[siem-soar-and-detection-engineering]] for the hunt-to-detection pipeline.

### Detection engineering as software engineering

| Practice | Traditional "rules in a UI" | Detection-as-code |
|---|---|---|
| Version control | None (UI save overwrites) | Git, PR review, signed commits |
| Testing | "Deploy and see" | CI pipeline: Sigma → replay against historical data → FP/TP report |
| Rollback | Manual, error-prone | Git revert, automated deployment |
| Ownership | Unknown | Git blame, CODEOWNERS |
| Audit | Log of UI clicks (unreliable) | Immutable Git history |
| Stale rules | Forgotten, accumulate noise | Quarterly review gate, auto-FP-threshold retirement |

The Sigma rule format is the vendor-neutral source of truth — it compiles to SPL, KQL, Lucene, etc.

### SOAR playbooks — the automation layers

| Layer | Example | Maturity |
|---|---|---|
| Alert enrichment | Pull asset owner from CMDB, IP from threat intel, user from IdP | Minimum viable |
| Automated triage | Close known-FP with reason, auto-populate ticket fields | Level 2 |
| Semi-automated response | SOAR proposes containment; analyst approves with one click | Level 3 |
| Fully automated response | SOAR isolates host + disables user + blocks IOC without human | Level 4 (requires confidence) |
| Case management + metrics | End-to-end incident lifecycle tracking, time-per-phase reporting | Level 5 |

Most valuable playbook #1: phishing triage. #2: identity compromise. See [[siem-soar-and-detection-engineering]] for full playbook workflow.

### Forensic anti-forensics — the adversary's toolkit

| Technique | What the adversary does | Detection / counter |
|---|---|---|
| **Timestomping** | Modify MAC times (`$STANDARD_INFORMATION`) | Cross-correlate with `$FILE_NAME` (MFT), kernel timestamps |
| **Log tampering** | `wevtutil cl`, delete `auth.log`, inject false events | Forwarder-based centralization, gap detection, cross-source correlation |
| **Encryption** | Encrypt data at rest; encrypt C2 traffic | Memory dump → recover keys; lawful access where applicable |
| **Steganography** | Embed data in images, audio, slack space | Entropy analysis, stegdetect, file-size anomalies |
| **Slack space / ADS** | Hide data in NTFS alternate data streams, unallocated | `streams.exe`, forensic tools with ADS awareness |
| **Packer / obfuscation** | UPX, VMProtect, custom packers | Memory analysis (`malfind`), entropy analysis, sandbox detonation |
| **Rootkit / bootkit** | MBR infection, kernel hooking | TPM measured boot, Secure Boot, offline RAM acquisition |

See [[digital-forensics-process]] for the full counter-measure matrix.

### Ransomware negotiation considerations — the OFAC advisory

> ⚠️ OFAC advisory (2024 update): paying a sanctioned entity is a federal violation. OFAC maintains a public list of sanctioned threat actors. Ignore this at your peril.

| Consideration | Detail |
|---|---|
| Sanctions check | Verify the actor is not on the OFAC SDN list *before* any payment discussion |
| Cyber insurance | Many policies now require pre-approval, sanctioned-actor attestation, and use of approved negotiation firms |
| Decryptor reliability | ~30–40% of third-party decryptors recover all data; test on non-critical files first |
| Paying as repeat-target mark | Paying once increases probability of being targeted again (attackers share "payer" lists) |
| NoMoreRansom project | Free decryptors for known families — check before paying |
| Legal exposure | SEC: 4 business days disclosure. GDPR: 72h. NIS2: 24h early warning + 72h notification |

The decision to pay is a C-suite + board + legal decision, not a CISO decision. See [[ransomware-response-playbook]] for the full decision tree, payment flow, and communication matrix.

## Resilience

### Top operations mistakes — and how to avoid them

| Mistake | Consequence | Fix |
|---|---|---|
| **No playbooks** | Every incident improvised; IR takes 3× longer | Write and rehearse playbooks for top-5 scenarios |
| **Alert fatigue** | Analysts rubber-stamp FPs; real alert missed | Tune/retire top-10 noisy rules quarterly |
| **No after-hours escalation** | Sev-1 sits 8h because "no one was on call" | 24×7 on-call rotation with defined escalation policy |
| **Backup not tested** | Restore fails during real incident | Quarterly restore drill, hash-verify |
| **DR plan not tested** | Failover takes 4× RTO target | Parallel + cutover tests annually |
| **Change freezes accumulating risk** | 6 months of unfixed CVEs "because freeze" | Exemption process for security patches during freeze |
| **Mixing production and corporate networks** | Phishing credential → lateral to prod in minutes | Network segmentation, separate identity domains |
| **No asset inventory during IR** | Cannot scope the incident ("what did they touch?") | CMDB synced to SIEM, auto-enriched in SOAR |

### When the SOC burns out analysts

Signs: MTTA climbing, senior analysts leaving, 100% alert-close-as-FP, no one volunteers for hunt. Root causes: alert volume > human capacity, no career progression (T1 forever), shift-work health impact, no psychological support after traumatic incidents. Fix: automate Tier 0 aggressively, create T1→T2→T3 career path, rotate off-shift every 6 months, provide mental health resources. Burnout is cheaper to prevent than to recover from.

### When DR fails — the assumptions that kill

| Assumption | Reality |
|---|---|
| "People can access the building" | Fire, flood, civil unrest, pandemic — building is inaccessible. Have a remote DR runbook. |
| "The DR site is patched" | DR site often trails by months. Same baseline, same EDR, same patching cadence as primary. |
| "The runbook is on the DR site" | If the doc explaining how to fail over is on the failed-over system — you cannot read it. Store runbooks separately. |
| "We tested last year" | Environments drift. Networks change. People leave. Test at least annually. |
| "The vendor handles it" | Vendor DR SLA is not your DR capability. Validate independently. |

### When forensics destroys evidence

| Mistake | What it destroys | Fix |
|---|---|---|
| **Booting the system** | Overwrites pagefile, clears RAM, updates timestamps | Never boot. Image first. |
| **Not imaging RAM first** | Loses encryption keys, running malware, network state | Acquire RAM before disk |
| **Mounting disk read-write** | Timestamps change, $LogFile overwritten | Always use write-blocker |
| **Not logging actions** | Chain of custody broken; evidence inadmissible | Log every command, hash, time, examiner |
| **Waiting too long** | Volatile evidence gone before acquisition | Pre-staged forensic tooling on every endpoint |
| **Single hash** | No independent verification if collision alleged | Use two algorithms (SHA-256 + MD5 or BLAKE3) |

See [[digital-forensics-process]] for ACPO principles, chain-of-custody form, and tool validation.

## Context

### In-house SOC vs MSSP vs Hybrid

| Factor | In-house | MSSP | Hybrid |
|---|---|---|---|
| Control / IP retention | Full | Limited | Shared |
| Cost (3-year TCO) | $$$$ (headcount + tooling) | $$ (subscription) | $$$ |
| Context depth | Deep (your environment) | Shallow (their playbook) | Moderate |
| 24×7 coverage | Hard (6+ analysts per seat) | Default | Covered |
| Hiring burden | High | None | Moderate |
| Audit rights / exit | Full control | Must negotiate | Must negotiate |
| Best for | Large enterprise, regulated | SMB, compliance-driven | Mid-market scaling |

See [[security-operations-center-soc-models]] for detailed model comparison, staffing math, and MSSP SLA negotiation checklist.

### SOC in cloud-native environments

| Traditional SOC | Cloud-native SOC |
|---|---|
| Log sources: endpoints, network appliances, on-prem AD | Log sources: CloudTrail, Azure Activity, K8s audit, serverless, IdP |
| Response: isolate host on VLAN, block IP on FW | Response: revoke IAM role, terminate instance, block S3 public access |
| Asset inventory: CMDB, IP scan | Asset inventory: CSP inventory API, agentless, ephemeral |
| Detection: Windows/Linux process creation | Detection: IAM change, K8s pod creation, serverless invocation |
| Containment: network quarantine | Containment: IAM policy deny, security group removal, account isolation |

A cloud-native SOC needs different playbooks, runbooks, and hiring profiles.

### Physical security at scale — data center tiers

| Tier | Uptime % | Downtime/yr | Redundancy | Cost | Example |
|---|---|---|---|---|---|
| **Tier I** | 99.671% | 28.8 h | Single path, none | 1× | Small server room |
| **Tier II** | 99.741% | 22.0 h | N+1 power/cooling | 1.5× | Corporate data center |
| **Tier III** | 99.982% | 1.6 h | Concurrently maintainable | 3× | Colocation facility |
| **Tier IV** | 99.995% | 0.4 h | Fault tolerant, 2N+1 | 5×+ | Trading, defense, critical infra |

Per ANSI/TIA-942 and Uptime Institute. The tier is a physical expression of RTO: a Tier II facility with a 15-minute RTO is an architectural mismatch. See [[physical-and-environmental-security]] for full perimeter layers.

### Military vs civilian IR — key differences

| Aspect | Military / Government | Civilian / Enterprise |
|---|---|---|
| Authority | Legal mandate, may be classified | Contract, policy, regulatory |
| Attribution | National priority; classified intel used | Usually not pursued beyond IOC extraction |
| Rules of engagement | Offensive / counter-operation possible | Defensive only |
| Evidence handling | Classified chain, SCIF environment | Standard ACPO/FRE chain |
| Disclosure | Restricted, may be classified | May be mandatory (SEC, GDPR, NIS2) |

### When to call law enforcement

Call LE when: crime involves federal jurisdiction (wire fraud, CFAA), stolen data crosses borders, regulatory duty to notify, or incident threatens public safety. Handle internally when: internal policy violation, minor data exposure, or privileged investigation (attorney-client). Pre-arrange a relationship with your local FBI/NCSC field office and maintain a cyber-insurance panel counsel.

## Nuance

### Incident vs event vs alert

| Term | Definition | Example |
|---|---|---|
| **Alert** | Notification from a system that something occurred | "SIEM rule 1042 fired: multiple failed logins from IP X" |
| **Event** | An observable occurrence in a system | "User bob authenticates from IP 10.0.0.1 at 14:03:22 UTC" |
| **Incident** | A violation or imminent threat of violation of security policy, acceptable use, or law | "Adversary has valid credentials, lateral movement confirmed, data exfiltrated" |

Every incident is made of events; some events generate alerts. Most alerts are not incidents. Triage discriminates which alerts represent incidents.

### Containment vs eradication — different phases, different goals

| Aspect | Containment | Eradication |
|---|---|---|
| Goal | Stop the bleeding — prevent further damage | Remove the root cause |
| Question | "How do we stop the adversary now?" | "How do we ensure they cannot return?" |
| Actions | Network isolate, disable account, block IOC, revoke session | Image forensics, rotate all creds, reimage hosts, patch CVE |
| Speed | Minutes (urgent) | Hours to days (thorough) |
| Risk of rushing | Adversary escalates, destroys, or goes quiet (still present) | Residual malware, missed backdoor, incomplete patches |

Most IR failures come from skipping containment (adversary still active) or skipping eradication (adversary returns next week). The phases are sequential.

### Chain of custody — why it matters and how to break it

Chain of custody is the unbroken, documented trail proving evidence was in authorized custody from acquisition to courtroom. It is the single most common reason evidence is excluded.

**How to break it (and thus how to prevent):** a hand-off without signature/timestamp; a gap in the log ("it was in the locker, trust us"); evidence in an unsecured location; hash mismatch between acquisition and presentation; the examiner cannot explain what tool was used or its validation status.

**The form:** Evidence ID → Description → Hash (2 algorithms) → Acquired by (name, badge) → Date/time (UTC) → Method (tool, version) → Storage location → Released to → Released by → Returned/destroyed date. Print, sign, scan, store. Every row. Every time.

See [[digital-forensics-process]] for the full chain-of-custody form template and ACPO principles.

### Order of volatility — why CPU registers come first

The order reflects physical reality:

1. **CPU registers/cache** — nanoseconds. Current instruction and operands in-flight. Lost the moment the CPU executes the next instruction. Recoverable only with hardware debugger/JTAG (rarely practical but theoretically #1).
2. **RAM** — seconds to minutes. Process memory, network connections, encryption keys, clipboard. This is the most valuable layer in practice.
3. **Temp filesystems** — minutes to hours. Deleted-but-not-overwritten files in `/tmp`, swap, pagefile.
4. **Disk** — persistent. Full forensic image.

The rule: **if you have to choose between volatility and completeness, volatility first — disk can be re-imaged, RAM cannot.**

### DR test vs BCP exercise — different objectives

| Aspect | DR Test | BCP Exercise |
|---|---|---|
| Focus | Technology recovery | People and process continuity |
| Question | "Can we restore the ERP within 4 hours?" | "Can the finance team process payroll from the alternate site?" |
| Measures | RTO achieved, RPO achieved, data integrity | Decision latency, communication effectiveness, process completion |
| Participants | IT, ops, SRE, vendors | Business process owners, executives, HR, comms |
| Output | Technical runbook updates | BCP document updates, training gaps |
| Auditor looks for | Test evidence, RTO/RPO trend, remediation | Exercise attendance, after-action report, CAPA |

Confusing them means one or both are not being done properly.

## Resources

### IR playbook template (minimum viable)

| Section | Content |
|---|---|
| Detection triggers | List of alerts/reports that initiate the playbook |
| Declaration authority | Who can declare Sev-1/2/3 and how |
| Roles (RACI) | Incident Commander, Scribe, Legal, Comms, Technical Lead |
| Severity matrix | Sev-1/2/3/4 definitions and response times |
| Containment playbook | Per-scenario: network isolate, account disable, full shutdown |
| Eradication checklist | Image → hash → IOC push → cred rotation → reimage → patch → validate |
| Recovery playbook | Restore from backup → validate integrity → staged re-introduction |
| Communications matrix | Who tells whom, when, through what channel |
| Post-incident | Timeline, RCA, lessons learned, CAPA, runbook update |
| Escalation path | If IC unavailable → X; if CISO unavailable → Y |

### DR test schedule template

| Quarter | Test type | Systems | RTO target | Participants | Evidence |
|---|---|---|---|---|---|
| Q1 | Tabletop (ransomware) | All Tier-1 | — | IC + exec + DR coordinator | Attendance, AAR |
| Q2 | Parallel test | ERP, CRM | 4h | Ops + DR site team | RTO achieved, issues log |
| Q3 | Cutover test | Email, IdP | 2h | Ops + SRE | Failover log, DNS cut |
| Q4 | Full interruption (optional) | All Tier-1 | Per BIA | All + exec sign-off | End-to-end timeline |

### SOC tier responsibilities matrix

| Function | Tier 0 (Auto) | Tier 1 | Tier 2 | Tier 3 | Tier 4 |
|---|---|---|---|---|---|
| Alert acknowledge | Auto-ack | Manual ack | — | — | — |
| Enrichment | Auto-pull CMDB, TI | Manual context | — | — | — |
| Triage (FP/TP) | Auto-close known FP | Classify FP/TP, escalate | Review escalations | — | — |
| Investigation | — | — | Deep analysis, scoping | Complex, multi-host | — |
| Containment | Auto-isolate (pre-approved) | — | Manual containment | Custom containment | Crisis decision |
| Forensics | — | — | Host forensics | Memory, malware, ATT&CK | — |
| Threat hunting | — | — | — | Hypothesis-driven | — |
| Detection engineering | — | — | — | Author rules | Approve rules |
| Incident command | — | — | Sev-3 IC | Sev-2 IC | Sev-1 IC |
| Reporting | Auto-metrics | Ticket close | RCA | Hunt reports | Board reports |

### Change management workflow diagram

```
                    ┌─ Standard change ──── Pre-approved ──── Deploy ──── Close
                    │
RFC Submitted ──────┼─ Normal change ────── Assess ── CAB ── Approve ── Build ── Test ── Deploy ── Verify ── Close (PIR)
                    │
                    └─ Emergency change ─── ECAB (<15m) ── Risk-accept ── Deploy ── Retro-PIR
```

### Physical security checklist

| Layer | Check | Frequency |
|---|---|---|
| Perimeter | Fence integrity, gate operation, bollard condition | Monthly |
| Lighting | All fixtures functional, motion sensors tested | Monthly |
| CCTV | All cameras streaming, NTP sync, retention verified | Monthly |
| Access control | Badge readers, mantrap interlock, biometric performance | Quarterly |
| Visitor management | Log audit, NDA compliance, escort policy | Quarterly |
| Server room | Two-person rule, cabinet locks, environmental sensors | Monthly |
| Power | UPS battery test, generator full-load test, fuel level | Monthly (gen annual full-load) |
| Fire suppression | Clean agent cylinder weight, pre-action valve, smoke detector | Semi-annual |
| HVAC | Temp/humidity logging, leak detection test, filter replacement | Monthly |
| Badge audit | Who has access, still needs it, separation of duties | Quarterly |

<!-- END EXPANDED -->

## CISO / Risk Manager View

For the board, Domain 7 is best framed as **"the runbook that decides whether a bad day becomes a quarter or a line item."** Three board-level questions and the answers that earn trust:

1. **"How fast do we see an attack?"** → Track **MTTD** and **MTTA** as primary KPIs. Industry elite (per Mandiant M-Trends) is under 24 hours; many firms still measure in months.
2. **"How fast do we stop it?"** → Track **MTTR** and **containment-to-eradication ratio**. A good IR plan halves the cost of an incident (IBM Cost of a Data Breach 2024: $5.46M average, USD 1.5M savings for IR + AI tested teams).
3. **"Is our spend on ops proportional to the loss we are avoiding?"** → Compute the **expected loss reduction** vs. **SOC + tooling cost**:

$$ROI_{ops} = \frac{ALE_{no\ ops} - ALE_{with\ ops} - Cost_{ops}}{Cost_{ops}}$$

If the SOC costs $4M/yr and reduces expected loss by $30M/yr, $ROI = (30 − 4 − 4)/4 = 5.5x$ — a defensible board number.

**Hard truths for the CISO:**

- **The most expensive control is the one nobody uses.** A SIEM that fires 5,000 alerts/day and is triaged by one analyst is theater, not defense.
- **Tabletop exercises pay back in months.** The SANS "Security Operation Center" survey consistently finds that tested IR plans are the single biggest differentiator in breach cost.
- **DR is not a backup product; it is a recovery capability.** Backups that have never been restored are Schrödinger's data.
- **Ransomware is now an executive liability question.** Boards that have not rehearsed a ransomware scenario are betting the company on luck — see [[ransomware-response-playbook]].

## Related Connections

### L2 deep-dives (children)
- [[incident-response-lifecycle-nist]] — NIST SP 800-61 lifecycle, roles, severity matrix
- [[digital-forensics-process]] — order of volatility, chain of custody, ACPO
- [[change-and-configuration-management]] — CAB, baselines, ITIL
- [[disaster-recovery-and-bcp]] — BIA, RTO/RPO/MTPD, hot/warm/cold sites
- [[physical-and-environmental-security]] — perimeter, CCTV, HVAC, fire, TEMPEST

### L3 edge-cases / playbooks (children)
- [[ransomware-response-playbook]] — payment decision tree, recovery
- [[backup-strategies-3-2-1]] — 3-2-1-1-0, immutable, air-gapped
- [[siem-soar-and-detection-engineering]] — Sigma, MITRE ATT&CK, use cases
- [[security-operations-center-soc-models]] — tier 1/2/3, in-house vs MSSP, OODA, metrics

### Cross-domain
- [[domain-01-security-and-risk-management]] — risk register drives RTO/RPO/MTPD targets
- [[domain-02-asset-security]] — asset inventory is the SOC's source of truth
- [[domain-04-communication-and-network-security]] — NDR, NGFW, DNS, VPN telemetry
- [[domain-05-identity-and-access-management]] — PAM, JIT, MFA enforcement in ops
- [[domain-06-security-assessment-and-testing]] — vuln findings feed patch pipeline

## References

- (ISC)² CISSP CBK, 5th ed. — Domain 7
- NIST SP 800-61 Rev. 2 — *Computer Security Incident Handling Guide* (withdrawn April 2025; superseded by Rev 3)
- NIST SP 800-61 Rev. 3 — CSF 2.0 Community Profile (April 2025, replaces Rev 2; aligned to Govern/Identify/Protect/Detect/Respond/Recover functions)
- NIST SP 800-184 — Guide for Cybersecurity Event Recovery
- NIST SP 800-34 Rev. 1 — Guide to Test, Training, and Exercise Programs for IT Plans
- NIST SP 800-88 Rev. 1 — Guidelines for Media Sanitization
- NIST CSF 2.0 — Govern / Identify / Protect / Detect / Respond / Recover
- ISO/IEC 27035-1:2016 / 27035-2:2016 — Information security incident management
- ISO/IEC 27037:2012 — Guidelines for identification, collection, acquisition, preservation of digital evidence
- ISO 22301:2019 — Business continuity management
- ITIL 4 — Change Enablement, Incident Management, Service Continuity
- MITRE ATT&CK Enterprise / ICS / Mobile
- CIS Critical Security Controls v8 (esp. 1, 5, 6, 13, 17, 19)
- SANS PICERL / NIST lifecycle reconciliation
- ACPO Good Practice Guide for Digital Evidence (UK, 2012)
- Mandiant M-Trends (annual dwell-time benchmark)
- IBM Cost of a Data Breach Report (annual)
