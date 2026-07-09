---
title: SIEM, SOAR, and Detection Engineering
layer: 3
domain: 7
tags: [cissp, domain-7, siem, soar, sigma, mitre-attack, detection-engineering, use-case, ueba, xdr, edr-ndr, log-pipeline, threat-hunting]
related_domains: [4, 5, 6]
parent: incident-response-lifecycle-nist
---

# SIEM, SOAR, and Detection Engineering

## Feynman Explanation

A SIEM is the SOC's search engine for "did anything bad happen in our logs?" — a giant indexed database of every event from every system, queried in real time by detection rules and in retrospect by analysts. A SOAR is the SOC's robot — a workflow engine that, when an alert fires, opens a ticket, pulls context, asks a human for approval, and (sometimes) takes action. Detection engineering is the discipline of writing, testing, and tuning the rules so that the SIEM surfaces the right things and the SOAR does the right things. Without all three, you have 100,000 alerts per day and no time to look at any of them.

## Technical Details

### SIEM — the core capability

| Capability | What it does |
|---|---|
| **Log collection** | Agents, syslog, API pull, cloud subscriptions (S3, Event Hub) |
| **Normalization** | Field-level mapping to a common schema (CEF, OCSF, ECS) |
| **Indexing** | Full-text + structured (Elastic, KQL, SPL, AQL) |
| **Correlation** | Multi-event rules, threshold, sequence, UEBA |
| **Alerting** | Severity, dedup, suppression, routing |
| **Retention** | Hot (days), warm (weeks), cold (months–years) |
| **Dashboards** | Real-time KPI, hunt pivots |
| **Forensic pivots** | Pivots from a single host/user/IP across all log sources |

### Log source inventory (a baseline taxonomy)

| Category | Example sources | Typical volume |
|---|---|---|
| **Identity** | AD, Azure AD, Okta, Duo, Auth0, PAM | 1k–10k EPS |
| **Endpoint** | EDR (CrowdStrike, SentinelOne, Defender), Sysmon | 10k–100k EPS |
| **Network** | NGFW, NDR (Vectra, Darktrace, ExtraHop), DNS, proxy, VPN | 10k–100k EPS |
| **Cloud** | CloudTrail, Azure Activity, GCP Audit, M365, Okta, GitHub, OCI | 1k–50k EPS |
| **Email** | M365, Proofpoint, Mimecast, Exchange | 1k–10k EPS |
| **Application** | Web server, app logs, DB audit, API gateway, CRM | 1k–10k EPS |
| **Physical** | Badge, CCTV, access control | < 1k EPS |
| **Threat intel** | MISP, vendor feeds, ISAC, TAXII | < 1k EPS |

**EPS = events per second.** A 50,000-EPS SIEM is a serious deployment; the 2024 enterprise median is ~5,000 EPS.

### SIEM architectures

| Architecture | Strength | Weakness |
|---|---|---|
| **Centralized on-prem** (Splunk, QRadar, ArcSight) | Control, compliance | Capex, scale ceiling |
| **Cloud-native SaaS** (Sentinel, Chronicle, Sumo, Panther) | Scale, ML, less ops | Cost at scale, data egress |
| **Open-source stack** (Elastic + Wazuh + OpenSearch + Sigma) | Cost, customization | Ops burden, scale work |
| **Hybrid (ingest cloud, store on-prem)** | Compliance + scale | Complexity |
| **Lakehouse SIEM** (S3 + Athena + Sigma) | Cheap long-term, flexible | Latency for live correlation |

### The 2024–2025 trend: SIEM → "data lakehouse" + detection-as-code

| Trend | Implication |
|---|---|
| **Detection-as-code** | Detection rules versioned in Git, tested in CI, deployed via CI/CD (Splunk + Git, Sentinel + GitHub, Panther, Chronicle ruleset) |
| **Open schema** | OCSF (Open Cybersecurity Schema Framework) as vendor-neutral normalization |
| **Lakehouse** | Cheap storage (S3, ADLS) + query engine (Athena, Trino, BigQuery) + detection layer |
| **XDR consolidation** | EDR + NDR + Cloud + Identity correlated natively; SIEM becomes the "data layer" |
| **AI / LLM** | Natural-language hunt, alert summarization, playbook drafting (still early) |

### SOAR — security orchestration, automation, and response

| Component | Purpose |
|---|---|
| **Ingest** | Pull alerts from SIEM, EDR, email, ticketing |
| **Case mgmt** | Assign, prioritize, track |
| **Playbook** | Visual workflow: enrichment → decision → action |
| **Integration** | Pre-built apps for hundreds of security tools |
| **Action** | API calls to EDR (isolate host), IAM (disable user), firewall (block IP), ticketing, paging |

A typical SOAR playbook (phishing triage):

1. Receive email from user or gateway.
2. Extract URLs, hashes, sender.
3. Query threat intel (VirusTotal, internal).
4. Detonate URL in sandbox.
5. If malicious → search SIEM for clicks → isolate endpoint → disable user → notify.
6. Open ticket with all artifacts.
7. SOC analyst reviews and approves.

A more advanced SOAR playbook (ransomware response) integrates IR (see [[ransomware-response-playbook]]) and DR (see [[disaster-recovery-and-bcp]]).

### Detection engineering — the discipline

| Step | Output |
|---|---|
| **Threat model** | Top adversaries, top TTPs (MITRE ATT&CK) |
| **Hypothesis** | "If the adversary is doing T1059.001 (PowerShell), we should see event X" |
| **Data source** | Confirm log source exists and is ingested |
| **Rule** | Sigma / YARA / SPL / KQL rule |
| **Test** | Replay against historical data, atomic Red test |
| **Tune** | False positive analysis, threshold adjustment |
| **Deploy** | Promotion to production with version control |
| **Measure** | True positive rate, FP rate, time-to-fire |
| **Retire** | Quarterly review of stale or FP-heavy rules |

### The Sigma rule format (vendor-neutral)

```yaml
title: Suspicious PowerShell Encoded Command
id: 0c7b0d3e-1f0a-4e1d-9b6c-9e1f5e2d3c4b
status: stable
description: Detects suspicious encoded PowerShell execution often used by attackers
references:
  - https://attack.mitre.org/techniques/T1059/001/
author: Detection Engineering
date: 2026/07/08
tags:
  - attack.execution
  - attack.t1059.001
  - cve.2024.12345
logsource:
  product: windows
  category: process_creation
detection:
  selection:
    Image|endswith:
      - '\powershell.exe'
      - '\pwsh.exe'
    CommandLine|contains:
      - '-EncodedCommand'
      - '-enc '
      - 'FromBase64String'
  filter_legit:
    CommandLine|contains:
      - 'AppLocker'
      - 'WDAC'
  condition: selection and not filter_legit
fields:
  - User
  - Computer
  - CommandLine
  - ParentImage
falsepositives:
  - Legitimate admin scripts
level: high
```

A compiled rule (SPL, KQL, Lucene) is generated from the Sigma by converters; the **Sigma spec** is the single source of truth.

### MITRE ATT&CK mapping (the lingua franca)

ATT&CK organizes adversary behavior into:

| Level | Concept | Example |
|---|---|---|
| **Tactic** | Why the adversary is doing this | Initial Access, Execution, Persistence |
| **Technique** | How | T1059 PowerShell, T1486 Data Encrypted for Impact |
| **Sub-technique** | Specific variant | T1059.001 PowerShell, T1486.001 Stop Services |
| **Procedure** | Real-world instance | Conti uses T1059.001 with -EncodedCommand |

A detection mapped to a technique is far more valuable than a string-of-the-day rule. ATT&CK mapping is the *unit test* of detection coverage.

### Detection coverage measurement

$$Coverage = \frac{|Detected\ Techniques\ (mapped\ to\ a\ rule)|}{|Total\ Techniques\ relevant\ to\ organization|}$$

**Mature target:** 80%+ of the techniques most relevant to the org's threat model (top 20–50 techniques) have a working rule. "We have 100% coverage" usually means "we have 1,000 rules," not "we cover all 200 techniques in the relevant tactics."

**Detection coverage heatmap:** rows = techniques, columns = data sources, cell = covered (green) / partial (yellow) / gap (red). The output is a heatmap; the gaps drive the next sprint.

### Use case library (a starter set, ATT&CK-mapped)

| Tactic | Technique | Use case |
|---|---|---|
| **Initial Access** | T1566.001 Spearphishing | Suspicious attachment / link from external sender |
| **Initial Access** | T1190 Exploit public-facing app | WAF / NDR burst of exploit signatures |
| **Execution** | T1059.001 PowerShell | Encoded command, base64 decode in same script |
| **Execution** | T1047 WMI | wmic.exe with /node + process call create |
| **Persistence** | T1546.010 AppInit DLLs | Registry value change |
| **Persistence** | T1136.001 Create account | New local/domain user by non-admin |
| **Privilege Escalation** | T1068 Exploitation for priv-esc | Driver load, token impersonation |
| **Defense Evasion** | T1562.001 Disable security tools | Defender / EDR real-time monitoring turned off |
| **Credential Access** | T1003 OS credential dumping | LSASS access, mimikatz signatures |
| **Lateral Movement** | T1021.002 SMB/Windows admin shares | Admin share access from non-admin host |
| **Collection** | T1560.001 Archive via utility | 7z, rar with sensitive file paths |
| **Exfiltration** | T1041 Exfil over C2 | Anomalous outbound to known C2 |
| **Impact** | T1486 Data encrypted for impact | Mass file rename, vssadmin delete |
| **Impact** | T1490 Inhibit system recovery | bcdedit, wbadmin delete |

A SOAR playbook maps 1:1 to a use case, with branching logic and approvals.

### UEBA (User & Entity Behavior Analytics)

UEBA adds a baseline-and-anomaly layer to rule-based detection:

| Behavior | Baseline | Anomaly → alert |
|---|---|---|
| **Impossible travel** | User in NY, then Singapore 30 min later | Login from new geo within impossible window |
| **Off-hours access** | User works 9–5 PT | Access at 03:00 local time |
| **Volume spike** | User downloads 100 MB / day | 10 GB download in 1 hour |
| **New resource** | User accesses ERP, GitHub | First-time access to finance DB |
| **Privilege escalation** | User has read-only | Suddenly adds a group, creates an account |
| **Credential stuffing** | Failed logins rare | 10k failures from one IP, one hour |

UEBA complements rule-based detection but is not a replacement.

### Threat hunting (the proactive loop)

| Step | Output |
|---|---|
| **Hypothesis** | "Adversary uses WMI for lateral movement in our env" |
| **Data required** | Sysmon Event ID 19/20/21 (WmiPrvSE) |
| **Hunt query** | Process tree with WmiPrvSE → unusual child |
| **Findings** | New IoC, false positive, gap, or new tool needed |
| **Detection** | Turn findings into a Sigma rule |
| **Feedback** | Add to playbook, feed threat intel |

Hunt cycle time: weekly hunt, monthly report, quarterly review. A mature SOC has dedicated hunters, not just alert triagers.

### KPI for the SOC / detection engineering

| KPI | Target | Why |
|---|---|---|
| **Mean Time to Detect (MTTD)** | Hours (top decile < 24 h per Mandiant) | Speed of visibility |
| **Mean Time to Acknowledge (MTTA)** | < 15 min Sev-1 | Triage speed |
| **Alert volume per analyst per day** | < 50 (elite) | Sanity, not drowning |
| **False positive rate** | < 5% per rule | Quality of detection |
| **Detection coverage** (ATT&CK heatmap) | > 80% of top-50 techniques | Breadth |
| **% rules with a tested attack scenario** | > 80% | Trust |
| **% rules with version control + owner** | 100% | Auditability |
| **% rules reviewed in last 12 months** | 100% | Stale rules rot |
| **SOAR playbooks in production** | > 20 | Automation depth |
| **Time to onboard a new log source** | < 1 week | Speed of coverage |

### Storage math for SIEM

$$Storage_{GB/day} = EPS \times Avg\ Event\ Size_{KB} \times 86400 / 1024$$

Example: 10,000 EPS × 1 KB × 86400 / 1024 ≈ 845 GB/day ≈ 25 TB/month for 30-day hot retention. Cold retention multiplies this by months/years.

Splunk-style licensing: GB/day ingested. A 1 TB/day Splunk Enterprise Security deployment runs ~$300k–$500k/yr list, often negotiated lower.

### The "you are what you integrate" trap

A common SOC failure: SIEM is up, but only 40% of log sources are integrated. The most expensive problem is *not* lack of detection rules; it is **lack of data**. Before tuning rules, validate ingestion coverage.

A useful rule of thumb for **log source coverage audit**:

| Audit check | Pass criterion |
|---|---|
| All Tier-1 systems in CMDB log to SIEM | 100% |
| All EDR agents report | > 95% (informs MTTD) |
| All identity systems (AD, IdP) log | 100% |
| All cloud accounts log | 100% |
| All firewall / network sensors log | 100% of perimeter + core |
| Time sync (NTP) on all sources | 100% (off-by-1h breaks correlation) |
| Health monitoring (sourcetypes up) | > 99.5% (silent broken pipeline) |

## CISO / Risk Manager View

**The board conversation:**

1. **A SIEM is not a detection capability; it is a *platform* for detection capability.** Buying Splunk or Sentinel does not detect anything. The rules, the data, the people, and the process do.
2. **Detection coverage is a board topic, not a SOC topic.** A heatmap of "what we can detect" is now a quarterly board artifact. The board should see the red cells.
3. **Alert fatigue is the silent killer.** A SOC analyst who triages 500 alerts a day is a SOC analyst who will miss the real one. Tune, automate, retire stale rules.
4. **SOAR without detection is a robot that does nothing.** SOAR is a multiplier on whatever detection capability you have. If you have nothing, multiplying zero is still zero.
5. **MITRE ATT&CK coverage is the modern "are we good?" question.** Audit the heatmap, fund the gaps, retire the dead rules.
6. **AI / LLM in the SOC is real but early.** Useful for summarization, drafting, and natural-language hunt. Not yet trustworthy for autonomous response.

A useful ROI math:

$$Value_{SIEM} = (Incidents\ Detected \times Loss\ Avoided) - (Tooling\ Cost + Headcount)$$

A typical enterprise: 1 SOC analyst @ $200k, 1 SIEM @ $300k/yr = $500k/yr. Detection + response on a $5M average breach, at 50% probability per year, justifies the spend by orders of magnitude — **if** the detection actually fires.

**Top 5 things a CISO can do this quarter:**

1. **Audit log source coverage against the CMDB.** Identify the gap, fund the integration.
2. **Build the MITRE ATT&CK heatmap** for the top 20 adversary techniques relevant to your industry.
3. **Tune the top 10 highest-volume alerts.** Reduce noise, increase signal.
4. **Implement detection-as-code** (Git + CI + version-controlled Sigma).
5. **Add 3 SOAR playbooks** (phishing triage, identity compromise, ransomware containment) and measure time-to-respond before/after.

## Related Connections

### Parent
- [[incident-response-lifecycle-nist]] — Detection is the front door of IR

### Sibling L2
- [[change-and-configuration-management]] — Detection rules are CIs; rule changes go through CAB-lite process
- [[digital-forensics-process]] — SIEM is the data layer for forensic pivots
- [[disaster-recovery-and-bcp]] — SIEM in DR region; log retention during fail-over
- [[physical-and-environmental-security]] — Physical log sources (badge, CCTV) feed SIEM

### Sibling L3
- [[ransomware-response-playbook]] — Ransomware IoCs are detection content
- [[backup-strategies-3-2-1]] — Backup target logs feed tamper-detection
- [[security-operations-center-soc-models]] — The SOC consumes SIEM/SOAR

### Cross-domain
- [[domain-04-communication-and-network-security]] — NDR / NGFW / DNS are the largest log sources
- [[domain-05-identity-and-access-management]] — IAM events are high-signal log sources
- [[domain-06-security-assessment-and-testing]] — Red team / pen test findings drive detection content

## References

- MITRE ATT&CK — Enterprise, ICS, Mobile matrices — https://attack.mitre.org/
- Sigma Rules — https://sigmahq.io/ ; rule repository on GitHub
- Elastic Common Schema (ECS), Open Cybersecurity Schema Framework (OCSF) — vendor-neutral normalization
- Splunk Enterprise Security, Splunk SPL, Splunk SOAR (Phantom) — product references
- Microsoft Sentinel, Defender XDR, KQL — Microsoft stack
- Chronicle (Google Cloud), YARA-L — Google stack
- Sumo Logic, Panther, Devo, LogRhythm, IBM QRadar, Securonix, Exabeam — vendor references
- OpenSearch + Wazuh, Elastic Security + Sigma, Grafana + Loki — open-source stacks
- NIST SP 800-92 — *Guide to Computer Security Log Management*
- NIST SP 800-137 — *Information Security Continuous Monitoring (ISCM)*
- NIST CSF 2.0 — DE (Detect) function
- CIS Critical Security Controls v8 — Control 13 (Network Monitoring and Defense), Control 8 (Audit Log Management), Control 6 (Access Control Management)
- SOC-CMM — SOC Capability Maturity Model (by SOC Prime)
- Tidal Cyber / Picus / AttackIQ — breach and attack simulation (BAS) for detection validation
- Atomic Red Team (Red Canary) — open-source adversary emulation tests
- Caldera (MITRE) — automated adversary emulation
- "Detection Engineering" (Cubukcu, AI4SOAR) — practitioner references
- "SANS SEC555: SIEM with Tactical Analytics"
- Mandiant M-Trends (annual dwell time)
- IBM Cost of a Data Breach Report (annual)
- "The Defender's Dilemma" (Specter, et al., 2015 MITRE study) — empirical basis for detection coverage
- SANS "2024 SOC Survey"
