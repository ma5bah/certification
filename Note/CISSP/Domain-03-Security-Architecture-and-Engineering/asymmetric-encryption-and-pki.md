---
title: Asymmetric Encryption and PKI
layer: 2
domain: 3
tags: [cissp, asymmetric, rsa, diffie-hellman, ecc, dsa, x509, ca, ocsp, crl, pki, pqc, ml-kem, ml-dsa, kma]
related_domains: [1, 4, 5, 7, 8]
---

# Asymmetric Encryption and PKI

## Feynman Explanation
Asymmetric cryptography is the trick of *splitting* a single key into a public half anyone can see and a private half only the owner knows, so that what the public half encrypts only the private half can decrypt — like a mailbox with a slot anyone can drop letters into, but only the owner has the key. The whole internet (HTTPS, SSH, S/MIME, code signing) sits on this idea, plus a *chain of trust* — a hierarchy of digital notaries called Certificate Authorities that vouch that a public key really belongs to the server you think it does. PKI is the bureaucracy of cryptography: the math is elegant, the practice is messy, and the security is only as good as the weakest registrar.

## Technical Details

### 1. Asymmetric Primitive Catalog

| Algorithm | Math basis | Operation | NIST PQC replacement |
|-----------|-----------|-----------|----------------------|
| RSA | Integer factoring | Encrypt, sign | ML-KEM (KEM), ML-DSA (sign) |
| DH / DSA | Discrete log mod $p$ | KEM, sign | ML-KEM, ML-DSA |
| ECDH / ECDSA | ECDLP | KEM, sign | ML-KEM, ML-DSA |
| Ed25519 / Ed448 | ECDLP (twisted Edwards) | Sign (fast) | ML-DSA |
| X25519 / X448 | Montgomery ECDH | KEM | ML-KEM |
| Rabin | Squaring mod $n$ | Encrypt | (PQC) |

### 2. RSA — Re-derivation and Use

#### 2.1 Key Generation (2048-bit example)
1. Generate two 1024-bit primes $p, q$ (use Miller–Rabin primality test; rejection sampling on random odd).
2. $n = p \cdot q$, $\phi(n) = (p-1)(q-1)$.
3. Choose $e = 65537$ (a Fermat prime, fast exponentiation).
4. Compute $d \equiv e^{-1} \bmod \phi(n)$ via extended Euclidean algorithm.

#### 2.2 Encryption / Decryption
$$
C \equiv M^e \pmod{n}, \qquad M \equiv C^d \pmod{n}
$$

#### 2.3 Padding Schemes (RSA-OAEP for encryption, RSA-PSS for signing)

**PKCS#1 v1.5** is broken in practice (Bleichenbacher 1998: padding oracle against RSAES-PKCS1-v1_5). It must not be used in new code.

**RSAES-OAEP (Optimal Asymmetric Encryption Padding)** — RFC 8017:

$$
C = (M \,\|\, \text{pad}) \bmod n
$$

where $\text{pad} = \text{MGF1}(M, \text{seed})$ uses SHA-256. Indistinguishable under chosen-ciphertext attack (IND-CCA) when the hash is modeled as a random oracle.

**RSASSA-PSS** — the modern signature scheme. Random salt $r$ is hashed with the message, the signature is:

$$
\sigma = (H(M \,\|\, \text{pad}_1 \,\|\, \text{pad}_2))^d \bmod n
$$

Verifier checks $H(M \,\|\, \text{pad}_1 \,\|\, \text{pad}_2) \stackrel{?}{=} \sigma^e \bmod n$.

### 3. Diffie–Hellman (DH) and ECDH

#### 3.1 Classical DH Group
Public: safe prime $p$ such that $q = (p-1)/2$ is also prime; generator $g$ of order $q$ in $\mathbb{Z}_p^*$.

Alice: secret $a$, public $A = g^a \bmod p$. Bob: secret $b$, public $B = g^b \bmod p$.

$$
K = g^{ab} \bmod p = A^b \bmod p = B^a \bmod p
$$

#### 3.2 ECDH

ECDH on curve $E$ with base point $G$ of prime order $n$:

$$
K = [a] B = [b] A = [ab] G \in E
$$

The shared secret is the $x$-coordinate of $K$ (or both, depending on suite).

#### 3.3 Hardness Assumptions
- DH: Computational Diffie–Hellman (CDH) problem — given $g^a, g^b$, find $g^{ab}$ — assumed hard.
- DLP: given $g^a$, find $a$ — equivalent to CDH in the generic-group model.

#### 3.4 ECDH Variants in TLS 1.3
- **X25519** (RFC 7748) — Curve25519 ECDH; ~128-bit security; mandatory in TLS 1.3.
- **P-256 (secp256r1)** — NIST P-curve; ~128-bit.
- **P-384 (secp384r1)** — ~192-bit.
- **P-521** — ~256-bit.

**Avoid**: secp256k1 (Bitcoin curve) in non-blockchain contexts — its parameters have no known weakness but were not generated verifiably; P-256/X25519 are preferred.

### 4. Elliptic Curve Signatures: ECDSA and EdDSA

#### 4.1 ECDSA (FIPS 186-5)

Sign $H(M)$ (a 256-bit hash for P-256) with private key $d$:

1. Pick a *per-message* random nonce $k$ in $[1, n-1]$.
2. Compute point $R = [k] G = (x_R, y_R)$.
3. $r = x_R \bmod n$.
4. $s = k^{-1} (H(M) + d \cdot r) \bmod n$.
5. Signature is $(r, s)$.

Verify: $u_1 = s^{-1} H(M) \bmod n$, $u_2 = s^{-1} r \bmod n$, $R' = [u_1] G + [u_2] Q$, accept iff $r \equiv R'_x \bmod n$.

**Critical**: a reused or biased $k$ *leaks the private key*. The 2010 Sony PS3 ECDSA bug used a static $k$ for all signatures.

#### 4.2 EdDSA (Ed25519, Ed448) — RFC 8032

Ed25519 uses the twisted Edwards curve Curve25519 with a *deterministic* nonce derived from the private key and message:

$$
k = \text{SHA-512}(d \,\|\, M)_{[0..32)}
$$

This eliminates the $k$-reuse failure mode of ECDSA. Signing:

$$
R = [k] B, \qquad S = (r + \text{SHA-512}(R \,\|\, A \,\|\, M) \cdot s) \bmod \ell
$$

where $B$ is the base point, $A = [s] B$ is the public key, $s$ is the secret scalar, $r$ is the $x$-coordinate of $R$, and $\ell$ is the group order.

Ed25519 signatures are 64 bytes; verification is faster than ECDSA. Used in SSH, TLS 1.3 (Ed25519 is a permitted signature), JWT, GPG, Tor, Signal.

#### 4.3 Schnorr Signatures (BIP-340, Bitcoin Taproot)

Schnorr signatures are linear, enabling multi-signature and threshold signature aggregation. Bitcoin Taproot (2021) introduced BIP-340 Schnorr over secp256k1.

### 5. Digital Envelope (Hybrid Encryption)

Real systems rarely use asymmetric encryption for bulk data. They encrypt the data with AES, then encrypt the AES key with RSA/ECIES.

$$
\begin{aligned}
K_{\text{AES}} &\gets \text{CSPRNG}() \\
C_1 &= E_{K_{\text{AES}}}(M) \\
C_2 &= E_{K_{\text{pub}}}(K_{\text{AES}}) \quad \text{(RSAES-OAEP) or } C_2 = \text{ECIES}(K_{\text{AES}}) \\
\text{Output} &= (C_1, C_2)
\end{aligned}
$$

This is the basis of CMS/PKCS#7, S/MIME, PGP, and JWE (JSON Web Encryption).

### 6. Public Key Infrastructure (PKI)

#### 6.1 X.509 Certificate

An X.509 v3 certificate binds a public key to a subject (DNS name, IP, email, organization) via a digital signature. Profile (RFC 5280):

```
Certificate ::= SEQUENCE {
  tbsCertificate       TBSCertificate,
  signatureAlgorithm   AlgorithmIdentifier,
  signatureValue       BIT STRING
}

TBSCertificate ::= SEQUENCE {
  version         [0]  Version DEFAULT v1,
  serialNumber         CertificateSerialNumber,
  signature            AlgorithmIdentifier,
  issuer               Name,
  validity             Validity,         -- notBefore, notAfter
  subject              Name,
  subjectPublicKeyInfo SubjectPublicKeyInfo,
  ...
  extensions      [3]  Extensions OPTIONAL
}
```

**Critical extensions**:
- `subjectAltName` (SAN) — DNS names, IPs, URIs. Modern CA/B Forum rules *require* SAN; CN is ignored.
- `keyUsage` — `digitalSignature`, `keyEncipherment`, `keyAgreement`, `keyCertSign`, `cRLSign`.
- `extendedKeyUsage` (EKU) — `serverAuth`, `clientAuth`, `codeSigning`, `emailProtection`.
- `basicConstraints` — `CA:TRUE` and `pathLen` for CAs.
- `authorityInformationAccess` — URL to OCSP responder and CA issuer cert.
- `CRLDistributionPoints` — URL to the CRL.

#### 6.2 CA Hierarchy

A typical chain (TLS server cert):

```
Root CA (self-signed, in trust store)
  └── Intermediate CA (offline; signs intermediates and OCSP)
        └── Issuing CA (online; issues leaf certs)
              └── leaf cert (your server)
```

- **Root CA** — top of chain; embedded in OS trust stores. Kept offline.
- **Intermediate / Sub-CA** — issues leaf certs. Compromise is bounded.
- **Issuing CA** — high-volume issuance. Often fronted by an RA (Registration Authority) that performs identity vetting.

**Path validation** (RFC 5280):
1. Build chain from leaf to a trusted root.
2. Verify each certificate's signature against the parent's public key.
3. Check validity dates, key usage, EKU.
4. Check revocation (CRL or OCSP).
5. Check name constraints, policy constraints.
6. Reject if any check fails.

#### 6.3 Trust Models

| Model | Use | Description |
|-------|-----|-------------|
| Hierarchical (X.509) | Web PKI | One root chain, top-down |
| Bridge CA | Cross-org interop (US Federal PKI) | Multiple roots cross-certified through a bridge |
| Mesh / Web of Trust | PGP / GPG | No central authority; users sign each other's keys |
| Trust on First Use (TOFU) | SSH | First connection's key is trusted; changed key = warning |
| DANE / DNSSEC | DNS-bound cert pinning | TLSA records pin certs by hash in DNSSEC-signed DNS |
| Account-bound / FIDO | End-user | Public key bound to account, not certificate |

#### 6.4 Certificate Lifecycle

```
Generation → Request (CSR) → Identity Vetting (RA) → Issuance (CA signs)
       → Publication (CT logs) → Distribution (delivered to server)
       → Usage (TLS handshake) → Monitoring (CT, cert transparency monitors)
       → Renewal / Re-key → Revocation (CRL/OCSP) → Expiration → Archival
```

#### 6.5 Revocation: CRL and OCSP

**Certificate Revocation List (CRL)** — a signed, time-stamped list of revoked cert serial numbers, published by the CA. RFC 5280.

- *Pros*: one fetch reveals all revocations.
- *Cons*: list grows; latency = the next scheduled CRL; browsers historically ignored CRLs due to size and freshness issues.

**Online Certificate Status Protocol (OCSP)** — RFC 6960. A real-time query: "is serial number X still valid?" OCSP responder signs the answer.

- *Pros*: real-time, smaller response.
- *Cons*: privacy (CA learns which sites you visit); latency and availability (OCSP responder down = either fail-open or fail-close).

**OCSP Stapling** (RFC 6066) — server periodically fetches a signed OCSP response and *staples* it to the TLS handshake. Browsers accept the stapled response. Avoids the privacy and availability problem.

**Must-Staple** (RFC 7633) — the certificate's TLS feature extension requires a stapled response; failure → reject. Defends against CA-side compromise of a *non-stapled* cert.

**CRLite** (Mozilla) — a Bloom-filter-based compression of all revocations, pushed to browsers. Practical at scale.

**CRL/OCSP are still required by the CA/B Forum for server certs** even though browsers often rely on Certificate Transparency + short cert lifetimes (≤ 90 days).

#### 6.6 Certificate Transparency (CT) — RFC 9162

Every public TLS certificate must be logged to a set of *CT logs* (append-only Merkle trees, gossip protocol, signed tree heads). Monitors (e.g., Google Pilot, Facebook CT Monitor) detect mis-issuance.

Chrome requires:
- **CT log inclusion** for all public TLS certs (≥ 90 days validity).
- **SCT** (Signed Certificate Timestamp) delivered in TLS handshake.
- Chrome rejects certificates not in qualifying CT logs.

**Effect**: mis-issued certs are detected within hours, not years.

#### 6.7 Pinning

- **HPKP** (HTTP Public Key Pinning) — RFC 7469. Browser pins a cert/SPKI hash. **Removed from Chrome 2018** because of operational risk (pin loss = locked out).
- **HSTS preload** — still used; pin that *only* HTTPS may be spoken on a domain.
- **DNSSEC/DANE TLSA** — the *intended* modern pin; requires DNSSEC deployment.

### 7. Key Management Lifecycle (NIST SP 800-57)

```
Pre-operational:
  User registration → User enrollment → Key generation
  → Key establishment (manual, DH, KEM) → Key entry into HSM/KMS

Operational:
  Normal use → Key storage (HSM) → Key backup
  → Key recovery (escrow) → Key rotation
  → Key replacement after compromise or end-of-cryptoperiod

Post-operational:
  Key archival (long-term) → Key destruction (zeroize)
  → Key revocation (publish, log)
  → Logging and audit throughout
```

**Cryptoperiod**: the time a key may be used. Examples:
- TLS server RSA-2048: 2 years (CA/B Forum)
- TLS server ECDSA P-256: 825 days (Apple) / 398 days (CA/B Forum 2024)
- Signing keys: 1–3 years
- Data-encrypting keys: 1–2 years
- Root CA: 20+ years
- Email signing (S/MIME): 2–3 years

### 8. Key Escrow and Recovery

- **Key escrow** — a copy of the private key is held by a trusted third party (e.g., government, enterprise) for lawful access or disaster recovery.
- **M of N control** — recovery requires M of N key custodians to authorize.
- **Shamir Secret Sharing** — split a key into $n$ shares, $k$ of which reconstruct:

$$
\text{secret} = f(0) \quad \text{from a polynomial } f \text{ of degree } k-1
$$

Any $k$ shares uniquely determine $f$; any $k-1$ reveal nothing.

- **NIST SP 800-57 Part 2 Rev. 1** — recommends *not* escrowing keys that may have been used in authenticated mode; argues for *re-encrypting* data with a new key, not retrieving the old one.

### 9. Quantum Impact and Migration

Shor's algorithm breaks RSA, DH, and ECC in polynomial time on a CRQC. Mitigation:

| Use | Classical | PQC replacement |
|-----|-----------|-----------------|
| Key exchange | ECDH X25519, P-384 | ML-KEM-768 (Kyber768) — hybrid in TLS 1.3 |
| Identity signatures | Ed25519, RSA-3072, P-256 | ML-DSA-65 (Dilithium3), SLH-DSA (SPHINCS+) |
| Hash-only | SHA-256 | SHA-256 (Grover halves strength: use SHA-384+) |

**Hybridization** is the safe path: combine a classical KEM (X25519) with a PQC KEM (ML-KEM-768). The connection is secure if *either* is unbroken.

See [[post-quantum-cryptography]] for full details.

### 10. Algorithm Selection Quick Reference (2025)

| Need | Recommended |
|------|-------------|
| TLS 1.3 KEM (interim) | X25519 |
| TLS 1.3 KEM (PQC-ready) | X25519 + ML-KEM-768 hybrid |
| TLS 1.3 signature | Ed25519, RSA-3072+PSS, ECDSA-P-256 |
| Document signature (long-term) | ML-DSA-65, SLH-DSA-SHAKE-256s |
| Code signing (current) | Ed25519 or RSA-3072+PSS |
| SSH key | Ed25519 (default) or RSA-4096 for legacy |
| JWT signing | EdDSA, RS256/PS256 with RSA-3072 |
| PGP / OpenPGP | Ed25519 + X25519 subkey |
| Email (S/MIME) | RSA-3072+PSS+SHA-256, or Ed25519+ML-KEM-768 hybrid |

## CISO / Risk Manager View

**Board framing**: "If our CA is compromised, how long until the world knows, and how long until the cert is no longer trusted?" The answer is *minutes* (CT logs) and *the next browser update* (revocation). The board needs to understand that PKI is a *trust network*, not just a technology buy.

**Strategic imperatives**:
1. **No private CAs for public-facing certs.** Use Let's Encrypt, DigiCert, Sectigo, or your CA/B Forum-approved public CA. Private CAs only for internal domains.
2. **Mandate CT for every public cert.** A mis-issued cert shows up in CT logs within hours; without CT, you find out only when a browser user does.
3. **Mandate OCSP stapling + Must-Staple** for high-value servers.
4. **Mandate 90-day cert lifetimes** so that revocation is a *fail-safe*, not the primary mechanism. Automation via ACME (Let's Encrypt protocol) makes this free.
5. **Mandate PQC-hybrid key exchange** for all new TLS endpoints within 24 months. The cost of *not* doing this is captured ciphertexts that a CRQC will eventually decrypt.
6. **HSM for every CA, every root key, every code-signing key.** FIPS 140-3 Level 3+ for any key that signs what other systems trust.

**Key management program**:
- **One owner** (the CISO) of the key management policy.
- **One KMS** (or a small federation) for *all* application keys.
- **One set of rotation policies** (e.g., "data keys: 90 days; CA signing: 2 years; root: 20 years").
- **One audit log** that is append-only and reviewable.

**Crypto-agility mandate**: do not let an application call `RSA_sign()` directly; require `crypto.sign(alg="RSA-PSS-3072", key=...)`. Code that names a specific cipher cannot migrate when the algorithm is broken.

**Common audit finding the board should expect**:
- Self-signed certs on production endpoints (no chain to a public CA).
- Certs > 398 days (CA/B Forum violation).
- 1024-bit or 2048-bit RSA-with-v1.5 padding.
- 3DES or RC4 in legacy service.
- Private keys checked into source control (mitigated by *pre-commit* scanning and HSM-backed commit signing).

**Single highest-ROI action**: deploy ACME-based issuance + automated renewal across the entire public-facing estate. This eliminates ~80% of the certificate-related incidents a typical enterprise sees in a year.

## Related Connections
- Crypto fundamentals: [[cryptography-fundamentals-and-math]]
- Symmetric algorithms: [[symmetric-encryption-algorithms]]
- TLS 1.3: [[tls-1-3-deep-dive]]
- Post-quantum: [[post-quantum-cryptography]]
- FIPS 140-3: [[fips-140-3]]
- Secure boot / TPM: [[secure-boot-and-trusted-platform-module-tpm]]
- Domain hub: [[domain-03-security-architecture-and-engineering]]

## References
- NIST FIPS 186-5 (2023) — *Digital Signature Standard*.
- NIST SP 800-56A Rev. 3 (2018) — *Pair-Wise Key Establishment Using DH*.
- NIST SP 800-57 Part 1 Rev. 5 (2020) — *Recommendation for Key Management*.
- NIST SP 800-57 Part 2 Rev. 1 (2019) — *Recommendation for Key Management: Key Establishment*.
- NIST SP 800-57 Part 3 Rev. 1 (2015) — *Application-Specific Key Management Guidance*.
- RFC 5280 (2008) — *Internet X.509 PKI Certificate and CRL Profile*.
- RFC 6960 (2013) — *Online Certificate Status Protocol (OCSP)*.
- RFC 6962 (2013) — *Certificate Transparency*.
- RFC 7748 (2016) — *Elliptic Curves for Security* (X25519, X448).
- RFC 8017 (2016) — *PKCS #1: RSA Cryptography Specifications Version 2.2*.
- RFC 8032 (2017) — *Edwards-Curve Digital Signature Algorithm (EdDSA)*.
- RFC 8446 (2018) — *The Transport Layer Security (TLS) Protocol Version 1.3*.
- RFC 9162 (2022) — *Certificate Transparency Version 2.0*.
- CA/Browser Forum — *Baseline Requirements for the Issuance and Management of Publicly-Trusted Certificates*.
- Bleichenbacher, D. (1998). *Chosen Ciphertext Attacks Against Protocols Based on RSA Encryption Standard PKCS #1*.
- Shor, P. W. (1994). *Algorithms for Quantum Computation: Discrete Logarithms and Factoring*.
- NSA CNSA 2.0 (2022).
