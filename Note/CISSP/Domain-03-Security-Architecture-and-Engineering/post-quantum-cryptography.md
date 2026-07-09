---
title: Post-Quantum Cryptography — NIST PQC Standards (ML-KEM, ML-DSA, SLH-DSA) and Quantum Threat
layer: 3
domain: 3
tags: [cissp, pqc, post-quantum, kyber, dilithium, sphincs, ml-kem, ml-dsa, slh-dsa, shor, grover, nist, cnsa-2, hybrid-kem, quantum-threat, harvest-now-decrypt-later, hndl]
related_domains: [1, 2, 4, 5, 6, 7, 8]
parent_note: cryptography-fundamentals-and-math
---

# Post-Quantum Cryptography

## Feynman Explanation
A *cryptographically relevant quantum computer* (CRQC) is a future machine that, when it exists, will use Shor's algorithm to factor large numbers and compute discrete logarithms in polynomial time — instantly breaking every RSA, Diffie-Hellman, and elliptic-curve system in use today. Defenders cannot wait; the *harvest-now-decrypt-later* threat means an adversary recording today's TLS traffic, then decrypting it in 2035 when the CRQC is online. NIST's post-quantum standards (ML-KEM for key exchange, ML-DSA for signatures, SLH-DSA for long-life signatures) replace the math with *lattice* and *hash-based* problems that are believed hard even for quantum computers. The migration plan is: hybridize now (combine classical + PQC key exchange in TLS 1.3), and roll forward as standards mature.

## Technical Details

### 1. The Quantum Threat Model

#### 1.1 Shor's Algorithm (1994)

A polynomial-time quantum algorithm for factoring integers and computing discrete logarithms:

$$
T_{\text{Shor}}(n) = O\!\left((\log n)^3\right) \quad \text{on a quantum computer}
$$

Compared to classical:

$$
T_{\text{GNFS}}(n) = \exp\!\left((c + o(1)) (\ln n)^{1/3} (\ln \ln n)^{2/3}\right)
$$

**Effect on asymmetric primitives**:

| Primitive | Hard problem | Classical (RSA-3072, P-256) | Quantum |
|-----------|--------------|------------------------------|---------|
| RSA | Integer factoring | $\approx 2^{128}$ | Polynomial |
| DH / DSA | DLP mod $p$ | $\approx 2^{128}$ | Polynomial |
| ECDH / ECDSA / EdDSA | ECDLP | $\approx 2^{128}$ | Polynomial |
| AES-128 | Symmetric | $2^{128}$ | $2^{64}$ (Grover) |
| AES-256 | Symmetric | $2^{256}$ | $2^{128}$ (Grover) |
| SHA-256 | Pre-image | $2^{256}$ | $2^{128}$ (Grover) |
| SHA-256 | Collision | $2^{128}$ | $2^{85}$ (BHT) |

#### 1.2 Grover's Algorithm (1996)

Quadratic speedup for unstructured search:

$$
T_{\text{Grover}}(N) = O\!\left(\sqrt{N}\right)
$$

Implication: AES-128 and SHA-256 pre-image are *halved* in security. Mitigation: double the key / output size (AES-128 → AES-256, SHA-256 → SHA-384). Lattice schemes have an even smaller Grover penalty because they are *structured*.

#### 1.3 The Harvest-Now-Decrypt-Later (HNDL) Threat

Adversaries (nation-states, well-funded cyber-mercenaries) are recording *today's* TLS-encrypted traffic. The data is stored. When a CRQC is online, the recorded ciphertexts are decrypted.

**Critical data classes for HNDL**:
- Government / military communications (Top Secret, compartmented).
- Healthcare records (HIPAA-protected; 50+ year confidentiality).
- Financial trading flows (proprietary for years).
- Industrial IP (chemical formulas, defense designs).
- Legal communications (attorney-client privilege).
- Personal biometric data (irrevocable).

A rule of thumb: any data with a confidentiality life of more than **5–10 years** (i.e., past 2030–2035) is at HNDL risk *today*.

#### 1.4 Timeline Estimates (NSA, 2024)

| Capability | Rough estimate (no public confirmation) |
|------------|------------------------------------------|
| CRQC of 25 million physical qubits | 5–10 years |
| CRQC of 100 million physical qubits | 10–15 years |
| Logical qubits for RSA-2048 | ~4,000 (with error correction overhead) |
| Logical qubits for ECC-256 | ~2,500 |
| Time to break RSA-2048 (estimated) | 8 hours on a 1M-physical-qubit CRQC |

These are *guesses* — public announcements by Google, IBM, and the NSA in 2024–2025 suggest 5–15 year horizons.

### 2. NIST Post-Quantum Standards

After a 6-year competition (2017–2022) and a 4th-round selection, NIST published the first three PQC FIPS standards in 2024:

| FIPS | Algorithm (original) | Use | Construction | Status (2024) |
|------|----------------------|-----|--------------|----------------|
| FIPS 203 | ML-KEM (Kyber) | Key encapsulation | Module-LWE (lattice) | Standard |
| FIPS 204 | ML-DSA (Dilithium) | Digital signatures | Module-LWE + Module-SIS (lattice) | Standard |
| FIPS 205 | SLH-DSA (SPHINCS+) | Stateless hash-based signatures | Hash + Merkle | Standard |

Fourth round (HQC, BIKE) and additional signature schemes (FN-DSA / FALCON-512, FN-DSA-1024) remain in evaluation or draft.

### 3. ML-KEM (FIPS 203) — Kyber

Module-Lattice-based Key Encapsulation Mechanism. The KEM pattern:

```
KeyGen():
  pk, sk = generate_keypair()      //  pk = 1184 B (ML-KEM-768), sk = 2400 B

Encapsulate(pk):
  m  = 256 random bits
  K, c = encapsulate(m, pk)        //  shared secret K, ciphertext c

Decapsulate(sk, c):
  K' = decapsulate(sk, c)
  // Implied: K' == K if c valid
```

#### 3.1 Underlying Problem

ML-KEM's security reduces to the **Module Learning With Errors (M-LWE)** problem:

> Given a public matrix $\mathbf{A} \in R_q^{k \times k}$ and a noisy inner product $(\mathbf{A}\mathbf{s} + \mathbf{e}) \bmod q$, find $\mathbf{s}$.

where $R_q = \mathbb{Z}_q[X] / (X^n + 1)$, $n = 256$, $q = 3329$. The lattice parameter $\eta_1$ controls the error distribution.

The best known quantum attack on M-LWE runs in time:

$$
T_{\text{MLWE}} = 2^{\Theta(n)} \quad \text{vs} \quad 2^{O(n^{2.5})} \text{ classical}
$$

For $n = 256$ (ML-KEM-768), $\approx 2^{192}$ quantum operations — *above* 128-bit security. The scheme is conservatively parameterized for ~192-bit classical and ~170-bit quantum.

#### 3.2 Parameters

| Parameter set | Security (classical) | pk | sk | ct | Shared secret |
|---------------|----------------------|------|------|------|---------------|
| ML-KEM-512 | 128 | 800 B | 1632 B | 768 B | 32 B |
| ML-KEM-768 | 192 | 1184 B | 2400 B | 1088 B | 32 B |
| ML-KEM-1024 | 256 | 1568 B | 3168 B | 1568 B | 32 B |

**Recommended**: ML-KEM-768 for general use; ML-KEM-1024 for top-secret.

#### 3.3 Implementation Notes

- **Side-channel resistance**: ML-KEM is a constant-time lattice operation; the major decapsulation failure check uses a *silent* reject (returns a pseudo-random key on failure) to avoid timing oracles.
- **Performance**: ML-KEM-768 keygen ~20 μs, encaps ~25 μs, decaps ~30 μs on a modern x86 (OpenSSL / liboqs).
- **Footprint**: 1.4 KB client hello extension. Modest compared to TLS 1.3's full handshake.
- **Hybridization** is the recommended deployment mode (see §6).

### 4. ML-DSA (FIPS 204) — Dilithium

Module-Lattice-based Digital Signature Algorithm.

#### 4.1 Signature Operation

Given a message $M$ and a private key $\mathbf{s}_1, \mathbf{s}_2, t$:

1. Hash the message with SHAKE256 (or a context-bound domain): $\mu = H(M)$.
2. Random mask $\mathbf{y} \leftarrow$ short polynomial vector.
3. Compute $\mathbf{w} = \mathbf{A}\mathbf{y}$ (high norm), $\mathbf{w}_1 = \text{HighBits}(\mathbf{w}, 2\gamma_2)$, $c = H(\mu \,\|\, \mathbf{w}_1)$.
4. Compute $\mathbf{z} = \mathbf{y} + c \mathbf{s}_1$.
5. Rejection sampling — if $\mathbf{z}$ is too high, retry.
6. Signature is $(c, \mathbf{z}, h)$.

Verification:

1. Recompute $\mathbf{w}' = \mathbf{A}\mathbf{z} - c \mathbf{t}$.
2. Check that $c = H(\mu \,\|\, \text{HighBits}(\mathbf{A}\mathbf{z} - c \mathbf{t}, 2\gamma_2))$.
3. Check that the norms of $\mathbf{z}$ and the hint $h$ are within bounds.

#### 4.2 Parameters

| Parameter set | Security (classical) | pk | sig |
|---------------|----------------------|-----|-----|
| ML-DSA-44 (Dilithium2) | 128 | 1312 B | 2420 B |
| ML-DSA-65 (Dilithium3) | 192 | 1952 B | 3293 B |
| ML-DSA-87 (Dilithium5) | 256 | 2592 B | 4595 B |

#### 4.3 Properties

- **Deterministic** signing variant available (with a random seed) — eliminates RNG dependence.
- **Fiat-Shamir with Aborts**: rejection sampling is normal; performance is competitive.
- **Hash function**: SHAKE256 / SHA-3 family. (Makes ML-DSA a *SHA-3* scheme.)

#### 4.4 EdDSA vs ML-DSA Tradeoff (2025)

| Property | Ed25519 | ML-DSA-65 |
|----------|---------|-----------|
| Sign speed | ~70 μs | ~200 μs |
| Verify speed | ~250 μs | ~150 μs |
| Signature size | 64 B | 3293 B (50× larger) |
| Public key | 32 B | 1952 B (60× larger) |
| PQC-safe | No (ECDLP → broken by Shor) | Yes |
| Mature | Yes (since 2011) | Yes (FIPS 204, 2024) |

For new *classical-only* systems: Ed25519 is the default. For *PQC* requirements: ML-DSA-65 (or higher) is the standard.

### 5. SLH-DSA (FIPS 205) — SPHINCS+

Stateless Hash-Based Signature Scheme. The signature is a one-time signature (WOTS+ or FORS) extended with a Merkle tree.

#### 5.1 Underlying Idea

Each WOTS+ key signs exactly one message; the public key is a single Merkle tree leaf; the signature is the leaf + the authentication path. For $h$-bit security, the tree has $2^h$ leaves — $2^{128}$ leaves for 128-bit security. To sign many messages, SPHINCS+ uses *hyper-trees*: a tree of trees, with each layer using a different WOTS+ key.

#### 5.2 Parameters

| Parameter set | Security | pk | sig |
|---------------|----------|-----|-----|
| SLH-DSA-SHA2-128s | 128 (cat. 1) | 32 B | 7,856 B |
| SLH-DSA-SHA2-128f | 128 (cat. 1) | 32 B | 17,088 B |
| SLH-DSA-SHAKE-128s | 128 (cat. 1) | 32 B | 7,856 B |
| SLH-DSA-SHA2-192s | 192 (cat. 3) | 48 B | 16,224 B |
| SLH-DSA-SHA2-256s | 256 (cat. 5) | 64 B | 49,216 B |

(`s` = small signature, slower signing; `f` = fast signing, larger signature.)

#### 5.3 Tradeoffs

- **Conservative**: security reduces to *second-pre-image resistance* of the underlying hash (SHA-256, SHA-512, SHAKE256). The weakest hash assumption of any signature scheme.
- **Slow and large**: signatures are 7–49 KB; signing is 50–1000× slower than Ed25519.
- **Stateless**: unlike XMSS / LMS (stateful hash-based, NIST SP 800-208), SPHINCS+ requires no state per signer. Simpler key management, slightly larger signatures.

**Use cases**: firmware signing, software signing, root CA signing, code signing with 20+ year integrity, document signing for long-life evidence.

### 6. Hybrid KEM in TLS 1.3

The 2024 IETF draft (Stebila, Fluhrer, Gunn) defines *hybrid* key exchange for TLS 1.3:

```
key_share:
  ecdh { X25519: client_pubkey_X }      // classical
  kem { ML-KEM-768: client_pubkey_K }   // PQC
```

The shared secret is the concatenation:

$$
\text{SS} = \text{ECDH}(X25519) \,\|\, \text{ML-KEM-768}(K_{\text{enc}})
$$

then fed through the standard TLS 1.3 key schedule. The connection is secure if *either* primitive is unbroken — quantum-safe if ML-KEM is unbroken, classically safe if X25519 is unbroken.

**Real deployments (2024–2025)**:
- Chrome 124+ with `X25519Kyber768Draft00`.
- Cloudflare (default for compatible clients).
- AWS CloudFront.
- Apple iMessage (PQ3 protocol, 2024).

### 7. Other PQC Schemes (Active in Standards Process)

| Algorithm | Use | Family | Status (2024) |
|-----------|-----|--------|----------------|
| FN-DSA (FALCON-512) | Signatures | NTRU-lattice | NIST draft FIPS 206 |
| FN-DSA (FALCON-1024) | Signatures | NTRU-lattice | NIST draft |
| HQC | KEM | Code-based | 4th-round alternate |
| BIKE | KEM | Code-based | 4th-round alternate |
| Classic McEliece | KEM | Code-based | Alternate; 1 MB public key |
| XMSS / LMS | Signatures | Stateful hash | NIST SP 800-208 |

FALCON's advantage over ML-DSA: smaller signatures (666 B for FALCON-512 vs 2420 B for ML-DSA-44). Disadvantage: Gaussian sampling is harder to implement side-channel-free; complex parameter selection.

### 8. CNSA 2.0 — NSA's Quantum Mandate (2022)

The Commercial National Security Algorithm Suite 2.0 mandates PQC for *all* national-security systems:

| Use | Classical | CNSA 2.0 PQC |
|-----|-----------|---------------|
| Software / firmware signing | RSA-3072, ECDSA-P-384 | ML-DSA-65, ML-DSA-87 |
| Web / API / TLS | RSA-3072, ECDH-P-384 | ML-KEM-768, ML-KEM-1024 (hybrid) |
| Low-end IoT | RSA-2048, ECDSA-P-256 | ML-DSA-44, ML-KEM-512 |
| Hash | SHA-384, SHA-512 | SHA-384, SHA-512 (SHA-3 acceptable) |
| Symmetric | AES-256 | AES-256 |

**Timeline**:
- 2025 — non-binding; PQC available.
- 2027–2030 — software/firmware signing *must* use PQC.
- 2030–2033 — web/TLS, *all* new products and acquisitions.
- 2033 — full PQC for national security.

**Software-signing-only transition** is *sooner* than TLS — the threat is *retroactive forgery*, which means an attacker with a CRQC can re-sign old firmware if the algorithm is broken.

### 9. Crypto-Agility — The PQC Migration Prerequisite

PQC migration is a *software* problem, not a *math* problem. The math is ready. The migration challenges are:

1. **Larger keys and signatures** — TLS handshake grows by 1–2 KB.
2. **New APIs** — `ML-KEM-768.Encaps(pk)` is not a drop-in for `RSA.Encrypt(pk)`.
3. **Hybridization** — every call site has to combine two primitives.
4. **HSM / KMS support** — vendors must integrate ML-KEM into their APIs.
5. **Certificate format** — X.509 must encode ML-DSA public keys (256-byte SubjectPublicKeyInfo).
6. **FIPS validation** — FIPS 140-3 validated modules must re-validate with PQC.

The architectural property that makes all of this manageable is **crypto-agility**: every cryptographic call accepts the algorithm as a parameter. If your code does `RSA_sign(SHA256, ...)`, you cannot migrate. If your code does `crypto.sign(alg="RSA-PSS-3072", hash="SHA-256", key=...)`, you change one config and the entire estate migrates.

### 10. PQC Migration Roadmap (CISO Template)

**Year 0 — Inventory (90 days)**:
- Catalog all cryptographic assets: which algorithms, which libraries, which endpoints, which data classifications.
- Identify *long-confidentiality-life* data (HNDL targets).
- Inventory HSMs, KMS, smart cards, tokens.
- Identify vendor dependencies (which products use which crypto).

**Year 0 — Crypto-Agility Audit (90 days)**:
- Static analysis: find all hard-coded algorithm names.
- Refactor to a central crypto-API.
- Set up `crypto.update(alg=...)` to be a one-line config change.

**Year 1 — TLS Hybridization (12 months)**:
- Enable `X25519+ML-KEM-768` on all public TLS endpoints.
- Maintain classical-only fallback (X25519) for legacy clients.
- Test, monitor, profile.

**Year 2 — PQC Signatures for High-Value Use (12 months)**:
- Code signing pipeline → Ed25519+ML-DSA-65 (dual-signed, dual-verified).
- Document signing for 10+ year integrity → SLH-DSA-SHA2-192s.
- Root CA → ML-DSA-87.

**Year 3 — Internal TLS and East-West (12 months)**:
- Internal mTLS → hybrid KEM.
- Service mesh sidecars → ML-KEM-768.
- Internal CAs → ML-DSA-65.

**Year 4 — Deprecation of Classical for HNDL-relevant Data**:
- Identify the 5–10 systems with the highest HNDL exposure.
- Move *only those* to PQC-only KEM.
- Monitor CRQC milestones (NIST PQC migration advisories).

**Year 5+ — Continued Migration**:
- As libraries and vendors mature, broaden PQC adoption.
- Track NIST guidance and standards updates.

### 11. PQC in Specific Protocols

| Protocol | PQC migration path |
|----------|---------------------|
| TLS 1.3 | Hybrid KEM (X25519+ML-KEM-768); IETF draft 2024 |
| IKEv2 (IPsec) | IKEv2 hybrid KE; RFC 9370 (ML-KEM, ML-DSA, Falcon) |
| SSH (RFC 4253) | `ssh-mlkem768x25519-sha256` (2024) |
| S/MIME (RFC 8551) | CMS with ML-KEM, ML-DSA |
| OpenPGP (RFC 9580) | v6 keys: ML-KEM-768 + Ed25519 |
| DNSSEC | Conjectural; not standardized |
| WebAuthn / FIDO2 | Conjectural; classical ECC for now |
| KMIP | Draft PQC support |
| TLS 1.2 | Not recommended for PQC migration; upgrade to 1.3 |

### 12. Quantum-Resilient Architectures Beyond Cryptography

Even with PQC, certain architectural problems are *not* solved:

- **Replay**: signatures can still be replayed in absence of counters/nonces.
- **Compromise of pre-quantum data**: data captured before PQC migration remains vulnerable to a future CRQC.
- **Quantum side channels**: not all PQC implementations are constant-time.
- **HSM re-validation**: a CRQC may not break a key, but the FIPS 140-3 validation is for a specific algorithm set; re-validation is needed.

## CISO / Risk Manager View

**Board framing**: "What is our quantum-readiness posture, and what is our HNDL exposure?" The board is asking: "If the news in 2031 says a CRQC was demonstrated, what do we lose, and what do we do about it?" The answer must be specific, dollar-denominated, and date-bounded.

**Board-level HNDL exposure estimate** (the math the CFO will want):
- Identify *long-life* data classes (IP, PII, government).
- Estimate volume: $V$ TB.
- Confidentiality life: $L$ years.
- Probability of CRQC within $L$ years: $p$.
- Expected loss: $V \times p \times \text{cost-per-record}$.
- Mitigation cost: PQC migration of *only* the relevant systems.

The cost of PQC migration of an external TLS estate is typically **1–3 % of the cost of a single compliance breach**. The board math supports action.

**Quantum-readiness program (CISO charter)**:
1. **Inventory** (90 days) — all crypto assets, all HNDL-relevant data.
2. **Crypto-agility mandate** (180 days) — refactor hard-coded algorithms.
3. **Hybrid PQC deployment** (12–18 months) — TLS 1.3, code signing.
4. **Vendor PQC contract clause** (immediate) — any new procurement must include a PQC migration roadmap.
5. **HSM / KMS PQC upgrade** (24–36 months) — phased, vendor-driven.
6. **Continuous CRQC tracking** — quarterly briefing; monitor IBM, Google, NSA, NIST PQC migration advisories.

**Board-level callouts**:
- **HNDL is the single most underappreciated threat in enterprise security** — every CFO, every CISO, every General Counsel should understand it.
- **PQC is not optional** — it is a regulatory and procurement requirement (CNSA 2.0 for US gov; EU BSI TR-02102 for Germany; UK NCSC PQC migration; ANSSI for France).
- **Crypto-agility is the only sustainable defense** — even with PQC, the next break (or the next quantum upgrade) is coming. Code that names algorithms cannot survive; code that accepts algorithm identifiers can.

**Common audit finding the board will expect to see addressed**:
- "System X uses RSA-2048 for customer data confidentiality with a 25-year data retention."
- "System Y hard-codes `RSA_sign` in 47 call sites."
- "Vendor Z has no public PQC roadmap."

**Board question to prepare for**: "When was the last CRQC threat briefing? When is the next PQC migration milestone? What is the residual HNDL exposure after the FY26 plan executes?" The CISO's three-quarter answer is the board's confidence signal.

## Related Connections
- Crypto fundamentals: [[cryptography-fundamentals-and-math]]
- Asymmetric and PKI: [[asymmetric-encryption-and-pki]]
- TLS 1.3: [[tls-1-3-deep-dive]]
- Hashes & HMAC: [[hash-functions-and-hmac]]
- FIPS 140-3: [[fips-140-3]]
- Security architecture patterns: [[security-architecture-patterns]]
- Domain hub: [[domain-03-security-architecture-and-engineering]]

## References
- NIST FIPS 203 (Aug 2024). *Module-Lattice-Based Key-Encapsulation Mechanism (ML-KEM)*.
- NIST FIPS 204 (Aug 2024). *Module-Lattice-Based Digital Signature Algorithm (ML-DSA)*.
- NIST FIPS 205 (Aug 2024). *Stateless Hash-Based Digital Signature Algorithm (SLH-DSA)*.
- NIST SP 800-227 (2024). *Migration to Post-Quantum Cryptography*.
- NIST SP 800-208 (2020). *Stateful Hash-Based Signature Schemes* (XMSS, LMS).
- NIST SP 800-131A Rev. 2 (2023). *Transitioning the Use of Cryptographic Algorithms and Key Lengths*.
- NSA CNSA 2.0 (Sept 2022). *Announcing the Commercial National Security Algorithm Suite 2.0*.
- NSA (2024). *The Commercial National Security Algorithm Suite 2.0 — Frequently Asked Questions*.
- IETF draft-ietf-tls-hybrid-design (2024). *Hybrid Key Exchange in TLS 1.3*.
- IETF draft-ietf-ipsecme-ikev2-multiple-ke (RFC 9370, 2023). *Multiple Key Exchanges in IKEv2*.
- Shor, P. W. (1994, 1997). *Polynomial-Time Algorithms for Prime Factorization and Discrete Logarithms on a Quantum Computer*.
- Grover, L. K. (1996). *A Fast Quantum Mechanical Algorithm for Database Search*.
- CRYSTALS team — *Kyber*, *Dilithium* specifications.
- Bernstein, D. J.; Lange, T. (2017). *Post-quantum cryptography*.
- Chen, L. et al. (2016). *Report on Post-Quantum Cryptography*, NISTIR 8105.
- ENISA (2023). *Post-Quantum Cryptography: Integration Study*.
- ETSI TR 103 616 (2021). *Quantum-Safe Cryptography and Security*.
- BSI TR-02102 (2024). *Cryptographic Mechanisms: Recommendations and Key Lengths* (Germany).
- ANSSI (2024). *PQC migration guidance* (France).
- Cloudflare (2024). *Post-Quantum at Cloudflare*.
- Google (2024). *Post-Quantum Cryptography in Chrome* (X25519Kyber768).
- Signal (2024). *PQ3 — Post-Quantum Ratchet for iMessage*.
