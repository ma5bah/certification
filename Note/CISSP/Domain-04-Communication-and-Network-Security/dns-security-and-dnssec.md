---
title: DNS Security & DNSSEC
layer: 3
domain: 4
tags: [cissp, domain-4, dns, dnssec, cache-poisoning, amplification, doh, dot, rpz, dane, tsig, edns]
related_domains: [3, 7]
parent: network-security-controls
---

# DNS Security & DNSSEC

## Feynman Explanation

DNS is the **phone book of the internet** — when you type `bank.com`, your computer asks "what's the IP for bank.com?" and the answer decides where your browser actually goes. The problem is that the original DNS was designed in 1983 and answered that question **without signing the answer**. So an attacker who could respond faster than the real DNS server could say "bank.com is at *my* IP" and your bank login form would land in the attacker's lap. That is **cache poisoning**. **DNSSEC** is the equivalent of a notarized phone book — every answer is digitally signed, and you can verify the chain of trust all the way to the root. **DoH/DoT** are private phone calls — they wrap DNS in encryption so the snooping café Wi-Fi can't even see who you are calling.

## Technical Details

### 1. DNS protocol — quick refresher

| Component | Port | Notes |
|---|---|---|
| DNS (clear) | 53/UDP (queries), 53/TCP (large/zone xfer) | Default; unencrypted; unsigned |
| DoT (DNS over TLS) | 853/TCP | RFC 7858; TLS 1.2+; SNI often uninformative (host = IP) |
| DoH (DNS over HTTPS) | 443/TCP | RFC 8484; looks like regular HTTPS; great for bypassing captive portals but bad for IT visibility |
| DoQ (DNS over QUIC) | 853/UDP | RFC 9250; faster handshake |
| DNSCrypt | 443/TCP | Older alternative; not standard |

#### 1.1 DNS message structure (RFC 1035)

```
+---------------------+
|       Header        | 12 bytes: ID, QR, Opcode, AA, TC, RD, RA, Z, AD, CD, RCODE,
|                     |          QDCOUNT, ANCOUNT, NSCOUNT, ARCOUNT
+---------------------+
|      Question       |  QNAME (labels), QTYPE, QCLASS
+---------------------+
|       Answer        |  RRset (NAME, TYPE, CLASS, TTL, RDLENGTH, RDATA)
+---------------------+
|      Authority      |  NS records
+---------------------+
|      Additional     |  A, AAAA, OPT (EDNS), TSIG, RRSIG, NSEC, ...
+---------------------+
```

Header flags of security interest:

| Flag | Meaning |
|---|---|
| QR | 0 = query, 1 = response |
| AA | Authoritative answer |
| TC | Truncated (use TCP) |
| RD | Recursion desired |
| RA | Recursion available |
| AD | Authentic Data (DNSSEC) — "data has been verified" |
| CD | Checking Disabled (DNSSEC) — client says "don't validate for me" |
| RCODE | 0=NOERROR, 1=FORMERR, 2=SERVFAIL, 3=NXDOMAIN, 4=NOTIMP, 5=REFUSED |

### 2. DNS threats (CISSP taxonomy)

| Threat | Description | Impact |
|---|---|---|
| **Cache poisoning (Kaminsky 2008)** | Attacker injects forged RR into resolver cache | Victim goes to attacker's IP for victim domain |
| **DNS spoofing** | Attacker replies to a query with bogus answer (on-path) | Same as poisoning; transient |
| **DNS amplification** | Small query → huge response (up to ~50x); reflect off open resolvers | DDoS target |
| **DNS reflection** | Spoofed source IP in query → response to victim | DDoS |
| **DNS tunneling** | Encode data in subdomain labels (e.g. `aaa.bbb.ccc.example.com`) | C2 channel, data exfil |
| **DGA (Domain Generation Algorithm)** | Malware calls thousands of random domains daily | C2 resilience, C2 detection |
| **NXDOMAIN flood** | Bombard victim authoritative server with non-existent names | CPU exhaustion on auth server |
| **Phantom domain attack** | Slow responses → client keeps retrying | Client-side DoS |
| **Subdomain takeover** | Dangling CNAME in authoritative zone → attacker registers the alias | Account takeover, cookie theft |
| **Typosquatting / combosquatting** | `g00gle.com`, `bankcom-secure.com` | Phishing |
| **DNS rebinding** | Attacker DNS resolves *to internal IP* after first answer | SSRF via browser, internal recon |
| **DNSSEC zone walking** | NSEC / NSEC3 records enumerate all names | Information disclosure |
| **DoH to malicious resolver** | Malware uses DoH to talk to attacker DNS, bypassing corporate DNS | Blind spot for defenders |

### 3. Cache poisoning in detail (Kaminsky's bug)

#### 3.1 Original (pre-2008) poisoning

```
Client                          Resolver                  Attacker
  | -- query id=0x1234 bank.com ---> |                          |
  |                                  | -- bank.com? -> root --> |
  |                                  | <-- NS for .com ------- |
  |                                  | -- bank.com? -> auth --> |
  |   (attacker tries to race the    | <-- A=1.2.3.4 ---------- |
  |    legitimate response with a    |                          |
  |    forged packet id=0x1234,      |                          |
  |    A=evil IP) ---------------->  |                          |
  |                                  |   (if guess matches and  |
  |                                  |    no source port check, |
  |                                  |    resolver caches evil) |
```

The flaw: 16-bit transaction ID + predictable source port = small enough to brute force.

#### 3.2 Kaminsky's amplification (2008)

Instead of poisoning an existing record, the attacker:
1. Queries for `random.bank.com`
2. Resolver goes to authoritative for `bank.com` to find it
3. While that's happening, attacker floods resolver with thousands of fake responses, each containing NS records + glue A for `attacker.com`
4. Resolver caches `attacker.com` NS at the resolver level
5. Resolver caches evil glue A; any future query for `attacker.com` goes to attacker

Fix: **source port randomisation** (now standard) makes the search space ~2^32 (16-bit ID × ~16-bit port); bailiwick checking (RFC 5452) prevents caching records outside the original question.

#### 3.3 Defenses against cache poisoning

| Defense | Effect |
|---|---|
| Source port randomization | 2^16 extra bits to guess |
| Transaction ID randomization | 2^16 (already there) |
| 0x20 encoding (mixed case) | Extra bit entropy (mostly deprecated) |
| Bailiwick check (RFC 5452) | Reject answers with out-of-zone data |
| DNSSEC | Cryptographically impossible |
| Query rate-limit per server | Limits racing window |
| Resolver hardening (BIND `dnssec-validation auto`) | Validates DNSSEC |
| Response Policy Zones (RPZ) | Override known-bad names |

### 4. DNS amplification / reflection DDoS

```
Attacker --(spoofed src = victim)--> Open Resolver
                                       |
                                       └── 60B query → 4000B response (66x amp)
                                                  → 60B
                                                  → victim
```

Amplification factors (approximate, 2024):

| Protocol | Query size | Response size | Factor |
|---|---|---|---|
| DNS ANY | 60 B | 4000+ B | 70x |
| NTP monlist | 60 B | 482 B × 600 | 5500x |
| Memcached | 15 B | 750 kB | 51000x |
| SSDP | 70 B | 3000 B | 30x |
| CLDAP | 60 B | 2000 B | 56x |

**Mitigations**:
- BCP38 (RFC 2827) ingress filtering at ISP
- Response Rate Limiting (RRL) at authoritative server
- Disable open recursion on resolvers
- DDoS scrubbing for the victim
- Anycast + capacity

### 5. DNSSEC — the cryptographic chain of trust

#### 5.1 New RR types

| Type | RFC | Purpose |
|---|---|---|
| **DNSKEY** | 4034 | Zone's public key (KSK and ZSK) |
| **RRSIG** | 4034 | Signature over a record set |
| **DS** | 4034 | Delegation Signer — hash of child KSK in parent zone |
| **NSEC** | 4034 | Authenticated denial of existence (linked list) |
| **NSEC3** | 5155 | Hashed denial of existence (prevents zone walking) |
| **NSEC3PARAM** | 5155 | NSEC3 parameters (algo, iterations, salt) |
| **CDNSKEY** | 7344 | Child → parent signaling for DS update |
| **CDS** | 7344 | Child DS record (signaled to parent) |
| **CAA** | 8659 | Authorized CA for this domain (only listed CAs may issue) |

#### 5.2 The chain of trust (root → TLD → zone)

```
Root zone (.)
  └── Signs .com with root ZSK; root KSK is in browser/OS trust store
       │
       └── .com zone
            └── DS record = hash(.com KSK)
                 └── bank.com zone
                      └── DS record = hash(bank.com KSK)
                           └── bank.com KSK → ZSK
                                └── ZSK signs bank.com A/AAAA/MX/...
```

Validation algorithm (RFC 4035):

1. Query for `bank.com A` returns RRset + RRSIG
2. Resolver has `bank.com DNSKEY` (ZSK) + parent DS (in `.com`)
3. Verify RRset against RRSIG using ZSK
4. Verify DS == hash(KSK) for `bank.com`
5. Recursively: parent chain must validate
6. Chain ends at the **trust anchor** — root KSK

#### 5.3 Key signing key (KSK) vs Zone signing key (ZSK)

| Key | Purpose | Lifetime | Stored where |
|---|---|---|---|
| **KSK** | Signs DNSKEY RRset | Long (years) | Hardware HSM, offline |
| **ZSK** | Signs all other RRsets | Short (weeks/months) | Online signing server |
| **CSK** (Combined) | Single key for both | Varies | Some registries |

KSK rollover (ZSK) is routine; KSK rollover (root) is a *major* event (ICANN coordinates).

#### 5.4 NSEC vs NSEC3 (denial of existence)

| Mechanism | How it works | Issue |
|---|---|---|
| **NSEC** | Lists the *next* name in canonical order | Walks the zone: query for each name to enumerate |
| **NSEC3** | Hashed name; opt-out (NSEC3-PARAM) | Harder to walk; opt-out makes unsigned delegations invisible |
| **NSEC5 / Black Lies** | Research | Avoids walking without opt-out abuse |

NSEC3 parameters: `1 SHA-1 5 0123456789abcdef` = algo, iterations, salt.

#### 5.5 DNSSEC validation outcome

| Status | Meaning | Client action |
|---|---|---|
| **Secure** | All sigs verified | Use data |
| **Insecure** | Chain not signed (no DS at parent) | Use data (best effort) |
| **Bogus** | Signature invalid | SERVFAIL to client (or NXDOMAIN with aggressive NSEC) |
| **Indeterminate** | Unable to validate (timeout) | SERVFAIL |
| **Indeterminate (no signer)** | Authoritative says "unsigned subtree" | OK |

AD flag: set by validating resolver to indicate "I verified this". CD flag: client says "send it anyway".

#### 5.6 DANE / TLSA (RFC 6698, 7671, 7672, 7673)

- Use **DNSSEC** to authenticate a service's certificate or CA
- RR type **TLSA** at `_port._proto.hostname`
- Selector: `0` = full cert, `1` = SPKI
- Matching-type: `0` = exact, `1` = SHA-256, `2` = SHA-512
- Usage: 0=PKIX-TA, 1=PKIX-EE, 2=DANE-TA, 3=DANE-EE

**DANE-EE (usage 3)**: the cert *must* match the TLSA — pins a cert via DNS, no CA required (e.g. SMTP, PGP keys).

**DANE for SMTP** (RFC 7672) — SMTP servers can use DANE-EE to assert their cert is the only valid one. Pairs with **MTA-STS** (RFC 8461) for TLS downgrade protection.

#### 5.7 SMTP security stack (modern)

| Layer | Mechanism | Effect |
|---|---|---|
| **SPF** (RFC 7208) | TXT record listing allowed senders | Stops spoofed envelope-from |
| **DKIM** (RFC 6376) | Signed message hash in header | Stops tampering; ties identity to domain |
| **DMARC** (RFC 7489) | Policy: reject/quarantine; reporting | Closes gap between SPF and DKIM |
| **MTA-STS** (RFC 8461) | TLS-only mode for SMTP | Strips opportunistic TLS downgrade |
| **DANE / SMTP** (RFC 7672) | Pin cert via DNSSEC | Removes CA dependency for SMTP |
| **ARC** (RFC 8617) | Forwarded-message chain of trust | Maintains DKIM through intermediaries |

### 6. DoT and DoH (encrypted DNS)

| Property | DoT (RFC 7858) | DoH (RFC 8484) | DoQ (RFC 9250) |
|---|---|---|---|
| Port | 853/TCP | 443/TCP (HTTPS) | 853/UDP |
| Visible to network operator? | Yes (port 853) | No (looks like HTTPS) | No (UDP 443 indistinguishable from QUIC) |
| SNI | Not used (IP-based) | SNI = `cloudflare-dns.com` (can be blocked) | Same |
| Inspection | Easy (block 853) | Hard (proxy intercept) | Very hard |
| Latency | +1-RTT TLS | +1-RTT TLS | 0-RTT possible |
| Application support | Stub-resolver: systemd-resolved, dnscrypt-proxy, Unbound | Browsers, getdns, Android (Private DNS), iOS | Stub resolvers (AdGuard) |
| Privacy from ISP | Yes | Yes | Yes |
| Privacy from DoH provider | Provider can log | Provider can log | Provider can log |
| Compliance (corp) | Easy to log/disable | **Hard** to log/disable | Hard |
| Bootstrap IP | Required (e.g. 1.1.1.1, 9.9.9.9) | Can use discovery (RFC 8612) | Required |

**DoH URI templates** (`https://dns.example.com/dns-query{?dns}`).

**ODoH** (Oblivious DoH) — adds a proxy hop so the resolver doesn't see the client IP. RFC 9230.

**Discovery of DoH resolvers** — `dns.resolver.arpa` (RFC 9462).

### 7. DNS in the enterprise (CISSP perspective)

#### 7.1 Resolver architecture

```
                  ┌────────────────┐
User device ───►  │  Stub resolver │ (Windows DNS client, systemd-resolved, glibc)
                  └────────┬───────┘
                           │
                  ┌────────▼───────┐
                  │ Recursive      │   ← caching, DNSSEC validation
                  │ resolver       │     (Unbound, BIND, Windows DNS, Infoblox)
                  │ (corp)         │
                  └────────┬───────┘
                           │
              ┌────────────┼─────────────┐
              ▼            ▼             ▼
         Root (.)     TLDs (.com)   Authoritative (bank.com)
```

#### 7.2 Split-horizon DNS

| User | Sees | Use |
|---|---|---|
| Internal (corp net) | Internal IPs (e.g. `intranet 10.0.0.5`) | Access internal services |
| External (internet) | Public IPs or filtered | Limited info disclosure |
| Guest Wi-Fi | External only | Prevents guest reconnaissance of internal |

#### 7.3 DNS firewall / RPZ (Response Policy Zones)

| Action | Example |
|---|---|
| Block known C2 domains | `nxdomain` |
| Sinkhole malicious | Reply with 127.0.0.1 or block page |
| Redirect ad/tracker | `0.0.0.0` |
| Whitelist | Allow-list critical services |

RPZ providers: Spamhaus, SURBL, PhishTank, Palo Alto DNS Security, Cisco Umbrella, Quad9.

### 8. Common DNS misconfigurations (CWE / CISSP)

| Misconfig | CWE | Risk |
|---|---|---|
| Open recursion on authoritative server | CWE-1385 | Reflection/amplification |
| Zone transfer (AXFR) to anyone | CWE-200 | Recon / zone walking |
| Glue records stale | – | Takeover |
| Wildcard record | – | Subdomain takeover |
| Dangling CNAME | – | Subdomain takeover |
| DNSSEC validation off | – | Cache poisoning |
| Lame delegation | – | Service outage |
| TTL too long | – | Slow poisoning recovery |
| TTL too short | – | Authoritative server overload |
| No CAA record | – | Mis-issuance of certs |
| DoH without logging | – | Insider exfil |

### 9. Hardening checklist (CISSP-grade)

| Item | Standard |
|---|---|
| Disable open recursion on authoritative servers | RFC 5358 / BIND `allow-recursion` |
| Restrict zone transfer | `allow-transfer { trusted; }` |
| Enable Response Rate Limiting (RRL) | BIND `rate-limit` |
| Enable DNSSEC validation on resolver | `dnssec-validation auto` (BIND/Unbound) |
| Sign your own zones | KSK + ZSK; automate with OpenDNSSEC, BIND `auto-sign` |
| Set CAA records | Restrict to your CA(s) |
| Sign outbound mail (DKIM), publish SPF, set DMARC policy | `p=reject` eventually |
| Use DoH or DoT on user devices | With corp-aware resolver |
| Log all queries (or sample) | Feed SIEM; NXDOMAIN surge = C2 |
| Monitor for subdomain takeover | Detect dangling records |
| Set sane TTLs | 300–3600 for most records; lower for short-lived |
| Use RPZ / threat intel | Block known-bad at resolver |

### 10. Real-world incidents (illustrative)

| Year | Incident | Mechanism | Impact |
|---|---|---|---|
| 2008 | **Kaminsky bug** | Cache poisoning, low entropy | Universal vulnerability in DNS |
| 2010 | Operation Ghost Click / DNSChanger | Malware changed host resolver | ~4M machines, $14M FBI op |
| 2012 | **DNS amplification attacks** | Open resolvers | 60 Gbps attacks on banks |
| 2016 | **Dyn DDoS** | Mirai botnet → Dyn DNS | Took down Twitter, GitHub, etc. |
| 2017 | **Sea Turtle / DNSpionage** | Registrar / DNSSEC misconfig | Government targets |
| 2018 | **DNSpionage** | Middle East gov targets | Cert issuance for proxy domains |
| 2019 | **Sea Turtle** continued | Registrar hijack | .gov, .int targets |
| 2021 | **AvosLocker** ransomware | DNS over HTTPS for C2 | Bypass corp DNS |
| 2023 | **Phishing via Cloudflare Workers** | DoH + short-lived cert | Credential theft |
| 2024 | **Edge devices (SOHO router hijack)** | DNS hijack | Persistent redirection |

### 11. Algorithm/agility for DNSSEC

| Algorithm | RFC | Status (2024) |
|---|---|---|
| RSA/SHA-1 | 4034 | **Deprecated** |
| RSA/SHA-256 | 5702 | Acceptable |
| RSA/SHA-512 | 6605 | Acceptable |
| ECDSA P-256/SHA-256 | 6605 | **Recommended** |
| ECDSA P-384/SHA-384 | 6605 | Recommended |
| Ed25519 | 8080 | **Recommended** for new deployments |
| Ed448 | 8080 | Recommended |
| **ML-DSA (Dilithium)** | 9371 (2023) | Future (PQC migration) |

## CISO / Risk Manager View

**Board framing**: DNS is *both* the simplest and the most overlooked high-impact service in your network. Every application depends on it; almost no application *validates* it. Three board narratives:

1. **"Cache poisoning is solved by DNSSEC, and we're at X%."** A clean KPI: percent of authoritative zones signed + percent of recursive resolvers validating. Many enterprises have signed their zones (cheap), but if the *customer* doesn't validate, the protection doesn't reach them.
2. **"DNS DDoS is a capacity problem, not just a security one."** Be on anycast. Know your upstream's DDoS runbook. The 2016 Dyn attack is the canary in the mine.
3. **"Encrypted DNS is a double-edged sword."** DoH/DoT improves user privacy, but also blinds the corporate SOC to C2 and DGA traffic. Decide policy: *enforce* corporate resolver (with logging), or accept the visibility gap.
4. **"Email still works because of DNS."** DMARC + DKIM + SPF + DANE all sit on DNS. The 2024 baseline for a Fortune 500 is `p=reject` with reporting, and DANE for SMTP for high-value partners.

KPIs:

| KPI | Target |
|---|---|
| Authoritative zones signed (DNSSEC) | 100% of public zones |
| Recursive resolvers validating | 100% |
| % of corp devices using DoT/DoH to corp resolver | ≥ 90% (with logging) |
| DMARC `p=reject` at primary domain | 100% |
| CAA record for public-facing CAs | Yes |
| DANE / SMTP for top partners | Enabled for high-value domains |
| DNS query log → SIEM | 100% |
| Time to detect + block malicious domain | ≤ 1 hour (RPZ refresh) |

## Related Connections

### Parent (L2)
- [[network-security-controls]] — DNS firewall, RPZ

### Siblings (L3 in this domain)
- [[firewall-types-and-rule-evaluation]] — DNS queries traverse FW; FQDN-based rules depend on DNS
- [[vpn-types-and-ipsec-modes]] — split-tunnel DNS leaks; DNS inside tunnel
- [[5g-and-iot-network-security]] — 5G DNS, IoT resolution of OTA servers

### Cross-domain
- [[Domain 3 — Security Architecture and Engineering]] — DNSSEC crypto, PKI, CAA
- [[Domain 7 — Security Operations]] — DNS telemetry in SOC, log analysis, DGA detection

## References

- RFC 1034, 1035 — DNS (original)
- RFC 4033, 4034, 4035 — DNSSEC
- RFC 5155 — NSEC3
- RFC 6605 — ECDSA for DNSSEC
- RFC 7344 — CDS / CDNSKEY
- RFC 8080 — EdDSA for DNSSEC
- RFC 7858 — DoT
- RFC 8484 — DoH
- RFC 9250 — DoQ
- RFC 9230 — ODoH
- RFC 8612 — DHCP options for DoH discovery
- RFC 9462 — Discovery of Designated Resolvers
- RFC 6698, 7671, 7672, 7673 — DANE / TLSA / SMTP
- RFC 7208 — SPF
- RFC 6376 — DKIM
- RFC 7489 — DMARC
- RFC 8461 — MTA-STS
- RFC 8617 — ARC
- RFC 5452 — DNS forgery resistance
- RFC 6891 — EDNS(0)
- RFC 8945 — DNS Privacy Considerations
- RFC 8244 — DNS Privacy requirements
- RFC 8764 — DNS Cookies (DoC)
- RFC 8078 — Managing DS records from the Child
- RFC 8806 — Running a Root Server Local Copy
- RFC 9499 — DNS Terminology
- RFC 5625 — DNS Proxy Implementation Guidelines
- NIST SP 800-81r2 — Secure Domain Name System (DNS) Deployment Guide
- NIST SP 800-175B — Cryptographic Standards (DNSSEC)
- ICANN DNSSEC Practice Statements
- ENISA, *Domain Name System Security*
- IETF DNSOP working group drafts (RRL, DoH, encrypted resolver discovery)
- Kaminsky, D., *It's the End of the Cache as We Know It* (Black Hat 2008)
- Aitchison, *Pro DNS and BIND 10* (Apress)
- Cricket Liu & Paul Albitz, *DNS and BIND* (O'Reilly)
- IETF RFC 8945 — DNS Privacy
- UChicago ANL, *DNS Privacy Project* (DoH / DoT comparisons)
- Spamhaus, *RPZ documentation*
- *Suricata / Zeek DNS logs* — SOC detection content
- APNIC *DNSSEC validation rate by country* (live measurement)
- Akamai *DNS Threat Landscape Report* (annual)
- Vercara *Global DDoS Attack Report*
