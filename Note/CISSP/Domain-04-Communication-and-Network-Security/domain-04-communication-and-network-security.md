---
title: Domain 4 — Communication and Network Security (Hub)
layer: 1
domain: 4
tags: [cissp, domain-4, network-security, hub, osi, tcp-ip, firewall, vpn, dns, wireless]
related_domains: [3, 5, 7]
---

# Domain 4 — Communication and Network Security

## Feynman Explanation

Every conversation between two computers is really a chain of seven (or four) little conversations stacked on top of each other — a web request rides on TCP, which rides on IP, which rides on Ethernet, which rides on a fiber. Each layer is a separate trust boundary, and each layer has its own way of being attacked, eavesdropped, or impersonated. Domain 4 is about designing, securing, and verifying every one of those layers so that an adversary who breaks one boundary still faces the next one. The job of a network security architect is to make every hop a *decision point* — identity-checked, encrypted, logged, segmented.

## Technical Details

### Scope of Domain 4 (CISSP CBK 2024 outline)

| Subtopic | Weight guidance | L2/L3 children |
|---|---|---|
| OSI / TCP-IP models, encapsulation | Foundational | [[osi-tcpip-models-and-encapsulation]] |
| Network security controls (firewall, IDS/IPS, NDR, WAF, NAC, UTM) | Core | [[network-security-controls]] |
| Secure network architecture (VLAN, DMZ, microsegmentation, SDN, ZTNA) | Core | [[network-architecture-segments]] |
| Wireless & mobile (WPA3, 802.1X, LTE/5G, MDM) | Core | [[wireless-and-mobile-security]] |
| Secure protocols & services (IPsec, TLS, SSH, DNSSEC, S/MIME, Kerberos) | Core | [[secure-protocols-and-services]] |
| Firewall types & rule evaluation | Edge detail | [[firewall-types-and-rule-evaluation]] |
| VPN types & IPsec modes | Edge detail | [[vpn-types-and-ipsec-modes]] |
| DNS security & DNSSEC | Edge detail | [[dns-security-and-dnssec]] |
| 5G & IoT network security | Edge detail | [[5g-and-iot-network-security]] |

### The two reference models (one page reference)

| OSI (7) | TCP/IP (4) | PDU name | Example protocols | Security concern |
|---|---|---|---|---|
| 7. Application | Application | Data | HTTP, SMTP, DNS, Kerberos, S/MIME | App-layer auth, injection, malformed payloads |
| 6. Presentation | Application | Data | TLS, SSH, MIME, ASN.1 | Cipher selection, certificate trust |
| 5. Session | Application | Data | TLS handshake, RPC, NetBIOS | Session hijack, replay |
| 4. Transport | Transport | Segment / Datagram | TCP (6), UDP (17) | SYN flood, port scan, UDP reflection |
| 3. Network | Internet | Packet | IPv4, IPv6, ICMP, IPsec | Smurf, route hijack, IP spoof |
| 2. Data Link | Link | Frame | Ethernet, 802.11, ARP, PPP, L2TP | MAC flood, ARP poison, evil twin |
| 1. Physical | Link | Bits | Fiber, copper, RF, optical | Tapping, jamming, EMI |

For full encapsulation tables, PDU header maps, and SDU ↔ PCI flow see [[osi-tcpip-models-and-encapsulation]].

### Threat model (what the adversary is actually doing)

| Attack class | Layer | Brief | Defensive control |
|---|---|---|---|
| Sniffing / passive eavesdrop | 1–2 | Read cleartext frames | Link encryption (MACsec, WPA3), TLS |
| Active MITM | 2–7 | Insert self into path | Mutual auth (mTLS, 802.1X), certificate pinning |
| Spoofing | 2–3 | Forge source MAC/IP | DAI/DHCP snooping, BCP38 uRPF, signed DNS |
| DoS / DDoS | 1–4 | Saturate bandwidth or state tables | Scrubbing, rate-limit, SYN cookies, anycast |
| Lateral movement | 3–4 | Pivot inside flat network | Microsegmentation, east-west firewall, ZTNA |
| DNS abuse | 7 | Cache poison, amplification, tunneling | DNSSEC, DoH/DoT, response-rate-limit |
| Wireless rogue | 1–2 | Evil twin, KRACK, deauth | WPA3-Enterprise, 802.1X, WIDS |
| VPN abuse | 3 | Tunnel as a backdoor | Split-tunnel policy, posture check, IPsec + posture |
| Supply-chain firmware | 1 | Compromise router/switch | Signed firmware, secure boot, TACACS+ AAA |

### Mapping to other CISSP domains

| Cross-domain link | Where it lives | Why it matters here |
|---|---|---|
| TLS handshake → [[Domain 3 — Security Architecture and Engineering]] | D3 crypto primitives, PKI, random numbers | All transport crypto depends on D3 primitives |
| 802.1X, Kerberos, RADIUS → [[Domain 5 — Identity and Access Management]] | D5 identity, auth protocols | Network admission = identity decision |
| SIEM, log, NDR, vulnerability mgmt → [[Domain 7 — Security Operations]] | D7 ops, monitoring | Network telemetry feeds SOC |
| BC/DR networking → [[Domain 1 — Security and Risk Management]] | D1 governance | Network resilience = business resilience |

### The "defense-in-depth network" mental model

```
                       Untrusted
                          │
                ┌─────────┴─────────┐
                │  DDoS scrubbing   │  L3/L4
                └─────────┬─────────┘
                          │
                ┌─────────┴─────────┐
                │  Perimeter NGFW   │  L4–7
                └─────────┬─────────┘
                          │
              ┌───────────┴───────────┐
              │   DMZ (web/proxy)    │  Public-facing
              └───────────┬───────────┘
                          │
                ┌─────────┴─────────┐
                │  Internal FW +     │  East-West
                │  Microseg / ZTNA   │
                └─────────┬─────────┘
                          │
            ┌─────────────┴─────────────┐
            │   Endpoint + EDR + NAC    │  Identity-aware
            └───────────────────────────┘
```

Each box is a *control*; each boundary is a *policy enforcement point (PEP)*; each PEP logs to SIEM (D7) and uses identity from D5.

<!-- EXPANDED -->
## Expert Knowledge Expansion

### Foundations — The OSI Model as a Security Mental Map

Every security professional must be able to **"walk the stack"** from L1 to L7 on any network problem. The OSI model is not an academic reference — it is a triage and defense-in-depth framework. Ask: *at which layer did the attack succeed, and at which layers did defenses fail?*

**Encapsulation is the core security concept.** Each layer wraps the layer above, adding its own header (PCI). Security controls can be inserted at every wrapper boundary: MACsec at L2, IPsec at L3, TLS at L4–L7, application-layer auth at L7. An attacker who defeats one wrapper still faces the next. This is why [[osi-tcpip-models-and-encapsulation]] defense-in-depth works — compromise of L2 (ARP poison) does not transit to L7 if mTLS is enforced.

**The Three Data Planes** (critical for SDN and zero-trust):
| Plane | Function | Security implication |
|---|---|---|
| **Data plane** | Moves packets fast (ASIC/FPGA forwarding) | Integrity: IPsec ESP on every hop; in-band telemetry |
| **Control plane** | Decides routing (OSPF, BGP, SDN controller) | Hardening: BGP MD5/TCP-AO, RPKI, OSPFv3 IPsec, control-plane policing |
| **Management plane** | Configures (SSH, NETCONF, RESTCONF, SNMPv3) | AAA: TACACS+ per-command auth, out-of-band management VLAN, signed config diffs |

**"You can't secure what you can't see."** Network observability — SPAN/tap traffic → NDR, flow telemetry (NetFlow/IPFIX), packet capture (pcap), and DNS query logs — is the prerequisite to detection. Blind spots (DoH, QUIC, ECH) erode observability and must be compensated with endpoint telemetry.

### Architecture — Layered Defense and Zero-Trust in Networks

**The layered defense model** (see diagram above) progresses:
```
Perimeter NGFW → DMZ (public-facing) → Internal segmentation (VLAN + FW)
  → Microsegmentation (host-agent or service-mesh) → Host firewall (iptables/wf.msc)
```

**Zero Trust Architecture (NIST SP 800-207)** replaces the "trusted internal network" model:
| ZTA Principle | Network consequence |
|---|---|
| No implicit trust based on location | Internal VLAN = untrusted; every flow authenticated |
| Per-session, per-resource access | PEP + PDP per flow; no standing network access |
| Dynamic policy (identity + device posture + context) | IdP + MDM posture score + threat intel → allow/deny |
| Continuous verification | If posture degrades mid-session, terminate |

ZTNA (Zero Trust Network Access) is the practical implementation. **SDP (Software-Defined Perimeter)**: client must authenticate to a controller before receiving a per-app tunnel; the server is "dark" (no open ports) until authorized. **SWG (Secure Web Gateway)** and **CASB** fold into the SSE/SASE model: cloud-delivered security that collapses L3–L7 PEPs into the cloud.

**SDN architecture** changes the security model. Control plane is centralized (controller: ONOS, OpenDaylight, Cisco APIC, VMware NSX Manager) and data plane is distributed (OpenFlow switches, vSwitches). This means: (a) the controller becomes a Tier-0 asset — compromise = entire fabric, (b) microsegmentation can be enforced at the vSwitch per-VM, not at the physical firewall, (c) policy can be authored as code (Terraform, Ansible) and distributed instantly.

**CDN security**: edge distribution absorbs DDoS (anycast + capacity), TLS termination at edge (reduces origin server load but introduces edge trust — ensure CDN certs are short-lived and pinned), WAF federation (rules pushed to all PoPs). Risk: CDN edge becomes the new MITM point; logs at the edge are critical.

### Execution — Practical Security Deployment

**Writing Firewall Rules** — the dominant logic engines:
| Engine | Semantics | Example platforms |
|---|---|---|
| **First-match** | First rule whose selectors match → action; rest ignored | iptables, Cisco ACL, pf, PAN-OS, AWS NACL |
| **Policy-based (zone)** | Evaluate against zone-pair policy, then rule within zone | Juniper SRX, Fortinet, Check Point |
| **Best-match (longest-prefix)** | Routing table; not a firewall but often confused | BGP, OSPF |

**Critical ruleset hygiene**: default-deny with logged catch-all, most-specific-first ordering, egress filtering (block outbound from servers to internet — most C2 is outbound-initiated), anti-spoof (RFC 1918/5735 blocks on ingress), periodic rule review (≤90 d). Tools: Algosec, Firemon, Tufin detect shadowed and redundant rules. See [[firewall-types-and-rule-evaluation]].

**Configuring IPsec (IKEv2)** — the modern stack:
1. **IKE_SA_INIT** (2 messages): DH key exchange (group 19/20/21 or 31 Curve25519), proposal negotiation (AES-256-GCM, SHA-384, PRF)
2. **IKE_AUTH** (2 messages): mutual auth (cert preferred over PSK), first CHILD_SA
3. **NAT-T**: ESP wrapped in UDP/4500 (RFC 3948) — always enable; IKEv2 has it built-in
4. **MOBIKE** (RFC 4555): roaming client keeps SA when IP changes
5. **PFS rekey**: CREATE_CHILD_SA with new DH every X min/MB

**Transport vs Tunnel mode** — the exam-critical distinction:
- **Transport**: original IP header preserved, only payload protected. Host-to-host. Rare in practice.
- **Tunnel**: new outer IP header encapsulates entire original packet. Gateway-to-gateway or remote-access. Dominant.

The trick: ESP tunnel mode hides the *real* IP header; the outer header shows VPN gateway IPs only. AH is almost never used (can't NAT, no confidentiality). See [[vpn-types-and-ipsec-modes]].

**Deploying 802.1X** — the L2 port-security gold standard:
```
Supplicant (endpoint) ──EAPOL──► Authenticator (switch/WLC) ──RADIUS──► Auth Server (ISE/ClearPass → AD/IdP)
```

EAP type selection matrix (exam-essential):
| Scenario | Recommended EAP | Why |
|---|---|---|
| Windows domain, no client certs | EAP-PEAPv0 (MS-CHAPv2 inner) | Server cert only; lowest PKI burden |
| High-security, PKI in place | EAP-TLS | Mutual TLS; client cert required; strongest |
| BYOD / heterogeneous | EAP-TTLS or PEAP | Flexible inner methods |
| Carrier Wi-Fi offload | EAP-SIM / EAP-AKA' | SIM-based auth |
| Avoid | EAP-MD5, LEAP | No server auth; broken |

**DNS security deployment** — graduated approach:
1. **Baseline**: Disable open recursion, restrict zone transfers (AXFR), enable Response Rate Limiting (RRL), implement DNS firewall/RPZ (block known-bad at resolver)
2. **DNSSEC signing**: Generate KSK (offline HSM) + ZSK (online signer), publish DS at parent, automate ZSK rollover. Verify with `dnssec-validation auto` on recursive resolvers.
3. **Encrypted transport**: Deploy DoT (port 853, easy to log/block) or DoH (port 443, blends with HTTPS). Corporate decision: enforce DoH to a *corporate* resolver (retain visibility) or block external DoH entirely.
4. **Email posture**: SPF (`v=spf1`), DKIM (RSA 2048+ signing), DMARC (`p=reject` with aggregate reporting), MTA-STS (TLS-only), DANE (cert pinned via DNSSEC).

DNS edge cases that break security: split-horizon DNS leaks (internal IPs resolving externally), DoH as malware C2 (encrypted tunnel bypassing corporate DNS), DNSSEC bootstrap problem (if DNS is down, you can't validate DNS — chicken-and-egg), NSEC3 opt-out making zone walking possible again for unsigned delegations. See [[dns-security-and-dnssec]].

### Mastery — Advanced Network Security

**BGP security**: BGP is the internet's routing protocol — and it trusts by default.
| Control | What it does | Adoption (2024) |
|---|---|---|
| **RPKI (Resource Public Key Infrastructure)** | Validates route *origin* — ROA says "this AS is authorized for this prefix" | ~40% of routes validated by major ISPs |
| **BGPsec (RFC 8205)** | Validates entire AS *path* — each hop signs | Near-zero (operational burden) |
| **Prefix filtering** | Ingress: reject bogons, too-specific, your own prefixes. Egress: only advertise your allocations | Universal best practice |
| **BGP Flowspec (RFC 8955)** | Distribute DDoS drop rules via BGP to upstreams | Tier 1 ISPs, large enterprises |
| **Peering authentication** | BGP MD5 (deprecated) → TCP-AO (RFC 5925) | Should be universal |

**DDoS mitigation architecture**: Layers matter — volumetric (L3/L4) vs application (L7).
```
Anycast network (global PoPs) → Scrubbing centers (inspect + drop) → BGP Flowspec (upstream drops)
  → Rate-limit at edge (SYN cookies, conntrack tuning) → WAF (L7 challenge)
```

Architecture patterns: **Always-on** (Cloudflare Magic Transit, AWS Shield Advanced), **On-demand** (BGP redirect or DNS switch during attack), **Hybrid** (always-on L3/L4 + on-demand L7). The 2016 Dyn Mirai attack (1.2 Tbps) proved that even Tier 1 DNS infrastructure must be anycast + rate-limited.

**Network Traffic Analysis (NTA/NDR)**: Beyond signature-based IDS.
| Tool | Signature language / approach |
|---|---|
| **Zeek (Bro)** | Event-driven scripts (conn.log, dns.log, http.log, ssl.log) — protocol parsing |
| **Suricata** | Rule-based (alert, drop); emerging protocol parsing (QUIC, SMB, DNP3) |
| **JA3/JA4 fingerprinting** | TLS ClientHello hash → identify client application regardless of IP |
| **JA3S/JA4S** | Server-side fingerprint; detect impersonation |
| **RDP/RDP-sec fingerprinting** | Detect anomalous RDP use |

**QUIC/HTTP3 implications**: QUIC (RFC 9000) encrypts transport headers that firewalls historically relied on. TLS 1.3 is embedded in QUIC; there is no clear-text TCP handshake. Impact: (a) NGFW can no longer see TCP flags, SEQ/ACK numbers, (b) connection migration (QUIC session survives IP change — DPI loses session tracking), (c) 0-RTT replay vulnerability (firewalls must track and reject replayed 0-RTT data). NDR vendors compensate with JA4 fingerprinting, ML on encrypted traffic patterns, and endpoint telemetry.

**SD-WAN security**: Each SD-WAN edge device establishes encrypted tunnels (IPsec or proprietary) to all other edges via a central orchestrator. Security challenges: (a) trust model shifts from physical WAN circuits (MPLS) to zero-trust overlay, (b) every branch edge is now internet-facing, (c) centralized policy must ensure consistent security posture across all edges, (d) split-tunnel decisions determine which traffic goes through SSE vs direct-to-internet.

### Resilience — What Fails and Why

**Top network security mistakes** (from real incident patterns):
| Mistake | Mechanism | Consequence |
|---|---|---|
| Flat L2 networks (single VLAN) | No east-west barriers | Ransomware blast radius = entire org |
| No egress filtering | Outbound `any/any` from internal | C2 callbacks, data exfil over DNS/HTTPS |
| RDP/SSH exposed to internet | `permit tcp any any eq 3389/22` | Brute-force → entry vector; RDP is the #1 initial-access vector for ransomware (per Verizon DBIR) |
| Outdated SSL VPN concentrators | Unpatched Pulse Secure, Fortinet, Ivanti CVEs | Tier-0 bypass; attacker lands inside trust zone |
| IPv6 forgotten | Firewall rules IPv4 only; IPv6 traffic passes unmonitored | Complete bypass; enable and firewall IPv6 or disable it enterprise-wide |
| DNS running on a single server | Single point of failure | Anycast + secondary + recursive diversity |
| Shared credentials on network devices | `admin/admin`, shared `enable` | Lateral movement at L2/L3 |
| WAF in detect-only mode indefinitely | "We'll switch to block after tuning" — never happens | Prod WAF is window dressing; switch to block after 30 d of tuning |

**When WAF bypasses succeed**: WAFs parse HTTP — disagreements between WAF and backend parser enable smuggling and injection. Common bypasses: HTTP request smuggling (TE.CL or CL.TE), encoding tricks (double URL-encoding, Unicode normalization), parameter pollution, oversized headers (WAF truncates, backend reads full), and JSON/XML nesting (WAF only inspects first N levels). Continuous testing with tools like Burp Suite, OWASP ZAP, and WAF-bench (GoTestWAF) is essential.

**DNS as the Achilles heel**: Every service depends on DNS; almost no service *validates* DNS responses. Single points of failure: authoritative name servers (single provider), recursive resolvers (single cluster), registrar account takeover (if attacker owns your registrar, they own your DNS). Mitigations: multi-provider authoritative DNS, anycast resolver pools, DNSSEC from root to leaf, registrar with hardware MFA and registry lock.

### Context — Network Security in Different Environments

**Cloud network security** — primitives differ from on-prem:
| Primitive | AWS | Azure | GCP |
|---|---|---|---|
| **Stateful FW** | Security Group (allow only, per-ENI) | NSG (per-NIC/subnet) | VPC Firewall (hierarchical) |
| **Stateless FW** | NACL (subnet-level) | — | — |
| **WAF** | AWS WAF (on ALB/CloudFront) | Azure WAF (on App Gateway/Front Door) | Cloud Armor |
| **L3 isolation** | VPC | VNet | VPC |
| **Inter-VPC** | Transit Gateway | VNet Peering / VWAN | VPC Peering / NCC |
| **Private link** | PrivateLink / VPC Endpoint | Private Link | Private Service Connect |

Key difference: Security Groups are **stateful** and **allow-only** (return traffic auto-allowed). NACLs are **stateless** and support both allow and deny — but return traffic must be explicitly allowed (ephemeral ports). The combination of SG (per-resource) + NACL (per-subnet) provides layered defense.

**OT/ICS network security** — the Purdue model:
```
L5: Enterprise (ERP, email) ──── DMZ ──── L4: Site business (historian)
L3: Site operations (HMIs, engineering)     L2: Area supervisory (SCADA, PLCs)
L1: Basic control (sensors, actuators)      L0: Physical process
```
OT moves data *up*; control moves *down*. Any L3↔L4 crossing must go through an **industrial DMZ** (historian + jump host). Protocol-specific risks: Modbus TCP (no auth, no encryption — cleartext function codes), DNP3 (no auth by default; DNP3-SA adds TLS), PROFINET (real-time — any latency spike from security = process failure). Use passive IDS (Nozomi, Dragos, Claroty) for OT — never inline IPS that could drop control packets.

**Branch office vs headquarters design**: Branch is a *mini-HQ* but with weaker physical security, fewer staff, and lower bandwidth. Design pattern: local switching + SD-WAN edge + cloud SSE for internet egress (no on-prem firewall stack). HQ retains full perimeter + internal segmentation + NDR. Shared policy via cloud orchestrator.

### Nuance — Important Distinctions

**Stateful vs stateless** — both matter:
- **Stateful** (NGFW, SG, iptables ct): tracks flows, auto-allows return traffic, prevents half-open attacks. Vulnerable to state table exhaustion (SYN flood).
- **Stateless** (NACL, packet filter): evaluates every packet independently. Immune to state exhaustion; must explicitly allow both directions. Used at extreme scale (cloud edge, carrier ACLs).
- Best practice: stateful for per-flow policy; stateless for coarse subnet-level guards.

**Implicit deny vs explicit deny**: Implicit deny = the default behavior at end of ACL (if no rule matches, drop). Explicit deny = a written rule with `deny ip any any log` that makes the intent visible and generates logs. **Always use explicit deny with logging** — implicit deny silently drops without evidence.

**IDS vs IPS**: IDS (detect, alert) is out-of-band (SPAN/tap) and safe to deploy immediately. IPS (prevent, block) is inline and can break production — false positive = outage. **Most orgs run IDS mode first** (30–90 d) to tune signatures, then switch to IPS. In OT/ICS, **never** use inline IPS — a false positive blocking a SCADA packet can cause physical damage.

**Layered L2 security**: Port security (MAC limit + sticky) → MAC filtering (allow-list) → 802.1X (dynamic, identity-bound). Port security prevents CAM overflow; MAC filtering adds static allow-lists; 802.1X makes it identity-aware. Layered, not exclusive — if 802.1X auth server is unreachable, port security is the fallback.

### Resources — Quick-Reference Toolkit

**Firewall rule review checklist**:
- [ ] No `any/any/allow` rules on internal firewalls
- [ ] Catch-all `deny any any log` on all perimeter firewalls
- [ ] Anti-spoof rules (RFC 1918, 5735, loopback, multicast) on ingress
- [ ] No rules older than 90 d without review
- [ ] Logging enabled on deny rules (forwarded to SIEM)
- [ ] TLS inspection policy documented and legally reviewed
- [ ] Egress rules default-deny; allow-list by service/port
- [ ] Rule shadowing and redundancy analyzed (automated tool)
- [ ] IPv6 rules exist and match IPv4 policy
- [ ] Change control: FW rules in version control (GitOps)

**Network segmentation decision tree**:
```
Is the asset internet-facing? ──Yes──► DMZ (dual-FW sandwich for high-value)
    │No
Is it user endpoint? ──Yes──► VLAN + 802.1X + NAC (posture-checked)
    │No
Is it server-to-server only? ──Yes──► Microsegmentation (host-agent or SG)
    │No
Is it OT/ICS? ──Yes──► Purdue model + industrial DMZ + passive IDS
    │No
Is it IoT? ──Yes──► Isolated VLAN + internet-only egress + device cert
```

**Common port numbers quick reference**:
| Range | Classification | Examples |
|---|---|---|
| **0–1023** | Well-known / System | 22 (SSH), 53 (DNS), 80 (HTTP), 443 (HTTPS), 636 (LDAPS), 3389 (RDP) |
| **1024–49151** | Registered | 1812/1813 (RADIUS), 3389 (RDP), 5432 (PostgreSQL), 8080 (HTTP alt) |
| **49152–65535** | Dynamic / Ephemeral | Client-side source ports; NACL must allow return to ephemeral range |

**IPsec configuration template** (IKEv2, modern):
```
IKE proposal:
  encryption: aes-256-gcm16
  prf: sha384
  dh-group: 19 (ECP-256) or 31 (Curve25519)
  lifetime: 86400s

ESP proposal (CHILD_SA):
  encryption: aes-256-gcm16
  dh-group (PFS): 20 (ECP-384)
  lifetime: 28800s or 100MB

Auth: ECDSA P-256 certificate (avoid PSK in production)
NAT-T: enabled (UDP/4500)
MOBIKE: enabled (for roaming clients)
DPD: built into IKEv2
Avoid: 3DES, DES, MD5, SHA-1, DH group 1/2/5, aggressive mode
```

<!-- /EXPANDED -->

## CISO / Risk Manager View

For the board, Domain 4 is best framed as **"the network is the largest single attack surface we own, and the place where most of the high-impact breaches (lateral movement, ransomware spread, supply-chain pivot) succeed or fail."** Three questions the CISO must answer:

1. **Where are the trust boundaries, and what enforces them?** If the answer is "one firewall at the edge", the network is effectively flat. Lateral movement risk → ransomware blast radius.
2. **What is the encryption posture of data in transit?** Map it: TLS 1.2+ for all web/API, IPsec or MACsec for WAN, WPA3 for Wi-Fi, DoH/DoT for DNS. Anything older is a finding.
3. **Are we paying for visibility we don't use, or missing visibility we need?** NDR/SIEM licensing is a board topic; consolidation is a 2024–2025 trend (Gartner SASE/SSE).

**Segmentation ROI** is the single best network-security argument to a CFO: a single flat network can take a $4M ransomware event and turn it into a $40M one. The cost of microsegmentation (and even basic VLAN+FW work) is one to two orders of magnitude smaller than the avoided loss expectancy. **Remote-work risk** has become a permanent line item: every remote user is a new branch office, and the network stack (ZTNA, SASE, identity-aware proxy, MDM) must extend the corporate trust model to the kitchen table.

## Related Connections

### Layer 2 deep-dives (siblings)
- [[osi-tcpip-models-and-encapsulation]]
- [[network-security-controls]]
- [[network-architecture-segments]]
- [[wireless-and-mobile-security]]
- [[secure-protocols-and-services]]

### Layer 3 specifics (children)
- [[firewall-types-and-rule-evaluation]]
- [[vpn-types-and-ipsec-modes]]
- [[dns-security-and-dnssec]]
- [[5g-and-iot-network-security]]

### Cross-domain
- [[Domain 3 — Security Architecture and Engineering]] (crypto, PKI, TLS primitives)
- [[Domain 5 — Identity and Access Management]] (Kerberos, 802.1X, NAC identity)
- [[Domain 7 — Security Operations]] (SIEM, NDR, log, network ops)

## References

- ISC² CISSP CBK, 5th ed. — Domain 4
- NIST SP 800-41r1, *Guidelines on Firewalls and Firewall Policy*
- NIST SP 800-77r1, *Guide to IPsec VPNs*
- NIST SP 800-48r1, *Guide to Securing Legacy IEEE 802.11 Networks*
- NIST SP 800-63B, *Digital Identity Guidelines — Authentication*
- NIST SP 800-175B, *Cryptographic Standards* (TLS guidance)
- RFC 5246 / 8446 — TLS 1.2 / 1.3
- RFC 4301–4303 — IPsec (architecture, AH, ESP)
- RFC 4033–4035 — DNSSEC
- RFC 9596 — DoH (DNS over HTTPS)
- IEEE 802.1X-2010, 802.11ax-2021
- 3GPP TS 33.501 — 5G security architecture
- OWASP *Transport Layer Protection Cheat Sheet*
