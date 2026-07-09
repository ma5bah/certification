---
title: Incident Response Lifecycle (NIST SP 800-61)
layer: 2
domain: 7
tags: [cissp, domain-7, ir, nist, sp-800-61, incident-response, csirt, picerl, severity, playbooks]
related_domains: [1, 2, 5, 6, 7]
---

# Incident Response Lifecycle (NIST SP 800-61)

## Feynman Explanation

An incident response plan is a fire drill for computers: you rehearse the steps before the fire so that nobody argues about who holds the extinguisher when smoke is in the room. NIST SP 800-61 lays out six phases — **Preparation → Detection & Analysis → Containment, Eradication & Recovery → Post-Incident Activity** — that the team walks through for every security event, from a single malware detection to a multi-region ransomware outbreak. The same lifecycle covers both a 2 a.m. false positive and a 60-million-record breach; the difference is scale, not procedure.

## Technical Details

### NIST SP 800-61 Rev. 2 — six phases

| # | Phase | Output | Key activities |
|---|---|---|---|
| 1 | **Preparation** | People, policy, tooling ready | IR plan, contacts, jump bags, comms tree, training, tabletop |
| 2 | **Detection & Analysis** | Validated incident + severity | SIEM triage, scope, IOC correlation, root-cause hypothesis |
| 3 | **Containment, Eradication & Recovery** | Threat removed, business restored | Isolate, kill C2, rebuild, restore from clean backup, monitor |
| 4 | **Post-Incident Activity** | Lessons learned, control updates | After-action review, CAPA, runbook updates, threat intel |

(Note: SP 800-61 Rev. 2 collapsed "Containment / Eradication / Recovery" into one phase for narrative clarity; many organizations keep them as three distinct steps for runbook precision.)

### Alternate lifecycle: SANS PICERL

| P | I | C | E | R | L |
|---|---|---|---|---|---|
| Preparation | Identification | Containment | Eradication | Recovery | Lessons Learned |

The two frameworks map 1-to-1; the choice is stylistic. CISSP exam questions accept either.

### Roles and responsibilities (CSIRT model)

| Role | Responsibility | Authority |
|---|---|---|
| **Incident Commander (IC)** | Owns the response end-to-end; calls the shots | Declares incident severity, activates DR/legal/PR |
| **Security Analyst (Tier 1/2/3)** | Investigates alerts, scopes, executes playbooks | Triage, evidence collection, host containment |
| **Forensic Specialist** | Disk/memory acquisition, chain of custody | Evidence handling (see [[digital-forensics-process]]) |
| **Threat Intel** | IOC correlation, adversary TTPs, attribution | Feeds detection content, hunt hypotheses |
| **Legal / Privacy** | Regulatory notification, privilege, hold | Decides disclosure window (e.g., GDPR 72h) |
| **HR / Communications** | Insider cases, public statements | Controls external messaging |
| **Executive Sponsor** | Approves major actions (paid ransom, offline shutdown) | Final go/no-go on high-impact actions |
| **Scribe / Tracker** | Logs every action, decision, timestamp | Produces the timeline for the after-action review |

### Severity matrix (a common 4-tier scheme)

| Tier | Definition | Example | Response time | Page-out |
|---|---|---|---|---|
| **Sev-1 / Critical** | Active breach, exfil, ransomware, business down | Domain admin compromise, encryption of file shares | <15 min | CISO + IC + Legal + Exec |
| **Sev-2 / High** | Confirmed compromise, contained, impact moderate | Single host malware with C2, no lateral movement | <1 hr | CISO + IC |
| **Sev-3 / Medium** | Suspicious activity, possible policy violation | Policy violation by insider, single phishing click | <4 hr | IC + Analyst |
| **Sev-4 / Low / Informational** | Anomaly, no impact, tracking only | Unusual outbound DNS, port scan from known vendor | Next business day | Tier-1 only |

### Detection sources (where the alert comes from)

| Source | Class | What it finds |
|---|---|---|
| SIEM correlation | Log | Multi-stage patterns, impossible travel, beaconing |
| EDR | Endpoint | Process tree, file write, memory injection, LOLBin |
| NDR | Network | C2 callbacks, lateral SMB, DNS tunneling, JA3/JA3S |
| Cloud audit logs (CloudTrail, Azure Activity) | Cloud | IAM changes, key access, public bucket creation |
| Email security gateway | Email | Phishing, BEC, malware attachment |
| UEBA | Behavior | Anomalous data access, off-hours, impossible travel |
| User report | Human | Phishing, "weird popup", locked account |
| Honeypot / deception | Adversary | First-touch on a decoy asset |
| Third-party / ISAC / vendor | External | Dark-web mention, leaked creds, CVE weaponization |

### Containment strategies (decision tree)

| Choice | Pros | Cons | When to use |
|---|---|---|---|
| **Network isolation** (VLAN change, ACL, host quarantine) | Stops C2 fast | Disrupts user, may tip off attacker | Confirmed C2 / ransomware spread |
| **Account disable / session revoke** | Cuts access, preserves artifacts for forensics | User can't work; session ID may be re-used | Stolen creds, insider threat |
| **Block at perimeter (FW, DNS sinkhole)** | Surgical, low disruption | Adversary may have backup channel | Specific C2 domain/IP, malware family |
| **Application-layer kill switch** | Targets only malicious feature | Vendor-specific, may not exist | SaaS abuse, malicious OAuth app |
| **Full shutdown** | Maximum certainty | Total business outage, evidence loss | Active destruction in progress, last resort |

### Eradication checklist (what "clean" means)

- Identify **all** patient zero → last-hop systems (not just the one that paged).
- Pull and hash binaries, capture IOCs, push to EDR blocklist.
- Rotate **all** credentials, keys, and tokens the adversary touched.
- Reimage (don't "clean") compromised hosts; restore from known-good baseline.
- Patch the exploited CVE across the entire fleet (not just the host).
- Validate with rescans + threat hunt for the next 30 days.

### Recovery & validation

| Step | Validation |
|---|---|
| Restore from immutable backup | Hash compare to known-good |
| Rebuild app / OS from golden image | Vulnerability scan post-build |
| Bring service online in monitored staging | Synthetic transaction + log review |
| Move to production | Compare KPIs to pre-incident baseline |
| **Enhanced monitoring** (30/60/90 days) | Hunt for the same actor returning |

### Post-incident deliverables

| Artifact | Purpose | Owner |
|---|---|---|
| **Timeline** (UTC, second-precision) | Reconstruct story, support legal | IC + Scribe |
| **Root cause analysis (RCA)** | One-sentence "why" with evidence | Forensic lead |
| **Lessons learned (LL)** document | What worked, what didn't, what to change | IC |
| **CAPA** (Corrective and Preventive Action) | Owners + due dates for each gap | CISO |
| **Updated runbooks / playbooks** | Encode new findings into ops | Detection eng. |
| **Threat-intel package** | Share IOCs with ISAC, peers | Threat intel |
| **Notification package** | Regulators, customers, law enforcement | Legal + PR |
| **Cost report** | Hours, tools, downtime, recovery | Finance + CISO |

### NIST CSF 2.0 mapping (for cross-walk)

| NIST CSF function | IR phase that owns it |
|---|---|
| **Govern** | Preparation (policy, risk, roles) |
| **Identify** | Preparation (asset inventory, BIA) |
| **Protect** | Preparation + Recovery (controls restored) |
| **Detect** | Detection & Analysis |
| **Respond** | Containment, Eradication, Post-Incident |
| **Recover** | Recovery + post-incident restoration |

### Key metrics for IR

$$MTTD = \frac{1}{n} \sum_{i=1}^{n} (t_{detect,i} - t_{event,i})$$

$$MTTR = \frac{1}{n} \sum_{i=1}^{n} (t_{recover,i} - t_{detect,i})$$

$$Dwell\ Time = MTTD + MTTA + MTTR_{contain}$$

A widely cited board KPI: **Dwell Time < 24h** puts an organization in the top decile per Mandiant M-Trends.

## CISO / Risk Manager View

The single most expensive mistake a CISO can make is to have an incident response plan that lives in a binder. Mandiant's M-Trends and IBM's Cost of a Data Breach both show a multi-million-dollar cost differential between organizations that have a tested IR plan and those that improvise.

**Board talking points:**

1. **We test the plan, not the binder.** Two tabletop exercises per year minimum, one with ransomware scenario, one with supply-chain scenario. We capture MTTA, MTTR, decision latency, and decision quality.
2. **The IC has authority, not a meeting.** The IR plan delegates predefined authority to the Incident Commander up to a stated dollar / downtime threshold. Above the threshold, the IC has 30 minutes to convene the exec sponsor.
3. **Legal, privacy, and communications are at the table from minute zero.** Waiting for legal in a Sev-1 is what gets CISOs charged with obstruction of regulatory clocks (GDPR Art. 33 = 72h).
4. **MTTD + MTTR are board KPIs.** They map to dollars. We post them monthly with trend.
5. **The cost of a tabletop is the cost of a few pizzas and a quiet conference room.** The cost of *not* doing one is the cost of the incident.

A useful framing equation for the board:

$$Cost_{breach} \approx f(Records\ exposed,\ Dwell\ Time,\ Time\ to\ contain)$$

IBM's 2024 report pegs the cost at ~$169/record plus ~$1.5M of "infra cost" for IR/AI-tested teams vs. the same breach on an un-prepared team. **Detection + response capabilities are cheaper than the loss they prevent, by an order of magnitude.**

## Related Connections

### Sibling L2
- [[digital-forensics-process]] — Evidence handling during Containment/Eradication
- [[change-and-configuration-management]] — Emergency change procedure invoked from IR
- [[disaster-recovery-and-bcp]] — Activated when IR escalates to DR (e.g., ransomware wiping primary)
- [[physical-and-environmental-security]] — Physical breaches flow into the same IR funnel

### L3 children
- [[ransomware-response-playbook]] — The most common Sev-1 scenario
- [[siem-soar-and-detection-engineering]] — Detection is the front door of IR
- [[security-operations-center-soc-models]] — Tier 1/2/3 staffing is the IR factory

### Cross-domain
- [[domain-01-security-and-risk-management]] — IR plan is governance-mandated; BIA drives Sev definitions
- [[domain-02-asset-security]] — Asset inventory defines scope ("what did the attacker touch?")
- [[domain-05-identity-and-access-management]] — Account disable, session revoke, MFA re-enroll are IR actions
- [[domain-06-security-assessment-and-testing]] — Pen test + red team feed IR with realistic scenarios

## References

- NIST SP 800-61 Rev. 2 — *Computer Security Incident Handling Guide* (Cichonski et al., 2012)
- NIST SP 800-184 — *Guide for Cybersecurity Event Recovery* (2016)
- NIST CSF 2.0 (2024) — Govern, Identify, Protect, Detect, Respond, Recover
- ISO/IEC 27035-1:2016 / 27035-2:2016 — *Information security incident management*
- SANS Institute — *Incident Handler's Handbook* (Pirc et al.)
- ENISA — *Reference Incident Classification Taxonomy*
- (ISC)² CISSP CBK 5th ed. — Domain 7
- MITRE ATT&CK — Tactic/technique mapping for IR playbooks
- Mandiant M-Trends (annual)
- IBM Cost of a Data Breach Report (annual)
- CIS Critical Security Controls v8 — Control 17 (Incident Response Management)
