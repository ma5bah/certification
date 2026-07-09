---
title: 5G & IoT Network Security
layer: 3
domain: 4
tags: [cissp, domain-4, 5g, iot, mqtt, coap, lorawan, aka, slicing, nrf, nsc, edge-computing, private-5g]
related_domains: [3, 5, 7]
parent: network-security-controls
---

# 5G & IoT Network Security

## Feynman Explanation

5G and IoT are the **two big network frontiers** that arrived in the 2010s and 2020s. 5G replaces the radio that your phone uses with something faster, more programmable, and built around a *software-defined core* — meaning the network itself is now software that can be sliced into many independent virtual networks. IoT replaces the *device* — instead of a PC, you have a thermostat, a sensor, a meter, or a robot, each tiny, often poorly secured, and always on. The CISSP job is to know **how the trust model changes** in both cases: in 5G, the *operator* becomes part of your threat model (or your defense); in IoT, the *device* becomes the weakest endpoint you've ever had to manage.

## Technical Details

### 1. Mobile network evolution & security

| Gen | Air interface | Auth | Cipher | New risks | Counter |
|---|---|---|---|---|---|
| 1G (AMPS) | Analog FDMA | None (MIN) | None | Eavesdropping, cloning | Sunset |
| 2G (GSM) | TDMA/FDMA | SIM (COMP128-1) | A5/1 (broken) | IMSI catcher, SMS-MTI | Sunset 2G |
| 3G (UMTS) | W-CDMA | USIM + AKA | KASUMI, SNOW 3G | IMSI exposure (less) | Mutual auth (debut) |
| 4G (LTE) | OFDMA/SC-FDMA | EPS-AKA (mutual) | SNOW 3G, AES, ZUC | IMSI catch (still works), attach without integrity on some NAS | SUPI/SUCI in 5G |
| **5G (NR)** | OFDMA, mMIMO, mmWave | **5G-AKA, EAP-AKA'** | NEA1/2/3 (SNOW, AES, ZUC) | New: slicing, NEF, edge | Operator PKI, slice auth |

### 2. 5G security architecture (3GPP TS 33.501)

| Function | Role |
|---|---|
| **UE** | User Equipment (phone, IoT, C-V2X module) |
| **RAN / gNB** | 5G Node B (radio) |
| **AMF** | Access and Mobility Management Function (replaces MME) |
| **AUSF** | Authentication Server Function |
| **UDM** | Unified Data Management (subscriber data, ARPF — Authentication credential Repository and Processing Function) |
| **SMF** | Session Management Function |
| **UPF** | User Plane Function (data path) |
| **NEF** | Network Exposure Function (for 3rd party) |
| **NSSF** | Network Slice Selection Function |
| **SEAF** | Security Anchor Function (in AMF) — terminates auth with home network |
| **SUPI** | Subscription Permanent Identifier (like IMSI) |
| **SUCI** | Subscription Concealed Identifier (ECIES-encrypted SUPI) — **fixes IMSI catcher** |
| **5G-AKA** | Auth protocol (extensible) |
| **EAP-AKA'** | EAP variant (used for non-3GPP access: Wi-Fi, fixed) |

#### 2.1 5G-AKA flow (simplified)

```
UE (USIM)     gNB/SEAF       AUSF        UDM/ARPF
  | -- SUCI / SNN --> |          |             |
  |                    | -- 5G-AKA auth request (SUCI, SNN) --> |
  |                    | <-- auth vectors (RAND, AUTN, XRES*, KAUSF) -- |
  |                    | -- auth challenge (RAND, AUTN) ---> UE    |
  | <-- challenge <----|          |             |  (UE computes RES*)
  | -- RES* -------->  |          |             |
  |                    | -- RES* + SUPI to AUSF ---->             |
  |                    | <-- confirmation (HXRES*, KAUSF) ---->    |
  | <-- NAS SMC + key material ---|  (anchor key KSEAF derived)   |
  | <-- NAS SecurityModeCommand -->|                              |
  | -- NAS SecurityModeComplete -->|                              |
  | <-- AS SMC, AS SMC complete -->|                              |
  |  (secure: AS + NAS, UP via UPF)                              |
```

Key properties:
- **SUPI is never sent in the clear** — sent as SUCI (ECIES-encrypted with operator's public key)
- **Mutual authentication** — UE verifies AUTN contains expected SQN and MAC; network verifies RES* matches XRES*
- **Anchor key** (KAUSF → KSEAF → KgNB) chained for unidirectional derivation
- **SQN resync** protocol if SQN mismatch (timing attacks possible but bound)

#### 2.2 5G security features vs 4G

| Feature | 4G | 5G |
|---|---|---|
| Subscriber identifier | IMSI in clear | **SUPI/SUCI** (encrypted IMSI) |
| Mutual auth | Yes | **Yes + EAP-AKA'** for non-3GPP |
| Service-Based Architecture (SBA) | – | **Yes** (HTTP/2 + JSON between NFs) |
| Network slicing | – | **Yes** (each slice = isolated network) |
| Edge compute | EPC + LBO | **MEC** (Multi-access Edge) — UPF at edge |
| Subscriber privacy | IMSI catchable | SUCI prevents it |
| Lawful intercept | Vendor-specific | Standardized in TS 33.126 |
| Device attestation | – | **EARFCN / 5G-GUTI + optional secure element** |
| 256-bit algorithms | – | **NEA3/NIA3 (ZUC) + 256-bit AES** for high-assurance |
| AKMA | – | **Application Key Management for Applications** (apps get derived key) |

### 3. Network slicing (5G's biggest security surface)

| Concept | Meaning |
|---|---|
| **S-NSSAI** | Single Network Slice Selection Assistance Information (slice ID) |
| **Network slice** | End-to-end logical network with its own SLA/security policy |
| **Slice isolation** | Compute, storage, transport separated between slices |
| **Slice-specific auth** | Optional, in NSSAI |
| **Cross-slice attack** | Bug in shared control plane → lateral across slices |
| **MNO-operated slicing** | All slices in one operator |
| **Multi-operator slicing** | Each operator hosts part of a slice (e.g. for global IoT) |

**Threats**:
- Slice ID spoofing (UE claims to be in a higher-priority slice)
- Cross-slice resource exhaustion (DoS one slice to affect another)
- Mismatched slice auth between operator and tenant

**Defenses**:
- Slice-specific authentication and authorization (SAML, OAuth between operators)
- Shared infrastructure: microsegmentation at NFV level
- SBOM and patch SLA for all shared control plane components

### 4. Private 5G (enterprise-owned)

| Property | Private 5G | Public 5G (operator) |
|---|---|---|
| Spectrum | CBRS (US), local licensed, shared | Operator licensed |
| Core | On-prem (AWS Wavelength, Azure Edge Zones) | Operator |
| Use case | Factory, port, mine, defense, healthcare | Consumer, mobile |
| Security benefit | Sovereign; isolated from public internet | Operator-managed |
| Cost | High (CAPEX + radio engineers) | OPEX (subscription) |
| Risk | You own patching of the core | Operator patching |

### 5. IoT architecture

```
                           Cloud / Backend
                                ▲
                  (MQTT broker, time-series DB, control plane)
                                │
                            (MQTT, CoAP, HTTPS, AMQP)
                                │
                            Gateway (optional)
                                │
                            (BLE, Zigbee, LoRa, Wi-Fi)
                                │
                          End devices (sensors, actuators)
```

#### 5.1 IoT reference model (ISO/IEC 30141)

| Layer | Examples |
|---|---|
| **Sensing / actuation** | Temp sensor, relay, camera |
| **Connectivity** | Wi-Fi, BLE, Zigbee, LoRa, NB-IoT, LTE-M, 5G RedCap |
| **Edge / gateway** | Local aggregator, protocol translation |
| **Digital service / cloud** | Ingest, store, analyze |
| **Application** | User-facing app |
| **Business** | Enterprise integration |

### 6. IoT protocols — what to know for CISSP

| Protocol | Layer | Transport | Security |
|---|---|---|---|
| **MQTT** (ISO/IEC 20922) | L7 (pub/sub) | TCP 1883 (clear), 8883 (TLS) | Username/password, mTLS, ACLs |
| **MQTT-SN** | L7 (sleepy) | UDP | Same as MQTT |
| **CoAP** (RFC 7252) | L7 (RESTful) | UDP 5683 (clear), 5684 (DTLS) | DTLS, OSCORE (RFC 8613) |
| **AMQP** | L7 (queue) | TCP 5672 (clear), 5671 (TLS) | SASL, TLS |
| **DDS** | L7 (RT pub/sub) | UDP/TCP | Built-in PKI, RTPS-DS |
| **HTTP/REST** | L7 | TCP 80/443 | TLS, mTLS |
| **Modbus TCP** | L7 | TCP 502 | **No security** — replace |
| **DNP3** | L7 (utility) | TCP/UDP 20000 | DNP3-SA (TLS) |
| **IEC 60870-5-104** | L7 (utility) | TCP 2404 | TLS option |
| **IEC 61850** (substation) | L7 | TCP | TLS + multicast security |
| **OPC UA** | L7 (industrial) | TCP 4840 | Built-in PKI, auth, encryption |
| **Zigbee** | L2-L7 | 802.15.4 | Trust Center, Install Code, Zigbee 3.0 |
| **Z-Wave** | L2-L7 | 908.42 MHz | S2 security, ECDH key exchange |
| **Thread** | L2-L7 | 802.15.4 | MLE, 802.1X/PSK, DTLS |
| **LoRaWAN** | L1-L7 | sub-GHz | AppKey, AppSKey, NwkSKey, frame counter |
| **NB-IoT / LTE-M** | L1-L7 | Cellular | LTE security (EPS-AKA); 5G with 5G-AKA |

### 7. IoT-specific security issues

| Issue | Why it's hard | Mitigation |
|---|---|---|
| **Default credentials** | Manufacturers ship `admin/admin` | Bootstrapping framework (BCP 1.0, BRSKI) |
| **No patches** | Devices not designed to be updated | OTA update + signed firmware |
| **No identity** | Random MAC, no cert | Device cert (SCEP, EST) per device |
| **Insecure protocols** | Modbus, BACnet cleartext | Protocol gateway, encryption wrap |
| **Long-lived secrets** | Symmetric keys in flash | Secure element (TPM, SE), hardware crypto |
| **Side-channel** | Timing, power analysis | HSM, balanced code |
| **Firmware theft** | Easy to read flash | Secure boot, encrypted flash |
| **Physical access** | Anyone can attach to UART/JTAG | Disable debug, signed firmware |
| **Botnet recruitment** | Default creds + no patches | Mirai-style: must change defaults at provisioning |
| **Data exfil** | Always-on uplink | DLP at gateway, device traffic profiling |

### 8. IoT security frameworks and standards

| Standard / Org | Scope |
|---|---|
| **NIST IR 8259** | Foundational cybersecurity for IoT devices |
| **NIST SP 800-183** | Network of Things |
| **ETSI EN 303 645** | Consumer IoT cyber security (13 baseline controls) |
| **IEC 62443** | Industrial automation (covers IoT in OT) |
| **OWASP IoT Top 10** | Weak/hardcoded passwords, insecure network, insecure ecosystem, etc. |
| **GSMA IoT Security Guidelines** | Cellular IoT |
| **OneM2M** | Service-layer platform standard |
| **IETF SUIT (RFC 9019)** | Secure update of IoT firmware |
| **IETF CORE / OSCORE (RFC 8613)** | End-to-end security for CoAP |
| **IETF ROLL / 6LoWPAN** | IPv6 over low-power PANs |
| **IETF BRSKI (RFC 8995)** | Bootstrapping Remote Secure Key Infrastructure |
| **IETF TLS 1.3 profile for IoT (RFC 8446 + draft)** | Constrained devices |
| **Matter (CSA)** | Smart-home (Thread + Wi-Fi), device attestation |

### 9. Constrained devices — protocol-level accommodations

| Protocol | Adaptation |
|---|---|
| **CoAP** | UDP-based; smaller headers than HTTP; observe pattern |
| **DTLS** (RFC 6347) | TLS over UDP; cookie-based to mitigate DoS |
| **TLS 1.3** | Smaller handshake, 0-RTT, fewer cipher suites |
| **OSCORE** (RFC 8613) | Object security for CoAP — end-to-end at app layer |
| **CBOR / COSE** (RFC 8949, 8152) | Compact binary JSON + signed/encrypted |
| **QUIC** | UDP-based, low-latency TLS 1.3 — adopted for IoT gateways |
| **MQTT 5** | Reason codes, shared subscriptions, topic aliases |
| **LwM2M** (OMA SpecWorks) | Device management over CoAP/DTLS |

### 10. IoT bootstrap, identity, and onboarding

| Mechanism | Description |
|---|---|
| **BRSKI / RFC 8995** | "MASA" — manufacturer-signed voucher proves device's identity to local network; device receives a cert + config |
| **EST / SCEP** | Enroll device cert from CA (RFC 7030 EST, RFC 8894 SCEP) |
| **Device Identity Composition Engine (DICE)** | Hardware root-of-trust, derives per-context identity |
| **ARM PSA** | Platform Security Architecture (secure boot, secure storage, attestation) |
| **FIDO Device Onboard (FDO)** | Industrial / IoT onboarding (FDO spec) |
| **TPM / SE / TEE** | Hardware root of trust |
| **Secure Element** | SIM, eUICC, embedded SE |
| **iUICC / eSIM** | Remote SIM provisioning for IoT (SGP.22, SGP.32) |

### 11. Edge / MEC (Multi-access Edge Computing)

| Concern | Description |
|---|---|
| **Physical** | Edge is in a closet, not a DC — easier to tamper |
| **Shared infra** | Multi-tenant edge (MEC platform runs multiple slices/tenants) |
| **Data sovereignty** | Where is the data? Cross-border implications |
| **Patch** | Edge nodes are often forgotten; long lifecycle |
| **Network exposure** | Public-facing UPF at edge |
| **APIs** | NEF, CAPIF, service-based interfaces are HTTP — TLS + OAuth mandatory |

CAPIF (Common API Framework, TS 33.122) — common auth for 5G APIs.

### 12. Smart-card / UICC / eUICC

| Term | Meaning |
|---|---|
| **UICC** | Universal Integrated Circuit Card (the physical SIM) |
| **USIM** | UMTS Subscriber Identity Module (the application) |
| **ISIM** | IMS Subscriber Identity Module |
| **eUICC** | Embedded UICC (M2M or consumer eSIM) |
| **RSP / eSIM** | Remote SIM Provisioning (SGP.22 consumer, SGP.32 IoT) |
| **eSE** | Embedded Secure Element (banking, transport, ID) |
| **TEE** | Trusted Execution Environment (e.g. TrustZone, SGX) |
| **iUICC** | Integrated UICC (soldered; IoT use) |

### 13. IoT-specific attacks (CISSP-style)

| Attack | Vector | Defense |
|---|---|---|
| **Mirai** (2016) | Default creds, Telnet brute | Change defaults, disable unused services |
| **Reaper / Okiru** | Exploits known vulns | Patch |
| **VPNFilter** (2018) | Compromised router firmware | Signed firmware, integrity check |
| **Triton / TRISIS** (2017) | ICS safety system | Strict OT segmentation, monitoring |
| **Verkada (2021)** | Stolen super-admin credentials | MFA, audit, least privilege |
| **Pegasus** | 0-click iMessage | OS patch, attestation, MVT-style scanning |
| **Modbus recon** | Cleartext SCADA queries | VPN / DMZ; protocol-aware monitoring |
| **MQTT broker takeover** | Open broker, no auth | Auth, ACL, TLS, broker in DMZ |
| **CoAP amplification** | UDP reflection | BCP38, rate-limit, drop open CoAP |
| **DDoS via IoT botnet** | Mirai-class | Network-level mitigations, IoT segmentation |

### 14. IoT in regulated industries (security mandates)

| Vertical | Standard | Requirement |
|---|---|---|
| Medical (US) | FDA premarket cybersecurity guidance; 21st Century Cures § 524B | SBOM, vulnerability handling, patch plan, bill of materials |
| Medical (EU) | MDR (Annex I) | Cybersecurity for medical devices |
| Automotive | ISO/SAE 21434 | Cybersecurity engineering; UNECE WP.29 R155 for type approval |
| Energy (US) | NERC CIP | Bulk Electric System; applies to IoT in substations |
| Energy (EU) | NIS2, RED (Radio Equipment Directive) 2014/53 | Cybersecurity baseline for connected devices |
| Payment | PCI PTS | PIN Transaction Security (HCE, MPOS) |
| Consumer (UK) | PSTI Act 2024 | Bans default passwords; transparency of support period |
| Consumer (EU) | Cyber Resilience Act (CRA) 2024 | Whole product lifecycle; CE marking |
| Industrial | IEC 62443 | Zones & conduits, security levels |
| Telecom | GSMA NESAS | Network equipment security assurance |

### 15. Mapping 5G + IoT to CISSP exam topics

| Topic | 5G / IoT aspect |
|---|---|
| Confidentiality | Operator PKI for SUPI, OTA encryption (NEA), MQTT over TLS, CoAP over DTLS |
| Integrity | NAS integrity (NIA), RRSIG in DNS for OTA, OSCORE for CoAP |
| Availability | Slice capacity, OT/IT segmentation, DDoS scrubbing for IoT botnets |
| Auth | 5G-AKA / EAP-AKA' / device cert / DICE |
| Access control | Network slice AAA, broker ACL (MQTT), BRSKI enrollment |
| Logging | 5G CDR + DESS (Detection of Encrypted Signaling in 5G), IoT syslog to SIEM |
| Cryptography | AES-256 for 5G, ECDSA P-256 for IoT, ML-DSA (PQC) for future |
| Physical security | Edge site hardening, tamper-evident IoT enclosures |

## CISO / Risk Manager View

**Board framing**: 5G and IoT are the two places where the **attack surface is growing fastest** while **security maturity is lowest**. The strategic questions are simple and the answers are not:

1. **"Which 5G and IoT use cases are we betting on, and what is the security governance?"** A factory with 5,000 sensors is a 5,000-endpoint security problem; the previous risk model assumed 5,000 sensors were *isolated*. They are not.
2. **"Are we ready for the regulatory bar?"** EU CRA (2024) and US PSTI (2024) require lifecycle security for consumer IoT. EU RED enforced 2025. Medical and automotive have always had it. If you sell in those markets, you must comply.
3. **"What is our IoT inventory?"** Most enterprises have no idea how many IoT devices are on the network. NDR + CMDB reconciliation. Goal: 100% inventory.
4. **"Are we using operator 5G or private 5G, and what is the security SLA?"** Both have trade-offs. Operator 5G shares infra (CISA 5G risk assessment); private 5G is sovereign but is your problem.
5. **"PQC migration timeline."** 5G is being designed for PQC migration (CNSA 2.0). IoT devices with 10-year lifecycles will be vulnerable to "harvest now, decrypt later." Asset inventory drives the upgrade plan.

KPIs:

| KPI | Target |
|---|---|
| % of IoT devices on inventory (CMDB + NDR) | 95% |
| % of IoT devices with signed firmware | 100% (where supported) |
| % of IoT devices with default-cred change at provisioning | 100% |
| Mean time to patch IoT CVE | ≤ 30 d (where patch exists) |
| 5G SIM/SUPI exposure (cleartext) | 0 |
| % of network slices with documented security policy | 100% |
| 5G core patch SLA | ≤ 14 d |
| Edge compute nodes in CMDB | 100% |

## Related Connections

### Parent (L2)
- [[network-security-controls]] — WAF, NAC, IoT-segmentation, MDM/MDM-like controls

### Siblings (L3 in this domain)
- [[firewall-types-and-rule-evaluation]] — IoT rules; FQDN filtering for IoT clouds
- [[vpn-types-and-ipsec-modes]] — IPsec / WireGuard for IoT aggregation
- [[dns-security-and-dnssec]] — IoT DNS resolution; private DNS for IoT

### Cross-domain
- [[Domain 3 — Security Architecture and Engineering]] — PQC, secure boot, hardware roots of trust
- [[Domain 5 — Identity and Access Management]] — device identity, certificates, BRSKI
- [[Domain 7 — Security Operations]] — 5G NDR, IoT visibility

## References

- 3GPP TS 33.501 — Security architecture and procedures for 5G
- 3GPP TS 33.102 — 3G security
- 3GPP TS 33.401 — LTE / EPS security
- 3GPP TS 33.122 — CAPIF (API security)
- 3GPP TS 33.126 — Lawful Interception
- 3GPP TS 33.535 — AKMA
- 3GPP TR 33.926 — Security Assurance Specification (SCAS)
- 3GPP TS 33.210 / 33.310 — NDS / IP, PKI
- GSMA FS.20 — NESAS (Network Equipment Security Assurance Scheme)
- GSMA TS.34 / TS.35 / TS.36 / TS.37 — IoT Security Guidelines
- GSMA SGP.22 / SGP.32 — eSIM
- 3GPP TS 23.501 — System architecture
- ETSI TS 23.558 — MEC
- ETSI EN 303 645 — Cyber Security for Consumer IoT
- ETSI TS 103 701 — Consumer IoT — implementation
- ISO/IEC 30141 — Internet of Things Reference Architecture
- ISO/IEC 27400 — IoT security and privacy guidelines
- ISO/IEC 27001/27002 / 27103 — IoT in ISMS
- NIST IR 8259 — Foundational Cybersecurity for IoT
- NIST SP 800-183 — Network-of-Things
- NIST SP 800-193 — Platform Resiliency
- NIST SP 800-187 — Guide to LTE/5G Cybersecurity
- NIST SP 800-57 — Key Management
- IETF RFC 7252 — CoAP
- IETF RFC 6347 — DTLS
- IETF RFC 8323 — CoAP over TCP/TLS
- IETF RFC 8613 — OSCORE
- IETF RFC 8995 — BRSKI
- IETF RFC 9019 — SUIT (Secure Update for IoT)
- IETF RFC 8152 — COSE
- IETF RFC 8949 — CBOR
- IETF RFC 7397 — Firmware update for IoT
- OMA SpecWorks LwM2M
- ISO/IEC 20922 — MQTT
- IEC 62443 — Industrial cybersecurity
- ISO/SAE 21434 — Automotive cybersecurity
- UNECE WP.29 R155 / R156 — Vehicle type approval
- FDA — *Cybersecurity in Medical Devices: Quality System Considerations* (2023)
- CISA — *5G Network Slicing Threat Analysis*
- ENISA — *5G Threat Landscape*, *IoT Security*
- OWASP IoT Top 10 (2018, ongoing)
- ENISA *Baseline Security Recommendations for IoT*
- UK PSTI Act 2024
- EU Cyber Resilience Act 2024
- EU RED 2014/53 (Delegated Reg 2022/30)
- Mirai source (Trend Micro / Krebs analysis)
- *Practical IoT Hacking* (Chantzis et al., No Starch Press)
- Lee, *Internet of Things (IoT) Security: Best Practices*
- Schneier, *Click Here to Kill Everybody*
- ARM Platform Security Architecture
- RISC-V Trusted Execution Environment
- NXP / ST / Microchip IoT security white papers
- CISA *IoT Catalog of Risks*
- IEEE 802.15.4 — LR-WPAN
- IEEE 802.15.4e — TSCH (used by Thread)
- Bluetooth SIG *Core Specification 5.x*
- IETF RFC 4492, RFC 8422, RFC 8032 — ECC for security
- NIST PQC (FIPS 203/204/205) — ML-KEM, ML-DSA, SLH-DSA
- CISA *Quantum-Readiness: Migration to Post-Quantum Cryptography*
