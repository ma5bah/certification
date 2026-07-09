---
title: TLS 1.3 Deep Dive — Handshake, Cipher Suites, Forward Secrecy, 0-RTT
layer: 3
domain: 3
tags: [cissp, tls, tls-1-3, rfc-8446, handshake, cipher-suite, forward-secrecy, 0-rtt, aead, hkdf, ecdhe, x25519, kyber, ml-kem, hybrid-kem]
related_domains: [4, 5, 7]
parent_note: asymmetric-encryption-and-pki
---

# TLS 1.3 Deep Dive

## Feynman Explanation
TLS 1.3 is the *one* round trip that turns a TCP connection into a *secret* channel: client and server each send a key share, both sides do a few multiplications on an elliptic curve, and out pops a key that an eavesdropper cannot compute even after recording the entire conversation — that's the *forward secrecy* promise. The earlier TLS 1.2 was a *two*-round-trip handshake with optional encryption of even the certificate, plus a long menu of ciphers including weak ones (RC4, 3DES) that had to be negotiated away. TLS 1.3 deleted the weak ciphers, deleted the extra round trip, and made *every* session key derived from an ephemeral key agreement that no long-term private key can retroactively decrypt.

## Technical Details

### 1. The Two Round Trips

#### 1.1 Round Trip 1 — Client Hello + Server Hello + Server Finished

```
Client                                              Server
------                                              ------
ClientHello
  client_version = 0x0304 (TLS 1.3 in TLS 1.2 compat)
  random = 32 random bytes
  cipher_suites = [TLS_AES_128_GCM_SHA256, ...]
  key_share: ecdh { X25519: pubkey_A }
  signature_algorithms: ed25519, rsa_pss_rsae_sha256, ...
  psk_key_exchange_modes: (optional, for 0-RTT or resumption)
  supported_versions: 0x0304
  supported_groups: x25519, secp256r1, ...
                  -------------------->
                                              ServerHello
                                                version = 0x0304
                                                random = 32 bytes
                                                cipher_suite = TLS_AES_256_GCM_SHA384
                                                key_share: ecdh { X25519: pubkey_B }
                                                ...
                                            {EncryptedExtensions}
                                            {Certificate}
                                            {CertificateVerify}
                                            {Finished}
                  <--------------------
```

The `{EncryptedExtensions, Certificate, CertificateVerify, Finished}` are encrypted under *handshake keys* derived from `(pubkey_A, pubkey_B)` — a key share shared in the clear in the hello messages.

#### 1.2 Round Trip 2 — Client Finished → Application Data

```
                  {Certificate}
                  {CertificateVerify}      (only if client auth requested)
                  {Finished}
                  <--------------------
                  Application Data  // server can send immediately
```

After the client's `Finished` is verified, the client sends *application data* under the same `application_traffic_secret`. The server can also send application data as soon as it has processed the client's `Finished` (concurrent with the client's).

**End-to-end**: a 1-RTT handshake completes in 1 round trip for a fresh connection.

### 2. The Key Schedule

The TLS 1.3 key schedule (RFC 8446 §7.1) is a sequence of HKDF (HMAC-based Key Derivation Function) operations on a transcript hash:

$$
H_0 = H(\text{ClientHello} \,\|\, \text{ServerHello})
$$

then for each step, an Extract followed by Expand:

$$
\begin{aligned}
\text{secret}_1 &= \text{HKDF-Extract}(\text{salt}=0, \text{IKM} = (X, Y) = (g^{xy}, g^x, g^y)) \\
\text{secret}_2 &= \text{HKDF-Expand-Label}(\text{secret}_1, \text{label} = \text{"c hs traffic"}, \text{context} = H_0, L) \\
\text{secret}_3 &= \text{HKDF-Expand-Label}(\text{secret}_2, \text{"s hs traffic"}, H_2, L) \\
\text{secret}_4 &= \text{HKDF-Expand-Label}(\text{secret}_3, \text{"c ap traffic"}, H_3, L) \\
\text{secret}_5 &= \text{HKDF-Expand-Label}(\text{secret}_4, \text{"s ap traffic"}, H_4, L) \\
\text{secret}_6 &= \text{HKDF-Expand-Label}(\text{secret}_5, \text{"exp master"}, H_5, L) \\
\text{resumption\_master} &= \text{HKDF-Expand-Label}(\text{secret}_5, \text{"res master"}, H_5, L) \\
\text{application\_traffic\_secret}_C &= \text{secret}_4 \text{ derived} \\
\text{application\_traffic\_secret}_S &= \text{secret}_5 \text{ derived}
\end{aligned}
$$

Each $H_i$ is the hash of the entire handshake transcript up to and including message $i$ (this is the *transcript hash*; it is what is signed in the `CertificateVerify`).

**Key separation**: each `secret` is used for a specific purpose (`c hs traffic` = client-to-server handshake; `s hs traffic` = server-to-client; `c ap traffic` = client-to-server application; etc.). Compromise of one does not imply compromise of any other.

### 3. HKDF — HMAC-based Key Derivation

RFC 5869. Two functions:

$$
\text{PRK} = \text{HKDF-Extract}(\text{salt}, \text{IKM}) = \text{HMAC-Hash}(\text{salt}, \text{IKM})
$$

$$
\text{OKM}(L) = \text{HKDF-Expand}(\text{PRK}, \text{info}, L) = T_1 \,\|\, T_2 \,\|\, \cdots \,\|\, T_t
$$

with

$$
T_i = \begin{cases} \text{HMAC-Hash}(\text{PRK}, \text{info} \,\|\, 0x01) & i=1 \\ \text{HMAC-Hash}(\text{PRK}, T_{i-1} \,\|\, \text{info} \,\|\, i) & i > 1 \end{cases}
$$

`HKDF-Expand-Label` is a TLS-specific wrapper that prepends the *label* length and a *context* (the transcript hash). The *info* field in TLS 1.3 is essentially the label.

### 4. The Finished Message

`Finished` is a MAC over the entire handshake transcript so far:

$$
\text{verify\_data} = \text{HMAC}(\text{finished\_key}, H(\text{transcript}))
$$

where

$$
\text{finished\_key} = \text{HKDF-Expand-Label}(\text{base\_key}, \text{"finished"}, \text{NULL}, L)
$$

and `base_key` is derived from the relevant handshake traffic secret. The Finished is the only place the handshake transcript is *authenticated*; the certificate and key share provide authentication of the *peer*, but Finished binds them to the actual session.

### 5. Cipher Suites — Five, All AEAD

TLS 1.3 reduced the cipher-suite list to **five** (RFC 8446), and they are *all* AEAD:

| IANA name | AEAD | Hash |
|-----------|------|------|
| TLS_AES_128_GCM_SHA256 | AES-128-GCM | SHA-256 |
| TLS_AES_256_GCM_SHA384 | AES-256-GCM | SHA-384 |
| TLS_CHACHA20_POLY1305_SHA256 | ChaCha20-Poly1305 | SHA-256 |
| TLS_AES_128_CCM_SHA256 | AES-128-CCM | SHA-256 |
| TLS_AES_128_CCM_8_SHA256 | AES-128-CCM (8-byte tag) | SHA-256 |

Notes:
- **No CBC, no 3DES, no RC4, no MD5, no SHA-1.** All the broken families are gone.
- The *hash* is used for HKDF, Finished, and signatures — not for MAC.
- The *AEAD* does the encryption and authentication in one primitive.

### 6. Forward Secrecy (PFS)

A session has *forward secrecy* if compromise of the long-term private key (the server's RSA or ECDSA signing key) does **not** allow decryption of past sessions.

In TLS 1.3, **all** cipher suites are forward-secret by construction because every connection uses an *ephemeral* ECDHE key share (X25519, P-256, P-384, P-521). The server's long-term signing key authenticates the handshake but is *not* used to derive any session key.

| Mode | PFS in TLS 1.3? |
|------|------------------|
| ECDHE (X25519, P-256, ...) | Yes (mandatory) |
| Plain RSA key transport | **Removed** in TLS 1.3 |
| Static DH | **Removed** in TLS 1.3 |
| Fixed DH | **Removed** in TLS 1.3 |
| PSK-only (no (EC)DHE) | No (no PFS, but no long-term-key exposure either) |
| PSK + (EC)DHE | Yes |

**Practical implication**: even if the server's long-term key leaks in 2030, all sessions recorded in 2024 remain confidential.

### 7. 0-RTT — The Performance Tradeoff

TLS 1.3 supports *zero round-trip resumption* — the client can send application data in the first flight (with the ClientHello) under a *pre-shared key (PSK)* derived from a previous session's `resumption_master_secret`.

#### 7.1 Mechanism
The client sends:
```
ClientHello
  early_data: <application data, encrypted under early_secret>
  psk: { identity = resumption_ticket, obfuscated_ticket_age, ... }
```

The server, if it accepts the PSK, decrypts the early data; if not, the client retries with a full 1-RTT.

#### 7.2 The Replay Problem
0-RTT data is **not** protected by the server's `Finished` until the second round trip. An attacker who captures the 0-RTT request can *replay* it to the server. The TLS 1.3 spec mandates that 0-RTT data must be:
- Idempotent (safe to replay; e.g., GET, but not POST).
- OR the application must detect replays (single-use ticket server-side, monotonic counters).

**Anti-pattern**: APIs that allow *any* HTTP method in 0-RTT (e.g., 0-RTT POST /transfer) — leads to double-spend. Most major browsers now send only idempotent 0-RTT data; some (Firefox) disable 0-RTT entirely.

#### 7.3 Forward Secrecy for 0-RTT
A 0-RTT connection **does not** provide forward secrecy for the early data because the early secret is derived from a static PSK. Once the 0-RTT is followed by a full (EC)DHE handshake, the rest of the connection has full PFS.

### 8. Post-Quantum TLS 1.3

TLS 1.3 has been *extended* (IETF draft, deployment in Chrome 124+ in 2024) to support **hybrid key exchange** combining:
- Classical ECDH: X25519 (always-on baseline).
- Post-quantum KEM: ML-KEM-768 (Kyber768, FIPS 203).

The shared secret is the concatenation:

$$
\text{SS} = \text{ECDH}(X25519) \,\|\, \text{ML-KEM-768}(K_{\text{enc}})
$$

The connection is secure if *either* primitive is unbroken. This is the recommended path for forward secrecy against both classical and quantum attackers.

**Real deployments** (2024–2025):
- Chrome / Edge with cloud-fronted domains (`X25519Kyber768`).
- Cloudflare (default for compatible clients).
- AWS (CloudFront supports it; SDK libraries opt in).
- Apple iMessage (pre-quantum readiness).

**Performance**: ML-KEM-768 adds ~1 KB to the ClientHello and ~1 KB to the ServerHello. For most networks this is negligible; for constrained IoT, it matters.

### 9. Server Name Indication (SNI) and Encrypted SNI (ESNI / ECH)

SNI is sent in the clear in TLS 1.3 — an observer learns which hostname is being requested. **Encrypted Client Hello (ECH)** (IETF draft, deployed 2024+) encrypts the SNI using a public key published in DNS (HTTPS record) and bootstrapped via DNSSEC.

### 10. Common TLS 1.3 Attacks and Mitigations

| Attack | Mitigation |
|--------|------------|
| **Bleichenbacher / RSA padding oracle** (TLS 1.2) | Removed: RSA key transport gone in 1.3. |
| **POODLE** (CBC padding) | Removed: no CBC in 1.3. |
| **Sweet32** (3DES birthday) | Removed: no 3DES in 1.3. |
| **RC4 biases** | Removed: no RC4 in 1.3. |
| **Heartbleed** (TLS heartbeat) | Disabled by default in 1.3; heartbeat extension removed. |
| **DROWN** (cross-protocol SSLv2) | Eliminated by disabling SSLv2/3 (1.3 forbids fallback). |
| **Logjam** (weak DH) | Eliminated: only strong ECDHE groups allowed. |
| **Lucky 13** (CBC timing) | Eliminated: no CBC. |
| **0-RTT replay** | Application must be idempotent; ticket must be single-use. |
| **SNI sniffing** | ECH (encrypted client hello). |
| **Quantum** | Hybrid KEM (X25519+ML-KEM-768). |

### 11. TLS 1.3 Implementation Notes (for the architect)

- **Library**: OpenSSL ≥ 1.1.1, BoringSSL, rustls, Go `crypto/tls` ≥ 1.17.
- **Tested suites**: AWS s2n, BoringSSL, NSS, OpenSSL 3.x.
- **Config rule**: `ssl_min_protocol_version = TLSv1.3` (Cloudflare-style), or `TLSv1.2 or TLSv1.3` (compat).
- **Cipher string** (Nginx): `TLSv1.3 TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_128_GCM_SHA256`.
- **0-RTT**: opt in only with strict idempotency or anti-replay at application layer.
- **Resumption tickets**: 24-hour lifetime, single-use, server-side replay detection.
- **Renegotiation**: forbidden in TLS 1.3 (the entire concept is removed).
- **Compression**: forbidden (CRIME attack). Use application-level compression (e.g., HTTP/2 HPACK in header, application-level zstd in body).

## CISO / Risk Manager View

**Board framing**: "Are we on TLS 1.3 everywhere, and is our handshake quantum-ready?" If the answer to the first is no, the answer to the second is also no — there is no quantum-ready TLS 1.2.

**Migration playbook** (the *minimum* viable TLS 1.3 posture):
1. **Inventory** every TLS endpoint (load balancers, API gateways, app servers, proxies, internal east-west, partner links).
2. **Force TLS 1.3 only** on *external* endpoints, allow TLS 1.2 + 1.3 on *internal* (for legacy clients) until end of FY.
3. **Disable weak groups** (P-192, P-224, secp256k1) and weak ciphers; force X25519 / P-256 / P-384 / P-521 + AES-GCM / ChaCha20.
4. **Mandate OCSP stapling** and **Must-Staple** on all public certs.
5. **Adopt hybrid KEM** (X25519+ML-KEM-768) for all *new* external endpoints within 12 months.
6. **Disable 0-RTT** unless the application can guarantee idempotency. Most browsers already restrict to GET.
7. **Enable ECH** (encrypted client hello) for public endpoints; reduces metadata leakage.
8. **Continuous monitoring**: scan quarterly; alert on any TLS 1.0/1.1 endpoint, any RC4 / 3DES / DES, any self-signed cert in production, any cert > 398 days, any RSA-1024 or smaller.

**KPIs / KRIs**:
- 100 % of public endpoints on TLS 1.3 (KPI).
- 0 endpoints on TLS 1.0/1.1 (KRI; alert on any).
- Mean cert lifetime ≤ 90 days (KRI).
- 100 % of public certs in Certificate Transparency logs (KRI).
- 0 endpoints using RSA-2048-with-PKCS1-v1.5 padding (KRI).
- 100 % of new external endpoints on hybrid KEM (KPI, FY26).

**What the auditor will look for**:
- Current Qualys SSL Labs / `testssl.sh` A+ rating.
- Named cipher suite list; no anonymous suites; no NULL suites.
- TLS 1.3 mandatory for new code; TLS 1.2 sunset by FY27.
- Forward-secrecy-only key exchange (no static RSA, no static DH).
- OCSP stapling configured.
- 0-RTT policy documented and enforced.
- PQC-hybrid roadmap documented.

**Board-level risk callout** — *harvest-now-decrypt-later*: any TLS-protected data with a confidentiality life extending past the PQC migration window is at risk of retroactive decryption. A CRQC does not need to exist *today* to motivate action.

## Related Connections
- Asymmetric and PKI: [[asymmetric-encryption-and-pki]]
- Crypto fundamentals: [[cryptography-fundamentals-and-math]]
- Symmetric algorithms: [[symmetric-encryption-algorithms]]
- Hashes & HMAC: [[hash-functions-and-hmac]]
- Post-quantum: [[post-quantum-cryptography]]
- FIPS 140-3: [[fips-140-3]]
- Domain hub: [[domain-03-security-architecture-and-engineering]]

## References
- RFC 8446 (2018). *The Transport Layer Security (TLS) Protocol Version 1.3*.
- RFC 5869 (2010). *HMAC-based Extract-and-Expand Key Derivation Function (HKDF)*.
- RFC 5077 (2008). *Transport Layer Security (TLS) Session Resumption without Server-Side State*.
- RFC 6066 (2011). *TLS Extensions: Extension Definitions* (OCSP stapling).
- RFC 7633 (2015). *X.509v3 Transport Layer Security (TLS) Feature Extension* (Must-Staple).
- RFC 7748 (2016). *Elliptic Curves for Security* (X25519, X448).
- RFC 7905 (2016). *ChaCha20-Poly1305 Cipher Suites for TLS*.
- RFC 8446 Appendix C — *Compliance Requirements*.
- IETF TLS WG drafts — *Hybrid Key Exchange in TLS 1.3* (Stebila, Fluhrer, Gunn).
- NIST FIPS 203 (2024). *Module-Lattice-Based Key-Encapsulation Mechanism (ML-KEM)*.
- Cloudflare TLS 1.3 deployment post-mortems (2017–2024).
- IETF ECH Working Group — *Encrypted Client Hello* drafts.
- Rescorla, E. (2018). *The Transport Layer Security (TLS) Protocol Version 1.3 — RFC 8446*.
- Jager, T. et al. (2015). *On the Security of TLS 1.3 — Formal Analysis*.
- Drucker, N.; Gueron, S. (2019). *A Toolbox for Software Optimization of QC-MDPC Late-Stage Cryptosystems*.
