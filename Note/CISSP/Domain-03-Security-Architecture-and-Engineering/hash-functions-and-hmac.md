---
title: Hash Functions and HMAC — MD5, SHA-1, SHA-2, SHA-3, Birthday Paradox
layer: 3
domain: 3
tags: [cissp, hash, sha-1, sha-2, sha-3, keccak, hmac, mac, birthday-paradox, merkle-damgard, sponge-construction, shake, kmac]
related_domains: [2, 4, 7, 8]
parent_note: cryptography-fundamentals-and-math
---

# Hash Functions and HMAC

## Feynman Explanation
A cryptographic hash function turns any message — a sentence, a movie file, a database row — into a short fixed-length fingerprint (e.g., 256 bits) in a way that makes it practically impossible to find two different messages with the same fingerprint or to reverse the fingerprint back into the message. The *birthday paradox* says collisions become likely after only $\approx 1.18\sqrt{H}$ trials (much fewer than you'd think), which is why 128-bit hashes (MD5) are dead and 256-bit hashes (SHA-256) are the modern floor. HMAC wraps a hash with a secret key to produce a *Message Authentication Code* — the primitive that proves "this message came from someone who knew the key and was not altered in transit," the workhorse of TLS, JWT, and API authentication.

## Technical Details

### 1. Definition of a Cryptographic Hash

A hash function $H: \{0,1\}^* \to \{0,1\}^n$ must satisfy:

1. **Pre-image resistance** — given $y = H(x)$, finding any $x'$ such that $H(x') = y$ is hard.
2. **Second-pre-image resistance** — given $x$, finding $x' \ne x$ with $H(x) = H(x')$ is hard.
3. **Collision resistance** — finding any $x \ne x'$ with $H(x) = H(x')$ is hard.

The three properties are *ordered by strength* and *ordered by how long they last under attack*:

| Property | Generic attack cost | Notes |
|----------|---------------------|-------|
| Pre-image | $O(2^n)$ | Held by SHA-1 (theoretical) |
| Second pre-image | $O(2^n)$ | Held by SHA-1 |
| Collision | $O(2^{n/2})$ | **Broken** for MD5, **broken** for SHA-1 |

### 2. The Birthday Paradox

For a hash with $n$-bit output, a random attacker needs on average:

$$
B(n) \approx 1.177 \sqrt{2^n} = 1.177 \cdot 2^{n/2}
$$

trials to find a collision. Derivation: the number of items needed for $\ge 50\%$ collision probability in a set of $2^n$ possible values.

Equivalently:

$$
P(\text{collision after } q \text{ trials}) \approx 1 - e^{-q(q-1)/(2 \cdot 2^n)}
$$

For $q \approx 1.177 \sqrt{2^n}$, $P \approx 0.5$.

| $n$ | $2^{n/2}$ | Real-world status |
|-----|-----------|-------------------|
| 64 (MD5) | $2^{32} \approx 4 \times 10^9$ | Collisions in seconds on a laptop |
| 80 | $2^{40} \approx 10^{12}$ | Birthday attack on legacy MACs |
| 128 (MD5/SHA-1 collision) | $2^{64}$ | MD5 collisions in seconds; SHA-1 collisions in $2^{63.1}$ (SHAttered, 2017) |
| 160 (SHA-1 pre-image) | $2^{80}$ | Not practically broken (yet) |
| 256 (SHA-256) | $2^{128}$ | Out of reach of classical computers |

**Implication**: a hash with $n$-bit output gives *only $n/2$ bits of collision resistance*. Hence SHA-256 gives 128 bits of collision security — which is the *minimum* for any signature or MAC application.

### 3. MD5 (1992, Rivest) — BROKEN

- 128-bit output, Merkle–Damgård construction.
- **Collisions found in seconds** since 2004 (Wang et al.). Used to produce a rogue CA certificate (Flame malware, 2012; chosen-prefix collision by Stevens, 2012).
- **Status**: deprecated. Legacy use only. Forbidden in FIPS since 2010.

### 4. SHA-1 (1995, NIST) — BROKEN for collision

- 160-bit output, Merkle–Damgård.
- **Theoretical collision attack** by Wang (2005); **practical collision**: SHAttered (Stevens et al., 2017) at $\sim 2^{63.1}$ SHA-1 evaluations. Cost: ~$110k cloud.
- **Chosen-prefix collisions** (Stevens 2012, Leurent-Peyrin SHAmbles 2020) extend to the *practical* forgery of signatures using SHA-1.
- **Status**: rejected by NIST since 2017. **Rejection deadline for digital signatures**: 2030 (NIST SP 800-131A Rev. 2). **For TLS certificates**: CA/B Forum 2017.
- **Collision-resistant use**: $\sim 2^{63}$ work. Pre-image: still $\sim 2^{160}$ work (not yet broken).
- Use SHA-1 only for non-security, e.g., Git *integrity* (where attacker can't choose both sides), and even there the industry is migrating to SHA-256.

### 5. SHA-2 (FIPS 180-4, 2002; 2008 expansion)

SHA-2 is a family of six variants built on the Merkle–Damgård construction with a Davies–Meyer compression function (block cipher used in a custom mode).

| Variant | Output | Block | Rounds | Word size |
|---------|--------|-------|--------|-----------|
| SHA-224 | 224 | 512 | 64 | 32 |
| SHA-256 | 256 | 512 | 64 | 32 |
| SHA-384 | 384 | 1024 | 80 | 64 |
| SHA-512 | 512 | 1024 | 80 | 64 |
| SHA-512/224 | 224 | 1024 | 80 | 64 |
| SHA-512/256 | 256 | 1024 | 80 | 64 |

#### 5.1 SHA-256 Compression Function

SHA-256 operates on a 512-bit message block and a 256-bit state $H_i$:

$$
H_{i+1} = H_i \,\|\, \text{Compress}(H_i, M_i)
$$

The compression function uses 64 rounds of:

$$
\begin{aligned}
\Sigma_0(a) &= \text{ROTR}^2(a) \oplus \text{ROTR}^{13}(a) \oplus \text{ROTR}^{22}(a) \\
\Sigma_1(e) &= \text{ROTR}^6(e) \oplus \text{ROTR}^{11}(e) \oplus \text{ROTR}^{25}(e) \\
\sigma_0(x) &= \text{ROTR}^7(x) \oplus \text{ROTR}^{18}(x) \oplus \text{SHR}^3(x) \\
\sigma_1(x) &= \text{ROTR}^{17}(x) \oplus \text{ROTR}^{19}(x) \oplus \text{SHR}^{10}(x)
\end{aligned}
$$

with 8 working variables $a..h$, 64 round constants $K_t$, and a message schedule $W_t$:

$$
W_t = \begin{cases} M_t & t < 16 \\ \sigma_1(W_{t-2}) + W_{t-7} + \sigma_0(W_{t-15}) + W_{t-16} & 16 \le t < 64 \end{cases}
$$

Security: no known collision attack; best pre-image attack covers ~52 of 64 rounds (2024); still considered secure for the next decade.

#### 5.2 SHA-512 / SHA-384

The 64-bit variant — same structure, larger working variables, more rounds of diffusion, ~50% faster on 64-bit CPUs than SHA-256. SHA-512/256 is *truncated* to 256-bit output for compatibility but uses the 64-bit internal state.

### 6. SHA-3 (FIPS 202, 2015) — Keccak

NIST held a 2007–2012 competition; the winner was **Keccak** (Bertoni, Daemen, Peeters, Van Assche) — a *sponge construction*, not Merkle–Damgård.

#### 6.1 Sponge Construction

The state is $b = r + c$ bits, split into *rate* $r$ (absorb/squeeze) and *capacity* $c$ (security). SHA-3-256 uses $r = 1088$, $c = 512$.

$$
\begin{aligned}
\text{Absorb:} \quad S_{i+1} &= f(S_i \oplus (P_i \,\|\, 0^c)) \\
\text{Squeeze:} \quad Z_i &= \text{trunc}_r(S_{i+r/r})
\end{aligned}
$$

where $f$ is the Keccak-f[1600] permutation (24 rounds of θ, ρ, π, χ, ι).

| SHA-3 variant | Output | r | c | Note |
|---------------|--------|---|---|------|
| SHA3-224 | 224 | 1152 | 448 | |
| SHA3-256 | 256 | 1088 | 512 | Standard |
| SHA3-384 | 384 | 832 | 768 | |
| SHA3-512 | 512 | 576 | 1024 | |
| SHAKE128 | extensible | 1344 | 256 | Variable output (XOF) |
| SHAKE256 | extensible | 1088 | 512 | Variable output (XOF) |

#### 6.2 Why a New Hash Family?

- SHA-2 (Merkle–Damgård) has *length-extension* vulnerability: given $H(M)$ and $\text{len}(M)$, an attacker can compute $H(M \,\|\, \text{pad} \,\|\, X)$ for any $X$ *without knowing $M$*. (Fixed by switching to SHA-512/256 or using HMAC.)
- SHA-3 (sponge) is *not* length-extendable because the capacity is never directly exposed.
- SHA-3 is structurally *different* from SHA-2, so a future break of SHA-2 (e.g., from a new differential attack) likely won't break SHA-3.

### 7. Length-Extension Attack

For Merkle–Damgård hashes (SHA-1, SHA-2):

$$
H(M \,\|\, \text{pad}_L \,\|\, M') = H(M \,\|\, M' \text{ with appropriate padding})
$$

an attacker can compute the hash of $M$ extended with $M'$ from $H(M)$ and $\text{len}(M)$.

#### 7.1 Mitigation
- **HMAC**: uses two keys, breaking the attack (see below).
- **SHA-512/256**: truncated output, so length extension doesn't expose enough.
- **SHA-3**: sponge construction is immune.
- **Truncated HMAC** (HMAC-SHA-256/128): the inner state is still secure.

### 8. HMAC — Keyed-Hash Message Authentication Code (RFC 2104)

HMAC uses an underlying hash $H$ and a key $K$:

$$
\text{HMAC}(K, M) = H\big((K \oplus \text{opad}) \,\|\, H((K \oplus \text{ipad}) \,\|\, M)\big)
$$

where
- $\text{ipad} = 0x36$ repeated $B$ times (block size of the hash; 64 for SHA-256).
- $\text{opad} = 0x5C$ repeated $B$ times.
- $K$ is zero-padded to $B$ bytes (or pre-hashed if longer than $B$).

#### 8.1 Security Properties
- *Unforgeability* under chosen-message attack (UF-CMA) if $H$ is a PRF.
- *Length-extension immunity* — HMAC is *not* vulnerable to the length-extension attack that plagues raw SHA-256.
- *Keyed* — only parties with the key can produce / verify the tag.

#### 8.2 Recommended Variants (RFC 8554 / NIST SP 800-107)
- HMAC-SHA-1: legacy, 160-bit output, acceptable for non-collision use only.
- HMAC-SHA-256: standard, 256-bit.
- HMAC-SHA-384: high-assurance, 384-bit.
- HMAC-SHA-512: high-assurance, 512-bit.
- HMAC-SHA3-256: alternative to HMAC-SHA-256 (sponge).

#### 8.3 Output Truncation

HMAC-SHA-256/128: truncate HMAC-SHA-256 to 128 bits. The *full* tag is computed, then truncated. The 128-bit security level is *against forgery* (the attacker must try $2^{128}$ tags). Useful for protocols with limited bandwidth.

### 9. KMAC — SHA-3 Variant of HMAC (NIST SP 800-185)

KMAC uses the cSHAKE (customizable SHAKE) function:

$$
\text{KMAC128}(K, M, L, S) = \text{cSHAKE128}(N = \text{"KMAC"}, S, L) \text{ with secret } K
$$

$$
\text{KMAC256}(K, M, L, S) = \text{cSHAKE256}(N = \text{"KMAC"}, S, L) \text{ with secret } K
$$

where $S$ is customization string (domain separation) and $L$ is output length. Variable-length output, no length-extension concern, native SHA-3.

### 10. Poly1305 (RFC 8439)

A *universal-hash* MAC designed by Bernstein. A 32-byte one-time key $r$ and a 32-byte secret key $s$ are used to authenticate a message of any length:

$$
\text{Poly1305}(r, s, M) = \big(((\sum_i m_i \cdot p^i) \bmod 2^{130} - 5) + s\big) \bmod 2^{128}
$$

where $p$ is a large prime and $m_i$ are blocks of $M$ interpreted in base $2^{128}$ (with a final partial block). The tag is 16 bytes.

Poly1305 is paired with ChaCha20 (in ChaCha20-Poly1305 AEAD) or AES.

### 11. GMAC (Galois Message Authentication Code)

GMAC is the authentication component of GCM. Uses the same GHASH as GCM but without the encryption. Suitable for authenticating packets without confidentiality (e.g., routing protocol authentication in IPsec AH-mode, though GCM-only ESP is the modern choice).

### 12. Hash-Based Message Authentication: Composition Risks

The classic construction $H(K \,\|\, M)$ (i.e., prepend the key) is **broken** for Merkle–Damgård hashes (length extension). The classic $H(M \,\|\, K)$ (append the key) is broken for collision attacks. Use HMAC; do not roll your own.

### 13. Hash Function Choice (2025)

| Use | Recommended |
|-----|-------------|
| TLS 1.3 (in suite) | SHA-256 or SHA-384 (chosen with the AEAD) |
| TLS 1.2 PRF | SHA-256 or SHA-384 |
| Digital signatures | SHA-256 minimum; SHA-384 for 192-bit; SHA-512 for 256-bit |
| HMAC | HMAC-SHA-256; HMAC-SHA3-256 for variety |
| Code signing | SHA-256 (with Ed25519) or SHA-384 (with ML-DSA-65) |
| Password storage | **Do not use a hash** — use Argon2id, scrypt, or PBKDF2 |
| Key derivation | HKDF-HMAC-SHA-256 |
| File integrity (Git, etc.) | SHA-256; SHA-1 for legacy compatibility only |
| PQC signature contexts | SHA-3-256, SHAKE256 |

### 14. Quantum Impact on Hashes

Grover's algorithm finds a pre-image in $O(2^{n/2})$ time. For an $n$-bit hash, this halves the effective security:

| Hash | Classical | Quantum (Grover) |
|------|-----------|-------------------|
| SHA-256 | 256 (pre-image) / 128 (collision) | 128 (pre-image) / 85 (collision) |
| SHA-384 | 384 / 192 | 192 / 96 |
| SHA-512 | 512 / 256 | 256 / 128 |

Mitigation: move to SHA-384/SHA-512 for *long-term* integrity needs; for *collision resistance* (signatures, MACs), use 256-bit hashes. SHA-3 (Keccak) is structurally different; it is the safe diversification bet.

NIST SP 800-208 (*Recommendation for Stateful Hash-Based Signature Schemes*) uses SHA-256 and SHA-512 as the underlying hash for SLH-DSA (SPHINCS+).

## CISO / Risk Manager View

**Board framing**: "Are we using broken hashes anywhere?" The auditor expects the answer to be *no* and the inventory to be on file. The most common audit findings are: (1) MD5 or SHA-1 used in a code-signing pipeline; (2) a custom `HMAC`-like construction `H(K || M)`; (3) a password database using SHA-256 instead of Argon2id.

**Strategic posture (CISO's hash policy)**:
1. **Forbid MD5, SHA-1, RIPEMD, and any pre-2010 hash for security use.** No exceptions.
2. **Mandate SHA-256 minimum** for new code; **SHA-384** for high-assurance.
3. **Mandate HMAC** for any keyed authentication; do not roll your own `H(K || M)`.
4. **Replace any password-hashing with MD5/SHA-1/SHA-2 with Argon2id** immediately.
5. **Diversify** with SHA-3 for at least one high-stakes use (e.g., code signing) so a future SHA-2 break doesn't break the entire estate.
6. **Adopt SHAKE256** for any extensible-output use (e.g., X.509 LTV hash trees, large file integrity).
7. **PQC**: pair SHA-256/SHA-512 with ML-DSA and SLH-DSA; SHA-3 with SPHINCS+ and SHAKE256-based schemes.

**Hash inventory and rotation program**:
- Scan the codebase quarterly (semgrep rules: `crypto/md5`, `crypto/sha1`).
- Scan dependency lockfiles for transitive crypto use.
- Run `testssl.sh` / `sslyze` against every public endpoint; alert on any SHA-1 in the chain.
- Code-signing pipeline: replace SHA-1 with SHA-256 *or* migrate to Ed25519 (which uses SHA-512 internally, then move to Ed448+SHA-3 in 2026+).

**Board-level callout**:
- The 2017 SHAttered collision cost ~$110k to compute. Today, well-funded attackers can do it in days. A *signed* document that uses SHA-1 in 2025 is, technically, forgeable for any adversary willing to spend the money.
- For *digital signatures* with a 10+ year integrity life, hash function is *not* the binding constraint — the math of the signature scheme is. SLH-DSA (SPHINCS+) gives 128-bit collision resistance from SHA-256 even after a structural break, because the security proof reduces to second-pre-image resistance of the underlying hash.

## Related Connections
- Crypto fundamentals: [[cryptography-fundamentals-and-math]]
- Symmetric algorithms: [[symmetric-encryption-algorithms]]
- Asymmetric and PKI: [[asymmetric-encryption-and-pki]]
- TLS 1.3: [[tls-1-3-deep-dive]]
- Post-quantum: [[post-quantum-cryptography]]
- Domain hub: [[domain-03-security-architecture-and-engineering]]

## References
- NIST FIPS 180-4 (2012, 2025 update). *Secure Hash Standard (SHS)*.
- NIST FIPS 202 (2015). *SHA-3 Standard: Permutation-Based Hash and Extendable-Output Functions*.
- NIST SP 800-107 Rev. 1 (2012). *Recommendation for Applications Using Approved Hash Algorithms*.
- NIST SP 800-185 (2016). *SHA-3 Derived Functions: cSHAKE, KMAC, TupleHash, and ParallelHash*.
- NIST SP 800-208 (2020). *Recommendation for Stateful Hash-Based Signature Schemes* (XMSS, LMS).
- RFC 2104 (1997). *HMAC: Keyed-Hashing for Message Authentication*.
- RFC 4231 (2005). *Identifiers and Test Vectors for HMAC-SHA-256, HMAC-SHA-384, HMAC-SHA-512*.
- RFC 8439 (2018). *ChaCha20 and Poly1305 for IETF Protocols*.
- Stevens, M. et al. (2017). *SHAttered* — first practical SHA-1 collision.
- Leurent, G.; Peyrin, T. (2020). *SHA-1 is a Shambles* — chosen-prefix collision.
- Bertoni, G.; Daemen, J.; Peeters, M.; Van Assche, G. (2011). *The Keccak SHA-3 Submission*.
- Bernstein, D. J. (2005). *The Poly1305-AES Message-Authentication Code*.
- Wang, X.; Yin, Y. L.; Yu, H. (2005). *Finding Collisions in the Full SHA-1* (theoretical break).
- Kelsey, J.; Schneier, B. (2005). *Second Preimages on n-bit Hash Functions for Much Less than 2^n Work*.
- NIST SP 800-131A Rev. 2 (2023). *Transitioning the Use of Cryptographic Algorithms and Key Lengths*.
