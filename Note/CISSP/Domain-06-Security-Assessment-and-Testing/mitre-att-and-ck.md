---
title: MITRE ATT&CK
layer: 3
domain: 6
tags: [cissp, domain-6, mitre, att-and-ck, ttps, threat-informed-defense, cyber-threat-intelligence]
related_domains: [1, 3, 7]
parent: domain-06-security-assessment-and-testing
---

# MITRE ATT&CK

## Feynman Explanation

ATT&CK is a *catalog of how real attackers actually behave*, organized as a matrix of *tactics* (the *why* of an attack step) and *techniques* (the *how*). Where the OWASP Top 10 is a list of broken things, ATT&CK is a list of *broken-into things — the chain of moves an adversary makes from "I have a phishing email" to "I own the database."* Its power is that it lets red teams describe what they're doing and blue teams describe what they're detecting in the *exact same vocabulary*. That's why it's the foundation of modern threat-informed defense.

## Technical Details

### The three matrices (and when to use each)

| Matrix | Scope | When |
|---|---|---|
| **Enterprise** | Windows, macOS, Linux, Cloud (AWS/Azure/GCP/Azure AD/Office 365), Containers, Network | Default — most engagements |
| **Mobile** | iOS, Android | Mobile-targeted operations |
| **ICS** (Industrial Control Systems) | SCADA, DCS, PLCs, HMI, engineering workstations | OT/ICS environments |

> **CISSP trap:** knowing that there is an *ICS* matrix is the testable fact. Most candidates only know Enterprise.

### The 14 Enterprise tactics (the "kill chain" generalized)

Tactics are the adversary's *objectives* in order. The full Enterprise list (v15+):

| # | Tactic ID | Tactic | Adversary goal (the "why") |
|---|---|---|---|
| 1 | TA0043 | **Reconnaissance** | Gather info on target |
| 2 | TA0042 | **Resource Development** | Build / acquire infrastructure |
| 3 | TA0001 | **Initial Access** | Get into the network |
| 4 | TA0002 | **Execution** | Run attacker code |
| 5 | TA0003 | **Persistence** | Survive reboot / re-auth |
| 6 | TA0004 | **Privilege Escalation** | Get higher permissions |
| 7 | TA0005 | **Defense Evasion** | Avoid detection |
| 8 | TA0006 | **Credential Access** | Steal creds / secrets |
| 9 | TA0007 | **Discovery** | Learn the environment |
| 10 | TA0008 | **Lateral Movement** | Pivot to other systems |
| 11 | TA0009 | **Collection** | Gather data of interest |
| 12 | TA0011 | **Command and Control** | Communicate with implants |
| 13 | TA0010 | **Exfiltration** | Steal data out |
| 14 | TA0040 | **Impact** | Disrupt, destroy, ransom |

> The left-to-right reading is *one* possible attack chain. Real adversaries skip, repeat, and re-order tactics constantly. The matrix is not linear.

### Technique, sub-technique, procedure (the hierarchy)

| Level | Definition | Example |
|---|---|---|
| **Tactic** | Adversary's goal (the *why*) | TA0006 Credential Access |
| **Technique** | *How* the goal is achieved (the *general* way) | T1110 Brute Force |
| **Sub-technique** | More specific *how* | T1110.001 Password Guessing, T1110.003 Password Spraying, T1110.004 Credential Stuffing |
| **Procedure** | The *specific* instance in the wild | "APT29 used password spraying against Office 365 in 2022" |

**T-numbers are stable identifiers.** T1566 = Phishing across the whole ATT&CK lifetime. Sub-techniques use `.NNN` suffix.

### Top techniques by tactic (illustrative cheat sheet)

| Tactic | Top techniques (T-numbers) | What they look like in logs |
|---|---|---|
| Initial Access (TA0001) | T1566 Phishing, T1190 Exploit Public-Facing App, T1078 Valid Accounts, T1133 External Remote Services | Mail gateway, WAF, auth logs |
| Execution (TA0002) | T1059 Command & Scripting, T1204 User Execution, T1047 WMIC, T1053 Scheduled Task | EDR process telemetry, Sysmon |
| Persistence (TA0003) | T1136 Create Account, T1543 Boot/Logon Autostart, T1098 Account Manipulation, T1133 External Remote Services | AD changes, scheduled task creation |
| Privilege Escalation (TA0004) | T1068 Exploit for Priv Esc, T1548 Abuse Elevation Control, T1055 Process Injection, T1134 Access Token Manipulation | EDR, LSASS access |
| Defense Evasion (TA0005) | T1027 Obfuscated Files, T1070 Indicator Removal (log clear), T1112 Modify Registry, T1562 Impair Defenses | EDR, log gaps, AV disable events |
| Credential Access (TA0006) | T1110 Brute Force, T1003 OS Credential Dumping (Mimikatz), T1555 Credentials from Password Stores, T1558 Steal/Forge Kerberos Tickets | EDR, AD events, LSASS access |
| Discovery (TA0007) | T1087 Account Discovery, T1083 File & Directory Discovery, T1057 Process Discovery, T1018 Remote System Discovery | EDR process command lines |
| Lateral Movement (TA0008) | T1021 Remote Services (RDP/SMB/WMI/WinRM), T1570 Lateral Tool Transfer, T1550 Use Alt Auth Material (Pass-the-Hash/Ticket) | EDR + auth + network |
| Collection (TA0009) | T1560 Archive Collected Data, T1005 Data from Local System, T1114 Email Collection, T1213 Data from Information Repositories | EDR, file-server audit, mail audit |
| C2 (TA0011) | T1071 Application Layer Protocol (HTTP/DNS), T1090 Proxy, T1572 Protocol Tunneling, T1095 Non-App Layer Protocol | NDR, proxy, DNS, NetFlow |
| Exfiltration (TA0010) | T1041 Exfil Over C2, T1567 Exfil to Web Service, T1052 Exfil over Physical Media, T1048 Exfil Over Alt Protocol | DLP, NDR, cloud egress logs |
| Impact (TA0040) | T1486 Data Encrypted for Impact, T1490 Inhibit System Recovery, T1485 Data Destruction, T1489 Service Stop, T1499 Endpoint DoS | EDR + backup system + service-monitor |

> CISSP expects you to know the *tactic* order (e.g., "Lateral Movement is between Discovery and Collection") and a few *iconic* techniques (T1566 Phishing, T1486 Ransomware, T1003 Credential Dumping, T1078 Valid Accounts).

### Threat-informed defense (how ATT&CK is used in practice)

| Use case | How ATT&CK is applied |
|---|---|
| **Detection engineering** | Map every SIEM rule to ≥ 1 technique; track detection coverage by tactic |
| **Gap analysis** | For each technique, ask: "Do we have a rule? an EDR signal? a process?" |
| **Red team / pen test** | Plan engagement as a sequence of techniques; report findings with T-numbers |
| **Threat intel** | Map adversary IOCs / TTPs to the matrix; share via STIX/TAXII |
| **Control assessment** | "Can we detect T1003 (LSASS dump)?" — empirical, technique-by-technique |
| **Tabletop exercises** | Pick 3–5 techniques; ask "what do we do?" |
| **Vendor evaluation** | EDR/NDR RFPs ask "which ATT&CK techniques do you cover?" |

### ATT&CK Navigator (the workhorse tool)

The ATT&CK Navigator is a web-based visualization layer over the matrix. Security teams use it to:

- **Color-code** techniques by detection coverage (green = covered, yellow = partial, red = gap)
- **Plan** adversary emulation paths
- **Compare** coverage over time / between business units
- **Export** JSON for tracking in Jira/ADO

Example coverage layer (text representation):

| Tactic | Coverage |
|---|---|
| Initial Access | 🟢 (T1566, T1190, T1078, T1133 covered) |
| Execution | 🟢 (T1059, T1204 covered) |
| Persistence | 🟡 (T1136 not detected on Linux) |
| Privilege Escalation | 🟡 (T1068 detection depends on CVE) |
| Defense Evasion | 🔴 (T1027 obfuscation + T1070 log clearing — high gap) |
| Credential Access | 🟢 (T1003 covered) |
| Discovery | 🟡 (T1087 / T1083 partial) |
| Lateral Movement | 🟢 (T1021 covered) |
| Collection | 🔴 (T1114 mailbox collection not detected) |
| C2 | 🟢 (T1071 HTTP/DNS covered) |
| Exfiltration | 🟡 (T1567 to personal cloud — DLP gap) |
| Impact | 🟡 (T1486 detection only post-encryption, not pre-staging) |

### ATT&CK Evaluations (formerly MITRE Engenuity ATT&CK Evaluations)

Independent, public tests of EDR / NDR / XDR products against emulated adversary playbooks. Results published annually as a matrix; the "Evaluations" report is one of the most-cited sources in enterprise security procurement.

| Year | Adversary emulated |
|---|---|
| 2018 | APT3 (default) |
| 2019 | APT29 (Cozy Bear) |
| 2020 | APT29, Carbanak+FIN7 |
| 2021 | Carbanak+FIN7, Wizard Spider |
| 2022 | Wizard Spider, Turla |
| 2023 | Turla, Lazarus, Vanguard Panda |
| 2024 | North Korean / Russian / Iranian / Chinese APTs |

> The Evaluations are *not* a buyer's guide — they test *detection visibility*, not prevention or operational quality. CISSP expects awareness of the Evaluations' existence and purpose.

### D3FEND (the defensive complement)

| ATT&CK | D3FEND |
|---|---|
| Adversary *offensive* techniques | Defensive *counter-techniques* |
| "How does the attacker do X?" | "How do we detect / prevent X?" |
| Adversary-centric | Defender-centric |

D3FEND is a useful adjunct for gap analysis: for every T1003 (OS Credential Dumping), D3FEND has D3-LAM (Local Account Monitoring), D3-OSM (Operating System Monitoring), D3-PM (Process Monitoring), etc. CISSP doesn't test it directly, but it appears in D3 deep-dive contexts.

### STIX / TAXII (the sharing format)

| | Purpose |
|---|---|
| **STIX 2.1** | Structured Threat Information eXpression — language for sharing IOCs and TTPs |
| **TAXII 2.1** | Trusted Automated eXchange of Indicator Information — transport for STIX |
| **MISP** | Open-source threat intel platform (alternative/companion) |

> CTI (Cyber Threat Intelligence) feeds from ISACs, vendors, and government (CISA AIS) are typically STIX/TAXII; consuming them in a TIP (e.g., MISP, Anomali) is the standard way to operationalize.

## CISO / Risk Manager View

ATT&CK is the CISO's single best tool for *translating between worlds*:

- **Board:** "We have 85% detection coverage against the techniques real attackers use."
- **Engineering:** "We have a green cell for T1059 in the Navigator."
- **Vendor:** "Show me your coverage against T1486 ransomware in the 2024 Evaluations."
- **Auditor:** "Our control set is mapped to MITRE ATT&CK techniques."

The CISO's strategic move is to publish an **ATT&CK coverage map** for the organization and update it quarterly. This single artifact changes the security conversation from "we have a SIEM" (vague) to "we detect 87% of the techniques adversaries use against our industry, with named gaps" (specific).

**Threat-informed prioritization** is the 2024–2025 board theme. The CISO should:
1. Pick the 3–5 adversary groups most relevant to the company's industry (e.g., FIN7 for retail, APT29 for government, Scattered Spider for telecoms).
2. Map their *known TTPs* (publicly available from MITRE Groups + vendor reports).
3. Compare to the company's ATT&CK coverage map.
4. Spend the next quarter's budget on the largest *delta*.

This is the difference between "we have a SOC" and "we defend against the threats that target us."

**Budget framing:** ATT&CK adoption is mostly *free* (the framework is open) but requires investment in:
- Detection engineering capability (people, training, ATT&CK Defender, SANS SEC497)
- TIP (free: MISP; commercial: Anomali, ThreatConnect, Recorded Future)
- Adversary emulation platform (free: Atomic Red Team + Caldera; commercial: SCYTHE, AttackIQ, Cymulate, Picus)
- EDR/NDR (where the Evaluations pay off)

**Compliance angle:** SOC 2 CC7.1, ISO 27001 A.8.16 (Monitoring activities), NIST CSF 2.0 DE.CM (Continuous Monitoring), and the SEC cyber-disclosure rule all benefit from ATT&CK-mapped evidence.

## Related Connections

### Parent
- [[domain-06-security-assessment-and-testing]] (D6 hub)

### L2 siblings
- [[penetration-testing-methodology]] (pentests should align to ATT&CK techniques)
- [[vulnerability-management-lifecycle]] (vuln prioritization can use technique *reachability*)
- [[logging-monitoring-and-siem]] (SIEM use cases mapped to ATT&CK; coverage metrics)
- [[security-audits-and-compliance-testing]] (audits increasingly demand technique coverage)

### Cross-domain
- [[Domain 1 — Security and Risk Management]] (threat intel drives risk treatment)
- [[Domain 3 — Security Architecture and Engineering]] (D3FEND is the defensive twin)
- [[Domain 7 — Security Operations]] (SOC detection engineering uses ATT&CK daily)

## References

- MITRE ATT&CK Enterprise v15+ (attack.mitre.org/matrices/enterprise)
- MITRE ATT&CK Mobile v15+ (attack.mitre.org/matrices/mobile)
- MITRE ATT&CK ICS v15+ (attack.mitre.org/matrices/ics)
- MITRE ATT&CK Navigator (mitre-attack.github.io/attack-navigator)
- MITRE D3FEND (d3fend.mitre.org)
- MITRE ATT&CK Groups (APT group catalog with mapped techniques)
- MITRE ATT&CK Software (malware/tool catalog)
- MITRE ATT&CK Mitigations (defensive guidance per technique)
- MITRE Engenuity ATT&CK Evaluations (annual EDR/NDR results)
- MITRE Caldera — adversary-emulation platform
- MITRE ATT&CK Threat Intelligence (CTI) repo
- Atomic Red Team (Red Canary) — technique-level test cases
- STIX 2.1 (OASIS standard) — threat intel language
- TAXII 2.1 (OASIS standard) — threat intel transport
- MISP (open-source TIP) — misp-project.org
- ATT&CK Defender (MAD / CTI certifications) — training and certification
- SANS SEC497 — Practical Open-Source Intelligence
- SANS SEC599 — Defeating Advanced Adversaries
- Verizon *DBIR 2024* — uses ATT&CK for breach-pattern analysis
- Microsoft, Mandiant, CrowdStrike annual threat reports — all use ATT&CK vocabulary
- CISA — Stop Ransomware, AA20-239A etc. — TTPs mapped to ATT&CK
- ENISA Threat Landscape (annual) — uses ATT&CK
- NIST SP 800-150 (Guide to Cyber Threat Information Sharing)
- NIST SP 800-30 Rev. 1 (Risk Assessment) — threat source characterization aligns
