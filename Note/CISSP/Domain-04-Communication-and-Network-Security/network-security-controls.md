---
title: Network Security Controls
layer: 2
domain: 4
tags: [cissp, domain-4, firewall, ids, ips, ndr, waf, nac, utm, ngfw, dlp, ddos]
related_domains: [3, 5, 7]
parent: domain-04-communication-and-network-security
---

# Network Security Controls

## Feynman Explanation

Network security controls are the *security guards and cameras* of the building that is your network. A firewall is a guard at the door who checks IDs; an IDS is a security camera that records everything but does not stop a thief in the act; an IPS is a guard who *intervenes* the moment he sees something suspicious; a WAF is a specialist guard who only watches the front desk of one specific store inside the building; a NAC is the turnstile that won't open until you prove you are an employee in good standing; an NDR is a forensic team watching every corridor for unusual behavior. Each guard has different rules, blind spots, and salary costs — the architect's job is to deploy the right guard in the right place.

## Technical Details

### 1. Control taxonomy (the CISSP mental map)

| Axis | Options |
|---|---|
| **Where** | Inline (in-path) vs passive (tap/SPAN) vs host-based |
| **When** | Preventive (block), detective (alert), corrective (respond), recovery |
| **What** | Network, application, data, identity, endpoint |
| **How** | Signature, anomaly/behavior, policy/rule, ML/AI |
| **Layer** | L2 (MAC, 802.1X), L3/L4 (IP, port), L7 (HTTP, DNS, SMTP) |
| **State** | Stateless (per-packet) vs stateful (per-flow) vs application-aware |

### 2. Firewall evolution (the most-asked family)

| Generation | Also called | Layer | State? | Exam signal |
|---|---|---|---|---|
| 1st | Packet filter | L3/L4 | No | ACLs on routers; cheap; spoofable |
| 2nd | Stateful inspection / circuit-level proxy | L3/L4 + session | Yes | Tracks TCP state; drops INVALID |
| 3rd | Application-layer gateway (proxy) | L7 | Yes (per-app) | One proxy per protocol (HTTP, FTP) |
| 4th | NGFW (Next-Generation Firewall) | L2–L7 | Yes + DPI | App-ID, user-ID (identity), IPS, SSL inspection in one box |
| 5th | UTM / Web Security Gateway | All-in-one | Yes | SME-friendly; performance trade-off |
| (Cloud) | Cloud-native FWaaS | L4–L7 | Yes | AWS NACL/SG, Azure NSG, GCP firewall |

> See [[firewall-types-and-rule-evaluation]] for rule-order semantics, NAT, and deep examples.

#### 2.1 Packet filter (1st gen)

- Looks at: src/dst IP, src/dst port, protocol, TCP flags
- Stateless: each packet evaluated independently
- Cannot enforce "no outbound SSH from server X to internet"
- Used at router ACLs; **first** and **last** line of defense (BGP edge, server-to-server)

Example Cisco IOS ACL:
```
ip access-list extended INBOUND
 deny   ip 10.0.0.0 0.255.255.255 any log     ! anti-spoof: no spoofed internal
 permit tcp any host 198.51.100.10 eq 443
 deny   ip any any log
```

#### 2.2 Stateful inspection (2nd gen)

- Builds a **state table**: `(src_ip, src_port, dst_ip, dst_port, proto, state)`
- Returns from inside to outside are allowed *only* if they match a tracked outbound flow
- Performs TCP state tracking (SYN → ESTABLISHED → FIN)
- ICMP errors (e.g. "port unreachable") tracked to the flow that triggered them
- Cannot inspect HTTP method, URL, payload

#### 2.3 Application-layer gateway / proxy (3rd gen)

- Terminates the client connection, opens a new one to the server
- Inspects payload: HTTP method+URL, FTP command, DNS query
- Can enforce: "no GET /admin from user in group X"
- Pros: full content inspection, auth, caching
- Cons: per-app scaling; TLS terminates here → re-encrypt or chain certs

#### 2.4 NGFW (4th gen)

- Stateful + DPI + identity + IPS in one chassis
- **App-ID**: identifies app regardless of port (e.g. Skype over 443)
- **User-ID**: maps IP → user via AD/LDAP integration
- **Content-ID**: AV, DLP, file-type, URL categorization
- TLS/SSL inspection: must MITM to inspect encrypted traffic → must be policy-justified (privacy/legal)

#### 2.5 UTM (Unified Threat Management)

- Bundle: FW + IPS + AV + DLP + URL filter + spam + VPN in one appliance
- Target: SME with no security team
- Risk: single point of failure + single bottleneck; performance collapses when all features on
- Trend: replaced by cloud SSE / SASE in mid-market (2024+)

#### 2.6 Cloud-native FWaaS

| Provider | L4 SG | L7 proxy | WAF | Notes |
|---|---|---|---|---|
| AWS | Security Group (stateful) + NACL (stateless) | ALB | AWS WAF | SG = allow-rules only (no deny), stateful; NACL = allow/deny, stateless |
| Azure | NSG | App Gateway | Azure WAF | NSG = stateful; can apply to subnet |
| GCP | VPC firewall rules | HTTPS LB | Cloud Armor | Rules have priorities; tag-based targets |

### 3. Intrusion Detection & Prevention (IDS/IPS)

| Class | Mode | Detection method | False positive | Latency impact |
|---|---|---|---|---|
| NIDS (Network) | Passive tap | Signature, anomaly | Medium | None (out of band) |
| NIPS (Network) | Inline | Signature, anomaly | Medium | +µs per packet |
| HIDS (Host) | On endpoint | Syscall, log, file integrity | Lower | CPU on host |
| HIPS (Host) | On endpoint | Same + block | Lower | CPU on host |
| WAF | Inline (HTTP) | Signature + ML + behavior | Lower (HTTP-specific) | +ms per request |
| NBA / NDR | Tap or SPAN | ML, behavioral baselining | Low | None |

#### 3.1 Signature vs anomaly

| Method | Strengths | Weaknesses |
|---|---|---|
| **Signature** | High precision on known attacks; easy audit | Zero-day miss; sig set grows |
| **Anomaly (statistical)** | Catches novel attacks | High false positives; needs clean baseline |
| **Stateful protocol analysis** | Understands protocol spec | Vendor-tied; misses application logic bugs |
| **ML / behavioral** | Catches low-and-slow | Adversarial training; explainability |
| **Hybrid (signature + ML)** | Best of both | Complex; tuning expertise needed |

#### 3.2 Snort rule anatomy (exam/blue-team familiar)

```
alert tcp $HOME_NET any -> $EXTERNAL_NET 80 (msg:"ET MALWARE Cobalt Strike Beacon"; \
  flow:established,to_server; http_uri; content:"/submit.php"; content:"id="; nocase; \
  classtype:trojan-activity; sid:2024001; rev:2;)
```

Fields: action, proto, src, dst, **options** (msg, flow, content, classtype, sid).

### 4. Network Detection & Response (NDR)

- **What it is**: a sensor appliance or VM that ingests SPAN/tap traffic, applies ML models, surfaces "patient zero" or "lateral movement chain" in near-real-time.
- **Differences from IDS**: more behavioral/ML, less signature; surfaces east-west visibility that perimeter NGFW misses.
- **Vendors (2024)**: ExtraHop (Reveal(x)), Darktrace, Vectra AI, Cisco (Secure Network Analytics / Stealthwatch), Corelight.
- **Trend**: convergence with SIEM/XDR. Many NDR vendors ship SOAR playbooks.
- **Caveat**: NDR is **detective**, not preventive; pair with segmentation or NGFW to *block*.

### 5. Web Application Firewall (WAF)

| Mode | How it decides |
|---|---|
| **Positive (allow-list)** | Default-deny, only documented inputs allowed; **preferred** |
| **Negative (block-list)** | Default-allow, block known bad; OWASP Top 10 signatures |
| **Hybrid** | Per-endpoint mode (login URL = allow-list, comment = block-list) |
| **Behavioral** | ML on legit traffic baseline |
| **Bot management** | JS challenge, CAPTCHA, fingerprint |

**OWASP Top 10 (2021) — what a WAF actually blocks**:

| # | Risk | WAF action |
|---|---|---|
| A01 | Broken Access Control | URL/role enforcement; IDOR block |
| A02 | Cryptographic Failures | Force TLS, HSTS, secure cookies |
| A03 | Injection (SQLi, XSS, CMDi) | Signatures + parser |
| A04 | Insecure Design | Hard to block alone; design review |
| A05 | Security Misconfiguration | Disable methods, hide banners |
| A07 | Identification & Auth Failures | Brute-force lockout, MFA enforcement |

Deployment modes: **reverse proxy** (default), **transparent bridge**, **sidecar / side-band (host WAF in container)**, **inline container WAF (e.g. ModSecurity with Coraza)**.

### 6. Network Access Control (NAC)

| Model | What it is | Examples |
|---|---|---|
| **Pre-admission** | Block/allow *before* granting L2 access | 802.1X EAP-TLS, EAP-PEAP |
| **Post-admission** | Allow, then continuously assess | Persistent agent (Cisco ISE, Forescout) |
| **Agent-based** | Software on endpoint reports posture | Cisco ISE, Aruba ClearPass |
| **Agentless** | DHCP fingerprinting, SNMP scan, passive | ForeScout, Auconet |
| **Captive portal** | Web auth for guests | Coffee-shop, BYOD |

802.1X roles:
- **Supplicant** = endpoint
- **Authenticator** = switch / WLC (transparent)
- **Authentication server** = RADIUS (RFC 2865) / TACACS+ (RFC 8907) → AD/LDAP/IdP

EAP types (exam-essential):

| EAP method | TLS? | Cert on client? | Auth strength | Notes |
|---|---|---|---|---|
| EAP-MD5 | No | No | Weak (no server auth) | Avoid |
| EAP-LEAP | No | No | Weak (deprecated) | Cisco proprietary |
| EAP-PEAP | Tunneled TLS | No (server cert) | Strong | Most common enterprise Wi-Fi |
| EAP-TLS | Mutual TLS | Yes | Strongest | Gold standard; needs PKI |
| EAP-TTLS | Tunneled TLS | No (server cert) | Strong | Like PEAP but more flexible |
| EAP-FAST | Tunneled TLS via PAC | No (PAC) | OK | Cisco proprietary |
| EAP-SIM / EAP-AKA | Tunneled TLS via SIM | No (SIM) | Mobile carriers | 4G/5G/Wi-Fi offload |

### 7. Unified Threat Management (UTM) vs NGFW vs SSE

| Property | UTM | NGFW | SSE/SASE |
|---|---|---|---|
| Form factor | Appliance | Appliance/virtual | Cloud (vendor-hosted) |
| Best for | SME, branch | Enterprise campus/DC | Hybrid work, multi-cloud |
| Performance | Mediocre at full features | High | Effectively infinite (cloud scale) |
| Visibility into east-west | Yes (on-net) | Yes (on-net) | Limited (off-net only via ZTNA) |
| Identity integration | AD | AD + IdP | IdP-native (OIDC/SAML) |

### 8. Email & DNS as security controls (often forgotten in D4)

| Control | What it does |
|---|---|
| Secure Email Gateway (SEG) | Spam, malware, DLP, BEC, DMARC/SPF/DKIM enforcement |
| DNS firewall | RPZ, threat-intel feed, sinkhole C2 |
| DNS-based DGA detection | NXDOMAIN surges → likely C2 |
| Encrypted DNS (DoH/DoT) | See [[dns-security-and-dnssec]] |

### 9. DDoS protection

| Layer | Attack | Mitigation |
|---|---|---|
| L3 | ICMP flood, IP fragments | uRPF, rate-limit, drop frag |
| L4 | SYN flood, UDP flood | SYN cookies, conntrack tuning, scrubbing |
| L7 | HTTP GET flood, Slowloris | CDN, WAF, JS challenge, rate-limit per IP/session |

Architectures:
- **Always-on**: vendor in path (Cloudflare, Akamai, AWS Shield Advanced)
- **On-demand**: BGP route announcement or DNS redirect during attack
- **Hybrid**: always-on L3/L4 + on-demand L7
- **Anycast + scrubbing centers** spread the load

### 10. Selecting controls — CISO decision matrix

| Question | Choice |
|---|---|
| Branch office with 30 users? | UTM + cloud SSE for web |
| Datacenter with 5,000 servers, east-west? | NGFW + NDR + microseg |
| Public-facing e-commerce? | WAF + CDN + DDoS + NGFW |
| OT / ICS environment? | Passive IDS + unidirectional gateway + zone segregation |
| Multi-cloud (AWS+Azure+GCP)? | Cloud-native SG/NSG + CSPM + transit gateway + NDR |

### 11. Logging and forwarding (cross to D7)

| Control | Log volume/day (typical) | Forwarded to |
|---|---|---|
| Perimeter NGFW | 1–10 GB | SIEM, SOAR |
| Internal NGFW | 5–50 GB | SIEM, NDR |
| NDR sensor | 10–100 GB raw → 100 MB enriched | SIEM |
| WAF | 0.5–5 GB | SIEM, AppSec dashboard |
| NAC | 0.1–1 GB | SIEM, IAM |

### 12. Common control failures (CISSP traps)

| Failure | Cause | Fix |
|---|---|---|
| "Firewall in stealth mode" | Nmap no response → attacker assumes hole exists | Don't rely on stealth; use allow-list |
| SSL inspection blind spot | Cert errors disabled, app whitelisted "to keep working" | Restrict to sanctioned categories; re-evaluate quarterly |
| East-west no logging | Internal firewalls log to /dev/null | Forward all; sample WAF when full |
| WAF in detect-only | Vendor default | Move to block once tuned |
| NAC as one-time check | Posture drifts after login | Continuous assessment |
| IDS without tuning | Alert fatigue → ignored | Tune to environment; measure FP rate |

## CISO / Risk Manager View

Board language: **"every dollar on network controls is buying reduction in blast radius, not prevention of initial access."** Initial access is statistically phishing or credential abuse (per Verizon DBIR), both of which are *identity* problems. What network controls buy us is the *contained detection and bounded loss* once the attacker is already inside.

Strategic themes for the next 24 months:

1. **Consolidation**. The average enterprise has 30+ security vendors (Gartner). NGFW + SSE + NDR + SIEM is converging into a single-vendor XDR or platform play; expect licensing simplification.
2. **East-west visibility is the new perimeter**. With cloud, IoT, and zero-trust, attackers land inside. NDR and microsegmentation are the new "perimeter."
3. **Encryption-everywhere is also an inspection problem**. TLS 1.3 + E2EE means NGFW/WAF are blind unless they MITM. The legal/privacy case must be made and signed off.
4. **OT/ICS catch-up**. Many brownfield environments still run flat L2 networks. NERC CIP (US grid) and IEC 62443 drive the funding case.
5. **SASE / SSE board question**: "Do we keep the data center for egress, or do we trust the cloud?" This is now a *financial* question (CapEx vs OpEx) with security implications (visibility, jurisdiction, latency).

KPIs to track:

| KPI | Target | Why |
|---|---|---|
| % encrypted traffic visible to NGFW (post-inspection) | >90% | If you're blind, you're not controlling |
| Mean time to detect (MTTD) east-west | <24 h | Lateral movement speed benchmark |
| Firewall rule review cycle | ≤90 d | Stale rules = vulnerabilities |
| NAC coverage (managed endpoints) | 100% | Every endpoint must be posture-checked |
| DDoS drill (table-top) | Yearly | Validate scrubbing runbook |

## Related Connections

### Siblings (L2 in this domain)
- [[osi-tcpip-models-and-encapsulation]] — which layer each control operates at
- [[network-architecture-segments]] — where to *place* these controls
- [[wireless-and-mobile-security]] — 802.1X is both a control and a wireless safeguard
- [[secure-protocols-and-services]] — secure protocols that *use* these controls

### L3 children
- [[firewall-types-and-rule-evaluation]] — rule semantics and types
- [[vpn-types-and-ipsec-modes]] — control for remote access
- [[dns-security-and-dnssec]] — DNS firewall and DNSSEC as a control
- [[5g-and-iot-network-security]] — IoT NAC, 5G slicing controls

### Cross-domain
- [[Domain 3 — Security Architecture and Engineering]] — cryptographic primitives behind TLS inspection
- [[Domain 5 — Identity and Access Management]] — NAC = identity decision
- [[Domain 7 — Security Operations]] — every control's log lands here

## References

- NIST SP 800-41r1, *Guidelines on Firewalls and Firewall Policy*
- NIST SP 800-94r1, *Guide to Intrusion Detection and Prevention Systems*
- NIST SP 800-125B, *Secure Virtual Network Configuration for Virtual Machine (VM) Protection*
- NIST SP 800-44v2, *Guidelines for Securing the Public Web Server*
- Cisco, *Next-Generation Firewalls for Dummies*
- Palo Alto *Admin Guide* (App-ID, User-ID, Content-ID)
- Snort Users Manual, 3.x
- OWASP *Web Application Firewall* documentation; OWASP Top 10 2021
- Verizon *Data Breach Investigations Report* (DBIR), 2023/2024 editions
- Gartner, *Market Guide for Network Detection and Response*, 2023
- 3GPP TS 33.501 — 5G security
- IEC 62443 — Industrial communication network security
- T. Socol, *Practical Packet Analysis* (Wireshark)
