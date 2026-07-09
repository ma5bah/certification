---
title: Security Operations Center (SOC) Models
layer: 3
domain: 7
tags: [cissp, domain-7, soc, tier-1, tier-2, tier-3, mssp, mdr, ooda, metrics, kpi, dwell-time, false-positive, alert-fatigue, fusion-center]
related_domains: [1, 4, 5, 6]
parent: incident-response-lifecycle-nist
---

# Security Operations Center (SOC) Models

## Feynman Explanation

A SOC is the team's team — the people, the processes, the tooling, the shifts, the metrics — that watches the environment 24×7 and decides, in seconds to minutes, whether the alert that just fired is a false positive, a real attack, or a test that someone forgot to silence. Like a hospital emergency room, a SOC is staffed in tiers (the nurse triages, the senior doctor diagnoses, the surgeon operates), it runs on shift work, and the quality of care is measured in time-to-treat, misdiagnosis rate, and patient outcomes. A great SOC prevents million-dollar incidents; a bad SOC is an empty room with a flickering monitor.

## Technical Details

### The tier model (industry standard)

| Tier | Role | Tools | Authority |
|---|---|---|---|
| **Tier 0 / Automated** | SOAR / EDR auto-contain, auto-enrichment, auto-ticket | SOAR, EDR, identity | Pre-approved actions (isolate host, disable user) |
| **Tier 1 — Triage Analyst** | Alert acknowledgment, basic enrichment, false-positive triage, escalation | SIEM, ticketing, EDR | Document; close as FP; escalate to T2 |
| **Tier 2 — Incident Analyst** | Deep investigation, scoping, host/network forensics, containment | SIEM, EDR, NDR, forensic tools, threat intel | Contain (network-isolate host, disable user, block IOC) with playbook |
| **Tier 3 — Threat Hunter / Subject-Matter Expert** | Hypothesis-driven hunt, complex investigation, custom tooling, red/purple team | All + custom scripts, malware analysis, ATT&CK mapping | Custom IOC creation, advanced containment, IR lead |
| **Tier 4 — SOC Manager / IR Lead** | Run the watch, manage escalations, board reporting, vendor management | Dashboards, BI, runbooks | Crisis decisions, exec comms |
| **Tier 5 — CISO / Executive Sponsor** | Strategic direction, budget, regulatory, board | Board reports, GRC | Authority for major action (ransom, breach disclosure) |

Most enterprise SOCs are T1–T3; T4–T5 are management/exec. The tier model is a *skill gradient*, not a strict job ladder — a strong T1 may do T2 work; a T3 may still T1 in a quiet watch.

### SOC operating models (org-structure comparison)

| Model | Description | Pros | Cons | When |
|---|---|---|---|---|
| **In-house, follow-the-sun (FTS)** | 3 sites, 8-h shifts, global coverage | Control, IP retention, deep context | Cost, hiring, 24×7 fatigue | Large enterprise, regulated |
| **In-house, business hours + on-call** | Day shift in-house, night/weekend on-call | Cheapest | Burnout, slow night-time MTTD | Small/medium org |
| **Hybrid (in-house + MSSP)** | Tier 1 outsourced, Tier 2/3 in-house | Cost balance, 24×7 coverage | Handoff friction, MSSP lock-in | Mid-market, scaling |
| **MSSP (full outsource)** | Vendor runs the watch | Capex savings, fast start | Loss of control, IP leakage, tail risk | Compliance-driven, low-maturity |
| **MDR (managed detection + response)** | Vendor brings tool + analyst + IR | Outcome-driven, fast MTTD | Vendor lock-in, less custom | Mid-market, ransomware focus |
| **Co-managed SOC** | Vendor extends in-house team | Flexibility, surge capacity | Cost, governance | Mature in-house adding capacity |
| **Virtual SOC** | Distributed analysts, no co-located room | Cost, geographic reach | No team cohesion, harder comms | SMB, global remote org |
| **Fusion center** | SOC + fraud + physical + intel | Holistic picture | Hard to build, governance heavy | Government, finance, critical infra |

### Staffing math (a useful rule of thumb)

A 24×7 SOC with N analysts covers 24 × 7 = 168 hours / week. Each analyst works ~40 hours / week. To cover one seat, you need ~168/40 = 4.2 analysts (for 24×7 single-shift, ignoring PTO, training, sick leave). With PTO + training + sick leave (multiplier ~1.4):

$$Analysts_{needed} = \frac{Shifts \times Hours/week}{Hours/analyst/week} \times 1.4$$

Example: 1 shift, 24×7 = 4.2 × 1.4 ≈ **6 analysts minimum** for one watch position. A 4-watch position (T1, T2, T3, manager) = ~24 analysts for a small 24×7 SOC.

Industry benchmarks:

| Org size | In-house SOC headcount | Approx. budget |
|---|---|---|
| SMB (< 1,000 emp) | 0–3 (often outsourced) | $0.5–2M |
| Mid-market (1k–10k) | 5–15 (hybrid) | $2–8M |
| Large enterprise (> 10k) | 20–100+ | $10–50M+ |
| Global enterprise | 100–500+ | $50–200M+ |

### The OODA loop in SOC context

Colonel John Boyd's decision loop maps to SOC work:

| OODA stage | SOC activity | Tool |
|---|---|---|
| **Observe** | Collect logs, alerts, intel | SIEM, EDR, NDR, threat intel feeds |
| **Orient** | Enrich, contextualize, prioritize | SOAR, UEBA, threat intel platform |
| **Decide** | Triage decision: FP, known, escalate | Playbook, runbook, analyst |
| **Act** | Contain, eradicate, recover, document | EDR, IAM, SOAR, ticketing |

A SOC's value is making the OODA loop tight:

$$Loop\ Time = t_{observe} + t_{orient} + t_{decide} + t_{act}$$

A great SOC runs the loop in minutes; a poor one runs it in days.

### The PICERL ↔ SOC tier mapping

| NIST/PICERL phase | SOC tier | Action |
|---|---|---|
| Preparation | T4 / T5 | IR plan, training, tabletop |
| Identification (Detection & Analysis) | T1 → T2 → T3 | Triage → investigation → hunt |
| Containment | T2 → T3 | Isolate host, disable user, block IOC |
| Eradication | T3 + ops | Remove malware, rotate creds, patch |
| Recovery | T2 + ops | Restore from backup, validate, monitor |
| Lessons Learned | T4 + all | PIR, CAPA, runbook updates |

### Alert triage flow (Tier 1 in practice)

| Step | Time target | Output |
|---|---|---|
| 1. Acknowledge (MTTA) | < 15 min | Ticket open, owner assigned |
| 2. Enrich (auto by SOAR) | < 1 min | Asset, user, prior alerts attached |
| 3. First pass | < 30 min | TP, FP, or unknown |
| 4. If FP | < 5 min | Close with reason, propose tuning |
| 5. If unknown | < 30 min | Escalate to T2 with context |
| 6. If TP | < 5 min | Page T2, begin playbook |

### Common SOC metrics (DORA, SANS, Gartner)

| Metric | Definition | Target | Why |
|---|---|---|---|
| **MTTD** (Mean Time To Detect) | Time from event to alert | Hours (top decile < 24 h) | Speed of visibility |
| **MTTA** (Mean Time To Acknowledge) | Time from alert to analyst engaged | < 15 min Sev-1 | Triage speed |
| **MTTR** (Mean Time To Respond) | Time from alert to containment | Hours | Speed of action |
| **MTTC** (Mean Time To Contain) | Time from alert to threat stopped | < 1 h Sev-1 | Speed of action |
| **Dwell time** | MTTD + MTTA + MTTC | < 24 h (top decile) | Composite |
| **Alert volume per analyst per day** | Tickets handled | < 50 | Sanity / fatigue |
| **% false positive** | FP / total alerts | < 5% per rule, < 20% overall | Quality |
| **% rules with FP rate > 25%** | Per rule | < 5% | Tuning needed |
| **Detection coverage (ATT&CK heatmap)** | % of relevant techniques covered | > 80% top-50 | Breadth |
| **% SOC analysts with certs** | Cert (GCIH, GCFA, OSCP, CySA+) | > 50% | Skill |
| **% analysts with annual training hours** | 40 hr / yr | 100% | Compliance + retention |
| **Turnover rate** | Annual % | < 15% (industry: 20–30%) | Burnout inverse |
| **Time on task vs. alert** | Active investigation time | > 50% | Idle time waste |

### SOC analyst KPIs (per-tier)

| Tier | KPI |
|---|---|
| T1 | Tickets closed (FP), MTTA, escalation quality |
| T2 | MTTC, RCA accuracy, false escalations |
| T3 | Hunts run, new detection rules authored, techniques covered |
| T4 | MTTR trend, board reports, vendor SLAs, hiring/retention |
| T5 | Risk burn-down, $ loss avoided, board narrative |

### MSSP / MDR SLAs — what to negotiate

| SLA | Acceptable target | Red flag |
|---|---|---|
| MTTA Sev-1 | < 15 min | > 30 min |
| MTTC Sev-1 | < 60 min | > 4 h |
| False positive rate | < 30% | > 50% |
| Coverage hours | 24×7 (or defined window) | "business hours" for Sev-1 |
| Reporting cadence | Weekly / monthly | Quarterly only |
| Onboarding timeline | < 60 days | > 90 days |
| Data residency | Defined | Vendor discretion |
| Audit rights | Right to audit | No audit rights |
| Tool ownership | Customer owns, MSSP operates | Vendor owns, customer leases |
| Exit clause | Data export in standard format | None |

**CISSP tip:** an MSSP without audit rights and exit clause is a long-term hostage scenario.

### Detection content maturity (the "rule lifecycle")

| Stage | Definition | Owner |
|---|---|---|
| **Hypothesis** | "If adversary does X, we should see Y" | T3 / threat intel |
| **Data validation** | Confirm log source exists | Detection eng. |
| **Draft rule** | Sigma / KQL / SPL | Detection eng. |
| **Atomic Red test** | Run known-good adversary emulation, verify rule fires | Detection eng. + red team |
| **Tune** | FP analysis, threshold, exclusions | Detection eng. |
| **Deploy** | Promote to prod with version control | Detection eng. |
| **Operate** | T1/T2 triages, FP rate tracked | SOC + detection eng. |
| **Retire** | Stale, replaced, too noisy | Detection eng. |

A mature program runs this loop weekly; an immature program adds 50 rules a quarter and never retires.

### Fusion center (the next-level SOC)

A fusion center integrates:

| Source | Beyond traditional SOC |
|---|---|
| **Cyber** | EDR, NDR, SIEM, cloud |
| **Fraud** | Banking, transaction monitoring, AML |
| **Physical** | CCTV, badge, GIS, drones |
| **Insider threat** | HR signals, DLP, behavioral |
| **Threat intel** | OSINT, ISAC, government, vendor |
| **Brand/reputation** | Social, dark web |
| **Geopolitical** | Country risk, sanctions, civil unrest |

The output is a *holistic* threat picture, not siloed alerts. Used in financial services, government, defense, critical infrastructure.

### SOC tooling (a 2024 stack example)

| Layer | Tool examples | Open-source alt. |
|---|---|---|
| **SIEM** | Splunk ES, Sentinel, Chronicle, Sumo, Panther, Exabeam | Elastic Security, Wazuh, OpenSearch |
| **EDR** | CrowdStrike, SentinelOne, Defender, Trellix, Trend | Wazuh, OSSEC, Velociraptor |
| **NDR** | Vectra, Darktrace, ExtraHop, Corelight, IronNet | Zeek, Suricata, Arkime |
| **SOAR** | Splunk SOAR, Sentinel Playbooks, Tines, Torq, Palo Alto XSOAR | n8n, Shuffle, StackStorm |
| **Threat intel** | Recorded Future, Mandiant, Anomali, MISP, ThreatConnect | MISP, OpenCTI |
| **BAS** | AttackIQ, SafeBreach, Cymulate, Picus | Atomic Red Team, Caldera |
| **Ticketing** | ServiceNow, Jira, Zendesk, Linear | TheHive, Zammad, osTicket |
| **Case mgmt** | ServiceNow SecOps, Resilient, Swimlane | TheHive |
| **PAM** | CyberArk, BeyondTrust, Delinea, HashiCorp Vault | Teleport, Apache Guacamole + Vault |
| **Hunt / IR** | Velociraptor, KAPE, GRR, MFT, MemProcFS | All open-source |
| **Sandbox** | CrowdStrike Falcon, ANY.RUN, Joe Sandbox, Hybrid Analysis | Cuckoo (deprecated), CAPA |
| **Vuln mgmt** | Tenable, Qualys, Rapid7 InsightVM | OpenVAS, Nuclei |

### Alert fatigue — the silent killer

| Symptom | Cause | Fix |
|---|---|---|
| Analysts close 100% of alerts as FP in 1 click | No tuning, broken signal | Tune, retire, or replace rule |
| 5,000 alerts / day, 5 analysts | Rule sprawl, no auto-closure | Aggregate, dedup, alert on trend |
| Mean time on alert = 30 sec | Rubber-stamping | Force minimum 5-min investigation window |
| Senior analysts leave | Burnout, no growth | Career path, hunt rotation, training |
| MTTD creeping up | Analysts missing real alerts | Re-baseline alerts, offload Tier 0 to SOAR |

A widely cited 2023–2024 SOC survey: alert fatigue is the #1 reason for SOC turnover, and turnover is the #1 reason for SOC quality decline. A virtuous cycle of detection engineering, automation, and retention is the answer.

### SOC maturity model (SOC-CMM, simplified)

| Level | Description |
|---|---|
| **1 — Ad hoc** | No SOC, no process, no metrics |
| **2 — Reactive** | SIEM exists, no tuning, no hunt |
| **3 — Proactive** | Playbooks, runbooks, basic metrics |
| **4 — Operational** | SOAR, hunt, ATT&CK coverage, KPIs |
| **5 — Strategic** | Fusion, threat-intel-driven, board-trusted, AI-augmented |

Most enterprises sit at 2–3. The journey to 4 takes 18–24 months and a multi-million-dollar investment in people and tooling.

## CISO / Risk Manager View

**The board conversation:**

1. **The SOC is the visible face of security operations.** When the board asks "are we safe?" the answer is in the SOC's metrics, not in the firewall rule count. MTTD, MTTR, and detection coverage are the right metrics.
2. **SOC quality is a hiring problem, not just a tooling problem.** The best detection rules in the world are useless if no one triages them. A CISO must invest in retention: career path, training, rotation, competitive pay.
3. **MSSP / MDR is a strategic decision, not a procurement decision.** Outsourcing triage works only if the contract has audit rights, exit clauses, and well-defined SLAs. Otherwise, the cost savings disappear in incident response.
4. **The fusion center is the future, not the present.** Most organizations are not ready; the path is integration (SOC + fraud + physical + insider) over years, not quarters.
5. **AI is a force multiplier, not a replacement.** LLM summarization, natural-language hunt, and autonomous Tier-0 actions are real; autonomous Tier-3 investigation is not. The CISO who over-promises AI loses credibility fast.

A useful board equation:

$$Value_{SOC} = Incidents\ Detected \times Loss\ Avoided\ per\ Incident - SOC\ Cost$$

A mid-market SOC ($5M/yr) that prevents 2 major incidents per year ($5M each) pays for itself 1×. If the same SOC prevents 1 ransomware event ($10M) plus 1 BEC ($1M), the ROI is 2.2×. The board's job is to ensure the SOC has the budget to be effective — not to starve it into a rubber-stamp factory.

**Top 5 things a CISO can do this quarter:**

1. **Post MTTD, MTTA, MTTR monthly to the board.** Set a target, show the trend.
2. **Audit alert volume and FP rate.** Tune or retire the top 10 noisy rules.
3. **Build the MITRE ATT&CK heatmap** for the top-30 techniques relevant to your industry.
4. **Negotiate MSSP/MDR SLAs** to match your risk tolerance, not the vendor's.
5. **Invest in analyst retention** — career path, training budget, on-call stipend. A burned-out SOC is a failed SOC.

## Related Connections

### Parent
- [[incident-response-lifecycle-nist]] — SOC is the front line of IR

### Sibling L2
- [[change-and-configuration-management]] — SOC-driven change correlation; change tickets feed detection context
- [[digital-forensics-process]] — SOC hands off to forensics for deep investigation
- [[disaster-recovery-and-bcp]] — SOC in DR region; SOC playbook for DR activation
- [[physical-and-environmental-security]] — Physical alarms feed SOC

### Sibling L3
- [[siem-soar-and-detection-engineering]] — The tooling the SOC runs
- [[ransomware-response-playbook]] — The most common Sev-1 scenario the SOC handles
- [[backup-strategies-3-2-1]] — SOC monitors backup health and tamper detection

### Cross-domain
- [[domain-01-security-and-risk-management]] — Risk register drives SOC priorities
- [[domain-04-communication-and-network-security]] — NDR / NGFW feed SOC
- [[domain-05-identity-and-access-management]] — IAM events are high-signal SOC inputs
- [[domain-06-security-assessment-and-testing]] — Red team + pen test feed SOC with realistic attack scenarios

## References

- SANS — *2019 / 2024 SOC Survey* (annual)
- SOC-CMM — SOC Capability Maturity Model (SOC Prime)
- Gartner — *Market Guide for Security Operations Centers*, *Market Guide for MDR Services* (annual)
- Forrester — *The Forrester Wave: Managed Detection and Response Services*, *SOC Analyst Landscape*
- MITRE — *11 Strategies of a World-Class Cybersecurity Operations Center* (Zimmerman, 2014)
- MITRE — *The Defender's Dilemma* (2015)
- NIST SP 800-61 Rev. 2 — *Computer Security Incident Handling Guide*
- NIST SP 800-137 — *Information Security Continuous Monitoring (ISCM)*
- NIST CSF 2.0 — DE (Detect), RS (Respond) functions
- ISO/IEC 27035-1:2016 / 27035-2:2016 — Incident management
- CREST — *SOC Operator Competency Framework*
- (ISC)² CISSP CBK 5th ed. — Domain 7
- Mandiant M-Trends (annual)
- IBM Cost of a Data Breach Report (annual)
- Devo — *SOC Performance Benchmark* (annual)
- Tines — *Automation Benchmarks for SOCs*
- Swimlane — *State of the SOC* (annual)
- Palo Alto Unit 42 — *Incident Response and Data Breach Report*
- "The Phoenix Project" + "The Unicorn Project" (Gene Kim) — DevOps culture for SOCs
- "Cybersecurity Operations Center Starter Kit" (Alissa Torres, SANS) — practitioner handbook
- Tidal Cyber / Picus / AttackIQ — BAS vendors for detection validation
- Atomic Red Team (Red Canary) — open adversary emulation
- Caldera (MITRE) — automated adversary emulation
- "Blue Team Handbook: Incident Response Edition" (Don Murdoch) — SOC operations handbook
- Open-source tooling: MISP, OpenCTI, TheHive, Cortex, Wazuh, Velociraptor, GRR, Arkime, Zeek
