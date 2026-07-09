---
title: Red Team vs Blue Team vs Purple Team
layer: 3
domain: 6
tags: [cissp, domain-6, red-team, blue-team, purple-team, bas, adversary-emulation, assumptions-of-compromise]
related_domains: [1, 3, 5, 7]
parent: domain-06-security-assessment-and-testing
---

# Red Team vs Blue Team vs Purple Team

## Feynman Explanation

A **red team** is the offense — paid attackers trying to break in for real, often without the defenders knowing when. A **blue team** is the defense — the SOC, the detection engineers, the incident responders. A **purple team** is not a *third* team; it's a *collaboration* where red and blue sit in the same room and the red team's actions are designed to *teach* the blue team what to detect. BAS (Breach and Attack Simulation) is the *automation* of red-team techniques so the blue team can test detection coverage continuously, not just once a year. The strategic message: red shows you the gap, blue plugs it, purple is the conversation that makes both sides better, and BAS keeps the loop running.

## Technical Details

### The three teams — definitional table

| Dimension | Red Team | Blue Team | Purple Team |
|---|---|---|---|
| Mission | Emulate real adversary; achieve objective | Detect, respond, defend | Maximize learning for both sides |
| Reports to | CISO, business owner (sponsor) | CISO / VP Security | CISO (facilitator) |
| Primary activity | Adversary emulation, exploitation, exfil | Monitoring, detection, IR, hardening | Joint exercises; immediate feedback |
| Notification | Often unannounced (red); or scheduled (purple) | Always knows they are being tested | Both know in real time |
| Cadence | Continuous or quarterly | 24/7 | Per-exercise / monthly |
| Output | Objective achievement narrative; TTP report | MTTD, MTTR, coverage, control health | New SIEM rules + new tradecraft + cultural trust |
| Tools | C2 (Cobalt Strike, Sliver), custom implants, MITRE Caldera, SCYTHE, social engineering | SIEM, SOAR, EDR, NDR, TIP, IR playbooks | All of the above, plus shared documentation |
| Frameworks | ATT&CK, PTES, NIST 800-115, adversary playbooks (APT29, FIN7, Scattered Spider) | NIST CSF DE.CM, CIS Controls 8/13/17, NIST SP 800-61 IR | ATT&CK Navigator as the shared scoreboard |

### The offense–defense spectrum

```
   Adversary ─────── Red ─── Purple ─── Blue ─────── Defender
   external          team     team       team
   threat            (offense)(bridge)   (defense)
```

- **Red = offense** in isolation, sometimes without defense knowing.
- **Blue = defense** against *all* threats, not just the red team.
- **Purple = neither team, both teams** — the facilitator role that ensures the red team *teaches* the blue team during the exercise, not after.

> CISSP trap: purple is **not a separate team you hire**. It's a function — the collaboration model. Most mature orgs have red and blue teams, and the CISO appoints a "purple-team lead" (or assigns it to the AppSec or Detection Engineering manager) to run joint exercises.

### Adversary emulation (the modern red-team discipline)

| Style | Description | Realism | Cost |
|---|---|---|---|
| **Red team engagement** | Time-boxed (4–12 weeks), objective-bound, RoE | High | High |
| **Adversary emulation** | Replay a *specific named adversary*'s known TTPs (e.g., APT29) | High (scripted) | Medium |
| **BAS (Breach and Attack Simulation)** | Automated, scheduled, technique-by-technique (Atomic Red Team, MITRE Caldera, AttackIQ, Cymulate, Picus) | Medium (synthetic) | Low per test, recurring |
| **Penetration test** | Scope-bound, time-boxed, "find vulns" | Medium | Medium |
| **Bug bounty** | Continuous, public or private, financial incentive | Variable (crowd-sourced) | Variable |

### Adversary emulation vs. BAS — the trade-off

| | Adversary Emulation | BAS |
|---|---|---|
| Realism | High — real human, real chain | Medium — automated, single-step |
| Frequency | Quarterly | Daily–weekly |
| Cost | $$$$ | $ per technique, scales |
| Output | "We can emulate APT29 in 2 weeks" | "We cover 80% of T1000–T1999" |
| Best for | Board-level assurance, objective testing | Continuous detection-coverage validation |
| Limitation | Expensive, infrequent | Canned — misses novel chain logic |

A mature program runs **both**: BAS daily for the 80% of commodity techniques, ad-hoc emulation for the top 3 named adversaries the company worries about.

### BAS (Breach and Attack Simulation) — detail

BAS tools execute techniques end-to-end in a controlled way and report what was *blocked*, *detected*, or *missed*. The leading platforms:

| Platform | Style | Notes |
|---|---|---|
| **MITRE Caldera** | Open-source, agent + server | Industry reference, manual tuning |
| **Atomic Red Team** | Open-source test cases | Per-technique scripts; run via Invoke-AtomicRedTeam |
| **AttackIQ** | Commercial, agent-based | Strong enterprise, ATT&CK-mapped |
| **Cymulate** | SaaS, agent + agentless | Email, web, endpoint, lateral |
| **Picus Security** | SaaS, prevention + detection validation | Strong NDR/SIEM/EDR integration |
| **SCYTHE** | Commercial, C2-focused | "Red-team in a box" |
| **SafeBreach** | SaaS | EDR validation, kill-chain playbook |
| **Mandiant Security Validation** | Commercial | Acquired Mandiant's BAS, very deep |

### Output of a BAS run (per technique)

| Field | Example |
|---|---|
| Technique | T1003.001 OS Credential Dumping: LSASS Memory |
| ATT&CK tactic | TA0006 Credential Access |
| Execution | Mimikatz `sekurlsa::logonpasswords` simulated |
| EDR detection | ✅ Blocked (CrowdStrike Falcon) |
| SIEM alert | ✅ Alert SOC-2024-1014-001 |
| Time to detect | 4 seconds |
| Time to contain | 18 seconds (auto-quarantine via SOAR) |
| Gap | None for T1003.001; flagged gap on T1003.002 (Security Account Manager) |

### Assumptions of compromise (the strategic posture)

The **assumption of compromise** (also called *assume breach*) is the foundational posture of modern red-team-aligned security. It holds:

> "Given sufficient time, motivation, and skill, an adversary *will* compromise some part of our environment. Our security is measured not by 'can they get in' but by 'can we *detect* them, *contain* them, and *recover* quickly enough to limit business impact?'"

Three operational implications:

1. **Detection and response get equal budget to prevention.** A pure prevention posture is failed security. The Verizon DBIR shows breaches routinely take weeks to months to discover — by then, prevention has already failed.
2. **Zero-trust architecture (D3 + D4)** — segment, verify, log, repeat — is the architectural expression of the assumption.
3. **Tabletop exercises** regularly rehearse "they're already in, what's the call?" — CEO, CISO, Legal, Comms, business.

### Continuous red-team / BAS metrics (the right ones for the board)

| Metric | Definition | Target |
|---|---|---|
| **Emulation success rate** | % of adversary chain executed end-to-end without blue team detection | < 10% (i.e., blue team catches them 90%+) |
| **Time to first detection** | Median time from red action to first SIEM/EDR alert | < 5 minutes |
| **Time to contain** | Median time from detection to containment | < 1 hour |
| **ATT&CK coverage** | % of relevant techniques with at least one validated detection | > 80% |
| **Purple-team exercise throughput** | Number of exercises + new rules generated per quarter | Trending up |
| **Mean gap half-life** | Average time between BAS gap and rule deployment | < 14 days |

### Roles and structure (org model)

| Role | Reports to | Mission |
|---|---|---|
| **Red Team Lead** | CISO (dotted line to business sponsor) | Adversary emulation, objective testing |
| **Red Team Operator** | Red Team Lead | Offensive tradecraft |
| **Blue Team Lead / SOC Manager** | CISO / VP Security | 24/7 monitoring, detection, IR |
| **Detection Engineer** | Blue Team Lead | SIEM/EDR rule development, BAS-driven gap closure |
| **Threat Intel Analyst** | SOC Manager or CISO | CTI, adversary tracking, ATT&CK mapping |
| **Purple-Team Lead** | CISO (often dual-hat with Detection Eng or AppSec) | Exercise design, joint learning |
| **CISO** | CEO/Board | Owns the assumption-of-compromise posture; reports metrics to board |

> **CISSP exam perspective:** the question is often "which team is responsible for X?" — the answer is *blue team* for monitoring/detection/IR, *red team* for offensive testing, and *purple team* for *collaboration on improvement*. The CISO is ultimately accountable for the *outcome*.

### The continuous security validation loop (purple + BAS)

```
     ┌─────────────────────────────────────────────┐
     │                                             │
     ▼                                             │
  ┌────────┐  TTPs  ┌────────┐  run  ┌────────┐    │
  │ Threat │───────▶│ Red /  │──────▶│ EDR/   │    │
  │ Intel  │        │ BAS    │       │ SIEM   │    │
  └────────┘        │ Engmt  │       └────┬───┘    │
                    └────┬───┘            │        │
                         │                │ alert? │
                         │                ▼        │
                    ┌────▼────┐      ┌────────┐    │
                    │ Report  │      │ Blue / │    │
                    │ (T-num) │      │ Detect │────┘
                    └────┬────┘      │ Eng    │  feedback
                         │           └────┬───┘  new rule
                         ▼                │
                    ┌─────────┐            │
                    │ Purple  │◀───────────┘
                    │ Review  │
                    └────┬────┘
                         │
                         ▼
                    New SIEM rule /
                    EDR policy / SOAR
                    playbook / control
                    ──→ re-test
```

The loop is: **threat intel** picks the relevant TTP → **red/BAS** executes it → **blue** must detect and respond → **purple** reviews → new rule closes the gap → **re-test** proves it.

## CISO / Risk Manager View

Red, blue, and purple are *organizational expressions* of Domain 6's verification mandate. The strategic CISO views them as a single investment with three outputs:

1. **Annual third-party pen test** — independent assurance (Domain 6, [[penetration-testing-methodology]]).
2. **Internal red team** — continuous adversary emulation, depth on top-3 named threats.
3. **Mature blue team / SOC** — detection and response at scale, measured by [[logging-monitoring-and-siem]] KPIs.
4. **Purple-team program** — the *glue*; if you have red and blue but no purple, the red team's findings die in a Jira backlog.

**Maturity model (board-facing):**

| Level | Red | Blue | Purple | Cost |
|---|---|---|---|---|
| 1 — Initial | Annual 3rd-party pentest | SIEM, basic IR | Ad-hoc | $ |
| 2 — Defined | Quarterly pentest | SOC 24/7 (internal or MSSP) | Quarterly purple exercises | $$ |
| 3 — Managed | Internal red team + adversary emulation | Mature SOC, SOAR, detection engineering | Monthly purple, integrated | $$$ |
| 4 — Optimized | Continuous BAS + named-adversary emulation | AI-augmented SOC, MTTD < 1h | Continuous validation, ATT&CK-mapped | $$$$ |

**The "red team without blue team" trap.** Some CISOs build a red team to "look mature" and the findings rot in spreadsheets. Worse, the red team *trains the red team* — adversaries get better practice than defenders. **Always pair the red investment with a blue investment, and force purple-team exercises that close the loop in days, not quarters.**

**The "blue team without red team" trap.** A blue team that has never been adversarially tested is operating on theoretical detection coverage. Without red, the "we have 80% coverage" claim is a guess. **Annual independent adversary emulation is the proof.**

**Budget framing (mid-size enterprise):**
- Internal red team (4–6 people + tooling): $1.5M–$3M/year
- BAS platform (commercial): $50K–$300K/year
- SOC (24/7, 8–12 people, SIEM/SOAR): $2M–$8M/year
- MSSP co-sourcing: $500K–$2M/year (reduces internal headcount)
- Joint purple exercises: $100K–$300K/year (mostly time)

> **CISO's board message:** "We assume compromise. We measure how fast we see it, how fast we contain it, and what we learn to make it faster. Those are the metrics we report, and they trend in the right direction."

## Related Connections

### Parent
- [[domain-06-security-assessment-and-testing]] (D6 hub)

### L2 siblings
- [[penetration-testing-methodology]] (pen test = scheduled, scope-bound; red team = objective-bound)
- [[vulnerability-management-lifecycle]] (red team findings feed the vuln queue)
- [[security-audits-and-compliance-testing]] (audits are static; red/blue is dynamic)
- [[logging-monitoring-and-siem]] (MTTD/MTTR is the blue-team vital sign)

### Cross-domain
- [[Domain 1 — Security and Risk Management]] (assumption of compromise = risk treatment posture)
- [[Domain 3 — Security Architecture and Engineering]] (zero-trust is the architectural expression)
- [[Domain 5 — Identity and Access Management]] (identity is the most-attacked control surface)
- [[Domain 7 — Security Operations]] (blue team is the SOC + detection engineering + IR)

## References

- MITRE ATT&CK — Enterprise, Mobile, ICS matrices (v15+)
- MITRE D3FEND (defensive techniques)
- MITRE Caldera — open-source adversary emulation
- MITRE ATT&CK Evaluations (annual EDR/NDR public results)
- Atomic Red Team — Red Canary (per-technique test scripts)
- SCYTHE — adversary-emulation platform
- AttackIQ — BAS platform
- Cymulate — BAS platform (SaaS)
- Picus Security — BAS + prevention/detection validation
- SafeBreach — BAS + kill-chain validation
- Mandiant Security Validation (post-acquisition) — BAS
- Cobalt Strike (Fortra) — commercial C2 / red-team platform
- Sliver (Bishop Fox) — open-source C2
- Brute Ratel — commercial C2 (now cracked, do not use in production engagements)
- SANS SEC565 / SEC660 / SEC760 — red-team courses
- SANS SEC555 / SEC511 — blue-team / detection engineering courses
- SANS SEC530 / SEC551 — purple-team courses
- ATT&CK Defender (MAD) — detection engineering certification
- GIAC GREM / GCFA / GDAT — incident response / detection certs
- CREST — red-team certification (UK, widely adopted)
- TIBER-EU (European Central Bank) — threat-intelligence-based ethical red-teaming framework
- CBEST (Bank of England) — red-team framework for UK financial sector
- AASE / iCAST (Singapore) — red-team framework
- NIST SP 800-115 — Testing and Assessment
- NIST SP 800-61 Rev. 2 — Incident Handling
- Verizon *DBIR 2024* — MTTD/dwell-time benchmarks
- Mandiant *M-Trends 2024* — industry MTTD
- IBM *Cost of a Data Breach Report 2024* — MTTR benchmarks
- Lockheed Martin *Cyber Kill Chain* (precursor to ATT&CK; still referenced in finance)
- Palo Alto *Unit 42* annual incident report (uses ATT&CK)
- CrowdStrike *Global Threat Report* (uses ATT&CK)
- Microsoft *Digital Defense Report* (uses ATT&CK)
- CISA — *Red Team Assessment* public summaries (post-2023)
- ENISA *Threat-Led Penetration Testing* (TLPT) guidance under DORA
