---
title: Cryptography Fundamentals and Math
layer: 2
domain: 3
tags: [cissp, cryptography, aes, rsa, ecc, work-factor, key-size, modular-arithmetic, one-time-pad, perfect-secrecy]
related_domains: [2, 4, 5, 6, 7]
---

# Cryptography Fundamentals and Math

## Feynman Explanation
Cryptography is just two functions — encrypt $C = E_k(P)$ and decrypt $P = D_k(C)$ — plus the science of making sure an attacker with the message and the algorithm but not the key cannot reverse them. The math underneath is two questions: *how much* work is the attacker forced to do (the **work factor**), and is that work exponential in the key length (good) or polynomial (broken)? Every other note in this domain — AES, RSA, TLS, PQC, TPM — is just a specific way of answering "yes, exponential."

## Technical Details

### 1. The Core Quadruple

For every cryptosystem, four sets are defined and three mappings:

$$
(\mathcal{P},\; \mathcal{C},\; \mathcal{K},\; \mathcal{E},\; \mathcal{D})
$$

| Set | Meaning |
|-----|---------|
| $\mathcal{P}$ | plaintext space |
| $\mathcal{C}$ | ciphertext space |
| $\mathcal{K}$ | key space |
| $\mathcal{E}: \mathcal{K} \times \mathcal{P} \to \mathcal{C}$ | encryption function |
| $\mathcal{D}: \mathcal{K} \times \mathcal{C} \to \mathcal{P}$ | decryption function |

The fundamental correctness condition:

$$
\forall\, k \in \mathcal{K},\, \forall\, p \in \mathcal{P}: \quad D_k(E_k(p)) = p
$$

### 2. Symmetric vs Asymmetric

| Property | Symmetric (private key) | Asymmetric (public key) |
|----------|-------------------------|--------------------------|
| Keys | One shared secret $K$ | Public $K_{\text{pub}}$, private $K_{\text{prv}}$ |
| Speed | ~1,000× faster | ~1,000× slower |
| Key establishment | Out-of-band / KDF / DH | Public directory / PKI |
| Confidentiality | $C = E_K(P)$ | $C = E_{K_{\text{pub}}}(P)$ |
| Confidentiality (priv) | n/a | $P = D_{K_{\text{prv}}}(C)$ |
| Signature | MAC: $T = \text{MAC}_K(M)$ | Sig: $\sigma = D_{K_{\text{prv}}}(H(M))$ |
| Verify | Same MAC: $T \stackrel{?}{=} \text{MAC}_K(M)$ | Verify: $H(M) \stackrel{?}{=} E_{K_{\text{pub}}}(\sigma)$ |
| Key length → security | $2^k$ work factor | Sub-exponential (RSA) / $2^{k/2}$ (ECC) |
| Example primitives | AES-256, ChaCha20 | RSA-3072, X25519, Ed25519 |
| Forward secrecy | Possible (with DH) | Requires ephemeral key exchange |

### 3. Work Factor and Key Size

#### 3.1 Work Factor Definition

The work factor $W(n)$ is the number of elementary operations an attacker must perform to break the system, expressed as a function of the cryptographic parameter $n$ (key length, modulus length, group order).

For a $k$-bit symmetric key, the generic attack is exhaustive key search:

$$
W_{\text{sym}}(k) = 2^{k-1} \quad \text{on average}
$$

(One trial per $2^{k-1}$ keys on average, $2^k$ in the worst case.)

#### 3.2 Symmetric vs Asymmetric Key Equivalence (NIST SP 800-57 Part 1 Rev. 5)

| Security strength (bits) | Symmetric | RSA / DH | ECC |
|--------------------------|-----------|----------|-----|
| 112 | 3DES (2-key) | RSA-2048, DH-2048 | 224-bit curve |
| 128 | AES-128 | RSA-3072, DH-3072 | 256-bit curve (P-256, X25519) |
| 192 | AES-192 | RSA-7680, DH-7680 | 384-bit curve (P-384) |
| 256 | AES-256 | RSA-15360, DH-15360 | 512-bit curve (P-521, Ed448) |

#### 3.3 Sub-exponential Attacks Against RSA / DH

The General Number Field Sieve (GNFS) factors an RSA modulus $n$ in time:

$$
L_n\left[\tfrac{1}{3},\, c\right] = \exp\!\left((c + o(1))\, (\ln n)^{1/3} (\ln \ln n)^{2/3}\right)
$$

For 2048-bit RSA, this is approximately $2^{112}$ operations — *much* smaller than the $2^{2048}$ naive work factor. Hence the asymmetric key sizes in the table above.

For ECC, only generic group attacks apply, giving $2^{k/2}$ work (Pollard-rho) — this is why a 256-bit ECC key matches 128-bit symmetric security.

#### 3.4 Quantum Impact (Shor's Algorithm)

Shor's algorithm factors $n$ and computes discrete logs in polynomial time:

$$
T_{\text{Shor}}(n) = O\!\left((\log n)^3\right) = \text{polynomial}
$$

This collapses RSA, DH, and ECC to *broken* the moment a cryptographically relevant quantum computer (CRQC) exists. See [[post-quantum-cryptography]].

Grover's algorithm gives a quadratic speed-up on symmetric search:

$$
T_{\text{Grover}}(k) = O\!\left(2^{k/2}\right)
$$

Mitigation: double the symmetric key size (AES-128 → AES-256).

### 4. Symmetric Cryptography: AES

#### 4.1 AES-128 Round Structure

AES-128 operates on a 16-byte (128-bit) state arranged in a $4 \times 4$ matrix, with 10 rounds:

$$
\text{State}_{i+1} = \text{MixColumns}(\text{ShiftRows}(\text{SubBytes}(\text{State}_i \oplus K_i)))
$$

The final round omits MixColumns. SubBytes is a non-linear S-box (the only non-linear step), ShiftRows is a permutation, MixColumns is a linear diffusion step, AddRoundKey is XOR with the round key $K_i$.

#### 4.2 AES Galois Field Math

SubBytes, MixColumns, and the inverse cipher operate in $\mathrm{GF}(2^8) = \mathbb{F}_{2^8}$ defined by the irreducible polynomial:

$$
p(x) = x^8 + x^4 + x^3 + x + 1 \quad (\text{0x11B})
$$

Multiplication is polynomial multiplication modulo $p(x)$:

$$
a \otimes b \;=\; a(x) \cdot b(x) \bmod p(x)
$$

MixColumns multiplies each 4-byte column by the circulant matrix over $\mathrm{GF}(2^8)$:

$$
M = \begin{pmatrix} 02 & 03 & 01 & 01 \\ 01 & 02 & 03 & 01 \\ 01 & 01 & 02 & 03 \\ 03 & 01 & 01 & 02 \end{pmatrix}
$$

The constants $02$, $03$ are the polynomials $x$, $x+1$ in $\mathrm{GF}(2^8)$; multiplication by $02$ is the xtime operation, and multiplication by $03$ is $02 \oplus 01$.

#### 4.3 AES Key Schedule

The 128-bit master key is expanded to 11 round keys of 16 bytes each (44 32-bit words $W[0..43]$):

$$
\begin{aligned}
W[i] &= W[i-1] \oplus W[i-4] \quad \text{if } i \not\equiv 0 \pmod{N_k} \\
W[i] &= \text{SubWord}(\text{RotWord}(W[i-1])) \oplus \text{Rcon}[i/N_k] \oplus W[i-4] \quad \text{if } i \equiv 0 \pmod{N_k}
\end{aligned}
$$

where $N_k = 4$ for AES-128. Rcon are round constants derived from $x^{i-1}$ in $\mathrm{GF}(2^8)$.

### 5. Asymmetric Cryptography: RSA

#### 5.1 Key Generation

1. Pick two large distinct primes $p$, $q$ of equivalent bit-length (e.g., 1024 bits each for RSA-2048).
2. Compute the modulus and Euler's totient:
$$
n = p \cdot q, \qquad \phi(n) = (p-1)(q-1)
$$
3. Choose the public exponent $e$ coprime to $\phi(n)$ (typical: $e = 65537 = 2^{16}+1$).
4. Compute the private exponent:
$$
d \equiv e^{-1} \pmod{\phi(n)} \quad \text{via extended Euclidean}
$$

#### 5.2 Encryption / Decryption

$$
C = M^e \bmod n, \qquad M = C^d \bmod n
$$

The proof of correctness uses Euler's theorem:

$$
C^d = (M^e)^d = M^{ed} = M^{1 + k\phi(n)} = M \cdot (M^{\phi(n)})^k \equiv M \pmod{n}
$$

since $\gcd(M, n) = 1$ for $M < n$ and $p, q$ not dividing $M$.

#### 5.3 RSA Signature (RSASSA-PSS, PKCS#1 v2)

Signature generation:

$$
\sigma = (H(M)\,\|\,\text{pad})^{d} \bmod n
$$

Verification:

$$
H(M) \stackrel{?}{=} \sigma^{e} \bmod n
$$

Modern padding (PSS — Probabilistic Signature Scheme) is required; textbook RSA without padding is *malleable* and vulnerable to existential forgery (Davies' attack).

#### 5.4 RSA Security Strengths

| Modulus | Symmetric equivalent | Status (2025) |
|---------|----------------------|---------------|
| 1024 | 80 | **Broken** (GNFS record 2009) |
| 2048 | 112 | Acceptable legacy; not future-proof |
| 3072 | 128 | Recommended today |
| 7680 | 192 | High-value long-term |
| 15360 | 256 | Maximum classical |

### 6. Elliptic Curve Cryptography (ECC)

#### 6.1 The Curve

Over a prime field $\mathbb{F}_p$ (or binary field $\mathbb{F}_{2^m}$), the Weierstrass curve:

$$
E: y^2 \equiv x^3 + ax + b \pmod{p}, \quad 4a^3 + 27b^2 \not\equiv 0 \pmod{p}
$$

The set of points $(x, y) \in \mathbb{F}_p^2$ satisfying $E$ together with the *point at infinity* $\mathcal{O}$ form an abelian group under the chord-and-tangent rule.

#### 6.2 Point Addition

Given $P = (x_1, y_1)$ and $Q = (x_2, y_2)$, $P \neq Q$:

$$
\lambda = (y_2 - y_1)(x_2 - x_1)^{-1} \bmod p
$$

$$
R = P + Q = (\lambda^2 - x_1 - x_2 \bmod p,\; \lambda(x_1 - x_R) - y_1 \bmod p)
$$

For $P = Q$ (doubling):

$$
\lambda = (3x_1^2 + a)(2y_1)^{-1} \bmod p
$$

The identity is $\mathcal{O}$, with $P + \mathcal{O} = P$ and the inverse $-P = (x, -y \bmod p)$.

#### 6.3 Scalar Multiplication and ECDLP

For integer $k$, the *scalar multiplication* $[k]P$ is defined by repeated doubling-and-adding:

$$
[k]P = \underbrace{P + P + \cdots + P}_{k \text{ times}}
$$

The **Elliptic Curve Discrete Logarithm Problem (ECDLP)** is the assumed-hard problem:

> Given $P$ and $Q = [k]P$, find $k$.

Best known classical attack: Pollard-rho, $O(\sqrt{n})$ group operations. This is why a 256-bit curve (group order $\approx 2^{256}$) gives $\approx 2^{128}$ security.

#### 6.4 Standardized Curves

| Curve | Field | Group order | Security | Use |
|-------|-------|-------------|----------|-----|
| P-256 (secp256r1) | 256-bit prime | $\approx 2^{256}$ | 128 | TLS, JWT (legacy) |
| P-384 (secp384r1) | 384-bit prime | $\approx 2^{384}$ | 192 | High-value TLS |
| P-521 (secp521r1) | 521-bit prime | $\approx 2^{521}$ | 256 | Top-secret TLS |
| Curve25519 (X25519) | Montgomery | $\approx 2^{252}$ | 128 | TLS 1.3 default, SSH, WireGuard |
| Ed25519 | Edwards | $\approx 2^{252}$ | 128 | Signatures (fast, deterministic) |
| Ed448 | Edwards | $\approx 2^{446}$ | 222 | High-value signatures |

### 7. Diffie–Hellman (DH) Key Exchange

#### 7.1 Classical DH

Public parameters: prime $p$, generator $g$ of $\mathbb{Z}_p^*$. Alice picks $a$, computes $A = g^a \bmod p$; Bob picks $b$, computes $B = g^b \bmod p$.

Shared secret:

$$
K_{AB} = B^a \bmod p = A^b \bmod p = g^{ab} \bmod p
$$

The Discrete Logarithm Problem (DLP) makes this hard to invert.

#### 7.2 ECDH (the modern variant)

$$
K_{AB} = [a]\, B = [b]\, A = [ab]\, P
$$

ECDH is approximately 10× faster at equivalent security; the active default in TLS 1.3 is **X25519** and **P-256/P-384**.

### 8. Information-Theoretic Security: One-Time Pad (OTP)

#### 8.1 Definition

Encryption and decryption:

$$
C_i = P_i \oplus K_i, \qquad P_i = C_i \oplus K_i
$$

with $K_i$ drawn uniformly at random from $\{0,1\}^n$, with $\gcd$ constraints and $K$ used exactly once.

#### 8.2 Shannon's Theorem (Perfect Secrecy)

Shannon (1949): a cipher has *perfect secrecy* iff for every $p \in \mathcal{P}$ and every $c \in \mathcal{C}$:

$$
\Pr[P = p \mid C = c] = \Pr[P = p]
$$

The OTP satisfies this:

$$
\Pr[C = c \mid P = p] = \Pr[K = p \oplus c] = 2^{-n}
$$

independent of $p$. Consequence: a one-time-pad ciphertext leaks *zero* information about the plaintext.

#### 8.3 Drawbacks
- Key must be as long as the message.
- Key must be perfectly random (true randomness; not pseudorandom).
- Key must never be reused.
- Key distribution is an unsolved logistics problem in general (solved for backhaul links via QKD, hybridized with PQC for key establishment).

### 9. Entropy, Randomness, and PRNGs

- **Entropy** $H(X) = -\sum_i p_i \log_2 p_i$ bits — measure of uncertainty.
- **True random**: based on physical noise (Intel RDRAND, ring oscillators).
- **CSPRNG** (cryptographically secure pseudorandom number generator): deterministic, but output unpredictable without the seed. Examples: ChaCha20-based, AES-CTR-DRBG, Hash_DRBG.
- **NIST SP 800-90A**: specifies DRBG algorithms.

**Key derivation function (KDF)**: turns a high-entropy secret into one or more keys:

$$
K_1, K_2, \ldots \gets \text{KDF}(S, \text{salt}, \text{info}, L)
$$

HKDF-Expand (used in TLS 1.3):

$$
\text{OKM} = T_1 \,\|\, T_2 \,\|\, \cdots \,\|\, T_L
$$

$$
T_i = \text{HMAC}(\text{PRK}, T_{i-1} \,\|\, \text{info} \,\|\, i)
$$

where $\text{PRK} = \text{HMAC}(\text{salt}, IKM)$ and $I = 1, 2, \ldots$ indexed output blocks.

### 10. Choosing Algorithms: A Quick Decision Tree

1. **Confidentiality of bulk data at rest or in transit** → AES-256-GCM (or ChaCha20-Poly1305 in software-only).
2. **Key exchange** → X25519 / P-384; quantum-migrate by hybridizing with ML-KEM-768.
3. **Identity signatures** → Ed25519 (modern) or RSA-3072+PSS (legacy interop).
4. **Long-term document signatures (10+ years)** → ML-DSA-65 (Dilithium) or SLH-DSA (SPHINCS+).
5. **Symmetric data authentication** → HMAC-SHA-256 or AES-GMAC.
6. **Password storage** → Argon2id (memory-hard KDF) with high time/memory cost.
7. **Hashing only** → SHA-256 (or SHA-3-256 for variety); SHA-1 / MD5 for legacy-only.

## CISO / Risk Manager View

**Board framing**: every number in this note corresponds to a *defensible budget line*. AES-256 vs AES-128 doubles key-management storage and is a one-time cost. Doubling RSA from 2048 to 4096 is a 6× latency hit on TLS handshakes and a recurring cloud cost. Choosing AES-GCM vs AES-CBC has measurable performance and side-channel implications.

**Strategic posture (the CISO's three-decade roadmap)**:
1. **Inventory** all cryptographic assets: which algorithms, which key sizes, which libraries, which applications.
2. **Refuse the broken**: deprecate MD5, SHA-1, DES, 3DES, RC4, RSA-1024, RSA-2048 with PKCS#1 v1.5 padding, static DH, ECB mode.
3. **Mandate the strong**: AES-256-GCM, RSA-3072+PSS, P-256/X25519+Ed25519, HMAC-SHA-256, TLS 1.3 only.
4. **Plan for PQC**: hybridize within 24–36 months; full migration within 5–7 years (CNSA 2.0 timeline).

**Crypto-agility** is the single most important architectural property for survival. If your code calls `RSA_encrypt()` directly, you cannot migrate. If it calls `crypto.sign(alg="RSA-PSS-3072", key=...)`, you can.

**Work factor interpretation for the board**:
- $2^{80}$ = "stop using this yesterday" (for symmetric; ~1 year on commodity hardware pre-2010)
- $2^{112}$ = "replace on next refresh"
- $2^{128}$ = "current baseline for 5–10 years of confidentiality life"
- $2^{192}$ = "top-secret / 25-year confidentiality"
- $2^{256}$ = "future-proof through PQC migration"

**Single biggest board risk** — *harvest-now-decrypt-later*: adversaries are capturing ciphertexts today to decrypt once a CRQC exists. Any data with a confidentiality life extending past the PQC migration window must use *post-quantum-confidential* channels now. See [[post-quantum-cryptography]].

## Related Connections
- Symmetric algorithms: [[symmetric-encryption-algorithms]]
- Asymmetric and PKI: [[asymmetric-encryption-and-pki]]
- Hashes & HMAC: [[hash-functions-and-hmac]]
- TLS 1.3: [[tls-1-3-deep-dive]]
- Post-quantum: [[post-quantum-cryptography]]
- FIPS 140-3: [[fips-140-3]]
- Domain hub: [[domain-03-security-architecture-and-engineering]]

## References
- NIST FIPS 197 — *Advanced Encryption Standard* (AES), 2001.
- NIST FIPS 186-5 — *Digital Signature Standard* (DSS), 2023.
- NIST SP 800-57 Part 1 Rev. 5 — *Recommendation for Key Management*, 2020.
- NIST SP 800-90A — *Recommendation for Random Number Generation Using DRBGs*.
- NIST SP 800-56A Rev. 3 — *Recommendation for Pair-Wise Key Establishment Using DH*.
- Stallings, W. (2022). *Cryptography and Network Security*, 8th ed.
- Shannon, C. E. (1949). *Communication Theory of Secrecy Systems*. Bell Sys. Tech. J.
- Menezes, A.; van Oorschot, P.; Vanstone, S. (1996). *Handbook of Applied Cryptography*.
- Bernstein, D. J.; Lange, T. (2017). *SafeCurves: choosing safe curves for ECC*.
- NSA CNSA 2.0 — *Commercial National Security Algorithm Suite 2.0*, 2022.
