---
title: OSI / TCP-IP Models and Encapsulation
layer: 2
domain: 4
tags: [cissp, domain-4, osi, tcp-ip, encapsulation, pdu, sdu, frame, packet, segment]
related_domains: [3, 7]
parent: domain-04-communication-and-network-security
---

# OSI / TCP-IP Models and Encapsulation

## Feynman Explanation

Imagine sending a letter inside seven nested envelopes. The outermost envelope has the address of the local post office; the next has the regional hub; the next has the country office; and so on — each layer only knows about its own envelope and trusts the layers above and below to do their job. The OSI seven-layer model is exactly that: each layer adds (or strips) its own envelope — called a **header** — and passes the result to the next layer. The TCP/IP four-layer model is the practical version the internet actually uses. Encapsulation is the act of *putting the inner envelope into the next one* on the way out; de-encapsulation is the reverse on the way in. Every network attack maps to a specific envelope being forged, torn, or peeked into.

## Technical Details

### 1. The two models side-by-side

| # | OSI layer | OSI name | TCP/IP layer | TCP/IP name | PDU name | Header (key fields) | Example protocols / RFC |
|---|---|---|---|---|---|---|---|
| 7 | Layer 7 | Application | Application | Application / Process | **Data** | HTTP method, URI, status | HTTP/1.1 (RFC 9110), HTTP/2 (RFC 9113), HTTP/3 (RFC 9114), DNS (RFC 1035), SMTP (RFC 5321), Kerberos (RFC 4120), S/MIME (RFC 8551) |
| 6 | Layer 6 | Presentation | Application | (folds into App) | **Data** | TLS record type, version, cipher suite | TLS 1.2 (RFC 5246), TLS 1.3 (RFC 8446), XDR/ASN.1, MIME |
| 5 | Layer 5 | Session | Application | (folds into App) | **Data** | Session ID, ticket | TLS session resumption, RPC, SIP, NetBIOS |
| 4 | Layer 4 | Transport | Transport | Host-to-host | **Segment** (TCP) / **Datagram** (UDP) | Ports, seq/ack, flags, checksum, window | TCP (RFC 9293), UDP (RFC 768), SCTP (RFC 9260) |
| 3 | Layer 3 | Network | Internet | Internetwork | **Packet** | Version, TTL, src/dst IP, protocol, checksum, options | IPv4 (RFC 791), IPv6 (RFC 8200), ICMP (RFC 792), IPsec (RFC 4301) |
| 2 | Layer 2 | Data Link | Link | Network access | **Frame** | MAC src/dst, EtherType, FCS/CRC | Ethernet II (IEEE 802.3), 802.11 Wi-Fi, PPP, ARP (RFC 826) |
| 1 | Layer 1 | Physical | Link | (hardware) | **Bits** | Preamble, line coding, voltage/frequency | 1000BASE-T, 802.11ax (Wi-Fi 6/6E), SONET, OTN |

> **Mnemonic top→bottom**: *All People Seem To Need Data Processing* (Application → Physical).
> **Mnemonic bottom→top**: *Please Do Not Throw Sausage Pizza Away* (Physical → Application).

### 2. Encapsulation: putting envelopes inside envelopes

```
   Application data
        │  ← HTTP request: "GET /index.html HTTP/1.1\r\n..."
        ▼
   TLS record  (type, version, ciphertext)
        │  ← PCI: TLSPlaintext → TLSCiphertext
        ▼
   TCP segment  (src port, dst port, seq, ack, flags, window, checksum, [TLS data])
        │  ← PCI: 20-byte (no options) TCP header
        ▼
   IP packet    (version, IHL, total len, ID, flags, frag, TTL, proto=6, cksum, src IP, dst IP, [TCP segment])
        │  ← PCI: 20-byte (no options) IPv4 header
        ▼
   Ethernet frame (dst MAC, src MAC, EtherType=0x0800, [IP packet], FCS=CRC-32)
        │  ← PCI: 14-byte header + 4-byte trailer
        ▼
   Bits on the wire (preamble + encoded symbols + IFG)
```

| Term | Meaning |
|---|---|
| **SDU** (Service Data Unit) | The payload received from the layer above — what this layer is asked to carry |
| **PDU** (Protocol Data Unit) | The unit this layer produces on the wire = SDU + this layer's header/trailer (PCI) |
| **PCI** (Protocol Control Information) | The header (and sometimes trailer) added by this layer |
| **ICI** (Interface Control Information) | Out-of-band signaling between adjacent layers on the same host |

### 3. Header sizes that matter

| Layer | Header (min) | Header (typical max) | Notes |
|---|---|---|---|
| Ethernet II | 14 B | 22 B (Q-tag adds 4) | 4-byte FCS trailer is **not** counted in length |
| IPv4 | 20 B | 60 B (IHL=15 → 60) | Options field is rarely used; can host attacks (Ping of Death, TearDrop) |
| IPv6 | 40 B | 40 B (fixed!) | Extension headers (RFC 8200) replace options; Hop-by-Hop, Routing, Fragment, ESP, AH, Destination |
| TCP | 20 B | 60 B (data offset = 15) | MSS typically 1460 (Ethernet − 40 IP/TCP) |
| UDP | 8 B | 8 B | No handshake → cheap to spoof |
| TLS 1.3 record | 5 B | 5 B | ContentType, version, length, fragment, AuthTag (16) |

**MTU cascade** (typical Ethernet path):

```
App MSS:           1460 bytes  (default)
TCP segment:       1480 bytes  (MSS + 20 TCP)
IP packet:         1500 bytes  (segment + 20 IP)  ← Ethernet MTU
Ethernet frame:    1518 bytes  (packet + 14 hdr + 4 FCS)
```

If any hop has MTU < 1500 (PPPoE 1492, IPsec tunnel mode often 1420), IPv4 fragments; IPv6 requires Path MTU Discovery (RFC 8201) and drops oversized silently.

### 4. De-encapsulation on receive

| Step | NIC | Driver / OS | Network stack | App |
|---|---|---|---|---|
| Strip preamble + FCS, check CRC | ✓ | | | |
| Parse EtherType → hand to IP | | ✓ | | |
| Strip IP header, check cksum, route | | | ✓ | |
| Strip TCP header, reassemble segments in order | | | ✓ | |
| Strip TLS record, decrypt | | | (TLS lib) | |
| Deliver plaintext to socket | | | | ✓ |

This is why **defense in depth works**: an attacker who can spoof L2 (ARP poison) is *still* stopped at L7 if mutual TLS is enforced; a TLS-stripping MITM is still caught if the application validates the certificate.

### 5. Where attacks map to layers

| Attack | Layer | What the attacker manipulates | Mitigation |
|---|---|---|---|
| ARP spoof / poison | L2 | ARP reply forging IP↔MAC | Dynamic ARP Inspection (DAI), DHCP snooping, static ARP for critical hosts |
| MAC flooding | L2 | CAM table exhaustion | Port security, 802.1X, sticky MAC |
| VLAN hopping | L2 | Double-tagged frames, DTP negotiation | Disable DTP, set trunk to non-negotiate, native VLAN ≠ user VLAN |
| IP spoofing | L3 | Source IP field | uRPF (RFC 3704), BCP38 ingress filtering, IPsec AH |
| Smurf / ping flood | L3 | ICMP to broadcast | Disable directed broadcast, ICMP rate-limit |
| Teardrop / fragmentation overlap | L3 | Malformed frag | Modern OS drops, firewalls reassemble |
| SYN flood | L4 | Half-open TCP state | SYN cookies (RFC 4987), conntrack tuning, stateless SYN proxy |
| UDP reflection | L4 | Spoofed source IP → amp | BCP38, response-rate-limit, drop unused UDP services |
| Slowloris | L7 | Half-open HTTP request | Reverse proxy with timeout, WAF rule |
| HTTP smuggling | L7 | Disagreement on chunked vs content-length | Reject ambiguous requests at WAF, RFC 9112 parsing |

### 6. Protocol numbers and well-known ports (CISSP exam favorites)

| IP proto # | Name | Common use |
|---|---|---|
| 1 | ICMP | ping, traceroute, PMTU |
| 6 | TCP | HTTP, HTTPS, SSH, SMTP, Kerberos |
| 17 | UDP | DNS, NTP, SNMP, VoIP, QUIC |
| 41 | IPv6-in-IPv4 | 6in4 tunnel |
| 47 | GRE | Generic Routing Encapsulation (RFC 2784) |
| 50 | ESP | IPsec Encapsulating Security Payload |
| 51 | AH | IPsec Authentication Header |
| 132 | SCTP | Telephony / signaling |

| Port | Proto | Service | Secure variant |
|---|---|---|---|
| 20/21 | TCP | FTP (control 21, data 20) | FTPS (990/989), SFTP (22) |
| 22 | TCP | SSH | SSH itself |
| 23 | TCP | Telnet | — (use SSH) |
| 25 / 587 / 465 | TCP | SMTP / submission / SMTPS | STARTTLS, implicit TLS |
| 53 | TCP/UDP | DNS | DoT (853), DoH (443), DNSSEC |
| 67/68 | UDP | DHCP | DHCPv4 — no built-in auth |
| 69 | UDP | TFTP | — (use SFTP) |
| 80 | TCP | HTTP | redirect to 443 |
| 110 | TCP | POP3 | POP3S (995) |
| 123 | UDP | NTP | NTS (RFC 8915) |
| 143 | TCP | IMAP | IMAPS (993) |
| 161/162 | UDP | SNMPv2c | SNMPv3 (auth + priv) |
| 389 / 636 | TCP | LDAP / LDAPS | LDAPS, StartTLS |
| 443 | TCP | HTTPS / QUIC / DoH | TLS 1.2+ |
| 445 | TCP | SMB | SMB signing, v3 (encrypted) |
| 500 / 4500 | UDP | IKEv2 / NAT-T | IPsec VPN |
| 514 / 6514 | TCP/UDP | syslog / syslog-TLS | — |
| 636 | TCP | LDAPS | TLS |
| 1812 / 1813 | UDP | RADIUS / RADIUS acct | — |
| 3389 | TCP | RDP | RDP + NLA, RDP over TLS |
| 5900 | TCP | VNC | tunnel over SSH |

### 7. IPv4 header (20 B, exam-essential)

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |    DSCP   |ECN|         Total Length          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |Flags|    Fragment Offset      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Time to Live |    Protocol   |         Header Checksum       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Source Address                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Destination Address                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (if IHL > 5)                       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

| Field | Bits | Purpose / security note |
|---|---|---|
| Version | 4 | 4 for IPv4, 6 for IPv6 |
| IHL | 4 | Header length in 32-bit words (5..15). IHL < 5 is illegal → drop |
| DSCP / ECN | 8 | QoS; ECN can be covert channel — block on untrusted ingress |
| Total Length | 16 | Length of IP packet incl. header; max 65535 |
| Identification | 16 | Reassembly key |
| Flags (3 bits) | 3 | Reserved (0), DF, MF |
| Fragment Offset | 13 | 8-byte units; overlapping frags = Teardrop |
| TTL | 8 | Decrement each hop; 0 → ICMP Time Exceeded. Limits loops + amplifies spoofed UDP |
| Protocol | 8 | Next-header (6=TCP, 17=UDP, 1=ICMP, 41=IPv6, 47=GRE, 50=ESP, 51=AH) |
| Header Checksum | 16 | Recomputed at every hop; weak by design (CRCs) |
| Source / Destination | 32 each | IPv4 address; spoofable without ingress filtering |
| Options | variable | Record Route, Source Route (loose/strict — **disable at edge**), Timestamp |

### 8. TCP header (20 B, exam-essential)

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (if Data Offset > 5)               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

| Flag | Name | Meaning |
|---|---|---|
| URG | Urgent | Urgent pointer valid |
| ACK | Acknowledgment | Ack number valid (always set after SYN) |
| PSH | Push | Deliver to app immediately |
| RST | Reset | Abort connection |
| SYN | Synchronize | Open connection (SYN flood target) |
| FIN | Finish | Graceful close |

**Three-way handshake** (security-critical):

```
Client               Server
   | --- SYN, seq=x ---> |
   | <-- SYN+ACK, seq=y, ack=x+1 --- |
   | --- ACK, seq=x+1, ack=y+1 ---> |
   |     <-- established -->         |
```

SYN flood: attacker never sends final ACK → server holds half-open state in SYN queue → exhaustion.

### 9. UDP header (8 B)

```
 0      7 8     15 16    31
+--------+--------+--------+
| Source | Dest   | Length |
| Port   | Port   | |Cksum |
+--------+--------+--------+
|           Data ...         |
+----------------------------+
```

No handshake, no seq, no ack. UDP is *unreliable by design* — used for DNS, VoIP, video, gaming, QUIC, IPsec NAT-T. Spoofed UDP is the engine of DNS amplification, NTP amplification, memcached amplification (peak ~51,000×).

### 10. Ethernet II / IEEE 802.3 frame (L2)

```
+---------+---------+--------+----------+---------+--------+
| Preamble| Dst MAC | Src MAC| EtherType| Payload |  FCS   |
| 7+1 B   | 6 B     | 6 B    | 2 B      | 46-1500 | 4 B    |
+---------+---------+--------+----------+---------+--------+
```

- Preamble + SFD: 8 B (1010...10 + 10101011) — clock sync
- MAC: 48-bit OUI + NIC; 802.1Q tag inserts 4 B (TPID 0x8100 + TCI) for VLAN
- EtherType ≥ 0x0600 = protocol (0x0800 IPv4, 0x86DD IPv6, 0x8863/0x8864 PPPoE)
- For 802.3 (not Ethernet II) the 2 B is Length + LLC/SNAP
- 802.1ad Q-in-Q: nested VLAN tags for provider bridging

### 11. Protocol↔layer↔OSI mapping (memory table for exam)

| Protocol | OSI layer | PDU at that layer | Notes |
|---|---|---|---|
| HTTP | 7 | Data | Request/response, stateless |
| TLS record | 6 | Data | Encapsulates 7 |
| TLS handshake | 5 | Data | Stateful session |
| TCP | 4 | Segment | Reliable, in-order, byte-stream |
| UDP | 4 | Datagram | Connectionless, best-effort |
| IP | 3 | Packet | Best-effort forwarding, fragmentation |
| IPsec AH | 3 | Packet (inserted) | Integrity, anti-replay, no confidentiality |
| IPsec ESP | 3 | Packet (inserted) | Confidentiality + integrity (auth+cipher) |
| ICMP | 3 | Packet | Control plane (ping, unreachable, redirect) |
| ARP | 2 | Frame payload | IP↔MAC resolution (broadcast) |
| 802.1Q | 2 | Frame tag | VLAN ID + PCP |
| 802.1X EAPOL | 2 | Frame payload | Port-based NAC |
| Wi-Fi 802.11 | 1–2 | Frame | CSMA/CA, MAC contention |

### 12. Service-oriented view: SAP, primitive, service

| Concept | Meaning |
|---|---|
| **Service Access Point (SAP)** | The address a layer above uses to hand data to the layer below. E.g., IP uses a *protocol number* as a SAP into IP; TCP uses a *port* as a SAP into the application |
| **Primitive** | The verb a layer offers: `Request`, `Indication`, `Response`, `Confirm` |
| **Confirmed vs unconfirmed service** | TCP is confirmed (ACK); UDP is unconfirmed |
| **Connection-oriented vs connectionless** | TCP is CO; UDP is CL; IP is CL; Ethernet is CL |
| **QoS guarantees** | IntServ (RSVP) vs DiffServ (DSCP) — best-effort IP can be carved into classes |

### 13. Why the models still matter (CISSP, 2024+)

- **Troubleshooting**: any anomaly is described by layer ("L3 issue", "L7 issue", "name resolution = L7/DNS").
- **Vendor architecture mapping**: a cloud load balancer sits in L4–L7; an NGFW adds L7 awareness on top of stateful L4.
- **Zero-Trust mapping**: every hop between PEPs is a *layer-crossing* with identity, device, and policy checks; design your PEPs to align with model boundaries.
- **Defense in depth**: compromise of L2 (rogue switch) should not transit to L7 (untrusted app) because identity+encryption are still enforced.

## CISO / Risk Manager View

The OSI/TCP-IP model is the **universal language of the incident report**. A board-ready incident summary is almost always written layer-by-layer: "an attacker exploited a misconfigured firewall rule (L4) to inject commands (L7) using a hijacked DNS record (L7/L3)." Without a common mental model, the SOC, the network team, and the auditors will literally not understand each other.

For the CISO, the value of the layered model is **risk stratification per layer**. Each layer carries a known class of risk (DoS at L3–L4, ransomware at L7, eavesdrop at L1–L2), and each has a known set of controls with measurable effectiveness. The board will fund defenses when the cost of the control is shown to be a fraction of the expected loss at that layer — and that math is impossible without the model.

Strategic points:
- **Converged vs separate stacks**: SASE/SSE collapses L3–L7 PEPs into the cloud; this reduces capex but **moves the trust boundary outside the enterprise WAN**. The CISSP exam tests that you can describe what *didn't* change: the layered model still applies — what changed is *where* the PEPs live.
- **Encryption-in-transit inventory**: a CISO deliverable. Show the % of L7 traffic that is TLS 1.2+, % of L2 that is WPA3, % of L3 that is IPsec. Push the percentages.
- **"Layer collapse" risk**: HTTP/3 (QUIC) encrypts L4 inside L7; observers who assume "TLS is L7" lose visibility. NDR vendors compensate with JA3/JA4 fingerprinting and ML.

## Related Connections

### Siblings (L2 in this domain)
- [[network-security-controls]] — devices that enforce policy at each layer
- [[network-architecture-segments]] — where we *place* PEPs in the layered topology
- [[wireless-and-mobile-security]] — L1–L2 specifics for radio
- [[secure-protocols-and-services]] — secure protocols operating at each layer

### L3 children
- [[firewall-types-and-rule-evaluation]] — where L3/L4 rules live
- [[vpn-types-and-ipsec-modes]] — L3 protection
- [[dns-security-and-dnssec]] — L7 protocol that breaks L3 trust
- [[5g-and-iot-network-security]] — layered model for new radios

### Cross-domain
- [[Domain 3 — Security Architecture and Engineering]] — TLS internals live in D3 crypto
- [[Domain 7 — Security Operations]] — SIEM normalization is layer-aware

## References

- ISO/IEC 7498-1 — OSI Basic Reference Model
- ITU-T X.200 — OSI model
- DARPA, *Internet Protocol Transition Plan* — TCP/IP model origin
- RFC 791 — IPv4
- RFC 8200 — IPv6
- RFC 9293 — TCP (modern successor to RFC 793)
- RFC 768 — UDP
- RFC 826 — ARP
- IEEE 802.3-2022 — Ethernet
- IEEE 802.1Q-2022 — VLAN
- Stallings, *Cryptography and Network Security*, ch. 8 (network security)
- Tanenbaum & Wetherall, *Computer Networks*, 6th ed.
- Kurose & Ross, *Computer Networking: A Top-Down Approach*, 8th ed.
