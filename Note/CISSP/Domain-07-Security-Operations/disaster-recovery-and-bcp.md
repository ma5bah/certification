---
title: Disaster Recovery and Business Continuity (Operational View)
layer: 2
domain: 7
tags: [cissp, domain-7, dr, bcp, bia, rto, rpo, mtpd, draas, hot-site, warm-site, cold-site, cloud-dr, continuity]
related_domains: [1, 2, 4, 5]
---

# Disaster Recovery and Business Continuity (Operational View)

## Feynman Explanation

When the building burns, the region goes offline, or the cloud region is unavailable, the operations team has to bring the business back online from a known plan, on a known clock, with known data. Domain 7 owns the *running* of that plan: testing it, exercising it, failing it on purpose, and being ready to execute it at 2 a.m. Domain 1 owns the policy and the math (RTO, RPO, MTPD); Domain 7 owns the muscle memory. Without both, you have a binder that does not work, or muscle memory that is solving the wrong problem.

## Technical Details

### What lives in D1 vs. D7

| Artifact | D1 owner | D7 owner |
|---|---|---|
| BCP/DR policy and BIA numbers | ✓ (sets) | consumes |
| Business risk acceptance for downtime | ✓ (signs) | executes |
| DR runbooks, system-level | — | ✓ (builds, tests) |
| DR site, replication topology, network failover | — | ✓ |
| Failover drill, tabletop, full-interruption test | — | ✓ |
| DRaaS vendor management | partial | ✓ (operational) |
| Cyber insurance alignment | ✓ | ✓ (post-event) |

This note is the **operational** view. For the BIA / RTO / RPO math and policy framing, see [[business-continuity-and-disaster-recovery]] (D1).

### RTO / RPO / MTPD — the operational clock

| Term | Definition | Drives |
|---|---|---|
| **RTO** (Recovery Time Objective) | How long from "disaster declared" to "system back online" | Recovery site type, automation, runbook maturity |
| **RPO** (Recovery Point Objective) | Maximum acceptable data loss, in time | Backup frequency, replication mode |
| **MTPD** (Maximum Tolerable Period of Disruption) | The hard wall: business dies after this | RTO + RPO must fit inside MTPD |

The hard relationship:

$$RTO + RPO \le MTPD$$

Operational targets by system tier:

| Tier | RTO | RPO | MTPD | Implication |
|---|---|---|---|---|
| Tier 1 — Mission critical | Minutes | Seconds–minutes | Hours | Active/active, sync replication, runbook on a phone |
| Tier 2 — Business critical | 1–4 hours | 15 min – 1 h | 24 h | Warm site, async replication |
| Tier 3 — Operational | 24 h | 4–24 h | 3 days | Cold site, daily backup |
| Tier 4 — Administrative | 72+ h | 24+ h | > 1 week | Restore from tape, no DR site |

### Cost of downtime — the math

$$Cost_{outage} = \sum_{t=0}^{RTO} (RevenueRate_t + PenaltyRate_t + ReputationDelta_t) \cdot dt$$

Empirical inputs (rule-of-thumb, customize per industry):

| Industry | Hourly downtime cost (USD) | Source / note |
|---|---|---|
| Brokerage / trading | $4–8M | Per hour of halted trading |
| E-commerce (large) | $250k–$1M | Revenue + SLA penalties |
| Healthcare (large hospital) | $50k–$250k | Patient safety, regulatory |
| Manufacturing | $50k–$500k | Per line per hour |
| SaaS B2B | $10k–$200k | Per tenant SLA |
| Public sector / citizen services | Reputational + legal | Hard to quantify |

A simplified risk-based decision:

$$ALE_{dr} = ARO_{disaster} \times SLE_{disaster}$$

where $SLE_{disaster}$ = (hourly downtime cost) × (RTO) + (data loss) × (rebuild cost) + (regulatory penalty).

If the annualized disaster expectancy is, say, $5M (1-in-10-year event), and DR for Tier 1 costs $2M/year, the ROI is obvious.

### Recovery sites — operational comparison

| Site | Cost | RTO | RPO | Data currency | Operational reality |
|---|---|---|---|---|---|
| **Hot site** (active/passive) | $$$$ | Minutes | Seconds | Sync replication | Second site always running; fail over with a button |
| **Active/active multi-region** | $$$$ | Seconds | Near-zero | Sync replication | No fail-over; just route around the bad site |
| **Warm site** | $$$ | Hours | Minutes–hours | Async replication | Site exists, data is "warm," boot in hours |
| **Cold site** | $ | Days | Hours–day | Backup-restore | Empty rack + power; you bring everything |
| **Mobile site** | Variable | Hours | Hours | Bring hardware | Truck with servers + generator |
| **Cloud DR (DRaaS)** | $$ | Minutes–hours | Minutes | Snapshots + replication | Modern default; pay-per-use |
| **Reciprocal agreement** | Free | Variable | Variable | Varies | Almost never actually works under stress |

### Cloud DR patterns (modern)

| Pattern | RTO | RPO | Cost | When |
|---|---|---|---|---|
| **Backup & restore** (S3 / Glacier) | Hours | Hours | ¢ | Tier 3/4 |
| **Pilot light** (core running, scale on demand) | Tens of minutes | Minutes | $ | Tier 2 |
| **Warm standby** (full env at low utilization) | Minutes | Seconds | $$ | Tier 1/2 |
| **Multi-site active/active** | <1 min | <1 min | $$$ | Tier 1, global |

Reference: AWS Well-Architected Reliability Pillar, Azure Architecture Center DR guidance.

### Backup types and operational reality

| Type | Restore speed | Backup speed | Storage | Operational note |
|---|---|---|---|---|
| **Full** | Fastest | Slowest | 1× | Weekly baseline |
| **Incremental** | Slowest (need all + full) | Fastest | Lowest | Daily |
| **Differential** | Medium (full + last diff) | Medium | Medium | Daily, weekly full |
| **Synthetic full** | Fastest | Synthesized | Medium | Modern; dedup-aware |
| **Snapshot** | Fast (mount) | Fast (copy-on-write) | Disk | VMs, DBs |
| **CDP** (Continuous Data Protection) | Any point | Continuous | High | Mission critical |

**CISSP trap:** incremental = fast backup, slow restore. Differential = medium both. The 3-2-1 rule is the policy; the type is the technique.

### The 3-2-1-1-0 rule (operational stance)

| Component | Operational meaning |
|---|---|
| **3** copies | Production + 2 backup copies |
| **2** media types | Disk + tape, or disk + cloud |
| **1** offsite | Geographic separation (≥ 100 km, different power grid) |
| **1** immutable | WORM storage, object lock, air-gapped (see [[backup-strategies-3-2-1]]) |
| **0** errors | Last restore test returned zero integrity errors |

### DR test taxonomy (operational)

| Test | Disruption | Frequency | Owner |
|---|---|---|---|
| **Document review** | None | Quarterly | DR coordinator |
| **Tabletop** | None | Bi-annual | DR coordinator + IC + business |
| **Walkthrough / simulation** | None | Annual | DR coordinator + tech leads |
| **Parallel test** | Low (DR runs alongside prod) | Annual | Ops + DR site |
| **Cutover test** | Medium (brief switch) | Annual | Ops + business |
| **Full-interruption test** | High (prod off) | Rare | Ops + business + exec sign-off |

**Mature programs run parallel + cutover tests in production-like conditions at least annually**; the un-tested backup is the Schrödinger's data problem.

### DR activation criteria (decision tree)

A documented threshold — not a phone call — decides when to activate DR:

| Trigger | Action |
|---|---|
| Building inaccessible, no ETA < 4 h | Activate cold site / cloud DR |
| Region fully down (cloud) | Fail over to secondary region |
| Ransomware encrypted prod (irrecoverable in 4 h) | Activate DR + legal + comms |
| Single Tier-1 system down > 30 min, no RTO | Initiate Tier-1 recovery runbook |
| Vendor (cloud, ISP) outage > 1 h | Activate alternate path |

**CISSP tip:** "When to declare a disaster" is a board-level pre-authorized decision, not a 3 a.m. CIO phone call.

### Failover / failback — the operational sequence

| Step | Action | Time budget |
|---|---|---|
| 1. Declare disaster | IC + exec sponsor sign | <15 min |
| 2. Communicate | Internal + customer + regulator as required | <30 min |
| 3. Activate DR site | Bring up systems per runbook | ≤ RTO |
| 4. Verify data integrity | Hash compare, end-to-end test | <30 min |
| 5. Cut traffic (DNS, BGP, LB) | Route to DR | <15 min |
| 6. Monitor | Compare KPIs to baseline | Continuous |
| 7. Stand down primary | Forensic capture, then power down | Days |
| 8. Rebuild primary | Reimage, restore, test | ≤ MTPD |
| 9. Failback | Reverse the cut | Planned window |
| 10. Post-mortem | PIR + runbook updates | <2 weeks |

### DR and security (often forgotten)

| Risk | Mitigation |
|---|---|
| DR site less hardened than primary | Same baseline, same EDR, same patching cadence |
| DR credentials in a break-glass account | PAM-controlled, dual-control, time-limited |
| Replica inherits malware/ransomware | Immutability, isolation, restore-time AV scan |
| DR vendor = single point of failure | Multi-cloud or contractually required multi-region |
| Failover exposes management plane to internet | Bastion, MFA, just-in-time, IP allowlist |
| Forensics lost during fail-over | Capture before fail-over (memory, logs) |

### Succession planning (a board-level concern)

| Role | Backup | Authority delegation |
|---|---|---|
| CEO | COO or designated exec | Up to $X spend without board |
| CIO | Designated deputy | Activate DR up to Tier-2 |
| CISO | Designated deputy | Activate IR, emergency CVE patching |
| Ops lead | Designated deputy | Failover on declared criteria |
| Vendor primary contact | Secondary contract owner | Pre-arranged |

**CISSP exam tip:** A common question is "what is the first thing to do in a disaster?" — answer is **people safety**, then **business continuity**, then **IT recovery**. Domain 7 owns the third, but the second and first are the business's call.

## CISO / Risk Manager View

**"A backup is not a recovery; a recovery is not a DR plan; a DR plan is not a tested capability."**

The board conversation:

1. **We can name the cost of an hour of downtime for each Tier-1 system, in dollars.** That number drives the DR investment for that system.
2. **We have tested fail-over for every Tier-1 system in the last 12 months, with evidence.** We post the test results to the audit committee.
3. **Cyber insurance is a financial backstop, not a continuity strategy.** Most policies now exclude state-actor attacks, gross negligence, and the ransoms themselves. Read your policy.
4. **Cloud DR is faster and cheaper, but it has new failure modes.** Vendor lock-in, data egress, shared responsibility, IAM misconfiguration at the DR site.
5. **The DR test is the rehearsal for the real event.** If we have never cut DNS, we will not cut DNS well in a real event.
6. **Third-party DR matters as much as ours.** The DR posture of our top-10 vendors is in our vendor risk register.

A CISO's DR scorecard (board-friendly):

| Metric | Target | Red flag |
|---|---|---|
| % Tier-1 systems with tested fail-over in last 12 mo | 100% | <80% |
| Median RTO achievement vs. target | ≥ 100% | <90% |
| Median RPO achievement vs. target | ≥ 100% | <90% |
| Last tabletop | < 6 months | > 12 months |
| Cyber insurance limit / annual loss exposure | ≥ 1.0× | < 0.5× |

## Related Connections

### Sibling L2 (D7)
- [[change-and-configuration-management]] — DR runbooks are versioned CIs; emergency change procedure invoked at failover
- [[incident-response-lifecycle-nist]] — IR and DR converge on ransomware / destructive attack
- [[digital-forensics-process]] — Capture evidence before fail-over
- [[physical-and-environmental-security]] — Site loss (fire, flood) is a DR trigger

### L3 children
- [[backup-strategies-3-2-1]] — Operational details of the 3-2-1-1-0 rule, immutable, air-gapped

### Cross-domain
- [[business-continuity-and-disaster-recovery]] — D1 note for policy / BIA / RTO-RPO math
- [[domain-02-asset-security]] — Asset inventory is the input to BIA
- [[domain-04-communication-and-network-security]] — Network failover (BGP, DNS, LB) is operational DR
- [[domain-05-identity-and-access-management]] — Break-glass accounts, JIT for DR

## References

- NIST SP 800-34 Rev. 1 — *Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities*
- NIST SP 800-184 — *Guide for Cybersecurity Event Recovery* (2016)
- NIST CSF 2.0 — Recover function (RP.RP, RP.CO, RP.AN, RP.MI, RP.IM)
- ISO 22301:2019 — Business continuity management
- ISO/IEC 27031:2011 — ICT readiness for business continuity
- DRII (Disaster Recovery Institute International) — Professional Practices
- BCI (Business Continuity Institute) — Good Practice Guidelines
- FFIEC BCP / IT Examination Handbooks
- AWS Well-Architected Framework — Reliability Pillar (RTO/RPO patterns)
- Azure Architecture Center — Disaster Recovery guidance
- (ISC)² CISSP CBK 5th ed. — Domain 7
- IBM Cost of a Data Breach Report (annual)
- "Continuous Delivery" (Humble, Farley) — DR via deployment discipline
