---
title: Ransomware Response Playbook
layer: 3
domain: 7
tags: [cissp, domain-7, ransomware, playbook, ofac, ransom, ir, csirt, recovery, decryptor, negotiation, double-extortion]
related_domains: [1, 5, 6, 7]
parent: incident-response-lifecycle-nist
---

# Ransomware Response Playbook

## Feynman Explanation

Ransomware is the most consequential incident a typical organization will face: in a few hours an attacker can encrypt every file share, exfiltrate customer data, and demand seven figures in cryptocurrency. The playbook is the pre-written answer to "what do we do in the first 60 minutes, who has authority, do we pay, do we restore, do we notify, and how do we keep the lights on while this is happening?" Rehearse it, or rehearse the breach in front of a regulator.

## Technical Details

### NIST CSF 2.0 ransomware response — function by function

| Function | Ransomware-specific action |
|---|---|
| **Govern** | Pre-authorized playbook, named decision-makers, legal pre-brief |
| **Identify** | Asset inventory (what can be encrypted?), data classification (what gets exfiltrated?), third-party map |
| **Protect** | EDR everywhere, MFA on all remote + admin, immutable backups, segmentation, least privilege, disable macros/RDP, deny email-forwarding, application allow-list |
| **Detect** | EDR + NDR + SIEM tuned for ransomware IoCs (volume shadow copy delete, mass file rename, unusual SMB write, lolbin abuse) |
| **Respond** | Isolate, eradicate, decrypt (if lucky), rebuild, communicate, decide on payment |
| **Recover** | Restore from immutable, air-gapped, tested backups; validate integrity; staged re-introduction |

### The 60-minute decision tree (IC view)

| T+ min | Action | Owner |
|---|---|---|
| 0–5 | Confirm encryption, capture ransom note, do **not** reboot | Tier 1 analyst |
| 5–15 | Declare Sev-1, page IC + CISO + IT director + legal | Tier 1 / manager |
| 15–30 | IC convenes war room (in person or bridge); legal + cyber insurance + comms looped in | IC |
| 30–60 | Decide: containment strategy (full network isolation vs. surgical), comms posture, regulator clock, law enforcement contact | IC + exec sponsor |
| 60–180 | Begin eradication (image, hash, IOC push, creds rotation); restore from backup in parallel if available | IR + ops |
| 180–480 | Customer / regulator / media communication (per legal hold) | Comms + legal |
| Day 1–7 | Recovery, validation, monitoring, after-action | All |

### Detection signatures (what to alert on)

| IoC | Source | Rule logic |
|---|---|---|
| **Shadow copy deletion** (`vssadmin delete shadows /all`) | EDR / Sysmon | Process + command-line pattern |
| **Mass file rename** (`.locked`, `.crypt`, `.encrypted`) | EDR | File create pattern, high rate |
| **Ransom note file create** (`README.txt`, `HOW_TO_DECRYPT.html`) | EDR / file-server | File-name + path pattern |
| **SMB write spike from one source** | NDR / file-server log | Volume threshold per minute |
| **Unusual outbound large transfer** (data exfil) | NDR / proxy | Bytes per flow > baseline × 5 |
| **Disabled security tools** (`net stop`, `Set-MpPreference -DisableRealtimeMonitoring $true`) | EDR / Sysmon | Service stop + cmd pattern |
| **Living-off-the-land binaries** (PowerShell, PsExec, WMI, RDP) | EDR | Process tree + parent/child |
| **Cobalt Strike / Beacon beaconing** | NDR | JA3, beacon interval, known CS domains |
| **New admin account created** | IAM | User-create + privilege grant |
| **Group policy push to disable AV** | AD / EDR | GPO change + AV disable |

### Containment options (and when to pick)

| Strategy | Best for | Risk |
|---|---|---|
| **Full network isolation** (per-VLAN shut, core switch quarantine) | Encryption spreading, no business alternative | Hard business stop |
| **Surgical host isolation** (EDR network-contain) | Confined spread, business must keep running | Spreads to non-EDR assets |
| **Account lockdown** (disable all admin accounts, force password reset via known-good DC) | Credential-driven attack | Recovery time, breaks SSO |
| **Internet egress block** (default-deny, allowlist only) | C2 callbacks, exfil | Breaks cloud services, update servers |
| **Power off** (last resort) | Active destruction, no time to image | Loses memory evidence |

### Eradication checklist (mandatory)

1. Identify **patient zero** (most often RDP, phishing, or unpatched edge device).
2. Image all affected systems (forensic capture, chain of custody — see [[digital-forensics-process]]).
3. Pull and hash **all** ransomware binaries, push to EDR blocklist and threat intel.
4. Capture **all** IOCs (file hashes, C2 domains/IPs, mutexes, registry keys, services) — feed to [[siem-soar-and-detection-engineering]].
5. Rotate **all** credentials the attacker may have touched (every AD account, every API key, every SaaS token, every SSH key).
6. Revoke **all** OAuth tokens and refresh tokens.
7. Reimage (do not "clean") every affected host.
8. Patch the exploited CVE across the **entire** fleet, not just the affected host.
9. Re-run red team / purple team exercise on the same vector in 30 days.

### Recovery: restore vs. decrypt vs. rebuild

| Option | When | Cost | Risk |
|---|---|---|---|
| **Restore from immutable backup** | Backup exists, RPO acceptable, validated | Operational time | If backups infected (rare with immutable), restore spreads again |
| **Decrypt with published decryptor** | Known family, free key available (NoMoreRansom project) | Free, slow | Data may be partially corrupt; no guarantee |
| **Pay ransom + use attacker decryptor** | All else fails, business cannot survive without data | $$$$, decryptor slow, no guarantee, OFAC risk | Decryptor is buggy; ~30–40% success rate; marks you as payer |
| **Rebuild from scratch** | Data is reconstructable, business can pause | Operational time | Loss of historical data |

### The ransom payment decision (legal & ethical)

This is a **C-suite + legal + board** decision, made in the first 24–48 hours. The decision tree:

```
                       Is data recoverable from backup?
                                │
                ┌───────────────┴───────────────┐
                Yes (validated)                 No
                │                               │
        Restore from backup              Is data critical for survival?
        (DO NOT PAY)                     │
                                  ┌──────┴──────┐
                                  No           Yes
                                  │             │
                          Accept loss.    Is decryptor publicly
                          Document.       available? (NoMoreRansom)
                                                │
                                       ┌────────┴────────┐
                                       Yes              No
                                       │                │
                                   Use free          Sanctions / OFAC check
                                   decryptor         on threat actor
                                                          │
                                              ┌───────────┴───────────┐
                                              Sanctioned              Not sanctioned
                                                    │                       │
                                              Cannot pay legally     Negotiation &
                                                    │                  decision
                                              Declare total loss
                                              & rebuild
```

**Critical legal constraints (US-centric, FYI — consult counsel):**

| Constraint | Detail |
|---|---|
| **OFAC sanctions** | US Treasury (OFAC) sanctions specific threat actors. Paying a sanctioned actor is a federal violation. (E.g., 2020 advisory, 2024 update.) |
| **Fincen reporting** | Suspicious Activity Report (SAR) within 30 days if payment made |
| **OFAC licensing** | Even for non-sanctioned actors, OFAC can require a license pre-payment |
| **State laws** | Several US states (NY, NC, FL) restrict ransom payment by public entities or require disclosure |
| **EU** | TIBER-EU, NIS2 reporting; ransom payment itself not banned but disclosure required |
| **Cyber insurance** | Many policies now exclude ransom OR require carrier pre-approval; some require use of approved negotiation firm |
| **Securities law** | Public companies must disclose material cyber incidents within 4 business days (SEC 2023 rule) |

### Communication matrix (who, when, what, by whom)

| Audience | When | Channel | Owner | Tone |
|---|---|---|---|---|
| Internal staff | Hour 1 | All-hands call + email | Comms | Calm, factual, "we are responding" |
| Board / exec | Hour 1 | Direct call | CISO + CEO | Realistic, options + recommendation |
| Customers | Hour 24–72 | Email + status page | Comms + legal | Apologetic, factual, no speculation |
| Regulator (GDPR / NIS2) | 72 h | Filing | Legal / DPO | Statutory, formal |
| Regulator (SEC, sectoral) | 4 business days (US public co.) | 8-K filing | Legal | Statutory |
| Law enforcement (FBI / IC3, NCSC, Europol) | 24 h | Direct | Legal | Cooperative |
| Cyber insurance carrier | Hour 1 | Hotline | CISO + risk | Factual, full disclosure |
| Payment processor / bank | Hour 24 | Direct | Treasury | If ransom considered |
| Media | 48–72 h | Press release | PR | Single spokesperson, no speculation |
| Suppliers / partners | 48 h | Direct email | Procurement + legal | What we know, what we need from them |

### Decryption reality (NoMoreRansom, public decryptors)

| Family | Decryptor available | Source |
|---|---|---|
| GandCrab | Yes (all versions) | NoMoreRansom |
| Shade / Troldesh | Yes | Kaspersky |
| Dharma / CrySiS | Partial | Various |
| LockBit | No (until 2024 LE-led) | — |
| BlackCat / ALPHV | Partial (2023 LE) | FBI |
| REvil / Sodinokibi | Partial (2022 LE) | NoMoreRansom |
| Conti | No | — |
| Akira | No | — |

Rule of thumb: ~30% of attacks involve a family with a known decryptor, but the decryptor may not recover all data.

### The cyber-insurance / OFAC trap

A 2023–2024 trend: insurance carriers require pre-approval and a no-sanctioned-actor attestation **before** paying. Skipping the check = policy void. The CISO + legal + carrier in concert, not the CISO alone.

### Cost benchmarks (IBM Cost of a Data Breach 2024)

| Metric | Average |
|---|---|
| Detection + escalation cost | $1.6M |
| Notification cost | $0.4M |
| Post-breach response | $1.7M |
| Lost business cost | $1.6M |
| **Total average per breach** | **$5.46M** |
| Ransomware subset | ~$5.1M excluding ransom |
| Avg ransom paid (when paid) | ~$0.5–1.0M; large incidents > $5M |
| Avg downtime for ransomware | 24+ days |
| Cost reduction with tested IR + AI | ~$1.5M |

### The "no-pay" path: rebuild cost vs. ransom

| Decision | Direct cost | Indirect cost | Time |
|---|---|---|---|
| **Pay ransom** | $$ (ransom) + $$$ (forensics + legal) | Reputation (paid = target), no guarantee, OFAC risk | Days to weeks |
| **Rebuild from backup** | $$$ (ops + rebuild) + $-$$$ (forensics) | Reputation (no data loss) | Days to weeks |
| **Rebuild from scratch** | $$$$ (rebuild) | Reputation (data loss), customer churn | Weeks to months |

The math usually favors restore-from-backup when backup is intact, **even if the ransom is "low."**

### Post-incident (the 30/60/90 day watch)

| Window | Action |
|---|---|
| 0–30 days | Enhanced monitoring on affected systems; threat-hunt for the same actor returning |
| 30–60 days | Run red team / purple team exercise on the same vector |
| 30 days | Cyber insurance debrief + carrier recommendations |
| 30–45 days | Board after-action report |
| 60 days | External security assessment / IR retainer renewal |
| 90 days | Update playbook; close CAPA items |

### Insurance — what to read

A CISO should read the **policy** for these specific terms:

| Clause | Question |
|---|---|
| Ransom payment | Covered? Excluded if sanctioned actor? Pre-approval required? |
| Business interruption | Triggered only by covered cyber event? Waiting period? |
| Forensic cost | Capped? Vendor-approved list? |
| Notification cost | Capped? Per-record or aggregate? |
| Reputational harm | Covered? How measured? |
| War exclusion | Does state-actor exclusion apply? (Hiscox, Merck v. ACE 2018 case) |
| Subrogation | Insurer's right to sue — affects disclosure |
| Retro date | When did the policy start; is a long-dormant infection covered? |
| Panel counsel | Must use carrier's lawyers? |

## CISO / Risk Manager View

**The board conversation:**

1. **Ransomware is now a quarterly board topic, not a "what if."** In 2024, more than 50% of incidents at large enterprises involved ransomware or extortion. The board's job is to ensure the playbook is rehearsed, the decision authority is pre-delegated, and the insurance is in order.
2. **The decision to pay or not is a board call, not a CISO call.** Pre-delegate authority: if recovery is impossible without paying and the data is existential, the CEO + CISO + General Counsel can authorize up to a stated threshold. Above the threshold, board approval.
3. **The cost of *not* having immutable, tested, offsite backups is the cost of the ransom plus the cost of the breach.** The cost of *having* them is a small fraction of one quarter's cyber budget.
4. **OFAC compliance is a legal line item.** A CISO who pays a sanctioned actor without a license exposes the company and themselves to federal enforcement.
5. **The 24/7/365 IR retainer + cyber insurance is the modern minimum.** The first 24 hours of a ransomware incident are worth more than the entire annual retainer.

A useful board framing:

$$Expected\ loss\ from\ ransomware = ARO_{ransomware} \times (Ransom + Downtime + Notification + Reputation)$$

A mature program targets $ARO_{ransomware} \le 0.1$ (one incident per decade) via controls (segmentation, immutable backup, MFA, EDR, training, patch). An immature program sees $ARO_{ransomware} \approx 0.3$–$0.5$/yr and pays the consequences in premiums, retention, and ultimately ransom.

**Top 5 things a CISO can do this quarter:**

1. **Validate immutable, air-gapped, tested backups** (see [[backup-strategies-3-2-1]]).
2. **EDR on every endpoint, including servers and cloud workloads.**
3. **MFA on every remote-access path, no exceptions.**
4. **Rehearse the playbook** — tabletop with exec sponsor in the room.
5. **Read the insurance policy** with the broker.

## Related Connections

### Parent
- [[incident-response-lifecycle-nist]] — Ransomware is a Sev-1 IR scenario

### Sibling L2
- [[disaster-recovery-and-bcp]] — Recovery path when backups are intact
- [[change-and-configuration-management]] — Emergency patching / emergency change procedure
- [[digital-forensics-process]] — Evidence collection during eradication
- [[physical-and-environmental-security]] — On-prem ransomware propagation often physical/RDP-driven

### Sibling L3
- [[backup-strategies-3-2-1]] — The single most important pre-emptive control
- [[siem-soar-and-detection-engineering]] — Detection content for ransomware IoCs
- [[security-operations-center-soc-models]] — SOC is the first responder

### Cross-domain
- [[domain-01-security-and-risk-management]] — Risk acceptance for ransom, regulatory notification policy
- [[domain-05-identity-and-access-management]] — Account disable, password reset, MFA enforcement
- [[domain-06-security-assessment-and-testing]] — Pen test for ransomware readiness

## References

- NIST SP 800-61 Rev. 2 — *Computer Security Incident Handling Guide*
- NIST SP 800-184 — *Guide for Cybersecurity Event Recovery*
- NIST CSF 2.0 (2024) — Govern, Identify, Protect, Detect, Respond, Recover
- CISA #StopRansomware Guide (2023 update) — Joint advisory, NSA + CISA + FBI + CCCS + NCSC-NZ + ACSC + JPCERT/CC
- NoMoreRansom project — https://www.nomoreransom.org/ (free decryptors)
- US Treasury OFAC Advisory 2020 (and 2024 update) — *Sanctions Compliance Guidance for the Virtual Currency Industry*
- FinCEN — Advisory on Ransomware and the Use of the Financial System (2020, 2023 update)
- SEC Final Rules — Cybersecurity Risk Management, Strategy, Governance, and Incident Disclosure (2023, effective 2023-12-26)
- EU NIS2 Directive (2022) — 24 h early warning, 72 h notification
- GDPR Article 33 — 72 h notification
- ISO/IEC 27035-1:2016 / 27035-2:2016 — Incident management
- (ISC)² CISSP CBK 5th ed. — Domain 7
- Mandiant M-Trends (annual)
- IBM Cost of a Data Breach Report (annual)
- Coveware Quarterly Ransomware Report (public, free)
- Palo Alto Unit 42 Ransomware Threat Report
- "Ransomware: A Survival Guide" (Falcon-Sensor) — non-commercial recovery field guides
- Hiscox Cyber Readiness Report (annual)
- Coalition Cyber Claims Report (annual)
