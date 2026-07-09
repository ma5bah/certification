---
title: Wireless & Mobile Security
layer: 2
domain: 4
tags: [cissp, domain-4, wireless, wifi, 802-11, wpa3, 802-1x, eap, lte, 5g, mdm, byod, bluetooth, rfid]
related_domains: [3, 5, 7]
parent: domain-04-communication-and-network-security
---

# Wireless & Mobile Security

## Feynman Explanation

Wireless is the same conversation as wired — but now anyone within a few hundred meters can listen in, and the medium itself can't tell a friend from a stranger. The history of Wi-Fi security is the history of increasingly better locks: WEP was a padlock that any attacker could pick in 60 seconds; WPA fixed it temporarily; WPA2 hardened it for 18 years; WPA3 finally makes the lock personalized for every user. Mobile security adds another concern: the device is *someone's personal property* that we're asking to do corporate work — so we need Mobile Device Management (MDM) to separate "your stuff" from "company stuff" on the same phone. 5G changes the radio again, and brings a brand-new trust model with operators, slices, and edge computing.

## Technical Details

### 1. Wi-Fi (IEEE 802.11) family

| Standard | Year | Band | Max PHY rate | MIMO | Notes |
|---|---|---|---|---|---|
| 802.11a | 1999 | 5 GHz | 54 Mbps | – | OFDM |
| 802.11b | 1999 | 2.4 GHz | 11 Mbps | – | DSSS |
| 802.11g | 2003 | 2.4 GHz | 54 Mbps | – | OFDM |
| 802.11n (Wi-Fi 4) | 2009 | 2.4/5 | 600 Mbps | 4×4 | HT, 40 MHz |
| 802.11ac (Wi-Fi 5) | 2013 | 5 | 6.93 Gbps | 8×8 MU-MIMO | VHT, 160 MHz |
| 802.11ax (Wi-Fi 6/6E) | 2019/20 | 2.4/5/6 | 9.6 Gbps | 8×8 MU-MIMO, OFDMA | 6E = 6 GHz |
| 802.11be (Wi-Fi 7) | 2024 | 2.4/5/6 | 46 Gbps | 16×16 MU-MIMO, MLO | 320 MHz |

### 2. Wi-Fi security generations

| Generation | Standard | Algorithm | Status | Key vulnerability |
|---|---|---|---|---|
| WEP | 802.11 (1997) | RC4 (40/104-bit), 24-bit IV | **Broken** | IV reuse; cracked in <60 s (Aircrack-ng) |
| WPA | 2003 | RC4 + TKIP, MIC (Michael) | Deprecated | TKIP weaknesses; KRACK ortho-attack |
| WPA2-Personal | 2004 | AES-CCMP, PSK | Widely deployed | Offline PSK dictionary attack on weak passphrases |
| WPA2-Enterprise | 2004 | AES-CCMP, 802.1X/EAP | Standard | Rogue AP, KRACK, downgrade |
| WPA3-Personal (SAE) | 2018 | AES-GCMP, SAE = Dragonfly handshake | Required for Wi-Fi 6E (since 2020) | Dragonblood (2021) — implementation flaws in some vendors |
| WPA3-Enterprise | 2018 | AES-GCMP, 192-bit minimum in CNSA mode | Gov/regulated | Operational complexity; not immune to rogue AP |
| OWE (Opportunistic Wireless Encryption) | RFC 8110 | AES-CCMP, no auth | For open Wi-Fi | No auth → MITM by design, but encrypted |
| Enhanced Open (OWE Transition) | 2020 | OWE + transition | "Wi-Fi Enhanced Open" | Same caveats as OWE |

#### 2.1 The four-way handshake (WPA2-Personal)

```
AP                      Client (STA)
 | <-- ANonce ----------- |
 |                        | derive PTK = PRF(PMK + ANonce + SNonce + MAC_AP + MAC_STA)
 | ---- SNonce + MIC ---> | AP also derives PTK
 | <-- GTK + MIC -------- | (Group key, rotated)
 | ---- ACK + MIC ------> |
```

WPA3 replaces this with **SAE (Simultaneous Authentication of Equals)**, a Dragonfly-based key exchange that resists offline dictionary attack by *requiring an active interaction per guess*.

#### 2.2 KRACK (2017) — Key Reinstallation Attack

- Attacker spoofs/replays message 3 of the 4-way handshake
- Forces reinstallation of the same key, resetting nonces
- Affects **WPA2** (in some implementations), not WPA3
- Patched in vendor updates
- Defense: firmware updates + WPA2-Enterprise with 802.11w (PMF)

#### 2.3 WPA3 modes (exam)

| Mode | Use case |
|---|---|
| **WPA3-Personal (SAE)** | Replaces WPA2-PSK; SAE = Dragonfly |
| **WPA3-Enterprise** | Replaces WPA2-Enterprise; optional 192-bit CNSA suite for classified |
| **WPA3-Enterprise with 192-bit** | Suite per NIST SP 800-193 — govt/defense |
| **Wi-Fi Enhanced Open (OWE)** | Open Wi-Fi, but encrypted — no auth, so still MITM-able |
| **OWE Transition Mode** | Mixed deployment |

### 3. Authentication: 802.1X and EAP

802.1X is a *port-based NAC* framework; the actual auth method is EAP. Frame is EAPOL (EAP over LANs) at L2.

| EAP method | Auth | Server cert | Client cert | Mutual? | Use |
|---|---|---|---|---|---|
| EAP-MD5 | MS-CHAPv2 hash | – | – | No | Avoid |
| EAP-LEAP | Cisco proprietary | – | – | Weak | Deprecated |
| EAP-PEAPv0/v1 | Tunneled TLS | Yes | No | Yes | Most campus Wi-Fi |
| EAP-TLS | Mutual TLS | Yes | Yes | Yes | **Gold standard** (PKI needed) |
| EAP-TTLS | Tunneled TLS | Yes | No | Yes | More flexible than PEAP |
| EAP-FAST | Tunneled via PAC | No (PAC) | No (PAC) | Yes | Cisco proprietary |
| EAP-SIM | SIM-based | – | – | Weak | Carrier Wi-Fi offload |
| EAP-AKA / EAP-AKA' | USIM-based (LTE/5G) | – | – | Yes (auth) | Carrier Wi-Fi + 5G |

**PMK caching** (802.11i) lets a returning client skip the full handshake.
**802.11r (Fast BSS Transition)** speeds roaming between APs in the same ESS.
**802.11k (Radio Resource Mgmt)** helps clients find better APs.
**802.11v (BSS Transition Mgmt)** lets the network tell clients to move.
**802.11w (Protected Management Frames)** prevents deauth/disassoc spoofing (mitigates "deauth attack").

### 4. Wireless attacks (defense plan)

| Attack | Description | Mitigation |
|---|---|---|
| **Evil twin** | Rogue AP with same SSID | 802.1X (auth), WIDS, cert pinning |
| **Rogue AP** | Unapproved AP plugged into network | NAC, switch port security, WIPS |
| **Deauthentication** | Spoofed deauth frame to kick client | 802.11w (PMF) |
| **KRACK** | Replay 4-way handshake msg 3 | Patch |
| **WEP IV reuse** | Crack WEP in seconds | Use WPA2/3 |
| **Jamming** | RF interference | Detect, locate, mitigate; 802.22 (WRAN) for cognition |
| **BlueBorne** | Bluetooth L2 vulns | Patch BT stack; disable when not used |
| **War-driving** | Find open APs | Don't run open APs in corp |
| **Packet injection** | 802.11 frames injected | 802.11w, WIPS |
| **WPA3 Dragonblood** | Implementation side channels in SAE | Vendor patches; reference impl from Wi-Fi Alliance |

### 5. Wireless IDS/IPS (WIDS/WIPS)

| Vendor | Detection method |
|---|---|
| Cisco (AireOS / Catalyst 9800) | wIPS embedded in APs |
| Aruba | RAPIDS + ClearPass integration |
| Fortinet | FortiWLC sensor |
| AirMagnet / Ekahau | Spectrum + WIDS sensor |

What they detect: rogue AP, evil twin, deauth flood, MAC spoof, hidden node abuse, RF jamming.

### 6. Mobile network generations

| Gen | Air interface | Auth | Cipher | Notes |
|---|---|---|---|---|
| 2G (GSM) | TDMA/FDMA | SIM (COMP128) | A5/1 (broken), A5/3 | No mutual auth; vulnerable to IMSI catchers |
| 3G (UMTS) | W-CDMA | USIM (AKA) | KASUMI, SNOW 3G | Mutual auth; still has IMSI exposure |
| 4G (LTE) | OFDMA / SC-FDMA | USIM (EPS-AKA) | SNOW 3G, AES, ZUC | All-IP; eNodeB; no cipher on integrity of some NAS |
| 5G (NR) | OFDMA, massive MIMO, mmWave | 5G-AKA, EAP-AKA' | NEA1/2/3 (SNOW, AES, ZUC), NIA1/2/3 | SUPI/SUCI (encrypted IMSI), slice-aware security, gNB |

#### 6.1 5G security architecture (3GPP TS 33.501)

| Function | What it does |
|---|---|
| **SUPI** | Subscription Permanent Identifier (like IMSI) |
| **SUCI** | Subscription Concealed Identifier (ECIES-encrypted SUPI) |
| **SEAF** | Security Anchor Function in AMF (home network anchor) |
| **AUSF / UDM / ARPF** | Auth server / subscription store / key repository |
| **5G-AKA / EAP-AKA'** | Auth protocols |
| **NEA / NIA** | Null / SNOW / AES / ZUC for ciphering and integrity |
| **Network slicing** | Independent logical network with own security policy |

The **SUPI → SUCI concealment** (using the home network's public key from the operator PKI) finally fixes the IMSI-catcher problem that has plagued GSM/UMTS/LTE.

### 7. Mobile Device Management (MDM)

| Function | What it does |
|---|---|
| **Inventory** | List devices, OS versions, IMEI, model |
| **Configuration** | Wi-Fi/VPN certs, email profile, passcode policy |
| **App management** | MAM: required apps, allow-list, remove-list |
| **Containerization** | Separate work/personal data on BYOD |
| **Remote wipe** | Full or selective (work only) |
| **Lost mode** | Lock + locate + message |
| **Posture check** | Jailbroken? Encrypted? OS up to date? |
| **Compliance enforcement** | Block corp data sync to non-compliant devices |
| **Conditional access** | AD/IdP requires compliant device for resource access |

Frameworks:
- **Apple Business Manager + Apple School Manager** (DEP, VPP, Managed Apple IDs)
- **Android Enterprise** (Profile Owner / Device Owner)
- **Microsoft Intune / Endpoint Manager**
- **VMware Workspace ONE (AirWatch)**
- **Jamf (Mac/iOS)**
- **Kandji, Mosyle** (Mac focus)

**BYOD vs COPE vs COBO**:

| Model | Device owner | Use case | Privacy |
|---|---|---|---|
| **BYOD** | Employee | Bring Your Own Device | Highest user pushback; need containerization |
| **COPE** | Company | Corporate Owned, Personally Enabled | Balance |
| **COBO** | Company | Corporate Owned, Business Only | Most control |
| **CYOD** | Employee chooses from list | Choose Your Own Device | Middle ground |

### 8. Other short-range wireless (CISSP coverage)

| Tech | Range | Security concern | Control |
|---|---|---|---|
| **Bluetooth** | 10–100 m | BlueBorne, BlueSmack, KNOB | Disable by default; only pair in secure space; BT 5.x mitigations |
| **BLE (Bluetooth Low Energy)** | 10–50 m | Unauthenticated GATT; tracking | Authenticated pairing, MAC rotation, privacy mode |
| **NFC** | 4–10 cm | Relay attacks; skimming | Short range = strong physical control; secure element |
| **RFID** | cm–m | Cloning, replay | Mutual auth, signed tags (e.g. NXP NTAG424) |
| **Zigbee (802.15.4)** | 10–100 m | Default keys known; touchlink | Custom key; Zigbee 3.0 mandates stronger key mgmt |
| **Z-Wave** | 30 m | Key exchange weak pre-500 series | Use S2; do not use legacy pairing |
| **LoRaWAN** | 2–10 km | AppKey compromised if not provisioned securely | Per-device AppKey; join server with mutual auth |
| **Sigfox** | km | Lightweight; replay possible | Sequence number + payload scrambling |

### 9. Cellular enterprise security (private LTE/5G)

| Use case | Why private |
|---|---|
| Factory floor | Coverage, determinism, asset control, spectrum guarantee |
| Mining / ports | Coverage in remote areas |
| Stadium / venue | Capacity + custom apps |
| Defense | Sovereign, no public network dependency |
| Healthcare | Reliability + IoMT (IoT medical) isolation |

**CBRS (Citizens Broadband Radio Service, US 3.5 GHz)** — three tiers: Incumbent (naval radar), Priority Access License (PAL), General Authorized Access (GAA). SAS (Spectrum Access System) coordinates.

### 10. Wi-Fi site survey & RF hygiene

| Concept | Purpose |
|---|---|
| **Predictive survey** | RF modeling in software (Ekahau, AirMagnet) before AP placement |
| **AP-on-a-stick survey** | Physical measurement during deployment |
| **Passive survey** | Listen to existing RF (find interference) |
| **Active survey** | Associate and measure throughput |
| **Spectrum analysis** | Identify non-Wi-Fi interferers (microwave, video bridge) |
| **Channel planning** | Reuse channels to avoid CCI; use 5/6 GHz when possible |
| **Tx power tuning** | Minimize cell overlap and bleed outside building |
| **RRM (Radio Resource Mgmt)** | Controller adjusts channels/power automatically |

### 11. Mobile threat landscape (per OWASP MASVS / NIST SP 800-163)

| Threat | Mitigation |
|---|---|
| Lost / stolen device | Remote wipe, encryption at rest (default iOS/Android 10+) |
| Malicious sideloaded app | Allow-list, MDM, attestation (Apple/Google Play Integrity) |
| Jailbreak / root | Attestation API; deny corp access on rooted |
| Spyware / stalkerware | Anti-stalkerware scanning |
| Smishing / phishing | Mobile threat defense (MTD) like Lookout, Zimperium, Defender for Endpoint |
| USB attack (juice-jacking) | Disable data on lock; only charge |
| SIM swap | Carrier-side control; hardware token (FIDO2) |
| Rogue base station | MTD detects; 5G SUPI/SUCI fixes |
| OS update lag | Patch SLAs in MDM policy |

### 12. Exam-essential wireless table

| Tech | Auth | Confidentiality | Integrity | Replay defense |
|---|---|---|---|---|
| WEP | Open/Shared | RC4 (broken) | CRC-32 (broken) | IV (broken) |
| WPA | PSK/802.1X | TKIP (RC4-based) | Michael MIC | TSC |
| WPA2-Personal | PSK | AES-CCMP | AES-CCM CBC-MAC | TSC + 48-bit PN |
| WPA2-Enterprise | 802.1X | AES-CCMP | AES-CCM CBC-MAC | TSC + 48-bit PN |
| WPA3-Personal | SAE | AES-GCMP | AES-GCM GMAC | PN + SAE |
| WPA3-Enterprise | 802.1X (Suite B/CNSA) | AES-GCMP-256 | AES-GCM GMAC-256 | PN + 192-bit |
| OWE | Open | AES-CCMP | AES-CCM | PN (no auth) |
| Bluetooth 5.x (LE Secure) | ECDH P-256 | AES-CCM | AES-CCM | PN |
| 4G LTE | EPS-AKA (mutual) | SNOW/AES | SNOW/AES | NAS COUNT + PDCP COUNT |
| 5G NR | 5G-AKA / EAP-AKA' | NEA1/2/3 | NIA1/2/3 | Same + SUCI |

## CISO / Risk Manager View

For the board, wireless is best framed as **"the longest unsecured wire into your network that you didn't install."** A 50-meter Wi-Fi reach from a public sidewalk past your lobby means an attacker with a Yagi antenna can attempt association from a parked car. Cellular and 5G shift the threat vector to *operator trust* and *device compromise*.

Three board narratives:

1. **"Our office Wi-Fi is a consumer-grade liability if we haven't moved to WPA3-Enterprise with 802.1X."** A common finding. Cost of upgrade: $0 if APs already support WPA3; $X if hardware refresh needed. Risk: known PSK = full network access; password leaks = full rebuild.
2. **"Mobile is the new endpoint."** PC shipments flat, phone shipments growing. The endpoint security budget that went to EDR on Windows must now be redirected to MTD/MDM on iOS/Android, and the IAM budget must fund strong, passwordless mobile auth (FIDO2, passkeys).
3. **"5G is a network architecture decision, not a connectivity upgrade."** Private 5G for OT/ICS or campus gives a sovereign, deterministic network, but is a multi-million-dollar decision and a long-tail integration project with operators. Don't drift into it without a clear use case.

KPIs:

| KPI | Target | Why |
|---|---|---|
| % of corporate Wi-Fi on WPA3-Enterprise | 100% | Eliminate PSK + KRACK exposure |
| % of corporate mobile devices under MDM | 100% | No unmanaged device touches corp data |
| % of new mobile apps reviewed against MASVS | 100% | Build/buy security baseline |
| Mean time to wipe a lost device | < 4 h | Limit data loss |
| 802.1X-protected management frames (802.11w) | 100% of APs | Deauth flood mitigation |
| Cellular IoT devices using SUCI | 100% (5G-capable) | IMSI-catcher defense |

## Related Connections

### Siblings (L2 in this domain)
- [[osi-tcpip-models-and-encapsulation]] — layers of the radio stack
- [[network-security-controls]] — WIDS/WIPS as a control
- [[network-architecture-segments]] — SSID/VLAN segmentation
- [[secure-protocols-and-services]] — EAP, RADIUS, 802.1X

### L3 children
- [[firewall-types-and-rule-evaluation]] — perimeter policy for wireless guest
- [[vpn-types-and-ipsec-modes]] — VPN over wireless
- [[dns-security-and-dnssec]] — DNS rebinding and Wi-Fi guest
- [[5g-and-iot-network-security]] — deep dive on 5G and IoT

### Cross-domain
- [[Domain 3 — Security Architecture and Engineering]] — WPA3/SAE crypto
- [[Domain 5 — Identity and Access Management]] — 802.1X/EAP/RADIUS = identity
- [[Domain 7 — Security Operations]] — WIDS telemetry → SIEM

## References

- IEEE 802.11-2020 — Wireless LAN
- IEEE 802.1X-2010 — Port-based NAC
- IEEE 802.11i-2004 — WPA2
- IEEE 802.11w-2009 — Protected Management Frames
- Wi-Fi Alliance, *WPA3 Specification v3.4*
- NIST SP 800-48r1 — Legacy 802.11 security
- NIST SP 800-97 — Securing WiMAX (legacy)
- NIST SP 800-153 — Guidelines for Securing WLANs
- NIST SP 800-193 — Platform Resiliency (CNSA)
- NIST SP 800-63B — Authentication (EAP types)
- NIST SP 800-187 — Guide to LTE/5G Security
- 3GPP TS 33.501 — 5G Security Architecture
- 3GPP TS 33.102 — 3G Security
- 3GPP TS 33.401 — LTE Security
- OWASP MASVS / MASTG
- ENISA, *5G Threat Landscape*
- Common Criteria MDM PP
- CIS Benchmarks for iOS, Android, macOS
- GlobalPlatform TEE specifications
- Apple Platform Security, iOS 17
- Android Enterprise Recommended
- RFC 5216 — EAP-TLS
- RFC 3748 — EAP
- RFC 4492 — ECC for TLS (now RFC 8422)
- Dragonfly RFC 7664 (SAE)
- Matt Wright, *Certified Wireless Security Professional (CWSP) Study Guide*
