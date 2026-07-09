---
title: Symmetric Encryption Algorithms and Modes
layer: 2
domain: 3
tags: [cissp, symmetric, des, 3des, aes, blowfish, twofish, ecb, cbc, ctr, gcm, ccm, block-cipher, mode-of-operation, aead]
related_domains: [2, 4, 7]
---

# Symmetric Encryption Algorithms and Modes

## Feynman Explanation
A symmetric cipher is a *scrambling machine* that takes a secret key and a plaintext block and spits out ciphertext, in a way that someone without the key cannot unscramble it even with a billion years and a million computers. Different machines have different designs (DES, AES, ChaCha20) and different speed/safety tradeoffs, but the most important decision is the **mode** — how you glue the machine together to encrypt more than one block, because *the mode is what turns a strong primitive into a broken or secure system*. ECB mode with AES is a famous foot-gun: every identical plaintext block produces the same ciphertext block, leaking the entire "Where's Waldo?" picture as visible silhouettes.

## Technical Details

### 1. Block Cipher Primitives

A block cipher is a family of permutations indexed by a key:

$$
E: \mathcal{K} \times \{0,1\}^n \to \{0,1\}^n, \qquad D = E^{-1}
$$

Block size $n$ is fixed (64 bits for DES, 128 bits for AES). Key size $k$ is independent.

### 2. DES (Data Encryption Standard) — DEPRECATED

#### 2.1 Structure
- Block: 64 bits; key: 56 bits (parity-stripped from 64-bit input).
- 16-round Feistel network.
- Round function uses 8 S-boxes (the only non-linear component).

$$
L_{i+1} = R_i, \qquad R_{i+1} = L_i \oplus f(R_i, K_i)
$$

Final round swaps halves (canceled by the identical swap in decryption).

#### 2.2 Why DES is Dead
- Key size 56 bits → exhaustive search $2^{56} \approx 7.2 \times 10^{16}$ keys.
- EFF "Deep Crack" (1998) cracked DES in 56 hours for $250k.
- NIST formally withdrew DES in 2005; FIPS 46-3 rescinded.

#### 2.3 3DES (Triple DES, TDEA)
Apply DES three times with two or three keys:

$$
C = E_{K_1}(D_{K_2}(E_{K_3}(P))) \quad \text{(EDE — Encrypt-Decrypt-Encrypt)}
$$

The middle $D$ is a compatibility hack so 3DES with $K_1 = K_2 = K_3$ is single DES.

- **2-key 3DES**: $K_1 = K_3$, 112 effective bits; vulnerable to *meet-in-the-middle* → effective security $\approx 2^{80}$ (2TDEA).
- **3-key 3DES**: 168 nominal bits, effective $\approx 2^{112}$ (3TDEA).
- Both **disallowed after 2023** (NIST SP 800-131A Rev. 2). The Sweet32 attack on 3DES+TLS demonstrated practical risk.

### 3. AES (Advanced Encryption Standard) — The Default

Adopted 2001 (FIPS 197) after a 5-year open competition (Rijndael by Daemen & Rijmen).

#### 3.1 Variants

| Variant | Key size | Block size | Rounds |
|---------|----------|------------|--------|
| AES-128 | 128 bits | 128 bits | 10 |
| AES-192 | 192 bits | 128 bits | 12 |
| AES-256 | 256 bits | 128 bits | 14 |

#### 3.2 Round Operations (4 steps per round)
1. **SubBytes** — non-linear byte substitution via fixed S-box (composite over $\mathrm{GF}(2^8)$).
2. **ShiftRows** — rotate each row of the $4 \times 4$ state matrix left by $0,1,2,3$ bytes.
3. **MixColumns** — multiply each column by the circulant MDS matrix over $\mathrm{GF}(2^8)$.
4. **AddRoundKey** — XOR state with the round key derived from the key schedule.

Final round omits MixColumns. See [[cryptography-fundamentals-and-math]] for the $\mathrm{GF}(2^8)$ details.

#### 3.3 AES Security
- **Best known attack** (2024) on AES-128: ~$2^{126}$ operations (biclique). Negligible margin below exhaustive.
- AES-192, AES-256 have no known attacks better than exhaustive search.
- **Side-channel attacks** on AES: cache-timing (TA-DA, CacheBleed), power analysis (DPA on smart cards). Mitigations: AES-NI hardware instructions, constant-time software, masking.

#### 3.4 AES Key Schedule Strength
- AES-256 has a related-key distinguisher (Biryukov–Khovratovich, 2009) requiring $2^{99}$ work — does not break the cipher for chosen-key attacks but is a structural concern.
- **Mitigation**: use AES-256-GCM with random per-message keys; do not use key-derivation chains on AES-256 with strong related-key properties (a reason some prefer AES-128 in pure-CBC contexts).

### 4. Other Modern Symmetric Ciphers

| Cipher | Block | Key | Structure | Notes |
|--------|-------|-----|-----------|-------|
| **Blowfish** (1993) | 64 | 32–448 | Feistel | Legacy; vulnerable to Sweet32; 64-bit block. Deprecated. |
| **Twofish** (1998) | 128 | 128/192/256 | Feistel + MDS | AES finalist; still considered strong; not widely used (AES won). |
| **3DES** | 64 | 112/168 | Feistel × 3 | Deprecated; slow. |
| **IDEA** | 64 | 128 | Lai-Massey | Legacy PGP; not in modern standards. |
| **CAST5/CAST-256** | 64/128 | up to 256 | Feistel | Used in PGP / S/MIME. |
| **Serpent** | 128 | 128/192/256 | SP-network | AES finalist; very conservative design. |
| **ChaCha20** | (stream) | 256 | ARX | Modern; high software performance, no cache-timing. |
| **Camellia** | 128 | 128/192/256 | Feistel | Used in TLS, IPsec; strong. |

**Rule of thumb (2025)**: use **AES-128-GCM** or **AES-256-GCM** in hardware, **ChaCha20-Poly1305** in software.

### 5. Stream Ciphers (RC4, ChaCha20)

#### 5.1 RC4 — BROKEN
RC4 (1987) generates a pseudo-random byte stream XORed with plaintext. Practical attacks on RC4+TLS (Bar Mitzvah, NOMORE) recovered plaintext from cookie values. Banned by RFC 7465 (2015). Never use.

#### 5.2 ChaCha20
A 256-bit-key, 96-bit-nonce, 32-bit-counter ARX stream cipher by Bernstein (2008). State: sixteen 32-bit words; quarter-round:

$$
\begin{aligned}
a &\mathrel{+}= b; \ d \oplus= a; \ d \lll 16 \\
c &\mathrel{+}= d; \ b \oplus= c; \ b \lll 12 \\
a &\mathrel{+}= b; \ d \oplus= a; \ d \lll 8 \\
c &\mathrel{+}= d; \ b \oplus= c; \ b \lll 7
\end{aligned}
$$

Twenty rounds (80 quarter-rounds) produce the keystream block. Used in TLS 1.3 (mandatory cipher), WireGuard, IPsec/IKEv2.

### 6. Modes of Operation

A mode of operation chains block cipher calls to encrypt messages longer than one block. **The mode determines whether your encryption is actually secure.**

#### 6.1 ECB (Electronic Codebook) — INSECURE

Each block encrypted independently:

$$
C_i = E_K(P_i)
$$

Same plaintext block → same ciphertext block. Reveals patterns. Famous failure: ECB-encrypted "Tux the Penguin" still recognizable. Never use ECB for messages longer than one block.

#### 6.2 CBC (Cipher Block Chaining)

$$
C_i = E_K(P_i \oplus C_{i-1}), \qquad C_0 = \text{IV}
$$

- Requires a **unique, unpredictable IV** for every encryption.
- Vulnerable to **padding oracle** attacks (Vaudenay, 2002) — if the decryptor leaks "valid padding" vs "invalid padding", an attacker recovers plaintext one byte at a time. (POODLE, Lucky 13, Bleichenbacher.)
- **Sequential** — cannot be parallelized on encryption; decryption parallelizable.
- No integrity — must be paired with HMAC or replaced by AEAD.

#### 6.3 CTR (Counter)

$$
C_i = E_K(N \,\|\, i) \oplus P_i
$$

$N$ is a nonce, $i$ is the counter (typically 64 bits). Turns a block cipher into a stream cipher. **No padding required.** Fully parallelizable. **No integrity** — must be paired with HMAC or replaced by AEAD.

Risk: nonce reuse → keystream reuse → XOR of two plaintexts recovered. (Heartbleed was not a nonce-reuse bug, but many CTR implementations fail in this way.)

#### 6.4 GCM (Galois/Counter Mode) — THE MODERN DEFAULT

GCM = CTR encryption + **GHASH** universal authentication. AEAD: *Authenticated Encryption with Associated Data*.

$$
\begin{aligned}
\text{Encryption:} \quad C_i &= P_i \oplus E_K(N \,\|\, i) \\
\text{Tag:} \quad T &= \text{GHASH}_H(A, C) \oplus E_K(N \,\|\, 0)
\end{aligned}
$$

where $H = E_K(0^{128})$, and GHASH is a polynomial evaluation in $\mathrm{GF}(2^{128})$:

$$
\text{GHASH}_H(A, C) = A_1 H^{m+n+1} \oplus C_1 H^{m+n} \oplus \cdots \oplus C_n^* H^2 \oplus A_1 H \oplus H
$$

(For TLS 1.3, GCM is a 16-byte tag; for IPsec, often 8–12 bytes.)

**Critical rule**: a 96-bit nonce must NEVER be reused with the same key — it leaks the authentication key $H$. NIST SP 800-38D mandates random nonces or a counter that cannot repeat.

- **Performance**: hardware AES-NI + CLMUL enables ~10 Gbps per core; software-only falls back to ChaCha20-Poly1305.

#### 6.5 CCM (Counter with CBC-MAC)

Used in 802.11i (WPA2), Bluetooth LE, Thread. AEAD combining CTR + CBC-MAC. Slower than GCM, more nonce-sensitive.

#### 6.6 Other Modes
- **XTS** — disk encryption (IEEE 1619). XEX-based tweaked codebook. No integrity; use with Merkle-tree integrity layer.
- **OCB** — offset codebook; fast AEAD; patent encumbered until 2021.
- **SIV** — synthetic IV; deterministic, nonce-misuse resistant. Used in RFC 5297.
- **Poly1305** — Wegman–Carter authenticator paired with ChaCha20.

#### 6.7 Mode Selection Decision Tree

| Need | Mode |
|------|------|
| TLS 1.3 | AES-GCM, AES-CCM, ChaCha20-Poly1305 |
| IPsec (ESP) | AES-GCM, ChaCha20-Poly1305 |
| Disk / file encryption | XTS (data) + HMAC-SHA-256 (metadata) or AEAD (Adiantum for low-power) |
| WireGuard | ChaCha20-Poly1305 |
| SSH (chacha20-poly1305@openssh.com) | ChaCha20-Poly1305 |
| Authenticated local data (no nonce) | SIV |

### 7. Padding

Block modes (CBC, ECB) require padding. **PKCS#7** is the standard:

For a block size $b$ and $r$ bytes of padding needed ($1 \le r \le b$):

$$
\text{pad} = \text{byte}(r) \text{ repeated } r \text{ times}
$$

A full padding block ($r = b$) is appended when the plaintext is already aligned — this disambiguates. Padding oracle attacks exploit the unpadding step: do not use CBC mode in new code.

### 8. Initialization Vector (IV) and Nonce Rules

| Mode | IV / Nonce requirement | Reuse consequence |
|------|------------------------|---------------------|
| ECB | n/a | n/a |
| CBC | Random, $b$-bit, never reused with same key | Catastrophic (predictable plaintext) |
| CTR | Nonce $\ne$ per message, never reused | Catastrophic (XOR-of-plaintext) |
| GCM | 96-bit nonce, never reused with same key | Catastrophic (auth key $H$ leaked) |
| XTS | Two tweaks (sector + block index) | Sector-level pattern leak |

### 9. Performance Comparison (Cycles per byte, Haswell, single core, ~2014)

| Cipher | Mode | Throughput |
|--------|------|------------|
| AES-128 | GCM | ~5.0 cpb (with AES-NI + CLMUL) |
| AES-256 | GCM | ~5.6 cpb |
| ChaCha20-Poly1305 | AEAD | ~3.5 cpb on Haswell |
| AES-128 | CBC + HMAC | ~8 cpb |
| 3DES | CBC | ~100 cpb (not a typo) |

ChaCha20 is faster on ARM (no AES-NI on most phones pre-2018); AES-GCM is faster on x86 with hardware acceleration.

## CISO / Risk Manager View

**Board framing**: "Are we using a 50-year-old cipher because the developer copied it from Stack Overflow in 2009?" Inventory the answer. Most organizations have hundreds of MD5, SHA-1, DES, RC4, RSA-1024, and `AES-ECB` calls hidden in legacy code.

**Cryptographic deprecation roadmap (CISO mandate)**:
| Year | Action |
|------|--------|
| 2005 | DES off |
| 2013 | RC4 off (TLS) |
| 2017 | SHA-1 off (certificates) |
| 2023 | 3DES off (TLS, IPsec) |
| 2024 | RSA-2048 with PKCS#1 v1.5 off (use PSS) |
| 2024 | Static DH off (use ECDHE) |
| 2025 | AES-CBC off in new code (use GCM) |
| 2030 | RSA off (replaced by ML-DSA / Ed25519) |
| 2030 | ECC off (replaced by ML-KEM) |

**Crypto-agility mandate**: every code path that touches a cipher must accept the algorithm as a parameter. Vendors that ship hard-coded `AES_encrypt` calls have already failed their next audit.

**Performance and cost**: ChaCha20 is mandatory on mobile (battery); AES-NI is mandatory on servers (throughput). Cloud data-egress costs dwarf CPU costs — favor hardware AES on every server.

**Single most important control**: forbid any new system that uses CBC, ECB, CTR without authentication, or static-key DH. Mandate AEAD (GCM, ChaCha20-Poly1305) for everything new.

**Board question to prepare for**: "If a CRQC is announced tomorrow, what is our cipher inventory and what is the one-week migration playbook?" The answer is *the* quantum-readiness test. See [[post-quantum-cryptography]].

## Related Connections
- Crypto fundamentals: [[cryptography-fundamentals-and-math]]
- Asymmetric and PKI: [[asymmetric-encryption-and-pki]]
- TLS 1.3: [[tls-1-3-deep-dive]]
- Hashes & HMAC: [[hash-functions-and-hmac]]
- FIPS 140-3: [[fips-140-3]]
- Domain hub: [[domain-03-security-architecture-and-engineering]]

## References
- NIST FIPS 197 — *Advanced Encryption Standard* (AES).
- NIST SP 800-38A — *Block Cipher Modes of Operation* (ECB, CBC, CFB, OFB, CTR).
- NIST SP 800-38D — *Galois/Counter Mode (GCM) and GMAC*.
- NIST SP 800-38E — *XTS-AES Mode for Block-Oriented Storage Devices*.
- NIST SP 800-131A Rev. 2 — *Transitioning the Use of Cryptographic Algorithms and Key Lengths*.
- RFC 7465 — *Prohibiting RC4 Cipher Suites* (2015).
- RFC 8439 — *ChaCha20 and Poly1305 for IETF Protocols*.
- Bernstein, D. J. (2008). *ChaCha, a variant of Salsa20*.
- Daemen, J.; Rijmen, V. (2002). *The Design of Rijndael*.
- Biryukov, A.; Khovratovich, D. (2009). *Related-key cryptanalysis of the full AES-192 and AES-256*.
- Joux, A. (2009). *How many GB do you need to break AES-128?* (biclique attack).
