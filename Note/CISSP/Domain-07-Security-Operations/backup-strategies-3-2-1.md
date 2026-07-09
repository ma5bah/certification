---
title: Backup Strategies — 3-2-1, Immutability, and Air-Gapping
layer: 3
domain: 7
tags: [cissp, domain-7, backup, 3-2-1, immutable, air-gap, cdp, snapshot, draas, restore-test, vtl, s3-object-lock, worm]
related_domains: [1, 2, 4, 6]
parent: disaster-recovery-and-bcp
---

# Backup Strategies — 3-2-1, Immutability, and Air-Gapping

## Feynman Explanation

A backup is a copy of your data. A recovery is the ability to put that copy back. Most of what people call "backup" is only the first half. The 3-2-1 rule — three copies, two media types, one offsite — is the discipline that makes the first half reliable; immutability and air-gapping are the discipline that makes it ransomware-resistant; testing is the discipline that turns the file into a capability. Skip any one, and the other two do not save you.

## Technical Details

### The 3-2-1 family of rules (progression over time)

| Rule | Components | Origin |
|---|---|---|
| **3-2-1** | 3 copies, 2 media, 1 offsite | Peter Krogh (2005), photo industry |
| **3-2-1-1** | + 1 immutable copy | Modern, post-ransomware |
| **3-2-1-1-0** | + 0 errors on last restore | Veeam / US-CERT / NIST CSF |

| Component | Operational meaning | Implementation |
|---|---|---|
| **3 copies** | Production + 2 backup copies | Production + on-prem backup + offsite backup |
| **2 media types** | Different attack surfaces | Disk + tape, or disk + cloud (object lock) |
| **1 offsite** | Geographic separation (≥ 100 km) | Cloud region, second datacenter, BaaS |
| **1 immutable** | Cannot be modified or deleted for a retention window | Object lock, WORM tape, air-gapped |
| **0 errors** | Last restore test had zero integrity errors | Quarterly restore drill, hash verify |

### Backup types (operational trade-off)

| Type | Backup speed | Restore speed | Storage cost | Best for |
|---|---|---|---|---|
| **Full** | Slow | Fast | High (1×) | Weekly baseline |
| **Incremental** | Fast | Slow (need all in chain) | Low | Daily |
| **Differential** | Medium | Medium (full + last diff) | Medium | Daily with weekly full |
| **Synthetic full** | Synthesized | Fast | Medium | Modern; dedup-aware |
| **Snapshot (CoW / RoW)** | Fast (instant) | Very fast (mount) | Disk | VMs, databases |
| **CDP** (Continuous Data Protection) | Continuous | Any point | High | Mission-critical DBs |
| **Application-aware** (VADP, VSS, RMAN-aware) | Medium | Fast, consistent | Medium | Databases, Exchange, AD |

**Restore time vs. backup time:**

- **Incremental** = fast backup, slow restore (must reconstruct the chain in order).
- **Differential** = slower backup, faster restore (full + last differential).
- **Full** = slowest backup, fastest restore (one file).
- **Snapshot** = fast both (instant copy, instant mount).
- **CDP** = any-point-in-time, but expensive and complex.

### Immutability — what it means and how to implement

| Technology | Mechanism | Erasure horizon |
|---|---|---|
| **Object lock (S3, Azure Blob, GCS)** | Bucket-level WORM, retention period enforced by service | Configurable, days to centuries |
| **WORM tape (LTO-WORM)** | Cartridge physically write-once | 30+ years |
| **Linux `chattr +i`** | Immutable attribute, root required to unset | Until root unsets (not real immutability) |
| **Enterprise backup software WORM** (Veeam, Commvault, Rubrik, Cohesity) | Application-level WORM with retention lock | Configurable |
| **Snapshot-based immutability** | Snapshot cannot be deleted before retention | Per snapshot |
| **Compliance mode vs. Governance mode** | Compliance = cannot be shortened even by admin; Governance = can with elevated role | Choose based on risk appetite |

**CISSP trap:** immutability requires the *credentials used by the adversary* to be **unable to delete** the backup. If the attacker compromises the AWS root account, even object lock can be bypassed unless it is in compliance mode AND the root credentials are in a separate break-glass vault.

### Air-gapping — physical or logical isolation

| Type | Mechanism | Threat mitigated |
|---|---|---|
| **Physical air-gap** | Tape in a vault; or disk in a powered-off enclosure | Network-borne ransomware, compromised admin |
| **Logical air-gap** | Backup account cannot reach production network via firewall | Compromised prod admin |
| **Time-based air-gap** | Backup target is only online during backup window (e.g., 02:00–04:00) | Persistent attacker presence |
| **Immutability-as-air-gap** | Object lock + MFA-protected bucket; adversary cannot delete even with creds | Account compromise |
| **Quorum-based deletion** (e.g., N-of-M approvers) | Single admin cannot delete | Single insider |

**Modern recommendation:** combine — immutable object lock + bucket isolated from production VPC + separate IAM identity (in a separate account / OU) + MFA enforced on every delete attempt + alerting on any delete attempt.

### Backup target comparison

| Target | Cost/GB/mo | RTO | RPO | Immutability | Air-gap | Notes |
|---|---|---|---|---|---|---|
| **Local disk (DAS, NAS)** | $0.05–0.20 | Minutes | Minutes–hours | Via software | None | Fast, but co-located risk |
| **Tape (LTO-9)** | $0.005–0.02 | Hours | 24 h (typical) | WORM (LTO-WORM) | Physical (offsite rotation) | Cheap, slow, manual |
| **VTL (virtual tape library)** | $0.05–0.15 | Minutes | Hours | Software | Software | Tape workflow, disk backend |
| **BaaS / Cloud (B2, S3 IA, Azure Cool)** | $0.01–0.04 | Minutes–hours | Minutes | Object lock | Logical (separate account) | Modern default |
| **DRaaS** | $0.10–0.50 | Minutes | Minutes | Object lock + replication | Logical | Active DR, not just backup |
| **Cold archive (Glacier Deep Archive)** | $0.00099 | Hours–12 h | Hours | Object lock | Logical | Compliance retention |

### Storage cost math (rule of thumb)

$$Monthly\ Cost = Storage_{GB} \times \$/GB + Egress_{GB} \times \$/GB + Request\ Cost$$

| Service | Storage $/GB/mo | Egress $/GB | Notes |
|---|---|---|---|
| S3 Standard | $0.023 | $0.09 | Frequent access |
| S3 Standard-IA | $0.0125 | $0.09 | ≥ 30 day, ≥ 30 KB |
| S3 Glacier Instant | $0.004 | $0.09 | ms retrieval |
| S3 Glacier Flexible | $0.0036 | $0.09 | minutes–hours |
| S3 Glacier Deep Archive | $0.00099 | $0.09 | 12+ hours |
| Azure Blob Hot | $0.0184 | $0.08 | |
| Azure Blob Cool | $0.010 | $0.08 | ≥ 30 day |
| Azure Archive | $0.00099 | $0.10 | hours |
| Backblaze B2 | $0.006 | $0.01 | Cheapest egress |
| Wasabi | $0.0069 | $0 (free egress) | Hot tier |
| Veeam Hardened Repository | Free (own disk) | Free | Linux + XFS, immutable |

**Worked example:**

- 100 TB production data.
- 30 daily incrementals + 4 weekly synth-fulls = ~3× raw = 300 TB.
- 30-day retention in S3 Standard-IA: 300,000 GB × $0.0125 = **$3,750 / month storage**.
- Egress for full restore once per quarter: 100 TB × 1024 = ~100,000 GB × $0.09 = **$9,000** (one-time per quarter).
- Plus 1 immutable copy in another region (cross-region replication, +20% storage cost = $750) = total ~$4,500 / month.

### Encryption of backups

| Layer | Mechanism | Note |
|---|---|---|
| **In-flight** | TLS 1.2+ to backup target | Standard |
| **At-rest, server-side** | AES-256, KMS-managed | Default on cloud |
| **At-rest, client-side** | Encrypt before send (e.g., Duplicity, restic) | Protects against cloud-provider compromise |
| **Application-level** | DB-native (TDE, MySQL AES) | Defense in depth |
| **Key management** | Separate from backup account (HSM, external KMS) | If the same key is in the same account, immutable storage is moot |

**CISSP trap:** if the attacker compromises the cloud admin and the KMS key is in the same account, server-side encryption offers no protection. Client-side encryption with key held outside the cloud account is the only true defense.

### Backup of the backup (metadata, catalogs, indexes)

| Artifact | Risk if lost | Mitigation |
|---|---|---|
| Backup catalog / index | Cannot find a backup to restore | Replicated, immutable |
| Backup software config (Veeam, Commvault) | Cannot run a restore | Off-host, versioned, immutable |
| Encryption keys | Cannot decrypt | Escrow, split-knowledge, HSM |
| Bootstrap media (boot ISO, agent) | Cannot connect to backup target | Off-host, versioned, hash-verified |
| Service account credentials | Backup cannot run | PAM-managed, rotated, break-glass copy |

### The "restore test" (the proof)

A backup is not a backup until it has been restored. The restore test hierarchy:

| Test | What it validates | Frequency |
|---|---|---|
| **Hash compare** | File integrity post-restore | Per restore |
| **Restore single file** | Catalog, dedup, decryption | Weekly (random sample) |
| **Restore single VM / app** | Full chain, RTO estimate | Monthly |
| **Restore to clean room** | Clean environment, no contamination | Quarterly |
| **Restore to DR site** | End-to-end RTO/RPO | Semi-annual |
| **Full failover test** | Process + tooling | Annual |
| **Adversary simulation** (e.g., "inject ransomware, see if we can restore") | Real-world resilience | Annual |

**Math:**

$$Restore\ Success\ Rate = \frac{Successful\ restores\ in\ last\ N\ tests}{N}$$

Target: 100% over rolling 12 months on Tier-1 systems.

### Recovery time vs. cost (the decision)

For a given RTO, recovery infrastructure has a cost. A useful heuristic:

| RTO target | Approach | Cost |
|---|---|---|
| Minutes | Active/active DR, sync replication, runbook on phone | $$$$ |
| < 1 hour | Warm DR, CDP, hot standby | $$$ |
| 1–4 hours | Pilot light + warm standby | $$ |
| 4–24 hours | Backup-restore from cloud | $ |
| 24+ hours | Tape restore | ¢ |

A 4-hour RTO system with $10M/hr of downtime justifies a warm DR. A 24-hour RTO system with $5k/hr of downtime does not.

### Ransomware-specific backup design (NIST + CISA + CIS)

| Control | Implementation |
|---|---|
| **Immutable, air-gapped backup** | Object lock + separate account + MFA on delete |
| **3-2-1-1-0** | Three copies, two media, one offsite, one immutable, zero errors |
| **Tested restore** | Quarterly full restore test on Tier-1 |
| **Offline / out-of-band credentials** | Backup admin creds in a separate vault, not in AD |
| **Backup-software patch** | Backup server patched within SLA (often a forgotten backdoor) |
| **Backup-software EDR** | Backup server in scope of EDR + monitoring |
| **Network segmentation** | Backup target on isolated VLAN, no internet egress |
| **Behavioral alerting** | Alert on mass file delete on backup target, or new backup account |
| **Restore validation** | Restored data scanned for malware before re-introduction |

### Backup operations KPIs (board-reportable)

| KPI | Target | Why |
|---|---|---|
| **Backup success rate** | ≥ 99.5% | Failed backups are a future restore failure |
| **% Tier-1 systems with tested restore in last 90 days** | 100% | The proof is in the test |
| **Mean restore time** (per Tier) | ≤ RTO | Operational reality |
| **RPO achieved** | ≤ RPO target | Data loss is binary |
| **Immutability coverage** | 100% of Tier-1, 80% of Tier-2 | Ransomware resilience |
| **Offline / air-gap coverage** | ≥ 1 immutable copy | The last line of defense |
| **Time since last restore drill** | < 90 days for Tier-1 | The drill is the muscle memory |

### Common backup anti-patterns (a CISO should know)

| Anti-pattern | Why it's bad | Fix |
|---|---|---|
| Backup to the same disk as production | Disk failure loses both | Separate target, separate media |
| Backup account is a domain admin | Ransomware uses it to delete backups | Dedicated, least-privilege service account |
| Backup target on the same VLAN as prod | Adversary pivots via compromised prod | Isolated VLAN / VPC / account |
| Object lock but root key in same account | Root compromise deletes everything | Object lock + compliance mode + key in external HSM |
| No restore test for 18 months | "Backup" is Schrödinger's data | Quarterly drill on Tier-1 |
| Tape not rotated offsite | Site loss loses tape too | Weekly rotation to vault |
| No monitoring on backup target | Adversary sits there for months | SIEM ingest of backup logs, behavioral alerts |
| One admin can do everything | Single insider wipes backup | Quorum / MFA on destructive ops |
| No alerting on missing backup | Silent failure | Per-job success/failure alerts, daily summary |

## CISO / Risk Manager View

**The board conversation:**

1. **Backups are the only control that directly bounds the cost of a ransomware attack.** EDR, MFA, segmentation reduce the *probability*; backups reduce the *consequence*. Without tested, immutable backups, every other control is a probabilistic hedge.
2. **Immutable + air-gapped + tested is the trifecta.** Pick any one, the adversary can defeat. Pick all three, the cost of attack is higher than the cost of payment.
3. **Restore is a capability, not a feature.** A backup product that has never been restored is a feature. A restore that has worked in production is a capability. The difference is the test.
4. **Backup budget is operational, not capital.** Most boards treat backup as "set and forget" and starve the restore-test budget. A tested restore costs a few engineer-hours per quarter; an untested backup costs the company.
5. **Ransomware actors target backups specifically.** Modern ransomware (Conti, LockBit, BlackCat) has a built-in module to find and delete backup targets. The CISO must assume the adversary is looking.

A useful board framing:

$$Risk_{ransomware\ loss} = ARO \times (Ransom\ + Downtime\ + Reputation)$$

Immutable + air-gapped + tested backups reduce the ransom line toward zero. A CISO who can demonstrate the trifecta on Tier-1 systems can credibly tell the board: "We do not pay ransoms."

**Top 5 things a CISO can do this quarter:**

1. **Inventory every backup target**, including the shadow IT one. (90% of organizations have an un-inventoried backup target.)
2. **Validate immutability and air-gap on every Tier-1 backup.** Test it — try to delete, observe that you cannot.
3. **Run a full restore drill on a Tier-1 system.** Document the actual RTO, compare to the RTO target.
4. **Move backup credentials out of the production IAM** into a separate, MFA-gated vault.
5. **Send backup-target logs to the SIEM** and alert on any delete attempt.

## Related Connections

### Parent
- [[disaster-recovery-and-bcp]] — Operational DR view, RTO/RPO, sites

### Sibling L2 (D7)
- [[incident-response-lifecycle-nist]] — Restore is the recovery step of IR
- [[change-and-configuration-management]] — Backup software config is a CI
- [[digital-forensics-process]] — Restore path must be free of malware (scan before re-introduction)

### Sibling L3
- [[ransomware-response-playbook]] — Backup is the #1 ransomware defense
- [[siem-soar-and-detection-engineering]] — Backup logs feed detection of adversary tampering
- [[security-operations-center-soc-models]] — SOC monitors backup health

### Cross-domain
- [[business-continuity-and-disaster-recovery]] — D1 policy and BIA drive backup tiering
- [[domain-02-asset-security]] — Data classification drives what gets backed up most
- [[domain-04-communication-and-network-security]] — Network segmentation isolates backup traffic
- [[domain-06-security-assessment-and-testing]] — Penetration test should include backup-target attack path

## References

- Peter Krogh — *The DAM Book: Digital Asset Management for Photographers* (origin of 3-2-1)
- Veeam — *3-2-1-1-0 Rule* (modern extension)
- NIST SP 800-34 Rev. 1 — IT Plans and Capabilities
- NIST SP 800-184 — Guide for Cybersecurity Event Recovery
- NIST CSF 2.0 — PR.IP (Information Protection Processes), PR.DS (Data Security), RC.RP (Recovery Planning)
- CISA #StopRansomware Guide (2023 update)
- US-CERT — Data Backup Options (TA02-Backup)
- ISO/IEC 27001:2022 Annex A.8.13 (Information backup)
- ISO/IEC 27031:2011 — ICT readiness for business continuity
- (ISC)² CISSP CBK 5th ed. — Domain 7
- Veeam, Commvault, Rubrik, Cohesity, Zerto, Druva, HYCU — commercial backup product references
- AWS S3 Object Lock — *User Guide* (S3 Glacier + Object Lock = WORM)
- Azure Blob Immutable Storage — *Overview*
- GCP — Object Lock & Retention Policies
- LTO Program — LTO-9 / LTO-WORM specifications
- "Backup & Recovery: Inexpensive Backup Solutions for Highly Available Environments" (W. Curtis Preston, Oracle Press) — practitioner reference
- "GDPR & Backups: A Survival Guide" — StorageCraft / Veeam
- NoMoreRansom.org (Crypto Sheriff) — for decryptor availability
- IBM Cost of a Data Breach Report (annual)
- Coveware Quarterly Ransomware Report (public)
