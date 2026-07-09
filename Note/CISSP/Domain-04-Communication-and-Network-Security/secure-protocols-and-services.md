---
title: Secure Protocols & Services
layer: 2
domain: 4
tags: [cissp, domain-4, tls, ipsec, ssh, s-mime, pgp, kerberos, dnssec, ntp, ldaps, radius, tacacs]
related_domains: [3, 5, 7]
parent: domain-04-communication-and-network-security
---

# Secure Protocols & Services

## Feynman Explanation

A secure protocol is a **conversation with rules** that makes it impossible (or at least very expensive) for an eavesdropper to read, alter, or impersonate one of the speakers. Plain HTTP is a conversation in a glass phone booth — anyone can hear and write on the glass. HTTPS is the same conversation inside a soundproof, locked booth where you show a passport before entering. IPsec builds those soundproof booths into the network layer so anything inside gets protection. SSH lets you remotely log in as if you walked up and sat at the keyboard. Kerberos hands out short-lived wristbands so you don't have to show your passport for every door. DNSSEC is the equivalent of a tamper-evident seal on every DNS answer, so nobody can rewrite the phone book while it's in transit.

## Technical Details

### 1. TLS — the dominant transport crypto

| Version | RFC | Year | Status (2024) | Key change |
|---|---|---|---|---|
| SSL 2.0 | (Netscape) | 1995 | Forbidden | Broken |
| SSL 3.0 | (Netscape) | 1996 | Forbidden (POODLE 2014) | Broken |
| TLS 1.0 | RFC 2246 | 1999 | Deprecated (IETF, 2020) | MD5/SHA-1/RC4 broken |
| TLS 1.1 | RFC 4346 | 2006 | Deprecated | Same |
| TLS 1.2 | RFC 5246 | 2008 | Standard; minimum allowed | SHA-2, AEAD mandatory; cipher suite flexibility |
| TLS 1.3 | RFC 8446 | 2018 | Preferred | 1-RTT handshake, 0-RTT (with replay caveats), removes static RSA, AEAD-only |

#### 1.1 TLS 1.2 handshake (2-RTT)

```
Client                                  Server
  | ---- ClientHello --------------------> |  (version, random, cipher suites, extensions)
  |                                       |  ServerHello (chosen cipher), Certificate,
  | <----- ServerHello, Cert, ServerHelloDone -- |  (server cert chain, key exchange)
  | ---- ClientKeyExchange, ChangeCipherSpec, Finished ----> |  (premaster secret)
  | <--- ChangeCipherSpec, Finished ---- |  derive keys
  | <---- application data encrypted ---->|
```

#### 1.2 TLS 1.3 handshake (1-RTT)

```
Client                                  Server
  | ---- ClientHello (key_share, PSK) --> |  (encrypted extensions inside encrypted handshake)
  | <--- ServerHello + Cert + CertVerify + Finished -- |
  | ---- Finished + (early data, optional) ----> |
```

Improvements: 0-RTT possible with PSK, no static RSA, all cipher suites AEAD, encrypted SNI (ESNI) / Encrypted Client Hello (ECH) in development.

#### 1.3 Key TLS concepts

| Term | Meaning |
|---|---|
| **Cipher suite (1.2)** | `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384` (kx, auth, cipher, MAC) |
| **Cipher suite (1.3)** | `TLS_AES_256_GCM_SHA384` (cipher, hash); key exchange implicit (always ECDHE/DHE) |
| **Perfect Forward Secrecy (PFS)** | Ephemeral keys per session (ECDHE/DHE); compromise of long-term key does not retroactively decrypt |
| **OCSP Stapling** | Server periodically fetches OCSP response and "staples" it to the handshake — clients don't need to contact CA |
| **Certificate Pinning** | Client only accepts a specific cert/subject — defense against rogue CA |
| **HPKP** (RFC 7469) | Pin in browser; **deprecated** and removed from browsers |
| **CT (Certificate Transparency)** | RFC 6962 — public log of all certs; detects mis-issuance |
| **OCSP Must-Staple** | Hard-fail if no stapled response |
| **ALPN** (RFC 7301) | Negotiate app protocol (h2) inside TLS |
| **SNI / ESNI / ECH** | Server Name Indication; encrypted variant for privacy |
| **Mutual TLS (mTLS)** | Client also presents cert; for service-to-service and zero-trust |

### 2. IPsec (suite of protocols, RFC 4301–4303)

| Protocol | Full name | Function |
|---|---|---|
| **AH** | Authentication Header (RFC 4302) | Integrity + auth, **no** confidentiality; covers IP payload + selected header fields |
| **ESP** | Encapsulating Security Payload (RFC 4303) | Confidentiality + integrity + auth of payload; **does not** cover outer IP header |
| **IKEv2** | Internet Key Exchange v2 (RFC 7296) | Mutual auth + key exchange; replaces IKEv1 |
| **SA** | Security Association | One-way logical channel: SPI, IP, protocol, mode, algorithms, lifetime |
| **SPD** | Security Policy Database | Defines: BYPASS / DISCARD / PROTECT for each selector |
| **SAD** | Security Association Database | Active SAs |
| **PAD** | Peer Authentication Database | Auth method per peer (PSK, cert, EAP) |

#### 2.1 IPsec modes (detailed in [[vpn-types-and-ipsec-modes]])

| Mode | What it protects | Header added | Use |
|---|---|---|---|
| **Transport** | IP payload only | AH/ESP header (no new IP header) | Host-to-host, end-to-end |
| **Tunnel** | Entire original IP packet | New outer IP header + AH/ESP | Gateway-to-gateway, remote access VPN |

#### 2.2 IKEv2 exchange (4 messages)

```
Initiator (I)            Responder (R)
  | -- IKE_SA_INIT (1) --> |  (proposals, nonces, key shares)
  | <-- IKE_SA_INIT (2) --- |
  | -- IKE_AUTH (3) ----> |  (identity, auth, cert, traffic selectors)
  | <-- IKE_AUTH (4) ----- |
  | <---- CHILD_SA / CREATE_CHILD_SA ---> |  (negotiates ESP/AH SA)
```

- **PSK**, **cert** (RSA/ECDSA), or **EAP** (RFC 4739) for auth
- **NAT-Traversal** (RFC 3948) wraps ESP in UDP/4500
- **MOBIKE** (RFC 4555) lets a roaming client change IP without tearing down
- **DPD** (Dead Peer Detection) detects dead peers; IKEv2 has it built-in

#### 2.3 IPsec encryption & integrity (common algorithms)

| Purpose | Algorithm | Notes |
|---|---|---|
| Confidentiality | AES-CBC (128/256) | Standard |
| Confidentiality | AES-GCM (128/256) | AEAD (combined with integrity) |
| Confidentiality | ChaCha20-Poly1305 | Software-only, fast on ARM |
| Confidentiality | 3DES / DES | **Deprecated** |
| Integrity | HMAC-SHA-256/384/512 | Standard |
| Integrity + Conf. | AES-GCM / ChaCha20-Poly1305 | AEAD |
| Auth (cert) | RSA, ECDSA P-256/384, Ed25519 | ECDSA/EdDSA preferred for size/speed |
| Key exchange | (EC)DH group 14/19/20/21 | DH group 14 = 2048-bit MODP; 19/20 = ECP |
| PFS | (EC)DHE in IKE_AUTH rekey | Rekeys with new DH every X minutes |

### 3. SSH (RFC 4251–4254, 4335, 4344, 5656, 6668, 8268, 8332, 8709, 8758)

| Component | RFC | Purpose |
|---|---|---|
| Transport | 4253 | Server auth, key exchange, encryption, integrity |
| Authentication | 4252 | Client auth (pubkey, password, host-based, keyboard-interactive) |
| Connection | 4254 | Multiplexing channels (shell, file, forwarded port) |
| Key exchange | 4344, 5656, 8268 | Algorithm negotiation; SHA-2, Curve25519, Ed25519 |
| File transfer (SFTP) | 4716 (not "FTP over SSH") | Separate protocol layered on SSH |
| Subsystem | 4254 | Run server-side subsystem (sftp-server, git-receive-pack) |

**Default TCP port: 22.** SSH replaces Telnet (23), rlogin (513), rsh (514).

#### 3.1 SSH key exchange (modern: Curve25519)

```
Client                                  Server
  | ---- KEXINIT (algorithms) ----> |
  | <--- KEXINIT ----------------- |
  | ---- KEXDH_INIT (public e_c) ---> |
  | <--- KEXDH_REPLY (public e_s, host sig) -- |
  | <---- new keys derived, encrypted traffic ----- |
```

#### 3.2 SSH hardening

| Setting | Recommendation |
|---|---|
| Protocol 1 | **Disable** (legacy, broken) |
| PermitRootLogin | no (or prohibit-password) |
| PasswordAuthentication | no (use keys) |
| KbdInteractiveAuthentication | as needed (e.g. 2FA) |
| AllowUsers / AllowGroups | restrict to known users |
| MaxAuthTries | 3 |
| LoginGraceTime | 30 |
| ClientAliveInterval / Count | 300 / 2 |
| AllowAgentForwarding | no for most users |
| X11Forwarding | no unless required |
| PermitEmptyPasswords | no |
| KexAlgorithms | curve25519-sha256, diffie-hellman-group16-sha512, etc. |
| Ciphers | chacha20-poly1305@openssh.com, aes256-gcm@openssh.com |
| MACs | hmac-sha2-512-etm@openssh.com, etc. |
| Banner | /etc/issue.net with legal warning |

### 4. Kerberos (RFC 4120, 4121, 4556, 6806, 8009, 8553, 8636)

Microsoft's default auth in Active Directory; widely used.

| Term | Meaning |
|---|---|
| **KDC** | Key Distribution Center (AS + TGS) |
| **AS** | Authentication Service (issues TGTs) |
| **TGS** | Ticket Granting Service (issues service tickets) |
| **TGT** | Ticket Granting Ticket (lifetime ~10 h) |
| **Service ticket** | Per-service, per-session, encrypted with service's secret key |
| **Principal** | Identity (user or service) |
| **Realm** | Administrative domain (e.g. EXAMPLE.COM) |
| **SPN** | Service Principal Name (e.g. HTTP/web.example.com) |
| **Pre-auth** | Required in modern Kerberos; proves user knows key without sending it |
| **PAC** | Privilege Attribute Certificate (Windows: group SIDs in ticket) |
| **KRB-AS-REQ / KRB-AS-REP** | Initial auth (TGT) |
| **KRB-TGS-REQ / KRB-TGS-REP** | Service ticket request |
| **KRB-AP-REQ / KRB-AP-REP** | Mutual auth to service |

#### 4.1 Kerberos flow

```
Client (C)        AS        TGS       Service (S)
  | -- AS-REQ (C, preauth) --> |               |
  | <-- AS-REP (TGT + session) |               |
  | --- TGS-REQ (TGT, S) ----> |               |
  | <-- TGS-REP (S-TKT + S-sess) |             |
  | --- AP-REQ (S-TKT) ---------------->       |
  | <-- AP-REP (mutual auth) ----------------- |
```

#### 4.2 Attacks on Kerberos

| Attack | Description | Defense |
|---|---|---|
| **Pass-the-Ticket (PtT)** | Replay stolen TGT/service ticket | TGT lifetime short; Protected Users group (no NTLM fallback) |
| **Golden Ticket** | Forge TGT with krbtgt hash | Reset krbtgt twice; rotate; monitor |
| **Silver Ticket** | Forge service ticket | Same; service account password rotation |
| **Kerberoasting** | Offline crack service ticket encrypted with service account hash | Strong service account passwords; AES keys |
| **AS-REP Roasting** | Offline crack if preauth not required | Always require preauth |
| **Overpass-the-Hash** | Convert NTLM hash → Kerberos TGT | Disable NTLM, Protected Users |
| **Unconstrained delegation** | Service can impersonate any user to any service | Disable, or constrained delegation with auth mechanism |

#### 4.3 PKINIT (RFC 4556)

Allows *certificate-based* initial Kerberos auth — used in smartcard, federated, and zero-trust scenarios.

### 5. DNSSEC (RFC 4033, 4034, 4035)

Adds authentication to DNS responses. See [[dns-security-and-dnssec]] for deep dive.

| Record | Purpose |
|---|---|
| **DNSKEY** | Zone's public key (KSK and ZSK) |
| **DS** | Delegation Signer — hash of child KSK, published in parent |
| **RRSIG** | Signature over a record set |
| **NSEC / NSEC3 / NSEC3-PARAM** | Authenticated denial of existence (NSEC3 hashed) |
| **CDS / CDNSKEY** | Child → parent signaling for automated DS maintenance |
| **TLSA** (DANE) | Pin a cert to a domain (RFC 6698) |

### 6. S/MIME (RFC 8551, 5750–5751, 5754)

- **What**: X.509 cert-based email signing and encryption
- **Encoding**: `Content-Type: multipart/signed; protocol="application/pkcs7-signature"`
- **Crypto**: RSA, ECDSA, AES, SHA-2
- **Versus PGP**: S/MIME is centralized (X.509 / CA), PGP is decentralized (web of trust)

| Property | S/MIME | PGP / OpenPGP (RFC 9580, 4880) |
|---|---|---|
| Trust model | Hierarchical (X.509 CA) | Web of trust (PGP) |
| Key distribution | Directory, SCEP, EST, manual | Key servers (pgp.mit.edu), manual, WKD |
| Use case | Enterprise, regulated | Personal, journalist, open-source |
| Standard | IETF, S/MIME | IETF, OpenPGP |
| Inline signing | No (always detached) | Yes (inline PGP) |
| Algorithm agility | Periodic profile updates (e.g. RFC 8551) | Algorithm preference in key itself |

### 7. RADIUS (RFC 2865, 2866, 3576, 5176, 6613, 8559, 8740)

- **Port**: 1812 (auth), 1813 (accounting) — legacy 1645/1646 deprecated
- **Transport**: UDP (TLS variant RFC 6614 on 2083)
- **Encoding**: AVP (Attribute-Value Pairs) including RADIUS attributes (User-Name, User-Password, NAS-IP-Address, etc.)
- **Auth methods**: PAP, CHAP, MS-CHAPv2, EAP
- **RADIUS/TLS** (RadSec): encrypt the RADIUS hop; also gives mutual auth

```
  Supplicant ---(EAPOL)---> Authenticator (NAS)
                                   |
                                   | (RADIUS UDP or RadSec TLS)
                                   v
                              RADIUS server
                                   |
                                   v
                          (Identity store: AD, LDAP, IdP)
```

**TACACS+** (RFC 8907) — Cisco proprietary, separates AAA (auth/author/acc); uses TCP/49, encrypts entire payload, more flexible authorization (per-command).

| Property | RADIUS | TACACS+ |
|---|---|---|
| Transport | UDP (TLS variant) | TCP/49 |
| Encryption | Only password (User-Password is hashed; rest cleartext) | Full payload encrypted |
| AAA | Combined | Separated (can authorize per cmd) |
| Multi-protocol | No (mostly IP) | Yes (modular) |
| Vendor | Multi-vendor | Cisco |
| Use | Network access, Wi-Fi | Device admin (router/switch) |

### 8. NTP / NTS (RFC 5905, 8915)

- **NTP**: UDP/123. **No auth in v3**; v3 symmetric keys; v4 MAC
- **NTS** (Network Time Security, RFC 8915): TLS 1.3 + AEAD for NTP — keys negotiated via TLS, attached to NTP packets
- **Security**: NTP amplification is a DoS vector; mitigate with NTP BCP (reduce monlist, etc.)

### 9. LDAPS / StartTLS

- **LDAP** (389, cleartext) → **LDAPS** (636, implicit TLS)
- **StartTLS** (RFC 4511, 4513): upgrade cleartext LDAP to TLS via StartTLS extended op
- Always require channel binding (GSSAPI/Kerberos) for SASL auth

### 10. Other CISSP-favorite secure services

| Service | Plain | Secure variant | Notes |
|---|---|---|---|
| Web | HTTP (80) | HTTPS (443), HSTS | TLS 1.2+ |
| Mail send | SMTP (25) | SMTPS (465), STARTTLS (587) | SPF, DKIM, DMARC |
| Mail receive | POP3 (110) | POP3S (995) | TLS |
| Mail receive | IMAP (143) | IMAPS (993) | TLS |
| File | FTP (21/20) | FTPS (990/989), SFTP (22) | SFTP is SSH, not TLS |
| Name | DNS (53) | DoT (853), DoH (443), DNSSEC | See L3 |
| Time | NTP (123) | NTS (4460) | AEAD |
| Directory | LDAP (389) | LDAPS (636), StartTLS | TLS |
| Remote shell | Telnet (23) | SSH (22) | Always SSH |
| Remote shell | rlogin/rsh (513/514) | SSH | Always SSH |
| File sharing | SMB (445) clear | SMB 3.1.1 (AES-GCM) | Signing + encryption |
| Remote desktop | RDP (3389) | RDP + TLS + NLA | NLA = Network Level Authentication |
| Routing | BGP, OSPF, RIPv2 | BGPsec (RFC 8205), RPKI (RFC 6480) | Origin + path validation |
| SNMP | SNMPv2c (161/162) | SNMPv3 (auth + priv) | USM, VACM |
| DHCP | DHCPv4 | DHCPv6 with SEND | SEND (RFC 3971) — cryptographically generated addresses |
| Syslog | syslog (514 UDP) | syslog-TLS (6514) | RFC 5425 |

### 11. Protocol cross-cut: mutual authentication matrix

| Protocol | Server auth to client | Client auth to server | Cipher agility | Replay defense |
|---|---|---|---|---|
| TLS 1.2 | Cert (X.509) | Optional (mTLS) | Per-suite | Random + nonce + epoch |
| TLS 1.3 | Cert (required) | Optional (mTLS) | AEAD only | Always PSK/ephemeral |
| SSH 2 | Host key | Pubkey or password | Algorithm negotiation | Session ID + seq |
| IPsec/IKEv2 | Cert/PSK/EAP | Cert/PSK/EAP | SPD/SAD | Anti-replay window |
| Kerberos v5 | KDC + service | Ticket | Enctype (AES) | Replay cache + time bound |
| S/MIME | Cert | Cert | Per-cert / per-msg | Signed receipts / timestamp |
| RADIUS | Shared secret (per-hop) | EAP method | AVP per RFC | Replay cache in NAS |
| TACACS+ | Shared secret | TACACS+ acct | Per-packet | Packet seq |
| 5G-AKA | Home network | USIM | NEA/NIA | SQN + COUNT |

### 12. Defensive protocols (the "control plane")

| Protocol | Used for | Security |
|---|---|---|
| RPKI | BGP origin validation | Sign ROA, RPKI cache |
| BGPsec | BGP path validation | Sign AS path |
| RADIUS / TACACS+ | AAA | Above |
| syslog-TLS | Log integrity | TLS + signed (e.g. RFC 5848) |
| OSPFv3 with IPsec | Routing auth | IPsec ESP (RFC 4552) |
| NTP / NTS | Time | NTS = AEAD |
| DHCPv6 + SEND | Address assignment | CGA, RSA-signed |

## CISO / Risk Manager View

**Board framing**: "the cryptographic state of our traffic is the single most measurable indicator of how much an attacker can passively learn about us." The classic KPI is the % of internal network traffic that is **opportunistic TLS** (best-effort) versus **mandatory TLS with mTLS and PFS** (always). In a properly secured enterprise, that number is 95%+.

Three strategic questions:

1. **"What is our posture for TLS 1.3 + PFS across all customer-facing and partner-facing APIs?"** TLS 1.2 is still allowed but TLS 1.3 is preferred; anything below 1.2 is a finding.
2. **"Are we still relying on NTLM, RADIUS shared secrets in cleartext, or IPsec with 3DES?"** Each of these is a known weakness. They are deprecation projects — fund them.
3. **"How do we know if any of our vendors drop TLS back to 1.0/1.1?"** Continuous scanning with a tool like **Qualys SSL Labs**, **testssl.sh**, or **SSLyze**; alert on downgrade.

KPIs:

| KPI | Target |
|---|---|
| % TLS 1.2 or higher | 100% |
| % HTTPS with HSTS preload | 100% of public domains |
| % Certificates managed by automation (ACME, SCEP) | 95%+ |
| mTLS coverage on service-to-service traffic | 80%+ in zero-trust programs |
| Mean cert lifetime | ≤ 90 d (drives automation) |
| IPsec tunnels using PFS (DHE/ECDHE) | 100% |
| SSH hosts allowing password auth | 0% |
| Kerberos accounts with preauth disabled | 0 |

## Related Connections

### Siblings (L2 in this domain)
- [[osi-tcpip-models-and-encapsulation]] — protocol layers
- [[network-security-controls]] — controls that *use* these protocols
- [[network-architecture-segments]] — placement of TLS terminators
- [[wireless-and-mobile-security]] — EAP, RADIUS, 802.1X

### L3 children
- [[firewall-types-and-rule-evaluation]] — TLS inspection policy
- [[vpn-types-and-ipsec-modes]] — IPsec VPN flavors
- [[dns-security-and-dnssec]] — DNSSEC, DoT, DoH
- [[5g-and-iot-network-security]] — 5G-AKA

### Cross-domain
- [[Domain 3 — Security Architecture and Engineering]] — TLS internals, PKI
- [[Domain 5 — Identity and Access Management]] — Kerberos, RADIUS, PKINIT
- [[Domain 7 — Security Operations]] — log integrity, syslog-TLS

## References

- RFC 5246 — TLS 1.2
- RFC 8446 — TLS 1.3
- RFC 6066 — TLS Extensions
- RFC 6960 — OCSP
- RFC 6962 — Certificate Transparency
- RFC 4301–4303, 4306, 7296 — IPsec / IKEv2
- RFC 4251–4254, 4344, 5656, 8268, 8709 — SSH
- RFC 4120, 4121, 4556, 6806, 8009, 8553, 8636 — Kerberos
- RFC 4033, 4034, 4035 — DNSSEC
- RFC 8551, 5750–5751, 5754 — S/MIME
- RFC 9580, 4880, 6637, 9580 — OpenPGP
- RFC 2865, 2866, 3576, 5176, 6614, 8559, 8740 — RADIUS
- RFC 8907 — TACACS+
- RFC 5905 — NTP
- RFC 8915 — NTS
- RFC 4511, 4513, 5803, 7596, 7597, 7671, 7814, 7859, 7929 — LDAP
- RFC 5424, 5425 — Syslog / Syslog-TLS
- NIST SP 800-52r2 — Guidelines for TLS Implementations
- NIST SP 800-77r1 — Guide to IPsec VPNs
- NIST SP 800-57 — Key Management
- NIST SP 800-63B — Authentication Guidelines
- ENISA, *Algorithms, Key Sizes and Parameters Report*
- OWASP *Transport Layer Protection Cheat Sheet*
- OWASP *Cryptographic Storage Cheat Sheet*
- Microsoft, *Kerberos explained*
- IETF *Using Kerberos 5 over TLS* (RFC 8446 + extensions in progress)
