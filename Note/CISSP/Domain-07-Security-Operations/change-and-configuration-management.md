---
title: Change and Configuration Management
layer: 2
domain: 7
tags: [cissp, domain-7, change-management, configuration-management, cab, baselines, itil, cmdb, sccm, ansible, iac]
related_domains: [2, 4, 5, 6, 7]
---

# Change and Configuration Management

## Feynman Explanation

Change management is the bouncer at the door of production: every change to a system, network, or application has to be requested, reviewed, approved, tested, and recorded before it goes live, so that when something breaks at 3 a.m. the on-call engineer knows exactly what changed and can roll it back. Configuration management is the photo album of what every system is supposed to look like — the golden image, the baseline, the drift detector — so that the bouncer can spot a system that no longer matches the album. Together, they turn production from "a pile of snowflakes" into a fleet you can defend, audit, and rebuild.

## Technical Details

### Why both?

| Discipline | Question it answers |
|---|---|
| **Change Management** | "Who changed what, when, why, and was it approved?" |
| **Configuration Management** | "Is this system configured the way it should be, right now, and how do I prove it?" |

They are inseparable: change management without configuration management cannot detect unauthorized drift; configuration management without change management cannot explain why the system differs from the baseline.

### ITIL 4 change management practice

| Type | Definition | Approval | Lead time | Example |
|---|---|---|---|---|
| **Standard change** | Low-risk, pre-approved, repeatable | Pre-authorized model | Hours | Add user to group, restart service |
| **Normal change** | Requires assessment + CAB | CAB review | Days to weeks | New firewall rule, new server, app deploy |
| **Emergency change** | Restore service or fix critical vuln | ECAB (small, fast) | Hours | Patch zero-day, hotfix, incident-driven |
| **Major change** | High impact / cost / risk | CAB + exec sponsor | Weeks | New data center, ERP upgrade |

### Change Advisory Board (CAB)

| Role | Responsibility |
|---|---|
| **Change Manager** | Runs the CAB, owns the change calendar |
| **CAB Chair** | Senior leader (often IT director) who signs off |
| **Requester** | The engineer / team proposing the change |
| **Technical reviewer** | Subject-matter expert for the affected system |
| **Security reviewer** | Security architect / GRC sign-off (mandatory for risk-bearing changes) |
| **Operations / SRE** | Capacity, runbook, on-call impact |
| **Business sponsor** | Authority for the business outcome the change enables |

**CISSP tip:** "Who is the change manager?" — not the CIO, not the CISO. The change manager is the process owner. The CIO/CISO can be a CAB member, but a separate role is required.

### The change lifecycle (ITIL)

```
Request → Assess → Approve → Plan → Build → Test → Schedule → Deploy → Verify → Close
   │                          │                                              │
   └─── RFC (Request for Change) ────── PIR (Post-Implementation Review) ─────┘
```

| Step | Output | Required artifact |
|---|---|---|
| Request | RFC logged | Ticket with business case, risk class, scope |
| Assess | Impact, resources, rollback | Risk assessment, back-out plan |
| Approve | CAB minutes | Approver list, signatures |
| Plan | Implementation plan | Test plan, comms plan, maintenance window |
| Build | Staging artifact | IaC / pipeline run / package |
| Test | Test results | Functional, security, performance |
| Schedule | Calendar entry | Maintenance window, blackout dates |
| Deploy | Change log | Pre/post snapshots, runbook |
| Verify | Acceptance | Smoke test, KPI diff |
| Close | PIR | What worked, what didn't, follow-ups |

### Configuration management building blocks

| Concept | Definition |
|---|---|
| **Configuration Item (CI)** | Any component that needs to be managed to deliver a service (server, app, doc, contract, vendor) |
| **CMDB** | Configuration Management Database — the system of record for CIs and their relationships |
| **Baseline** | Approved, version-controlled reference configuration (golden image, hardened OS, network config) |
| **Drift** | Any deviation from baseline detected on a managed asset |
| **Asset inventory** | Subset of CMDB focused on physical/virtual assets (cross-link to D2) |
| **Federation** | Linking multiple CMDBs (IT + security + facilities) for a unified view |

### Baseline management

| Type | Example | Tool |
|---|---|---|
| **OS image** | Hardened Windows 11, CIS-benchmark Ubuntu 24.04 | MDT, AutoPilot, Ansible, Packer |
| **Network device config** | ACLs, OSPF, BGP, VLANs | RANCID, Oxidized, Batfish |
| **Application config** | `application.yaml`, secrets in vault | Ansible, Chef, Puppet, Salt, Terraform |
| **Cloud config** | Account org, IAM, S3 bucket policy | AWS Config, Azure Policy, GCP Org Policy |
| **Database config** | User grants, parameter groups, audit policy | Native DB tooling |

**CISSP trap:** A "baseline" is a frozen reference, not a moving target. Every change to a baseline must itself go through change management.

### Automated CM / Infrastructure as Code (IaC)

| Tool | Layer | Style | State model |
|---|---|---|---|
| **Ansible** | Config mgmt | Procedural, agentless | Inventory |
| **Chef** | Config mgmt | Declarative Ruby DSL | Client-server |
| **Puppet** | Config mgmt | Declarative | Client-server |
| **Salt** | Config mgmt | Both | Master-minion |
| **Terraform** | IaC (provisioning) | Declarative HCL | State file |
| **Pulumi** | IaC | Code-first (TypeScript, Python, Go) | State file |
| **CloudFormation** | AWS IaC | JSON/YAML | Managed by AWS |
| **Packer** | Image build | Declarative template | Output image |
| **Kubernetes + Helm** | Container orchestration | Declarative | Cluster state |

**Pattern:** Terraform/Pulumi *provision*, Ansible/Chef *configure*, Packer *image*, Helm/Kustomize *app*. The four layers compose.

### Configuration drift detection

| Method | Tool | Frequency |
|---|---|---|
| Agent-based scan | Chef InSpec, OpenSCAP, CIS-CAT | Continuous / scheduled |
| Agentless scan | AWS Config, Azure Policy, GCP SCC | Continuous |
| Pull-based | Salt, Ansible pull | Hourly |
| Image diff | Tripwire, OSSEC, OSSEC-like | Daily |
| Network device | Batfish, Batfish-equiv, RANCID diffs | On commit |
| Runtime | Falco, Tracee, eBPF-based | Continuous |

A simple **drift detection rate** KPI:

$$Drift\ Rate = \frac{\#\ assets\ with\ unauthorized\ config\ diff}{\#\ assets\ scanned}$$

A 2024 industry benchmark for mature programs: drift rate < 2% on Tier-1 systems.

### Versioning, tagging, and change correlation

Every change carries:

| Field | Example |
|---|---|
| Change ID (CHG-…) | CHG-2026-07-0001 |
| CI affected | ci-12345, ci-67890 |
| Approver | CAB-2026-W27 |
| Deploy timestamp | 2026-07-08T14:00:00Z |
| Deployer | jenkins-deployer, change-requestor-uid |
| Git commit | a1b2c3d |
| Artifact hash | sha256:9f8e7d… |
| Rollback artifact | Same artifact @ previous tag |

If your SIEM is on fire with a Sev-1 at 14:05, you can pivot from alert → deploy → change → RFC → approver in under a minute. **Change correlation is the SOC's first line of investigation.**

### Separation of duties (SoD) in change management

| Role | Cannot also be |
|---|---|
| Requester | Approver (for own change) |
| Approver | Deployer |
| Deployer | Tester of own change |
| Operator (prod access) | Developer (code change) |
| Auditor (CMDB) | Any of the above |

SoD is enforced via RBAC (see [[domain-05-identity-and-access-management|D5]]) and via the change management workflow.

### Emergency change procedure (the "fast lane")

| Step | Time target | Who |
|---|---|---|
| 1. IC declares emergency + opens CHG | <5 min | IC + change manager on call |
| 2. ECAB convened (3-5 people, often a VP + CISO + SRE lead) | <15 min | ECAB |
| 3. Risk-accept note signed | Concurrent | ECAB chair |
| 4. Deploy in restricted window | <30 min | Implementer |
| 5. PIR + retroactive normal change ticket | <24 h | Change manager + requester |

Emergency changes **must** be retro-documented. A change that "happens" with no RFC is, by definition, unauthorized.

### Configuration management and the audit

Auditors (PCI, SOC 2, ISO 27001, SOX) look for:

| Control | Evidence |
|---|---|
| All production changes are approved | Change tickets with CAB sign-off |
| Changes tested before production | Test results, code-coverage report |
| Rollback plan exists and works | PIR + drill records |
| Baseline configs are version-controlled | Git repo, signed tags |
| Drift is detected and remediated | Drift tickets, mean time to remediate |
| SoD enforced | IAM role definitions + periodic access review |

## CISO / Risk Manager View

Change and configuration management is **"the boring backbone of a defensible security program."** A board that sees an unbroken chain of change tickets, tested baselines, and zero-drift Tier-1 systems should sleep well. A board that sees a flat CMDB, ad-hoc root SSH, and unsigned production changes should expect a breach.

**Board talking points:**

1. **80% of outages are change-related.** A healthy CAB + tested back-out is the single most cost-effective resilience investment. Gartner tracks "Configuration Management as a discipline" as a top-3 driver of MTTR for incidents.
2. **The CMDB is the SOC's brain.** Without a CMDB, the SOC is guessing which systems matter, which owners to call, which change to revert. Every alert without a CMDB row costs minutes — and minutes cost millions.
3. **Drift is a leading indicator of compromise.** Attackers change configs (disable AV, add user, open port) before they detonate. Continuous drift detection has caught many an intrusion before the payload ran.
4. **SoD is the cheapest control you will ever buy.** A CISO can implement SoD with a wiki + an RBAC change; the cost of a single insider-misuse case is hundreds of times more.
5. **Change correlation closes the loop.** When a Sev-1 fires, the first 5 minutes are about "what changed?" not "who is the adversary?" — make that answer a database query, not a Slack hunt.

A useful framing for the CFO:

$$Outage\ cost\ reduction\ from\ CM = \Delta Outages\ \times\ Cost\ per\ hour\ of\ downtime$$

If CM reduces outages by 30% and downtime is $200k/hour, even one avoided 4-hour outage per year covers a small CM platform's annual cost.

## Related Connections

### Sibling L2
- [[incident-response-lifecycle-nist]] — Emergency change invoked from IR; PIR feeds runbooks
- [[digital-forensics-process]] — Configuration timeline supports forensic RCA
- [[disaster-recovery-and-bcp]] — DR runbooks are versioned CIs
- [[physical-and-environmental-security]] — Physical changes (camera moves, door re-coring) are CIs

### L3 children
- (Change management is foundational; no L3 child currently — see [[siem-soar-and-detection-engineering]] for SIEM-driven change correlation)

### Cross-domain
- [[domain-02-asset-security]] — CMDB is the operational twin of the asset inventory
- [[domain-04-communication-and-network-security]] — Network device configs are a critical CMDB subset
- [[domain-05-identity-and-access-management]] — SoD, JIT, PAM are enforced in change workflow
- [[domain-06-security-assessment-and-testing]] — Vulnerability findings flow into the change pipeline

## References

- ITIL 4 — Change Enablement, Configuration Management, Release Management
- ISO/IEC 20000-1:2018 — Service management
- COBIT 2019 — Manage Change (BAI03), Manage Configuration (BAI04)
- NIST SP 800-128 — *Guide for Security-Focused Configuration Management of Information Systems*
- NIST SP 800-53 Rev. 5 — CM-2 (Baseline Configuration), CM-3 (Configuration Change Control), CM-6 (Configuration Settings)
- CIS Benchmarks — Community-driven OS, cloud, application hardening
- CIS Critical Security Controls v8 — Control 4 (Secure Configuration Management)
- (ISC)² CISSP CBK 5th ed. — Domain 7
- AWS Well-Architected — Operational Excellence Pillar
- "Accelerate" (Forsgren, Humble, Kim) — DORA metrics (deploy frequency, lead time, MTTR, change-fail rate) are the modern CM KPI suite
- Atlassian ITSM / ServiceNow change practice
- Open-source: Ansible, Puppet, Chef, Salt, Terraform, Pulumi, OpenSCAP, Chef InSpec, CIS-CAT
