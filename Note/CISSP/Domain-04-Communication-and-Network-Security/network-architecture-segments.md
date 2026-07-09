---
title: Network Architecture & Segments
layer: 2
domain: 4
tags: [cissp, domain-4, network-architecture, vlan, dmz, microsegmentation, sdn, sase, zero-trust, ztna]
related_domains: [3, 5, 7]
parent: domain-04-communication-and-network-security
---

# Network Architecture & Segments

## Feynman Explanation

A well-designed network is a building with many locked doors, where every room is only as big as the work that happens in it. A flat network is one giant open-plan office — once an attacker walks in, they can read every screen. A segmented network is a building with fire doors, badge-only access, and a reception desk that screens every visitor: even if one room is breached, the fire doors slam shut and the rest of the building keeps working. The art is to design the *walls* (segments), the *doors* (policy enforcement points), and the *keys* (identity, posture, encryption) so that the right people can move freely and the wrong people cannot move at all.

## Technical Details

### 1. The trust model

| Concept | Definition |
|---|---|
| **Trust boundary** | Any place where data changes ownership, identity, or sensitivity |
| **Policy Enforcement Point (PEP)** | A device/function that allows/denies/logs at a boundary |
| **Policy Decision Point (PDP)** | The brains that decide; usually identity+posture+context aware |
| **Policy Information Point (PIP)** | Source of context: IdP, asset DB, threat intel, GeoIP |
| **Subject** | Who/what is asking (user, device, service) |
| **Object** | Resource being requested |
| **Action** | Read/Write/Execute |
| **Policy** | Subject × Object × Condition → Allow/Deny |

> In Zero-Trust: the **PDP** is logically separate from the **PEP**, even if they live in the same vendor box.

### 2. Classic segmentation: VLAN

| Term | Meaning |
|---|---|
| **VLAN** (Virtual LAN, IEEE 802.1Q) | L2 broadcast domain; 12-bit VID → 4094 usable VLANs (0 and 4095 reserved) |
| **Trunk** | Multi-VLAN link between switches; tagged with 802.1Q |
| **Access port** | Single-VLAN port; untagged to host |
| **Native VLAN** | Untagged traffic on a trunk — **should never be a user VLAN** (jumping attack) |
| **Voice VLAN** | IP phone + PC; PC tags with data VLAN, phone tags with voice VLAN (CDP/LLDP) |
| **Private VLAN (PVLAN)** | Promiscuous / Isolated / Community ports on the same VLAN — common in DMZ |
| **VLAN hopping** | Attacker injects 802.1Q tag to access another VLAN; mitigated by strict trunk config and disabling DTP |

**Sample enterprise VLAN plan**:

| VLAN ID | Subnet | Purpose | Hosts |
|---|---|---|---|
| 10 | 10.10.0.0/24 | Mgmt (out-of-band) | Switches, APs, sensors |
| 20 | 10.20.0.0/22 | User/employee | PCs, laptops |
| 30 | 10.30.0.0/23 | Voice | IP phones |
| 40 | 10.40.0.0/22 | Printers | MFPs |
| 50 | 10.50.0.0/24 | Servers (internal) | App, DB |
| 60 | 10.60.0.0/24 | DMZ | Web, proxy, mail |
| 70 | 10.70.0.0/24 | Guest Wi-Fi | BYOD |
| 80 | 10.80.0.0/24 | IoT | Cameras, badges |
| 90 | 10.90.0.0/24 | OT/ICS | PLCs, HMIs |
| 100 | 10.100.0.0/22 | Dev/Test | CI runners, staging |

### 3. The DMZ (Demilitarized Zone)

A **buffered, screened subnet** between two trust zones (usually the untrusted internet and the trusted internal network).

```
   Internet
       │
  ┌────┴────┐
  │ Edge FW │   <-- Perimeter firewall (screen)
  └────┬────┘
       │
  ┌────┴─────────────── DMZ ───────────────┐
  │  web-proxy | mail-relay | DNS-resolver  │
  │  (any host has TWO interfaces:          │
  │    one in DMZ, one in Internal)         │
  └────┬──────────────┬─────────────────────┘
       │              │
  ┌────┴────┐   ┌────┴────┐
  │ Internal│   │ Backend │
  │   FW    │   │   FW    │
  └────┬────┘   └────┬────┘
       │              │
   Corporate      Database VLAN
```

| Topology | Trade-off |
|---|---|
| **Single-firewall, 3-leg** (FW has DMZ, Inside, Outside) | Cheaper; one failure = full breach |
| **Dual-firewall (sandwich)** (FW1 outside-DMZ, FW2 DMZ-inside) | More expensive; **defense in depth** — preferred for high-value |
| **Service mesh in DMZ** (Kubernetes ingress + mesh) | Cloud-native; east-west is mTLS only |

**Public-facing services that belong in the DMZ**: web (HTTP/HTTPS), reverse proxy, mail relay, DNS resolvers, VPN concentrator, jump hosts, jump boxes.

### 4. East-West vs North-South

| Direction | Meaning | Old control | New control |
|---|---|---|---|
| **North-South** | Traffic entering/leaving the data center | Perimeter FW | Cloud edge / SSE |
| **East-West** | Server-to-server, lateral | Internal VLANs | Microsegmentation, NDR, mTLS service mesh |

**Why east-west is the bigger problem (2018+)**: the assumption that "everything inside is trusted" is false once an attacker lands on a single endpoint. East-west is where ransomware spreads and where APTs move.

### 5. Microsegmentation

| Approach | Granularity | How |
|---|---|---|
| **Network-based (VLAN + FW)** | Subnet | Manually map workloads to VLANs; ACL on internal FW |
| **Host-based (agent)** | Per-workload | Agent on each VM/container; rules per process; Illumio, Guardicore, Cisco Tetration |
| **Hypervisor-based** | Per-VM | NSX, vSphere Distributed Switch; rules at vSwitch level |
| **Cloud-native (security group)** | Per-NIC | AWS SG, Azure NSG, GCP firewall |
| **Service mesh (sidecar)** | Per-service | Istio/Linkerd with mTLS; per-route policy |
| **Identity-based (ZTNA)** | Per-identity | Subject + device posture + resource policy; no L3 trust |

**Recommended layering** (a microsegmentation program):
1. **Inventory** all workloads (CMDB, cloud tags, NDR pass)
2. **Visualize** flows (agent reports or flow telemetry)
3. **Author** policy in allow-list (default deny)
4. **Stage** in monitor mode (shadow)
5. **Enforce** ring by ring (dev → test → prod → user)
6. **Maintain** as code (Terraform, policy as code)

### 6. SDN / SD-WAN / NFV

| Term | What | Vendor examples |
|---|---|---|
| **SDN** | Control plane decoupled from data plane; central controller programs switches | OpenFlow (research), Cisco ACI, VMware NSX, Open vSwitch |
| **SD-WAN** | WAN overlay over any underlay; tunnel mesh; path selection by app health | Viptela (Cisco), Velocloud (VMware), Fortinet, Versa |
| **NFV** | Network functions as VMs/containers on COTS hardware | ETSI MANO reference architecture |
| **SASE** (Secure Access Service Edge) | Cloud-delivered SWG + CASB + ZTNA + FWaaS | Zscaler, Netskope, Palo Alto Prisma |
| **SSE** (Security Service Edge) | SASE minus SD-WAN | Same vendors |

### 7. Zero-Trust Network Architecture (ZTNA / ZTA)

NIST SP 800-207 (2020) core principles:

1. **All data sources and computing services are considered resources** (no implicit trust of "internal" subnets)
2. **All communication is secured** regardless of network location
3. **Access to individual enterprise resources is granted on a per-session basis** (not per-machine)
4. **Access is determined by dynamic policy** — observable client state, identity, behavior
5. **Enterprise monitors and measures integrity and security posture** of all assets
6. **All resource authentication and authorization are dynamic and strictly enforced** before access
7. **The enterprise collects as much information as possible** about asset state and uses it to improve security

Logical components (NIST 800-207):

| Component | Role |
|---|---|
| **Policy Engine (PE)** | Computes allow/deny decision |
| **Policy Administrator (PA)** | Establishes/terminates the session; issues commands to PEP |
| **Policy Enforcement Point (PEP)** | Sits in the data path; enables/disables connection |

A ZTNA control plane interaction:

```
  Subject ──► PEP ──► PA ──► PE
              ▲        ▲      ▲
              │        │      │
            (gate)   (auth)  (policy: IdP + Device Posture + Threat Intel + Time-of-day)
              │        │      │
              └──── response ─┘
```

ZTNA 1.0 (early): device → ZTNA broker → app tunnel
ZTNA 2.0 (Gartner 2023+): same plus **continuous verification**, **broker-less** options, **device posture**, and **layer-7 deep inspection**.

### 8. Network architecture for cloud (AWS reference)

```
                Internet
                   │
              CloudFront / WAF
                   │
              ALB (public)
                   │
        ┌──────────┴──────────┐
        │                     │
   Web tier subnet     API tier subnet
   (10.0.1.0/24)       (10.0.2.0/24)
        │                     │
   SG: ingress:443      SG: ingress:443 from VPC CIDR
        │                     │
        └──────────┬──────────┘
                   │
              Internal ALB
                   │
              App tier subnet (10.0.3.0/24)
              SG: ingress: from web OR api SG
                   │
              Data tier (RDS, Elasticache)
              SG: ingress: from app SG only, port 5432
                   │
        ┌──────────┴──────────┐
        │                     │
   VPC Flow Logs ──► S3 ──► Athena/QuickSight
   CloudTrail  ──► CloudWatch / SIEM
   GuardDuty (anomaly)
```

Key cloud-segmentation primitives:
- **Security Group** = stateful, allow-only, attached to ENI
- **NACL** = stateless, allow/deny, attached to subnet
- **VPC** = L3 boundary; **subnet** = broadcast/L2 boundary inside VPC
- **Transit Gateway** = inter-VPC routing
- **PrivateLink** = L3 private connectivity to a service in another VPC/account

### 9. Specific architectures the CISSP exam tests

| Architecture | When used | Security value |
|---|---|---|
| **Three-tier (web/app/db)** | Classic enterprise | App tier isolated from DB; DB accepts only from app SG |
| **Hub-and-spoke (TGW)** | Multi-VPC | Central egress + inspection; spoke-to-spoke denied by default |
| **Mesh (Transit Gateway + RAM)** | Multi-account cloud | Shared services in hub; per-account spokes |
| **Belt-and-suspenders DMZ** | High-value assets | Dual-firewall sandwich |
| **Air-gapped OT** | ICS/SCADA | Physically separate; data diode egress only |
| **Purdue model** | OT | L0–L5 zones (process → enterprise) with strict crossing rules |
| **Zero-Trust (NIST 800-207)** | Modern hybrid | Identity-bound, no implicit L3 trust |
| **SASE** | Cloud-first enterprise | Cloud-delivered SWG/CASB/ZTNA/FWaaS; replaces DC egress |

### 10. Concrete segmentation example: enterprise campus

```
[Guest Wi-Fi] ──► [Captive portal] ──► [Internet only, no internal]
[BYOD]       ──► [NAC: cert+MDM]  ──► [Internet + limited internal]
[Employee]   ──► [802.1X PEAP]    ──► [VLAN 20, full internal]
[Voice]      ──► [LLDP-MED]       ──► [VLAN 30, QoS]
[IoT]        ──► [PSK + VLAN 80]  ──► [Internet only]
[OT/ICS]     ──► [Air-gapped, VLAN 90, jump-host only]
[Servers]    ──► [802.1X cert]    ──► [VLAN 50, microseg inside]
[DMZ]        ──► [DMZ FW sandwich] ──► [Internet-facing only]
```

Each transition is a **PEP**: switch port + FW + NAC + identity check.

### 11. Segmentation effectiveness metrics

| Metric | Target |
|---|---|
| Number of paths between user and DB tier | ≤1 explicit path |
| Number of east-west flows discovered | Decreases as policy tightens |
| Stale rules older than 90 d | 0 |
| Unmanaged assets | 0 (CMDB/NDR reconcile daily) |
| East-west policy violations / day | Decreasing trend |
| Mean time to push new rule | < 24 h, fully audited |

## CISO / Risk Manager View

Boards understand **blast radius**. A breach of a single workstation in a flat network becomes a $4M ransomware event; the same breach in a segmented network becomes a $400K contained incident. The math is simple and the cost of segmentation (one-time: $1–5M for a 5,000-employee enterprise; ongoing: 0.5–1 FTE per 1,000 segments) is dramatically smaller than the avoided expected loss.

Three board-level narratives:

1. **"The perimeter is dead"**. Cloud, IoT, and remote work have dissolved the corporate network edge. Perimeter alone is a 1990s defense; east-west and identity are the new control plane. Fund ZTNA, microsegmentation, and NDR accordingly.
2. **"Segmentation is the cheapest insurance we own"**. Once you have a cleanly segmented network, **every** future security control (M&A integration, incident containment, regulatory scope reduction) becomes cheaper. It is foundational, with multi-year ROI.
3. **"SASE is a financial decision, not a security one"**. Whether you keep the data center for egress or route everything through cloud SSE depends on (a) per-megabit cost, (b) latency for voice/video, (c) regulatory jurisdiction of cloud egress. Security is part of the requirement set, not the deciding factor.

KPIs to track at the board level:

| KPI | 2024 baseline | 2025 target | Why |
|---|---|---|---|
| % of internal east-west traffic with microseg policy | <30% | 80% | Limits blast radius |
| % of remote access via ZTNA (vs legacy VPN) | <50% | 95% | Identity-aware |
| % of internal subnets with documented classification | 60% | 100% | Audit/insurance |
| Mean time to isolate (MTTI) a compromised workload | 4 h | 30 min | Containment speed |
| Network-related audit findings | High | Trending to zero | Regulatory |

## Related Connections

### Siblings (L2 in this domain)
- [[osi-tcpip-models-and-encapsulation]] — layers the segments live at
- [[network-security-controls]] — what we put *in* each segment boundary
- [[wireless-and-mobile-security]] — wireless VLANs and SSID segmentation
- [[secure-protocols-and-services]] — protocols (TLS, IPsec) that are segmentation-relevant

### L3 children
- [[firewall-types-and-rule-evaluation]] — the rules that enforce segment policy
- [[vpn-types-and-ipsec-modes]] — how remote users cross segments
- [[dns-security-and-dnssec]] — DNS as a cross-segmentation dependency
- [[5g-and-iot-network-security]] — IoT/5G segment isolation

### Cross-domain
- [[Domain 3 — Security Architecture and Engineering]] — design principles behind defense in depth
- [[Domain 5 — Identity and Access Management]] — ZTNA = identity-aware PEP
- [[Domain 7 — Security Operations]] — segment telemetry lands in SIEM/SOAR

## References

- NIST SP 800-207 — Zero Trust Architecture (2020)
- NIST SP 800-204A — Building Secure Microservices-based Applications
- NIST SP 800-204B — Attribute-based Access Control for Microservices
- NIST SP 800-125B — Secure Virtual Network Configuration
- NIST SP 800-125A — Security Recommendations for Server-based Hypervisor Platforms
- ISO/IEC 27033 — Network Security (multi-part)
- IEC 62443-3-2 — Security risk assessment and system design
- Cisco, *Campus LAN and WAN Architecture Design Guide*
- Forrester / Gartner ZTNA and SASE market guides
- AWS Well-Architected Framework — Security pillar
- Microsoft Azure Architecture Center — Network security
- Google Cloud Architecture Framework — Security, privacy, compliance
- Purdue / ISA-95 reference model (OT/ICS)
- IEEE 802.1Q-2022 — VLAN
- RFC 4364 — BGP/MPLS IP VPNs
