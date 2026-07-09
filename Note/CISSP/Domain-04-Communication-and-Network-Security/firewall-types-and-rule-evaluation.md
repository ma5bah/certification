---
title: Firewall Types & Rule Evaluation
layer: 3
domain: 4
tags: [cissp, domain-4, firewall, packet-filter, stateful, proxy, ngfw, acl, rule-order, first-match]
related_domains: [3, 7]
parent: network-security-controls
---

# Firewall Types & Rule Evaluation

## Feynman Explanation

A firewall is a guard at a door with a list of rules. Stateless firewalls are guards who only check the badge of the *current* person in line; stateful firewalls remember who just went in and only let *the same person* come back out. Proxy firewalls are guards who physically go fetch the package for you, then walk it through a different door. NGFWs are guards with a magnifying glass who can read the letter inside the package. The dangerous part is the **rule list itself** — put the wrong rule first, and the guard turns into a welcome mat. That is why this is its own deep dive: the *type* of guard matters, but the *order and quality of the rulebook* matters even more.

## Technical Details

### 1. The four firewall generations, restated for this level

| Gen | Type | L | State? | Visibility | Decision unit | Strengths | Weaknesses |
|---|---|---|---|---|---|---|---|
| 1 | **Packet filter (stateless)** | L3/L4 | No | IP, port, proto, flags | Per packet | Fast, simple, cheap | No context; spoofable; can't do "outbound SSH from servers" |
| 2 | **Stateful inspection** | L3/L4 + L4-state | Yes (per flow) | Same + TCP state | Per flow (5-tuple + state) | Stops half-open / uninitiated inbound | No L7 understanding; subject to state-exhaustion DoS |
| 3 | **Circuit-level proxy** | L4 + session | Yes (per conn) | TCP handshake only | Per TCP session | Hides internal host; session filtering | Doesn't see payload |
| 4 | **Application-level gateway (ALG)** | L7 | Yes | Full payload | Per protocol (HTTP, FTP, DNS) | Deep inspection; auth; can enforce method/URL | Per-app scaling; TLS MITM required for HTTPS |
| 4+ | **NGFW** | L2–L7 | Yes + identity | App-ID, user-ID, content-ID | Per session | Best of all | Cost; SSL inspection privacy concerns |
| 5 / Cloud | **UTM / FWaaS** | L4–L7 | Yes | All | Multiple | Bundled; SME-friendly | Performance at full feature; single-bundle risk |

### 2. The state table — what stateful actually tracks

For each flow, a stateful FW creates a tuple:

```
[ src_ip, src_port, dst_ip, dst_port, proto, state, timeout, [byte_count] ]
```

Plus derived context: NAT, ALG, MAC, interface, TTL.

| TCP state (per RFC 793) | Meaning |
|---|---|
| NEW | First packet seen, no SYN prior |
| ESTABLISHED | Saw SYN, SYN+ACK, ACK |
| RELATED | New conn tied to existing (e.g. FTP-data, ICMP err) |
| INVALID | Doesn't match any tracked flow |
| UNTRACKED | Bypassed (admin) |
| SNAT/DNAT | NAT applied |

**State table exhaustion**: millions of half-open flows (SYN flood) → table full → drops new legitimate flows. Mitigations:
- **SYN cookies** (RFC 4987): encode state in seq#; no per-flow memory until ACK
- **State table size tuning**
- **Rate-limit per source**
- **Stateless SYN proxy** (Cisco uRPF + TCP intercept)

### 3. Rule evaluation: first-match is everything

**CISSP-critical**: most ACL/rule engines use **first-match** semantics. The first rule whose selector matches the packet is the rule that fires. Everything below is ignored for that packet.

```
rule_id  action  src            dst            proto  dport   notes
--------------------------------------------------------------------
10       deny    10.0.0.0/8     any            ip     *       RFC1918 must not enter from outside
20       permit  any            host 1.1.1.1   tcp    53      public DNS
30       permit  any            host 2.2.2.2   tcp    443     web
40       deny    any            any            ip     *       default deny
9999     deny    any            any            ip     *       explicit catch-all + log
```

If a packet to 2.2.2.2:443 matches rule 10 first (because src 10.x is in 10.0.0.0/8 — not necessarily, but conceptually), it is denied. Order matters.

**Best practice: most-specific first**, **default-deny last**, with logging on the catch-all.

**Shadowing**: a rule below another that can never be reached because the first match always wins. Shadowed rules are a code-smell; tools like **Firemon**, **Algosec**, **Tufin** find them.

**Redundancy**: two rules that match the same set of traffic; one is unnecessary.

**Generalization**: a rule that is a superset of another.

**Anomaly** (Cuppens & Miège 2000 framework): shadow, redundancy, generalization, irrelevance, correlation.

### 4. Vendor rule examples

#### 4.1 Linux iptables (stateful)

```bash
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 5/s -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP
iptables -A INPUT -p tcp --dport 443 -m state --state NEW -j ACCEPT
iptables -A INPUT -j DROP
```

#### 4.2 nftables (modern, replaces iptables)

```nft
table inet filter {
  chain input {
    type filter hook input priority 0; policy drop;
    ct state established,related accept
    iif lo accept
    tcp dport 443 ct state new accept
    tcp dport 22 ct state new accept
    ip saddr 10.0.0.0/8 drop
  }
}
```

#### 4.3 Cisco IOS ACL

```
ip access-list extended EDGE-IN
 remark ## anti-spoof ##
 deny   ip 10.0.0.0 0.255.255.255 any log
 deny   ip 172.16.0.0 0.15.255.255 any log
 deny   ip 192.168.0.0 0.0.255.255 any log
 deny   ip 127.0.0.0 0.255.255.255 any log
 deny   ip 169.254.0.0 0.0.255.255 any log
 deny   ip 224.0.0.0 15.255.255.255 any log
 remark ## allow public services ##
 permit tcp any host 198.51.100.10 eq 443
 permit tcp any host 198.51.100.11 eq 80
 remark ## default deny ##
 deny   ip any any log
!
```

#### 4.4 AWS Security Group + NACL

Security Group (stateful, allow only, attached to ENI):
```json
{
  "GroupName": "web-sg",
  "IpPermissions": [
    { "IpProtocol": "tcp", "FromPort": 443, "ToPort": 443,
      "IpRanges": [{ "CidrIp": "0.0.0.0/0" }] }
  ]
}
```

NACL (stateless, allow+deny, attached to subnet):
```
Rule #  Type   Protocol  Port  Source        Allow/Deny
100     Allow  TCP        443   0.0.0.0/0     Allow
200     Allow  TCP        1024-65535  10.0.0.0/16  Allow  (return traffic — must allow ephemeral)
*       Deny   ALL        ALL    0.0.0.0/0     Deny
```

### 5. NAT (often colocated with firewall)

| Type | What | Example |
|---|---|---|
| **SNAT / Masquerade** | Many private → one public | Home router |
| **DNAT / Port Forward** | One public → one private | Web host: 203.0.113.5:443 → 10.0.0.10:443 |
| **PAT / NAPT** | SNAT with port multiplexing | Default home/SOHO |
| **Twice NAT** | Both src and dst rewritten | Overlapping private ranges on M&A |
| **Carrier-Grade NAT** | ISP-side large-scale NAT | IPv4 exhaustion workaround |

NAT is **not a security control** in the sense of authentication, but it does hide topology. State stateful-NAT has the same DoS concerns as stateful FW.

### 6. Application-Layer Gateways (ALG) and protocol-specific gotchas

| Protocol | ALG behavior | Risk |
|---|---|---|
| **FTP** | Tracks `PORT` / `PASV` to open data channel | Insecure protocol; replace with SFTP |
| **SIP** | Rewrites SDP IP/port for media | One of the most buggy ALGs |
| **TFTP** | Trivial, no auth | Block at edge |
| **H.323** | Complex multi-channel | Replace with WebRTC |
| **DNS** | Inspect query/response for tunneling (long TXT, high entropy) | DNSSEC adds size; new EDNS |

Modern NGFW drops the legacy ALG model in favor of L7 parsers (HTTP/2, QUIC) that don't need to "open a new pinhole" via control channel.

### 7. Deep packet inspection and TLS inspection

NGFW typically does **TLS termination + inspection**:

```
   Client ── TLS ──► NGFW (decrypts) ── TLS ──► Server
                       │
                       └─ inspect plaintext with IPS/AV/DLP
```

**Privacy / legal considerations**:
- MITM requires deploying a private CA to clients
- Privacy-sensitive categories (health, banking) often excluded
- Some jurisdictions require notice/consent
- PFS in TLS 1.3 is *still* honored by NGFW; they sit in the middle

**Failure modes**:
- Cert pinning on client → NGFW can't see content
- HSTS preload → can't downgrade
- Encrypted SNI / ECH → hides server name even from NGFW (limitation for categorization)

### 8. WAF-specific rule patterns (CISSP app perspective)

| Pattern | OWASP rule | Action |
|---|---|---|
| `UNION SELECT` from user input | SQLi | Block |
| `<script>` in user input | XSS | Block / encode |
| `../../../etc/passwd` in path | Path traversal | Block / normalize |
| `; ls -la` in query | CMDi | Block |
| 1000+ byte header | Buffer overflow attempt | Drop |
| Mismatched `Content-Length` and `Transfer-Encoding` | HTTP smuggling | Reject ambiguous |
| Bot UA + missing JS | Scraping | JS challenge |

### 9. Testing the ruleset

| Method | What | How |
|---|---|---|
| **Rule review** | Manual analysis of every rule | Quarterly by security team |
| **Static analysis** | Find shadow/redundancy | Algosec, Firemon, Tufin, fwadmin |
| **Penetration test** | Exploit, then attempt to bypass | Annual external pentest |
| **Red team** | Targeted TTP | Annual |
| **Fuzz** | Send malformed packets | Sulley, Boofuzz, Scapy |
| **Active probing** | nmap, hping3, scapy | Internal audit |
| **Audit log analysis** | Check what is *actually* being hit vs what's *allowed* | SIEM |
| **Compliance scan** | PCI/CIS benchmark | Compliance scanner |
| **Bypass testing** | Fragmentation, tunneling, source routing | Pentest |

### 10. The "what does a firewall actually see" table

| Header field | Where in packet | FW can inspect? | Caveat |
|---|---|---|---|
| MAC src/dst | L2 | Yes | Spoofable; not authenticated |
| 802.1Q VLAN | L2 | Yes | Stripped on routed hop |
| IP src/dst | L3 | Yes | Spoofable; not authenticated (without IPsec) |
| DSCP / ECN | L3 | Yes | Covert channel possible |
| IP Options | L3 | Yes | Often dropped at edge (Source Route deprecated) |
| L4 src/dst port | L4 | Yes | Inferred; encrypted in QUIC |
| TCP flags | L4 | Yes | Used in state tracking |
| TLS SNI | L7 (in TLS ClientHello) | Yes (unless ESNI/ECH) | Privacy move |
| TLS ALPN | L7 | Yes | "h2" or "h3" |
| HTTP method / path / query | L7 | Yes (post-decrypt) | None after decrypt |
| DNS query | L7 | Yes (in clear DNS) | Lost in DoH to 3rd party |

### 11. Common firewall pitfalls (exam traps)

| Pitfall | Symptom | Fix |
|---|---|---|
| "Any/Any/Allow" on internal FW | Flat network, lateral movement | Default-deny; allow-list by app flow |
| Logging disabled on catch-all | No visibility of denied traffic | Enable logging; forward to SIEM |
| Shadowed rule | Real intent not enforced | Regular review; static analysis |
| Outbound any | Malware C2 succeeds | Default-deny egress; allow-list of services |
| Stateful FW with no SYN-flood protection | DoS on state table | SYN cookies, stateless SYN proxy |
| ALG for FTP/SIP mis-tuned | Voice breaks, data leaks | Disable legacy ALG; switch apps |
| TLS inspection blind-spot (uncategorized domains) | SSL pinned traffic passes blind | Categorize, allow-list, log |
| Stateful + asymmetric routing | Stateful doesn't see return | Stickiness; route asymmetry fix |
| High-availability without state sync | Failover drops all flows | Stateful HA (active/active) |
| Rule changes not version-controlled | Drift, no rollback | Treat FW as code; GitOps |

### 12. Cloud firewall specifics

| Provider | Stateful | Stateless | Layer | Default-deny? |
|---|---|---|---|---|
| AWS SG | ✓ (ENI) | – | ENI | No (default = deny *inbound*, allow *outbound*) |
| AWS NACL | – | ✓ (subnet) | Subnet | No (default = allow) — **must** add deny all |
| Azure NSG | ✓ (NIC/subnet) | – | NIC/subnet | No (default = allow from same VNet, deny inbound from internet) |
| Azure ASG | Application Security Group (tag) | – | NIC | Logical grouping for NSG rules |
| GCP VPC FW | ✓ (hierarchical + network) | – | VM/network | Yes (default = deny; must add allow) |
| OCI SecList | ✓ (VNIC) | – | VNIC | No (default = deny) |
| Akamai / Cloudflare | ✓ (edge) | – | Global | Yes |

### 13. NGFW vs. UTM — when to pick which (and SASE)

| Scenario | Pick |
|---|---|
| Single small office, <100 users, no security team | UTM or cloud SSE |
| Enterprise campus, identity-aware, deep app control | NGFW (Palo Alto, Fortinet, Check Point) |
| Datacenter, east-west, server-to-server | NGFW + NDR + microseg |
| Multi-cloud, hybrid work | SASE / SSE (Zscaler, Netskope, Prisma) |
| OT/ICS | Purpose-built OT firewall (e.g. Dragos, Nozomi, Cisco IE) — passive + zoning |
| Carrier / ISP edge | BGP-aware router ACLs + scrubbing |

## CISO / Risk Manager View

**Board-level story**: "Our firewall ruleset is the codified version of our security policy. The number of rules, the age of the rules, and the rate of change are all measurable." Three KPIs that map directly to risk:

1. **Number of "any/any" rules**. Trend to zero. Each one is a question to answer in the next audit.
2. **Stale rule age** (days since last review). Target: ≤90 d. Stale rules = unknown exposure.
3. **% of rules with logged hits in 90 days**. If a rule never fires, it can probably be removed. The 20/80 of firewall management.

Strategic considerations for the next 24 months:

- **From rule-by-rule to policy-as-code**. Terraform / Ansible / vendor APIs → GitOps → reviewable, roll-back-able, auditable.
- **From perimeter to identity-bound policy**. NGFW + identity provider = user-ID-based rules. The same rule set survives an employee changing laptops.
- **Cloud parity**. Native cloud SGs are *not* the same as on-prem NGFW. Don't pretend they are; use CSPM to find drift.
- **Logging to SIEM**. Firewall logs that don't go to SIEM are evidence that is never seen. Forward everything; sample in storage.

Board KPIs:

| KPI | Target |
|---|---|
| % of egress traffic on default-deny | 100% (perimeter) |
| Rule review cycle (full audit) | ≤ 90 d |
| Mean rule age | < 180 d |
| "Any/any" rules | 0 |
| Catch-all with logging | Yes, on all perimeter FWs |
| Firewall log → SIEM | 100% |

## Related Connections

### Parent (L2)
- [[network-security-controls]]

### Siblings (L3 in this domain)
- [[vpn-types-and-ipsec-modes]] — IPsec is the "any/any" risk to the firewall
- [[dns-security-and-dnssec]] — DNS tunneling can be hidden from FW
- [[5g-and-iot-network-security]] — IoT/5G endpoints challenge the rule base

### Cross-domain
- [[Domain 3 — Security Architecture and Engineering]] — TLS inspection = crypto in the path
- [[Domain 7 — Security Operations]] — FW logs land in SIEM; playbooks for FW changes

## References

- NIST SP 800-41r1 — Guidelines on Firewalls and Firewall Policy
- NIST SP 800-10 (legacy but still cited)
- Cisco *Firepower Management Center Configuration Guide*
- Palo Alto *PanOS Administrator's Guide*
- iptables / nft man pages
- AWS, *Security Groups vs NACLs* (AWS whitepaper)
- Cheswick, Bellovin, Rubin, *Firewalls and Internet Security* (classic)
- Zwicky, Cooper, Chapman, *Building Internet Firewalls* (O'Reilly, 2nd ed.)
- Ingham & Forrest, *A History and Survey of Network Firewalls* (2002)
- Cuppens, Miège, *Administration Model for OR-BAC* (2000) — formal anomaly framework
- Common Criteria — Firewall PP
- CIS Benchmark — pfSense, Cisco ASA, Palo Alto, FortiGate
- *Penetration Testing* — Firewalls chapter, Georgia Weidman
- RFC 2663 — NAT
- RFC 3022 — Traditional NAT
- RFC 4787 — NAT Behavioral Requirements
- RFC 7858 — DoT
- RFC 8484 — DoH
- RFC 6092 — Recommended Simple Security Capabilities in Customer Premises Equipment
