---
title: "Business Continuity and Disaster Recovery"
layer: 2
domain: 1
tags:
  - cissp
  - domain-01
  - bcp
  - drp
  - bia
  - rto
  - rpo
  - mtpd
  - continuity
related_domains:
  - "[[risk-management-frameworks]]"
  - "[[risk-formulas-and-quantitative-analysis]]"
  - "[[security-governance-policies-standards]]"
  - "[[legal-regulatory-compliance-landscape]]"
  - "[[gdpr-deep-dive]]"
  - "[[hipaa-security-rule]]"
---

# Business Continuity and Disaster Recovery

## Feynman Explanation

Business Continuity is the company's playbook for "what if we can't do business for a while?" Disaster Recovery is the IT-specific chapter of that playbook. The whole point is to know, in advance, how long the business can survive without a system, how much data we can afford to lose, and what the cost of downtime is per hour. Without this math, in a real incident, executives argue in a conference room for 4 hours before anyone is allowed to make a decision.

## Technical Details

### BCP vs DRP vs Continuity of Operations

| Document | Scope | Question It Answers |
|---|---|---|
| **BCP** (Business Continuity Plan) | Entire business, all functions | "How do we keep the business running?" |
| **DRP** (Disaster Recovery Plan) | IT systems, data, infrastructure | "How do we restore IT after a disaster?" |
| **COOP** (Continuity of Operations Plan) | Government/military term, similar to BCP | "How do we maintain mission-critical functions?" |
| **OEP** (Occupant Emergency Plan) | Personnel safety in a facility | "How do we evacuate the building?" |
| **CMP** (Crisis Management Plan) | Communications, leadership during crisis | "Who speaks, who decides, who goes public?" |
| **IRP** (Incident Response Plan) | Security incidents | "How do we handle a breach?" |

The BCP is the umbrella; DRP is one component of it. CISSP exam question: "DRP is part of what?" → **BCP**.

### NIST SP 800-34 - BCP/DRP Lifecycle

| Phase | Output |
|---|---|
| 1. Project Initiation | Charter, scope, budget |
| 2. Business Impact Analysis (BIA) | RTOs, RPOs, MTPDs, financial impact |
| 3. Recovery Strategy | Site type, backup type, staffing |
| 4. Plan Development | Written BCP and DRP |
| 5. Testing & Exercises | Tabletop, simulation, parallel, full interruption |
| 6. Plan Maintenance | Annual review, change management |
| 7. Awareness & Training | Cross-functional training |

### Business Impact Analysis (BIA)

The BIA is the **single most important artifact** in BCP. It identifies:

- **Critical business functions** — which ones stop the business?
- **Dependencies** — what systems, people, vendors do they need?
- **Outage impact** — what is the cost per hour of downtime?
- **Timeframes** — RTO, RPO, MTPD, MTTR

#### Key Time Variables

| Term | Definition | Example |
|---|---|---|
| **MTPD** (Maximum Tolerable Period of Disruption) | Total time business can survive without the system | 24 hours |
| **RTO** (Recovery Time Objective) | How long until system is *back online* (at minimum acceptable level) | 4 hours |
| **RPO** (Recovery Point Objective) | How much data we can afford to lose, measured in time | 1 hour |
| **MTTR** (Mean Time To Repair) | Average time to restore after a failure (not a BIA term, but related) | 30 min |
| **MTBF** (Mean Time Between Failures) | Reliability metric | 10,000 hours |

**Key relationships:**

$$MTPD \geq RTO + RPO\ (typically)$$

- RPO controls **backup frequency** — hourly backups = 1-hour RPO.
- RTO controls **recovery infrastructure** — hot sites = minutes RTO; cold sites = days RTO.

#### Recovery Cost vs Downtime Cost

The whole point of the BIA is to match recovery investments to actual business impact:

| System Tier | MTPD | RTO | RPO | Recovery Site Type |
|---|---|---|---|---|
| Tier 1 - Mission Critical | Minutes to hours | < 1 hour | Minutes | Hot site, active/active |
| Tier 2 - Business Critical | Hours | 4-24 hours | 1-4 hours | Warm site |
| Tier 3 - Operational | 1-3 days | 24-72 hours | 4-24 hours | Cold site |
| Tier 4 - Administrative | > 1 week | > 72 hours | 24+ hours | Cold site / no DR |

### Continuity Strategies (IT Recovery Sites)

| Site Type | Cost | RTO | Data Currency | Use Case |
|---|---|---|---|---|
| **Hot site** | High ($$$) | Minutes to hours | Real-time or near-real-time replication | Trading, healthcare, Tier 1 |
| **Warm site** | Medium ($$) | Hours to a day | Periodic replication (hourly/daily) | Tier 2 business apps |
| **Cold site** | Low ($) | Days to weeks | Restore from backup on site activation | Tier 4, archive only |
| **Mobile site** | Variable | Hours to days | Bring hardware, restore data | Short-term disaster response |
| **Cloud DR (DRaaS)** | Variable | Minutes to hours | Snapshots + replication | Modern default |
| **Reciprocal agreement** | Free | Variable | Varies | Almost never actually works |

**CISSP trap:** Reciprocal agreements are historically a bad idea (your disaster is probably theirs). Use real DR infrastructure.

### Backup Strategies

| Type | Storage Used | Recovery Speed | Cost | Best For |
|---|---|---|---|---|
| **Full backup** | All data | Fast restore | High storage, slow backup | Baseline |
| **Incremental** | Only changes since *last backup* (full or inc) | Slow restore (need full + all increments) | Low storage, fast backup | Daily cadence |
| **Differential** | Changes since *last full backup* | Faster restore (need full + last diff) | Medium storage | Common middle ground |
| **Synthetic full** | Server synthesizes a full backup from inc/diff | Fast restore | Medium-high compute | Modern enterprise |
| **Snapshot** | Point-in-time state copy | Very fast (just mount) | Disk space | VMs, databases |
| **Continuous Data Protection (CDP)** | Captures every change | Any-point-in-time recovery | High | Critical data |

**Restore time vs backup time trade-off:**

- Incremental = fast backup, slow restore.
- Differential = slower backup, faster restore.
- Full = slowest backup, fastest restore.

**The 3-2-1 Rule:**

- **3** copies of data
- **2** different media types
- **1** offsite (different geographic region)

Modern version: **3-2-1-1-0** (add 1 immutable, 0 errors on verification).

### DR Testing Types (in order of severity)

| Type | Description | Disruption Risk |
|---|---|---|
| **Document review / Read-through** | Walk through the plan on paper | None |
| **Tabletop exercise** | Conference room scenario, key players talk through response | None |
| **Walk-through / Simulation** | Simulated, role-played scenario in real-ish environment | Low |
| **Parallel test** | Recover at DR site, run alongside production | Low |
| **Cutover test** | Briefly switch production to DR site | Medium |
| **Full-interruption test** | Shut down production, force DR activation | **HIGH** - rarely used |

**CISSP test tip:** First test = document review. Full interruption test = last. Always move from simple to complex.

### Succession Planning

- **Senior management succession** — who takes over if the CEO is unavailable?
- **Technical succession** — who has the root password / recovery keys?
- **Delegation of authority** matrix — who can sign what, up to what limit, when the primary is unavailable.

This is one of the most overlooked BCP components and a common exam topic.

### Insurance

- **Business Interruption Insurance** — covers lost revenue during outage
- **Extra Expense Insurance** — covers the cost of the recovery (hot site, contractors)
- **Cyber Insurance** — covers breach response, fines (sometimes), ransom (debatable)
- **Errors & Omissions (E&O)** — covers professional liability
- **Property Insurance** — physical assets, usually excludes data

**Insurance does NOT replace a BCP.** It is a financial backstop, not a continuity strategy.

## CISO / Risk Manager View

**The board conversation:**

> "We can lose $X per hour of downtime. Our top 5 systems account for $Y per hour. A hot site for those 5 systems costs $Z per year. Payback: 1 disaster event."

**Hard truths for the CISO:**

1. **A BCP that has not been tested is a marketing document.** Run a tabletop at least once a year. Full interruption tests are rarely practical, but a "switchover to DR for 4 hours, switch back" is doable in most cloud environments.
2. **RTO/RPO are business decisions, not IT decisions.** IT proposes options, business sets the target, IT delivers. If business says "we need 4-hour RTO" and IT can only do 24 hours, that is a risk register item.
3. **Cyber insurance is not a substitute.** Policies are full of exclusions (act of war, state actors, your own negligence). Read the fine print. Many policies no longer pay ransoms or fines.
4. **The cloud changed everything, but did not eliminate the problem.** Cloud DRaaS is faster and cheaper than old-school hot sites, but vendor lock-in, data egress fees, and shared-responsibility gaps are new risks.
5. **Third-party risk IS your risk.** Your BCP is only as good as your critical vendors' BCPs. Include vendor resilience reviews in your program.
6. **Test the people, not just the technology.** Most BCP failures in real incidents are about who can make decisions, not whether the backups work.

**Top KPIs for the board:**

- **RTO/RPO achievement rate** in last test
- **Number of untested critical systems**
- **Time since last tabletop exercise**
- **Vendor resilience coverage** of Tier 1 systems
- **Cyber insurance limit vs. annual loss exposure** (ALE)

## Related Connections

### Sibling L2
- [[risk-management-frameworks]] - BIA feeds the risk register
- [[risk-formulas-and-quantitative-analysis]] - Downtime cost is an SLE/ALE input
- [[security-governance-policies-standards]] - BCP/DR Policy and Procedures live here
- [[legal-regulatory-compliance-landscape]] - SOX, HIPAA, GLBA all require BC/DR

### L3
- [[gdpr-deep-dive]] - 72-hour breach notification depends on IR/BC plans
- [[hipaa-security-rule]] - Contingency Plan is an Administrative Safeguard

### Cross-Domain
- [[domain-03-security-architecture-and-engineering]] - Redundancy, RAID, geographic diversity
- [[domain-07-security-operations]] - DR activation, IR, ops recovery
- [[domain-08-software-development-security]] - Resilience baked into SDLC (chaos engineering, etc.)

## Sources / References

- NIST SP 800-34 Rev. 1 - Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities
- NIST SP 800-184 - Guide for Cybersecurity Event Recovery
- ISO 22301:2019 - Security and resilience — Business continuity management systems
- ISO/IEC 27031:2011 - Guidelines for ICT readiness for business continuity
- DRII (Disaster Recovery Institute International) - Professional Practices
- BCI (Business Continuity Institute) - Good Practice Guidelines
- (ISC)² CISSP CBK, 2024
- FFIEC BCP Examination Handbook
- AWS Well-Architected Framework - Reliability Pillar
