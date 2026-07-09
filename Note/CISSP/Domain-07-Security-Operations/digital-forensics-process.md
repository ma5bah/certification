---
title: Digital Forensics Process
layer: 2
domain: 7
tags: [cissp, domain-7, forensics, chain-of-custody, acpo, e-evidence, order-of-volatility, isf, ffs, imager]
related_domains: [1, 4, 5, 7]
---

# Digital Forensics Process

## Feynman Explanation

Digital forensics is the discipline of "freezing the scene" on a computer so that a judge, a regulator, or an after-action reviewer can trust what the machine actually said. The rules are simple in concept — capture volatile state first, never touch original media, log every hand-off — but the discipline is unforgiving: a single reboot, an unrecorded time skew, or a missing hash can render an entire investigation inadmissible. The difference between a court-accepted exhibit and an "interesting log file" is whether the process was defensible from the first byte to the courtroom.

## Technical Details

### The ACPO (UK Association of Chief Police Officers) principles

Originally four principles, now five — they are the international baseline for defensible digital evidence handling:

1. **No action taken by law enforcement agencies, their employees, or their agents should change data held on a computer or storage media which may subsequently be relied upon in court.**
2. **In exceptional circumstances, where a person finds it necessary to access original data held on a computer or storage media, that person must be competent to do so, and be able to give evidence explaining the relevance and the implications of their actions.**
3. **An audit trail or other record of all processes applied to digital evidence should be created and preserved. An independent third party should be able to examine those processes and achieve the same result.**
4. **The person in charge of the investigation has overall responsibility for ensuring that the law and these principles are adhered to.**
5. *(Added 2012)* **All agencies, regardless of the DAP (Digital Asset Protection) tools they use, must be able to demonstrate that the tools they use produce consistent, repeatable, and accurate results, and are fit for the purpose for which they are used.**

### The four investigative phases (KR Germany model)

| Phase | Output |
|---|---|
| **Acquisition** | Bit-for-bit image of source media |
| **Identification** | Relevant artifacts, time lines, IOCs |
| **Evaluation** | Correlation with hypothesis, attribution, motive |
| **Presentation** | Court-ready report, exhibits, witness statements |

### Order of volatility (capture from most fragile to least)

The most volatile evidence is lost first if you wait. Capture in this order:

| Volatility | Source | Typical lifetime | Tool |
|---|---|---|---|
| 1. CPU registers, cache | CPU | Nanoseconds | Hardware debugger (rare) |
| 2. Routing table, ARP cache, process table, kernel statistics | RAM, kernel | Seconds to minutes | `volatility`, `memdump`, LiME |
| 3. Temporary file systems, tmp | Disk | Minutes to hours | `tar`, custom scripts |
| 4. Disk (running file system) | Disk | Days to years | `dd`, FTK Imager, EnCase, X-Ways |
| 5. Remote logging, monitoring data | SIEM, netflow | Days to years (retention) | Vendor APIs |
| 6. Physical config, network topology | Network gear config | Persistent | Backup exports |
| 7. Archival media, offline backups | Tape, optical | Persistent | Cold-storage mounts |

**Rule of thumb:** if you have to choose between volatility and completeness, volatility first — disk can be re-imaged, RAM cannot.

### Bit-stream imaging — the right way

| Step | Detail |
|---|---|
| **Acquire** | Use a hardware write-blocker (USB/IDE/SATA) or trusted boot media. Do **not** mount the original disk read-write. |
| **Hash** | Compute **at least two** independent hashes (e.g., MD5 + SHA-256) of both source and image. |
| **Verify** | Compare hashes; mismatch = invalid image, redo. |
| **Document** | Time, date, timezone, examiner, tool version, host, source serial, image file size. |
| **Wipe destination** | Sanitize target media (NIST SP 800-88 *Clear* or *Purge* levels) before each acquisition. |

The image is the evidence, not the original. The original is sealed in a Faraday bag / evidence locker and never re-powered.

### Chain of custody form (one row per hand-off)

| Field | Value |
|---|---|
| Evidence ID | e.g., EV-2026-07-0001 |
| Description | "Dell Latitude 7420, SN XYZ, HDD 512GB" |
| Hash (SHA-256) | `a4f3...` |
| Acquired by | "J. Smith, IR Analyst, badge #4421" |
| Date/time (UTC) | 2026-07-08T13:42:00Z |
| Method | FTK Imager 4.7, hardware write-blocker, USB |
| Storage location | Evidence vault, locker B12 |
| Released to | "M. Garcia, outside counsel" |
| Released by / received by (signature) | … / … |
| Returned / destroyed | Date, signature |

If any field is missing or forged, the evidence can be excluded. **Print, sign, scan, store.**

### E-evidence rules of admissibility (US/Federal Rule of Evidence 901 / Daubert)

| Test | Question | Forensic artifact |
|---|---|---|
| **Relevance** (FRE 401) | Does it make a fact more or less probable? | Document, log, packet capture |
| **Authenticity** (FRE 901) | Can the proponent lay foundation? | Hash, chain of custody, system logs |
| **Hearsay** (FRE 802) | Is it an out-of-court statement offered for the truth? | Business records exception (FRE 803(6)) applies to routine logs |
| **Best evidence** (FRE 1002) | Original or authenticated copy? | Bit-image + hash |
| **Expert testimony** (Daubert) | Method is reliable, peer-reviewed, known error rate | Methodology + tool validation |
| **Chain of custody** | Continuous, documented, tamper-evident | Custody log + tamper-evident bag |

### Forensic data sources (the "where to look" map)

| Source | Artifact | Tool |
|---|---|---|
| **RAM** | Processes, network connections, encryption keys, clipboard | `volatility`, `memprocfs`, Rekall |
| **Disk** | Files, MFT, $LogFile, $UsnJrnl, registry hives, $I30, prefetch, scheduled tasks | Autopsy, X-Ways, Plaso, KAPE |
| **Logs (Windows)** | Event logs (Security, System, Application, PowerShell, Sysmon), ETW | `wevtutil`, EVTX parsers, `chainsaw` |
| **Logs (Linux)** | `auth.log`, `syslog`, `journald`, `auditd`, `bash history` | `journalctl`, auditd parsers |
| **Network** | NetFlow, PCAP, DNS query logs, proxy logs, Zeek | Wireshark, Suricata, Arkime |
| **Cloud** | CloudTrail, Azure Activity, GCP Audit, M365 unified audit | Vendor exports, `aws-cli`, `o365-cli` |
| **Email** | Headers, message tracking, transport rules | `pffexport`, MFC |
| **Mobile** | iTunes backup, Android `adb` backup, JTAG/ISP | Cellebrite, Magnet AXIOM |
| **Memory of a process** | Malware, decrypted buffers | `memprocfs`, `volatility malfind` |
| **Browser** | History, cache, downloads, autofill, sessions | Hindsight, browser_sqlite parsers |
| **Endpoint telemetry** | EDR raw events, syscalls, file write history | Vendor APIs (CrowdStrike, SentinelOne, Defender) |

### Anti-forensics (what the adversary does) and counter-measures

| Anti-forensic technique | Detection / counter |
|---|---|
| Timestomp (modify MAC times) | Cross-correlate `$STANDARD_INFORMATION` with `$FILE_NAME` (MFT) — they update differently |
| Log wiping (event log clear) | Alert on `wevtutil cl`, EVTX gap detection, forwarder-based centralization |
| Secure delete (cipher wipe, `sdelete`) | Look for `cipher /w`, large NTFS MFT gaps, unallocated space patterns |
| Packers / obfuscation | Memory analysis (`malfind`), entropy analysis, sandbox detonation |
| Bootkit / rootkit | Secure boot, TPM measured boot, offline memory acquisition |
| Encryption of data at rest | Memory dump to recover keys; lawful access where applicable |
| Log injection (false logs) | Correlate across multiple sources (EDR + SIEM + netflow) |
| Timestomping across TZ | Look for impossible gaps, look for kernel-time source |

### Tool validation (NIST CFTT, ISO 17025)

For a tool to produce court-defensible output:

| Requirement | Evidence |
|---|---|
| Known input → known output | Test corpus (e.g., NIST CFTT, DFRWS) |
| Documented methodology | White paper, manual, academic reference |
| Peer review / open source | Source available, multi-vendor comparison |
| Repeatability | Two examiners, same input → same output |
| Error rate | Quantitative, published |
| Audit log | Tool produces evidence-of-tool log |

### Live-system vs. dead-system forensics

| Aspect | Live | Dead |
|---|---|---|
| When | Host must keep running (production) | Host is off / can be powered down |
| Captures | RAM, network state, mounted encrypted volumes, running processes | Disk, MFT, registry hive, logs |
| Risk | Mutates evidence; must justify | Misses volatile state |
| Tools | `winpmem`, LiME, KAPE, Velociraptor | `dd`, FTK Imager, hardware imager |
| Authority | **Highest** — every command must be logged and justified | Standard imaging |

**CISSP trap:** Many candidates forget that in modern enterprise (encryption everywhere, TPM, cloud) you often **cannot** do dead-system forensics on critical systems. Plan for live response.

### Legal holds and cross-border e-evidence

| Topic | Implication |
|---|---|
| **Legal hold** | Preserve all potentially relevant data; suspend normal retention/deletion |
| **GDPR / data minimization** | Conflict with preservation; legal basis (legitimate interest, legal obligation) required |
| **Cross-border** | MLAT, CLOUD Act (US), e-Evidence Regulation (EU) — lawful access without local presence |
| **Encryption keys** | Compelled disclosure may be required (jurisdiction-dependent) |
| **BYOD / personal accounts** | May require specific policy + consent + scope limitation |
| **Privileged data** | Attorney-client; apply privilege review before production |

## CISO / Risk Manager View

For the board, digital forensics is **"the difference between a $5M breach and a $50M judgment."** If your forensic process is not court-defensible, the same breach that cost you a few days of IR becomes a punitive-damages case.

**Board talking points:**

1. **Forensics starts at hire, not at incident.** The retention policy, legal hold process, and evidence-locker discipline are governance decisions made long before there is a breach.
2. **Tool validation is a one-time investment with annual return.** A validated toolchain (NIST CFTT, ISO 17025) is a force multiplier in court. Unvalidated tools are a defense lawyer's gift.
3. **The chain of custody is a chain of evidence, not a chain of trust.** When a hand-off is undocumented, the case is winnable by the defense on procedural grounds alone.
4. **Live-system forensics is the modern default.** Cloud + encryption + EDR-driven investigation means the disk image alone is no longer the entire story. Plan for it, rehearse it, train for it.
5. **The forensic report is the executive summary of the breach.** It is also a regulator-facing document. The same report must satisfy SEC, GDPR, customers, and a future court.

A useful board equation:

$$Cost\ of\ eDiscovery\ +\ Litigation = f(Records\ preserved,\ Defensibility,\ Time\ to\ produce)$$

The 2024 IBM Cost of a Data Breach report shows average breach lifecycle of ~290 days, with identification + containment taking the bulk. A defensible forensic process shaves weeks off that and reduces the per-record cost.

## Related Connections

### Sibling L2
- [[incident-response-lifecycle-nist]] — Forensic phase is nested in IR Containment / Eradication
- [[change-and-configuration-management]] — Image baselines needed to identify post-incident drift
- [[disaster-recovery-and-bcp]] — Post-breach recovery is informed by forensic RCA
- [[physical-and-environmental-security]] — CCTV and badge logs often corroborate digital evidence

### L3 children
- (Forensic deep-dives live in [[siem-soar-and-detection-engineering]] for log-derived evidence)

### Cross-domain
- [[domain-01-security-and-risk-management]] — Legal/regulatory frameworks govern evidence handling
- [[domain-04-communication-and-network-security]] — PCAP, netflow, DNS logs are forensic gold
- [[domain-05-identity-and-access-management]] — Identity events (authn, IAM change) are primary evidence

## References

- ACPO Good Practice Guide for Digital Evidence (UK, 2012)
- NIST SP 800-86 — *Guide to Integrating Forensic Techniques into Incident Response*
- NIST SP 800-88 Rev. 1 — *Guidelines for Media Sanitization*
- NIST CFTT — Computer Forensic Tool Testing project (https://www.cftt.nist.gov/)
- ISO/IEC 27037:2012 — Guidelines for identification, collection, acquisition and preservation of digital evidence
- ISO/IEC 27042:2015 — Guidelines for the analysis and interpretation of digital evidence
- ISO/IEC 27050 — Electronic discovery
- Federal Rules of Evidence (FRE) 401, 801–803, 901, 1002 (US)
- Daubert v. Merrell Dow Pharmaceuticals, 509 U.S. 579 (1993)
- CLOUD Act (US, 2018), EU e-Evidence Regulation (2023)
- RFC 3227 — *Guidelines for Evidence Collection and Archiving* (IETF)
- (ISC)² CISSP CBK 5th ed. — Domain 7
- SANS FOR508 — Advanced Incident Response, Threat Hunting, and Digital Forensics
- Open-source: `volatility`, `plaso/log2timeline`, `kape`, `chainsaw`, `velociraptor`, `autopsy`
