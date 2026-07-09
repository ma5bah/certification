---
title: VPN Types & IPsec Modes
layer: 3
domain: 4
tags: [cissp, domain-4, vpn, ipsec, ikev2, transport-mode, tunnel-mode, site-to-site, remote-access, ssl-vpn, wireguard, mpls, l2tp, gre]
related_domains: [3, 5, 7]
parent: network-security-controls
---

# VPN Types & IPsec Modes

## Feynman Explanation

A VPN is a **private tunnel inside a public network** — like sending a locked briefcase through a public mail system. The briefcase is the encrypted payload; the locks are the keys negotiated between you and the receiver. IPsec builds the tunnel at the *network layer* (so any application can use it without changes), SSL/TLS VPNs build it at the *application layer* (so they only protect browser traffic), and MPLS builds it at the *carrier layer* (the carrier itself isolates your traffic from others). The most important choice is **who** ends the tunnel: end-to-end (transport mode, your laptop) or gateway-to-gateway (tunnel mode, your office firewall). That single decision changes the threat model, the trust boundary, and the compliance story.

## Technical Details

### 1. The two fundamental IPsec modes

| Mode | Header added | What is protected | Where it ends | Original IP header | Typical use |
|---|---|---|---|---|---|
| **Transport mode** | AH or ESP header inserted between original IP header and payload | IP payload (L4 + data) | Host-to-host | Preserved, visible | End-to-end between two hosts (rare in practice) |
| **Tunnel mode** | New outer IP header + AH or ESP | Entire original IP packet | Gateway-to-gateway (or host-to-gateway) | Encapsulated, hidden | Site-to-site VPN, remote access VPN |

#### 1.1 Visual: transport vs tunnel

**Transport mode (AH + ESP, hypothetical):**
```
[ IP hdr | AH | TCP | data | ESP | ... | ]
          └ protected, but IP src/dst visible
```

**Tunnel mode (ESP, common):**
```
[ Outer IP hdr | ESP | Inner IP hdr | TCP | data | ESP trailer | ESP auth ]
                  └ entire inner packet encrypted & authenticated
```

- **ESP transport**: `[IP | ESP | TCP | data | ESP-trail | ESP-auth]`
- **ESP tunnel**:  `[Outer-IP | ESP | Inner-IP | TCP | data | ESP-trail | ESP-auth]`
- **AH transport**: `[IP | AH | TCP | data | AH-auth]`
- **AH tunnel**:  `[Outer-IP | AH | Inner-IP | TCP | data | AH-auth]`

**AH covers** the IP header (read-only fields) + payload → binds identity to packet at L3.
**ESP covers** the *payload* (not the outer IP header) → can NAT.

AH is **rarely used** in practice; ESP with authentication is the standard.

### 2. IPsec protocol suite — the components

| Component | RFC | Function |
|---|---|---|
| **AH** | 4302 | Integrity + auth, NO confidentiality |
| **ESP** | 4303 | Integrity + auth + confidentiality |
| **IKEv2** | 7296 | Key exchange, peer auth, SA negotiation |
| **IKEv1** | 2407, 2408, 2409 | Older; legacy |
| **NAT-T** | 3948 | Encapsulate ESP in UDP 4500 to traverse NAT |
| **MOBIKE** | 4555 | Roaming client changes IP; keep SA |
| **DPD** | built-in to IKEv2 (RFC 3706 for v1) | Dead peer detection |
| **SA** | 4301 | Security Association — one-way logical channel |
| **SPD** | 4301 | Policy: BYPASS / DISCARD / PROTECT |
| **SAD** | 4301 | Active SAs |
| **ESP NULL** | 4303 | Auth-only ESP (no encryption) |
| **Combined-mode** | 4303 | ESP with AEAD (AES-GCM); one pass for both |

### 3. ESP header (RFC 4303)

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|               Security Parameters Index (SPI)                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                      Sequence Number                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    IV (cipher-suite specific)                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Payload Data (variable)                     |
~                                                               ~
+               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|               |     Padding (0-255 bytes)                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Pad Length | Next Header |   ICV (Integrity Check Value)       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- **SPI**: 32-bit; with dst IP + proto = SA identifier
- **Seq#**: 32-bit; anti-replay window (default 32–128 packets)
- **Next Header**: TCP(6), UDP(17), IPv4(4), IPv6(41)
- **ICV**: integrity checksum (e.g. AES-GCM 16-byte tag)

### 4. IKEv2 handshake in detail

#### 4.1 IKE_SA_INIT (1-RTT key exchange)

```
Initiator                                Responder
  | -- HDR, SAi1, KEi, Ni -->           |  (proposal, DH public, nonce)
  | <-- HDR, SAr1, KEr, Nr, [CERTREQ] -- |  (chosen, DH public, nonce, optional cert req)
```

After this: keys derived, but peers not yet authenticated.

#### 4.2 IKE_AUTH (mutual auth + first CHILD_SA)

```
  | -- HDR, SK { IDi, [CERT,] [CERTREQ,] AUTH, SAi2, TSi, TSr } -->
  | <-- HDR, SK { IDr, [CERT,] AUTH, SAr2, TSr, TSi } --
```

- **IDi/IDr**: identity (ID_FQDN, ID_RFC822_ADDR, ID_IPv4_ADDR, ID_DER_ASN1_DN)
- **AUTH**: payload proves possession of private key / PSK
- **SAi2/SAr2**: child SA proposal (ESP, algorithms, mode)
- **TSi/TSr**: traffic selectors (e.g. 10.0.0.0/8 ↔ 10.1.0.0/8)

#### 4.3 IKEv2 vs IKEv1

| Property | IKEv1 | IKEv2 |
|---|---|---|
| Exchange | 6+ messages (Main Mode, Aggressive Mode) | 4 messages |
| Auth methods | PSK, cert, K3 (legacy) | PSK, cert, EAP |
| NAT-T | Optional, vendor | Built-in |
| MOBIKE | None | Yes (RFC 4555) |
| DPD | Optional (RFC 3706) | Built-in (RFC 5996) |
| Reliability | UDP, manual retry | UDP, windowed retransmit |
| PFS rekey | Optional, re-IKE | Built-in via CREATE_CHILD_SA |
| NAT issues | Breaks without NAT-T | Works with NAT-T always on |
| Mobile | Poor | Strong |

### 5. Authentication options for IPsec

| Auth | Pros | Cons | When |
|---|---|---|---|
| **Pre-Shared Key (PSK)** | Simple, no PKI | Doesn't scale; weak if low-entropy | Small/test labs |
| **RSA / ECDSA / EdDSA certificate** | Scalable, revocable, identity-based | Need PKI | Enterprise site-to-site |
| **EAP** (RFC 4739) | Works with backend (RADIUS/AD) | Slightly more complex | Remote access with AD auth |
| **Hybrid (PSK + cert)** | Defense in depth | Operational complexity | High-assurance |

### 6. VPN architecture taxonomy

#### 6.1 By where the tunnel terminates

| Type | Tunnel ends at | Use |
|---|---|---|
| **Host-to-host (transport)** | Each host | Server-to-server, host-to-host |
| **Host-to-gateway (remote access)** | User laptop → corp gateway | Remote worker (legacy) |
| **Gateway-to-gateway (site-to-site)** | Branch FW → HQ FW | Branch office |
| **Hub-and-spoke** | Branch spokes → central hub | Multi-site enterprise |
| **Full mesh** | Every site ↔ every site | Low-latency multi-site |
| **Partial mesh** | Hybrid | Most enterprises |
| **DMVPN / FlexVPN** | Dynamic spoke-to-spoke | Cisco architectures |
| **SD-WAN overlay** | CPE → cloud / on-prem | WAN transformation |

#### 6.2 By what layer the tunnel operates at

| Type | Layer | Protocol | Pros | Cons |
|---|---|---|---|---|
| **L2 VPN (L2TP)** | L2 | L2TPv3 | Ethernet extension | No crypto; pair with IPsec |
| **L3 VPN (IPsec, GRE, MPLS)** | L3 | IPsec, GRE+IPsec, MPLS | Network extension | Per-flow state |
| **L4 VPN (WireGuard, OpenVPN)** | L4 | UDP-wrapped | App-agnostic | Single-purpose |
| **L7 VPN (TLS, SSH tunnel)** | L7 | HTTPS, SSH SOCKS, stunnel | Easy to deploy; survives NAT | App-specific; harder to enforce |
| **Carrier L2/L3 (MPLS, VPLS, EVPN)** | L2/L3 | MPLS with BGP/RFC4364 | SLA; deterministic | Expensive; carrier-managed |

### 7. IPsec vs SSL/TLS VPN

| Property | IPsec | SSL/TLS VPN |
|---|---|---|
| Layer | L3 (transparent to apps) | L4–L7 (per app or browser) |
| Client | OS / third-party (Cisco, strongSwan) | Browser only (clientless) or thin client |
| NAT traversal | NAT-T (UDP 4500) | Native (TCP 443) |
| Firewall traversal | UDP 500/4500 (sometimes blocked) | TCP 443 (almost always open) |
| Granular access | Network-level (per host/subnet) | App-level (URL or path) |
| User experience | Heavy client; full network access | Light; portal or per-app |
| Clientless mode | No | Yes (web proxy) |
| Mobile | Strong with MOBIKE | Better on shared devices |
| Split tunneling | Yes (carefully) | Yes (per-app) |
| Modern trend | Still dominant site-to-site, WAN | Dominant remote access (ZTNA successor) |

### 8. Other VPN protocols — exam relevant

| Protocol | RFC | Notes |
|---|---|---|
| **L2TP** (Layer 2 Tunneling Protocol) | 2661 | Tunnel only, no crypto; typically paired with IPsec → "L2TP/IPsec" |
| **PPTP** | Deprecated | **Insecure** (MS-CHAPv1 broken; MPPE weak) |
| **GRE** | 2784 | Generic encapsulation; tunnels multicast, IPv6; no crypto; typically paired with IPsec |
| **DMVPN** | Cisco | mGRE + NHRP + IPsec; dynamic spoke-spoke |
| **SSL VPN (legacy)** | Vendor-specific | OpenVPN, Cisco AnyConnect, Pulse Secure, F5 APM |
| **OpenVPN** | – | TLS-based; UDP or TCP; widely used; client app |
| **WireGuard** | – (becoming RFC) | UDP-only, modern crypto (Noise IK, ChaCha20-Poly1305), minimal config; kernel module |
| **SSH tunneling** | 4254 | Local/remote/dynamic port forwarding; quick win for sysadmin, not enterprise-grade |
| **SSTP** | Microsoft | SSL over TCP 443; Windows-only client; useful when IPsec/UDP blocked |
| **SoftEther** | – | Multi-protocol, educational, also used in some prod |

### 9. WireGuard — the modern minimalist

| Property | WireGuard |
|---|---|
| Crypto | Curve25519 (ECDH), ChaCha20-Poly1305 (AEAD), BLAKE2s (hash), HKDF, Noise IK |
| Key model | Static keypairs (no PKI); peers listed explicitly with allowed IPs |
| Configuration | Plaintext INI-like file |
| Transport | UDP only |
| Roaming | Excellent (just re-bind on new IP) |
| State | Stateless on server side, per-peer |
| Throughput | Very high (kernel implementation) |
| Audit | Small codebase (~4k LoC) — auditable |
| Caveats | No IKEv2-style auth negotiation; no dynamic IP discovery without NAT tricks |
| Production | Mainline Linux kernel 5.6+; iOS, macOS, Android, Windows |

### 10. Remote-access VPN specifics

| Concern | IPsec RA | TLS RA | ZTNA |
|---|---|---|---|
| Posture check | Limited (NAC separate) | Some clients check | **Built-in** |
| Per-app tunnel | No (full network) | Some (per-host) | **Per-app** |
| Identity binding | IP/PSK/cert | User cert | **IdP + device + context** |
| Lateral movement risk | **High** (full LAN) | Medium | **Low** (broker-mediated) |
| Compliance | Good | Good | **Best** |
| Trend | Stable | Declining | **Dominant** |

#### 10.1 ZTNA as VPN replacement

| Capability | ZTNA |
|---|---|
| Tunnel type | Reverse (server-to-client) or broker-mediated |
| Identity | IdP (OIDC, SAML) + device posture + context |
| Access | Per-app, per-session |
| Visibility | Broker sees metadata + can apply DLP |
| Connectivity | UDP/TCP, often MASQUE/QUIC under the hood |
| Examples | Zscaler ZPA, Cloudflare Access, Netskope Private Access, Prisma Access, Microsoft Entra Private Access |

### 11. Site-to-site VPN design

#### 11.1 Hub-and-spoke vs full mesh

```
   Hub-and-spoke (3 sites):        Full mesh (3 sites):
   S1──┐                          S1═══S2
       ├──Hub (HQ)                 │   │
   S2──┘                          └───S3
                                   (S1═══S3)
   Pros: simpler config,            Pros: no hairpin
   Cons: hairpin latency            Cons: N*(N-1)/2 tunnels
```

#### 11.2 High availability

- **Dual-tunnel**: each site has two tunnels to two remote endpoints
- **IPsec DPD + IKEv2 keepalive** detect failure
- **Routing**: BGP or IP SLA + floating static
- **Equal-Cost Multi-Path (ECMP)**: load-balance over both tunnels

#### 11.3 Crypto / IKE proposal — modern best practice

**IKE proposal** (Phase 1 / IKE_SA_INIT):
- Encryption: AES-256-GCM (or ChaCha20-Poly1305)
- PRF: SHA-384
- Integrity: (implicit if GCM) or HMAC-SHA-384
- DH group: 19 (ECP-256), 20 (ECP-384), 21 (ECP-521), or 31 (Curve25519)
- Lifetime: 86400 s (24 h)

**ESP proposal** (Phase 2 / CHILD_SA):
- Encryption: AES-256-GCM (or ChaCha20-Poly1305)
- Integrity: (implicit if GCM)
- DH group (PFS): 20, 21, 31 — **rekey with new DH**
- Lifetime: 28800 s (8 h) or 100 MB

Avoid: 3DES, DES, MD5, SHA-1, DH group 1/2/5, aggressive mode, PSK in IKEv1 Main Mode with weak key.

### 12. VPN in cloud

| Cloud scenario | Pattern |
|---|---|
| Branch to AWS VPC | Site-to-site IPsec (AWS VPG supports IKEv2) |
| AWS VPC to Azure VNet | Cloud-native peering or IPsec (e.g. Cisco CSR, Aviatrix, Strongswan) |
| SaaS to corp | ZTNA broker; some vendors offer IPsec for fixed egress |
| Cloud SD-WAN | Vendor CPE + multi-cloud backbone (e.g. Viptela, Versa, Cato) |
| Transit VPC | Hub VPC with NGFW + IPsec; spokes connect via peering or transit gateway |

### 13. Common VPN failure modes (CISSP traps)

| Issue | Symptom | Fix |
|---|---|---|
| **Phase 1 mismatch** (proposal) | SA never establishes | Match encryption/PRF/DH/lifetime exactly |
| **Phase 2 mismatch** (transform) | Tunnel up, no traffic | Match ESP proposal, PFS, lifetime |
| **Asymmetric routing** | Traffic one-way | Add return path; route map |
| **MTU issues** (tunnel reduces MTU) | Some sites work, others don't | Set TCP MSS clamping to 1340; lower PMTU |
| **NAT + AH** | Drops | Use ESP, not AH; enable NAT-T |
| **Fragmentation** | Slow, drops | DF-bit clear, fragment before encrypt |
| **PSK with weak entropy** | Offline crackable | Use certificates or strong PSK (≥ 22 random chars) |
| **Tunnel never rekeys** | Stuck after lifetime | Verify peer supports rekey; check rekeying config |
| **Phase 1 ID mismatch** | AUTH fails | Match IKE ID type and value |
| **Routing recursion** | Packets bounce | Avoid routing tunnel dst via tunnel |
| **Tunnel with overlapping subnets** | Half works | Use NAT on tunnel or re-IP |

### 14. Compliance angle (regulators love this)

| Control framework | What it says about VPN |
|---|---|
| **PCI DSS 4.0** | Strong crypto for transmission over open networks; trusted keys/certs |
| **HIPAA Security Rule** | §164.312(e)(1) — transmission security |
| **GDPR** | Art. 32 — appropriate technical measures; pseudonymization considered |
| **NIST SP 800-53** | SC-8 (Confidentiality), SC-9 (Integrity), SC-12/13 (Key management) |
| **NIST SP 800-77r1** | IPsec deployment guide |
| **CIS Controls** | 12.x — network infrastructure; 13.x — encryption |

## CISO / Risk Manager View

**Board framing**: VPN is the *de facto* remote-work plumbing — and the *de jure* liability if breached. A compromised VPN gateway (think Pulse Secure 2019–2021, Fortinet 2022, Ivanti 2024) is one of the highest-impact single events an enterprise can suffer, because it bypasses every perimeter control and lands the attacker *inside* the trust zone.

Strategic narratives:

1. **"Our remote access is the most-targeted surface in our network."** Invest in patching VPN concentrators, hard crypto, network segmentation behind the VPN, and replace with ZTNA where possible. Treat VPN as Tier 0.
2. **"Site-to-site VPN is a CAPEX/OPEX problem, not just a security one."** SD-WAN has replaced most enterprise IPsec WANs by 2024 for performance reasons; security follows. Keep IPsec as backup; do not run a business-critical app over a single IPsec tunnel.
3. **"VPN bypasses are how the big incidents start."** Post-VPN, the attack surface moves to IdP compromise (MFA fatigue, session theft). Fund passkeys (FIDO2), conditional access, and ZTNA.
4. **"Compliance doesn't equal security."** An IPsec tunnel with PSK and 3DES is *PCI compliant* if the auditor is asleep. Mandate AES-GCM, ECDH group 19+, and PFS.

KPIs:

| KPI | Target |
|---|---|
| % of VPN concentrators patched within 7 d of CVE | 100% |
| % of remote-access users on ZTNA | ≥ 70% (and growing) |
| % of IPsec tunnels using PFS + AES-GCM | 100% |
| % of tunnels with high-availability (dual) | 100% for Tier 0 |
| Mean time to revoke user access | < 4 h |
| VPN authentication via MFA | 100% |
| VPN client posture check coverage | 100% (or move to ZTNA) |

## Related Connections

### Parent (L2)
- [[network-security-controls]]

### Siblings (L3 in this domain)
- [[firewall-types-and-rule-evaluation]] — VPN traffic traverses FW; rules must allow UDP 500/4500
- [[dns-security-and-dnssec]] — split-tunnel DNS leaks
- [[5g-and-iot-network-security]] — private 5G/IoT replaces some VPN use cases

### Cross-domain
- [[Domain 3 — Security Architecture and Engineering]] — IPsec / TLS crypto primitives
- [[Domain 5 — Identity and Access Management]] — VPN auth = user+device identity
- [[Domain 7 — Security Operations]] — VPN concentrator is a Tier-0 asset to monitor

## References

- RFC 2401–2412 (IPsec v1 legacy)
- RFC 4301 — Security Architecture for IP
- RFC 4302 — AH
- RFC 4303 — ESP
- RFC 4306 — IKEv2
- RFC 4718 — IKEv2 clarifications
- RFC 4945 — IPsec Profile
- RFC 3948 — UDP Encapsulation of ESP (NAT-T)
- RFC 4555 — MOBIKE
- RFC 3706 — DPD
- RFC 4739 — EAP in IKEv2
- RFC 7296 — IKEv2 (current)
- RFC 9370 — Multiple Key Exchanges in IKEv2
- RFC 8247 — Algorithm Implementation Requirements for IKEv2
- RFC 7321 — IPsec Architecture Profile
- RFC 5685 — IKEv2 Redirect
- RFC 7383 — IKEv2 Renegotiation
- RFC 7427 — IKEv2 Signature Authentication
- NIST SP 800-77r1 — Guide to IPsec VPNs
- NIST SP 800-57 — Key Management
- NSA *Commercial National Security Algorithm Suite 2.0* (CNSA 2.0, 2022) — recommends ML-KEM/ML-DSA/EdDSA
- WireGuard whitepaper, J. Donenfeld
- CIS *IPsec VPN Benchmark*
- Cloud Security Alliance — *Software-Defined Perimeter* (SDP / ZTNA 1.0)
- ENISA, *Algorithms, Key Sizes and Parameters Report*
- IETF *draft-ietf-ipsecme-ikev2-mutual-eap* (extended EAP)
- Cisco *IPsec configuration best practices*
- Palo Alto *GlobalProtect* and *Prisma Access* admin guides
- F5 *APM VPN* admin guide
- Zscaler *ZPA Architecture*
- Microsoft *Always On VPN*
