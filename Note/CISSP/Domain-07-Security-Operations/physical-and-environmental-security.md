---
title: Physical and Environmental Security
layer: 2
domain: 7
tags: [cissp, domain-7, physical-security, perimeter, cctv, mantrap, hvac, fire-suppression, tempest, fencing, lighting, guards]
related_domains: [1, 3, 4]
---

# Physical and Environmental Security

## Feynman Explanation

Physical security is the layer of defense that protects the machines the software runs on, the wires the data travels through, and the people who use the keyboards. All the firewalls, encryption, and PAM in the world are useless if someone can walk into the data center, unplug a server, and walk out. Domain 7 owns the operational reality of that layer — the fences, cameras, mantraps, badge readers, fire systems, and HVAC that keep the bits safe from bullets, water, heat, and human curiosity.

## Technical Details

### The physical defense layers (outside → inside)

```
Site selection (crime, flood, quake, civil unrest)
   └── Outer perimeter (fence, wall, berm, bollards, lighting)
         └── Perimeter barrier (K-rated, anti-ram)
               └── Building exterior (walls, doors, glazing)
                     └── Lobby / reception (visitor mgmt, signage)
                           └── Mantrap / airlock (two doors, one at a time)
                                 └── Floor / suite access (badge + PIN / biometric)
                                       └── Server room / SCIF (multi-factor)
                                             └── Cabinet / rack (lock + audit)
                                                   └── Server / device (TPM, drive lock, BIOS password)
```

Each boundary has its own control, its own logging, and its own failure mode.

### Physical controls — the standard list

| Layer | Control | Threat mitigated |
|---|---|---|
| **Site** | Crime/flood/quake risk assessment, redundancy, dual utility feeds | Regional / natural disaster |
| **Perimeter** | Fence (8 ft min for industry), wall, bollards (K4/K12), vehicle gate, anti-ram | Unauthorized entry, vehicle-borne attack |
| **Lighting** | 1 fc / 10 lux minimum, motion-activated, redundant power | Night intrusion, video quality |
| **CCTV** | Coverage of all entry/exit, parking, server room, 30-day retention minimum | Tailgating, theft, evidence |
| **Guards** | 24×7 for Tier-1, periodic for Tier-2, augmentation of electronic | Tailgating, social engineering |
| **Reception** | Visitor sign-in, NDA, escort, badge | Insider-assisted intrusion |
| **Mantrap** | Two doors, interlock, occupancy sensor, weight mat | Tailgating, piggybacking |
| **Badge / card** | Wiegand, OSDP, encrypted badge, anti-clone | Unauthorized access |
| **Biometric** | Fingerprint, iris, face, palm; FAR/FRR tuned | Stolen badge |
| **PIN + biometric / 2-factor** | Something you have + something you are | Lost badge |
| **Cabinet lock** | Mechanical + electronic, audit log | Internal theft |
| **Tamper-evident seal** | Bag / sticker, photo before/after | Insider tampering |
| **Drive lock / SED** | Self-encrypting drive, pre-boot auth, OPAL | Stolen hardware |

### Mantraps and turnstiles

| Type | Best for | Failure mode |
|---|---|---|
| **Optical turnstile** | Office lobby | Tailgating (must pair with guard) |
| **Full-height turnstile** | Datacenter, industrial | Slow, can trap |
| **Speed gate** | Office lobby, mid-security | Cost, single-person enforcement |
| **Mantrap (two doors, interlock)** | Datacenter, SCIF, lab | Throughput, requires staff |
| **Vestibule with guard** | High-security | Cost of guard |

**CISSP trap:** a turnstile is *not* a mantrap. A mantrap is two interlocked doors that allow only one person to pass at a time; a turnstile allows piggybacking.

### CCTV design

| Parameter | Recommendation |
|---|---|
| Resolution | 4K for entry/exit; 1080p for area |
| Frame rate | 15–30 fps (entry); 7–15 (area) |
| Codec | H.265 (HEVC) or H.264 with retention |
| Retention | 30 days minimum; 90 for regulated (PCI: 90 days on-demand + 90 days searchable) |
| Storage | Tamper-resistant, offsite, hashed |
| Coverage | All entry/exit, lobby, server room door, parking |
| Lighting | Sufficient for nighttime; IR for low-light |
| Privacy | Mask sensitive areas (bathrooms, screens) where legal |
| Audit | Time-sync via NTP, hash chain for tamper evidence |

**Math for storage:**

$$Storage_{GB/day} = \frac{Cameras \times Mbps \times 86400}{8 \times 1024}$$

Example: 100 cameras × 4 Mbps × 86400 / (8 × 1024) ≈ 4220 GB/day ≈ 4.2 TB/day ≈ 126 TB/month for 30-day retention.

### Power and UPS

| Source | Purpose | Sizing |
|---|---|---|
| **Utility (mains)** | Normal power | Per IT load + 30% |
| **Surge protector / TVSS** | Transient voltage | At panel |
| **UPS (battery)** | Ride-through (minutes) | IT load × desired runtime |
| **Generator (diesel / gas / fuel cell)** | Long-term (hours–days) | IT load + HVAC + lighting |
| **Transfer switch (ATS / STS)** | Automatic switchover | <10 sec typical |
| **Dual utility feeds (A + B)** | Two substations, two paths | For Tier-1 / Tier-3/4 datacenters |

Generator fuel rule of thumb:

$$Runtime_{hours} = \frac{Tank_{gallons} \times FuelRate_{gal/hr} \times\eta}{Load_{kW}}$$

For a 1 MW data center at 50% load with 1,000-gallon tank burning 60 gal/hr, runtime ≈ 1,000 / 60 × 0.5 ≈ 8 hours. Most Tier-1 DCs spec 24–48 hours of fuel with priority refuel contracts.

### HVAC and environmental

| Variable | ASHRAE A1 range | Operational note |
|---|---|---|
| **Temperature** | 18–27 °C (64.4–80.6 °F) | Most modern equipment tolerates wider; setpoint 22–24 °C |
| **Humidity** | 40–55% RH (low) to -9 °C dew point (high) | Below 40% = static; above 55% = condensation |
| **Airflow** | Front-to-back, hot/cold aisle | Containment doubles efficiency |
| **Particulate** | ISO 14644-1 Class 8 or better | Sub-floor dust is a killer |
| **Leak detection** | Under-floor + above-ceiling, rope or spot | Rope water sensors at every AC unit |
| **Smoke detection** | Aspirating smoke detection (VESDA) for Tier-1 | Pre-alarm at 0.05% obscuration/ft |

**CISSP tip:** A common cause of downtime is **HVAC failure**, not cyber attack. The operational team must treat HVAC with the same discipline as a firewall.

### Fire suppression

| Class | Threat | Suppressant |
|---|---|---|
| **A** | Ordinary combustibles (paper, wood) | Water, foam |
| **B** | Flammable liquids (fuel) | Foam, CO2, dry chem |
| **C** | Energized electrical | CO2, dry chem, clean agent |
| **D** | Combustible metals | Dry powder (specialty) |
| **K** | Cooking oils | Wet chem |

For a **server room** the right answer is almost always a **clean agent**:

| Agent | Notes |
|---|---|
| **FM-200 (HFC-227ea)** | Older, decommissioning in some jurisdictions |
| **Novec 1230 (FK-5-1-12)** | Modern, low-GWP, safe for occupied |
| **Inergen** (IG-541, N2/Ar/CO2) | Oxygen reduction; safe at design concentration |
| **Argonite** (IG-55) | Similar to Inergen |
| **CO2** | Life safety risk in occupied spaces; data center only if unoccupied |

**CISSP trap:** Water-based sprinklers are appropriate for offices but can destroy a server room. Pre-action or clean-agent systems are standard for data centers. Pre-action = dry pipe until smoke is detected twice, then fills — gives staff time to investigate.

### Voltage and frequency

| Phenomenon | Effect | Mitigation |
|---|---|---|
| Surge | Component damage | TVSS, surge protector |
| Spike | Damage | TVSS, isolation transformer |
| Sag / brownout | Reboot, data loss | UPS |
| Blackout | Downtime | Generator |
| Noise / EMI | Data corruption, hardware stress | Grounding, shielding, line conditioner |
| Frequency drift | Motor / PSU damage | UPS (clean output) |

### TEMPEST and emanation security

| Topic | Detail |
|---|---|
| **TEMPEST** | NATO term for compromising electromagnetic emanations |
| **Threat** | Adversary reconstructs screen content from unintentional RF emissions |
| **Standard** | SDIP-27 Level A (formerly NATO TEMPEST Level A), Level B, Level C |
| **Mitigations** | Faraday cage (shielded room / SCIF), TEMPEST-rated equipment, RF filters on power, distance (red/black separation) |
| **Red/black separation** | Equipment handling plaintext ("red") physically separated from ciphered / public ("black") |
| **Who cares** | Government, military, financial, sensitive research; rarely cost-effective for SMB |

### Mobile / laptop / endpoint physical

| Control | Purpose |
|---|---|
| Cable lock (Kensington) | Theft deterrence (low effort, medium effect) |
| LoJack / absolute | Theft recovery |
| TPM + BitLocker / FileVault | Drive encryption (data at rest) |
| BIOS / UEFI password | Boot-time gate |
| Self-encrypting drive (SED, OPAL) | Always-on, fast crypto |
| Geofencing + remote wipe | BYOD / lost device |
| Privacy screen | Shoulder-surfing |
| Secure bag / sleeve | Travel |

### Audit and testing

| Activity | Frequency |
|---|---|
| Badge access review (who has access, do they still need it) | Quarterly |
| Mantrap / turnstile test | Semi-annual |
| CCTV coverage walkthrough | Quarterly |
| Fire system inspection | Annual (by jurisdiction) |
| Generator test (full load) | Monthly (no-load) / Annual (full-load) |
| UPS battery test | Annual (replace on schedule, typically 3–5 yr) |
| Water leak detection test | Semi-annual |
| HVAC failure simulation (chaos) | Annual |

## CISO / Risk Manager View

Physical security is the **most undersold layer of the security stack** and the easiest to defer because it does not generate alerts in a SIEM. The board often has to be reminded that a single insider with a USB stick can defeat a $10M cyber program, and a single HVAC failure can take out a $50M data center.

**Board talking points:**

1. **Physical security is a leading indicator of insider risk.** A clean physical access review, mantrap enforcement, and CCTV coverage correlate strongly with overall security maturity. Sloppy physical security is a warning sign.
2. **HVAC and power failures are the top non-cyber cause of downtime.** Generators, UPS, and dual feeds are cheaper than an outage. Insurance will not pay back reputation.
3. **TEMPEST is rare but real.** For high-value data (government, IP, financial), shielding is a board-level topic.
4. **The clean-agent vs water decision is a CISO/CFO call, not a building manager call.** A water-based suppression system in a data center is a board-level mis-design.
5. **The badge audit is the cheapest compliance check you have.** Quarterly, automated, near-zero cost.
6. **CCTV is not a security control if no one watches it.** Live monitoring or intelligent analytics (line crossing, loitering) is the difference.

A simple risk equation for the board:

$$Risk_{physical} = Threat \times Vulnerability \times Asset\ Value$$

- A $50M data center behind a 4-ft fence with no CCTV = $50M × 1.0 × high = unacceptable.
- A $5M office behind a 6-ft fence + mantrap + 24×7 guard = $5M × 0.1 × medium = manageable.

The cost of a fence is the cheapest dollar in the entire security budget.

## Related Connections

### Sibling L2 (D7)
- [[disaster-recovery-and-bcp]] — Fire, flood, earthquake are DR triggers
- [[incident-response-lifecycle-nist]] — Physical breach flows into the IR funnel
- [[change-and-configuration-management]] — Physical changes (door re-core, camera move) are CIs
- [[digital-forensics-process]] — CCTV and badge logs are physical evidence that corroborates digital

### L3 children
- (none — physical security is largely an L2 domain)

### Cross-domain
- [[domain-01-security-and-risk-management]] — Physical security policy lives in D1
- [[domain-03-security-architecture-and-engineering]] — SCIF design, TEMPEST shielding, secure facilities engineering
- [[domain-04-communication-and-network-security]] — Wiring closets, telecom rooms, fiber runs are physical attack surface

## References

- (ISC)² CISSP CBK 5th ed. — Domain 7 (Physical and Environmental Security)
- ASHRAE TC 9.9 — *Thermal Guidelines for Data Processing Environments* (5th ed., 2024)
- NFPA 75 — *Standard for the Fire Protection of Information Technology Equipment*
- NFPA 76 — *Standard for the Fire Protection of Telecommunications Facilities*
- NFPA 10 / 12 / 2001 — Portable extinguishers / CO2 / clean agent
- ANSI/TIA-942 — *Telecommunications Infrastructure Standard for Data Centers* (rated tiers I–IV)
- Uptime Institute Tier Standard: Topology / Operational Sustainability
- BICSI 002 — *Data Center Design and Implementation Best Practices*
- FIPS 140-3 — Security requirements for cryptographic modules (covers SEDs)
- IEEE 1100 — *Recommended Practice for Powering and Grounding Electronic Equipment*
- TIA-568 — Commercial building cabling
- ISO 27001:2022 Annex A.7 (Physical) and A.8 (Environmental)
- NIST SP 800-53 Rev. 5 — PE family (Physical and Environmental Protection)
- SDIP-27 (formerly AMSG 720B) — NATO TEMPEST standard
- UL 2050 — *Standard for National Industrial Security Systems* (certified installers)
- IPMVP / ISO 50015 — Measurement and verification of power savings
