---
title: Penetration Testing Methodology
layer: 2
domain: 6
tags: [cissp, domain-6, pentest, ptes, osstmm, nist-800-115, rules-of-engagement, recon, exploitation]
related_domains: [1, 3, 4, 5, 7, 8]
parent: domain-06-security-assessment-and-testing
siblings: [vulnerability-management-lifecycle, security-audits-and-compliance-testing, logging-monitoring-and-siem]
---

# Penetration Testing Methodology

## Feynman Explanation

A vulnerability scan is "run a tool and read the list." A penetration test is "pretend to be a real attacker, chain the holes, and try to actually steal the data." The point of a pen test isn't the tool output — it's the *story*: "I walked in through an unpatched VPN, escalated to domain admin using a misconfigured ACL, and exfiltrated the customer database from the internal share." That story tells the board something the scan can't: whether your *combined* defenses actually hold. The catch is rules of engagement — you have to write a contract that says "I am allowed to do X but not Y" or you'll either under-test or break the business.

## Technical Details

### Testing type matrix (CISSP exam favorite)

| Type | Info given to tester | Realism | Cost | Time | Use case |
|---|---|---|---|---|---|
| **Black-box** (zero-knowledge) | No internal docs, no credentials | High (matches external attacker) | High (more time) | Long | External perimeter, real-world threat model |
| **Grey-box** (partial-knowledge) | Limited user-level credentials, network diagram | Medium-high | Medium | Medium | Most common — simulates authenticated insider or phished user |
| **White-box** (crystal / full-knowledge) | Source code, configs, network diagrams, admin creds | Lowest realism but highest coverage | Lowest | Shortest | Code-level audit, internal controls, SDLC gate |
| **Double-blind** (black-box + team not told) | Tester gets no notice, defenders don't know | Highest realism | High coordination | Long | Red-team / adversary-emulation engagements |
| **Targeted** (known vuln) | Specific CVE, system, or scenario | Test depth on one thing | Low | Short | Validating a specific fix, regulatory retest |

### The five canonical phases (NIST SP 800-115 + PTES)

| Phase | PTES name | NIST 800-115 name | Goal | Example technique |
|---|---|---|---|---|
| 1 | Pre-engagement | Planning | Define scope, RoE, authorization | Sign "get out of jail free" letter |
| 2 | Intelligence Gathering | Discovery | Map attack surface | OSINT, Shodan, subfinder, certificate transparency |
| 3 | Threat Modeling | Attack Planning | Choose attack paths | STRIDE / PASTA / ATT&CK navigator |
| 4 | Vulnerability Analysis | Discovery (cont.) | Find exploitable flaws | Authenticated scan, manual review, fuzzing |
| 5 | Exploitation | Attack | Prove impact | Metasploit, custom exploit, C2 (Cobalt Strike, Sliver) |
| 6 | Post-Exploitation | Attack (cont.) | Show what attacker can do | Priv-esc, lateral movement, persistence, exfil |
| 7 | Reporting | Reporting | Communicate findings | Executive summary + technical report + re-test |

> PTES has 7 phases; OSSTMM has 5 channels (physical, wireless, telecommunications, data networks, human); NIST 800-115 has 4 (Planning, Discovery, Attack, Reporting). CISSP expects you to know that the *phases* are the same — what differs is the channel of attack and the level of structure imposed.

### Rules of Engagement (RoE) — the contract

| Clause | What it specifies | Example |
|---|---|---|
| Scope | In-scope IPs, apps, networks, people, time | "Production web tier, 09:00–17:00 Mon–Fri" |
| Out-of-scope | What is forbidden | "OT/ICS network, third-party SaaS, DoS" |
| Authorization | Who signed, who is the sponsor | CISO, Legal, business owner signature |
| Testing window | Calendar dates, maintenance windows | "Q1, after change-freeze ends" |
| Communication | Primary contact, escalation, daily standups | "Slack #pentest, daily 09:00 standup" |
| Notification | Will defenders be told? | "Yes (purple-team) / No (red-team)" |
| Data handling | What to do with captured data | "Encrypt at rest, destroy after 30 days" |
| Stop conditions | Hard "stop the test" triggers | "Stability event, accidental prod data exposure" |
| Insurance & liability | E&O / cyber / legal shield | $2M E&O, hold-harmless clause |
| Cleanup | Removal of artifacts, persistence, tools | "Remove all webshells, accounts, scheduled tasks" |

> A test without a signed RoE is, legally, an attack. CISSP exam scenarios often hinge on "is the test authorized?".

### Attack framework alignment (threat-informed)

Modern pen tests should align to **MITRE ATT&CK** (see [[mitre-att-and-ck]]) so findings map to adversary techniques the SOC can actually detect. For example:

| ATT&CK Tactic | Example technique | Pen-test goal |
|---|---|---|
| TA0001 Initial Access | T1190 Exploit Public-Facing App | Find unpatched web vuln |
| TA0003 Persistence | T1136 Create Account | Show you can plant a backdoor account |
| TA0004 Privilege Escalation | T1068 Exploitation for Priv Esc | Local-admin → SYSTEM → DA |
| TA0008 Lateral Movement | T1021 Remote Services (RDP/SMB/WinRM) | Pivot to second subnet |
| TA0010 Exfiltration | T1041 Exfil over C2 | Prove data can leave the network |

The adversary-emulation libraries (MITRE Evaluations, Atomic Red Team, Caldera) drive engagements that are repeatable and measurable.

### Common tools (organized by phase)

| Phase | Tools |
|---|---|
| Recon (passive) | Shodan, Censys, crt.sh, theHarvester, Maltego, OSINT framework |
| Recon (active) | nmap, masscan, rustscan, subfinder, httpx |
| Vuln ID | Nessus / Qualys, Nuclei, Burp Suite Pro, Nikto |
| Exploit | Metasploit, Cobalt Strike, Sliver, custom scripts |
| Post-ex / AD | Mimikatz, BloodHound, Rubeus, Impacket, CrackMapExec |
| Web | Burp Suite, ZAP, sqlmap, ffuf |
| Wireless | aircrack-ng, Kismet, bettercap |
| Reporting | Dradis, PlexTrac, Serpico, custom templates |

> **CISSP trap:** knowing the tool name is not a CISSP requirement. Knowing *which phase* it belongs to and *what kind of test* it's appropriate for, is.

### Output: what a pen-test report must contain

A defensible pen-test report (per PTES / NIST 800-115) has:

1. **Executive summary** — risk to the business in board-level language.
2. **Scope and methodology** — what was tested, how, and when.
3. **Findings** — one per finding: title, severity (CVSS or custom), affected asset, description, evidence (screenshots, command output), business impact, remediation recommendation, references.
4. **Attack narrative** — the chained story (initial access → priv-esc → crown jewel).
5. **Remediation roadmap** — what to fix first, in what order.
6. **Re-test report** — separate deliverable, after fixes.

### Pen-test vs. red-team (distinction CISSP loves)

| | Pen Test | Red Team |
|---|---|---|
| Goal | Find *all* vulns in scope | Achieve a specific objective (e.g., "exfil CFO email") |
| Scope | Bounded, time-boxed, explicit | Objective-bound, may use any vector |
| Reporting | Comprehensive list | Narrative + objective achievement |
| Detection by blue team | Often accepted / expected | Goal is to *avoid* detection |
| Cadence | Annual | Continuous / quarterly |
| Output metric | # vulns, severity distribution | Time-to-objective, time-to-detect |

Red team is covered in [[red-team-vs-blue-team-vs-purple-team]].

## CISO / Risk Manager View

A pen test is the **board's most expensive truth-telling event** in the security program. It is also the most over- and under-used. Two strategic points the CISO must own:

**1. The scope question.** A pen test is only as good as its scope. A "black-box web app pen test" of the public marketing site is theater — the real risk is the API tier or the internal ERP. The CISO must personally approve scope and force the conversation *"what are our crown jewels?"* with business owners. Scope creep into "everything" is the other trap: pen tests become long, expensive, and produce reports no one reads.

**2. The follow-through question.** Pen-test reports routinely recommend fixes that are not made; the next year's test finds the same things. **Mean Time to Remediate (MTTR) of pen-test findings** is the right board KPI. Industry MTTR for High findings from a third-party test: ≤ 30 days is mature, ≤ 90 days is acceptable, > 90 days is negligent.

**Budget framing:** A reputable external pen test for a mid-size enterprise is $50K–$250K depending on scope and depth (web, network, mobile, cloud, social). It is one of the highest-confidence inputs to the risk register and is a *mandatory* input to SOC 2, PCI DSS, ISO 27001, and most cyber-insurance applications. **A pen test that has no follow-up is wasted spend** — allocate the fix budget in the same fiscal cycle.

**Red team / purple team** is a *complement* to a pen test, not a replacement. Pen tests are breadth; red team is depth on objective. The CISO's mature program has both.

## Related Connections

### Sibling L2 deep-dives
- [[vulnerability-management-lifecycle]] (pen-test findings feed the vuln queue)
- [[security-audits-and-compliance-testing]] (pen-test is a SOC 2 / PCI / ISO 27001 evidence item)
- [[logging-monitoring-and-siem]] (blue-team detection-rate is measured by red team)

### L3 specifics
- [[owasp-top-10-and-web-app-testing]] (the canonical web pen-test checklist)
- [[mitre-att-and-ck]] (technique mapping for modern pen tests)
- [[red-team-vs-blue-team-vs-purple-team]] (offensive exercises are a superset of pen tests)

### Cross-domain
- [[Domain 1 — Security and Risk Management]] (pen test = risk-identification mechanism)
- [[Domain 3 — Security Architecture and Engineering]] (architectural flaws surface here)
- [[Domain 4 — Communication and Network Security]] (network pen test is core deliverable)
- [[Domain 5 — Identity and Access Management]] (IAM is the most-tested control)
- [[Domain 7 — Security Operations]] (blue team consumes pen-test findings)
- [[Domain 8 — Software Development Security]] (web/API pen test for app release gate)

## References

- NIST SP 800-115, *Technical Guide to Information Security Testing and Assessment* (2008)
- PTES — Penetration Testing Execution Standard (pentest-standard.org)
- OSSTMM 3 — Open Source Security Testing Methodology Manual (ISECOM, 2010)
- PCI DSS v4.0 §11.4 (penetration testing)
- OWASP Testing Guide (WSTG) v4.2
- OWASP ASVS 4.0
- MITRE ATT&CK Enterprise v15+ (technique-aligned testing)
- MITRE ATT&CK Evaluations (empirically tested EDR / NDR)
- SANS SEC560 / GPEN — network pen testing
- SANS SEC542 / GWAPT — web app pen testing
- CREST — Council of Registered Ethical Security Testers (UK, widely adopted standard)
- NIST SP 800-53A, *Assessing Security and Privacy Controls* — penetration test as an assessment method
- Lockheed Martin Cyber Kill Chain (precursor to ATT&CK)
- Common Weakness Enumeration (CWE) — categorization of vuln classes
- AICPA SOC 2 — pen-test evidence per Common Criteria CC7.1
- MITRE Caldera / Atomic Red Team / Vectr — adversary-emulation platforms
