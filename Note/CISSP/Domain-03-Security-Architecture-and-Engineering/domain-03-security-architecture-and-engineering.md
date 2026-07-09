---
title: Domain 3 — Security Architecture and Engineering
layer: 1
domain: 3
tags: [cissp, domain-3, hub, security-architecture, cryptography, security-models, pki, secure-design]
related_domains: [1, 2, 4, 5, 6, 7, 8]
---

# Domain 3 — Security Architecture and Engineering

## Feynman Explanation
Security Architecture is the discipline of designing systems so that the *structure itself* resists attack — locks, walls, alarms, and procedures chosen to match the value of what they protect. *Engineering* is the part where cryptography, hardware roots-of-trust, and reference monitors turn policy into provable guarantees. Together, this domain is where CISSP candidates learn to think like both an architect and an adversary: pick the right primitive, the right key length, the right model, and the right pattern, then prove nothing can be subverted without detection.

## Technical Details

### 3.1 Weight in the Exam
Domain 3 is one of the most technically dense of the eight CISSP domains and carries heavy weighting on engineering fundamentals, security models, and applied cryptography. Mastery here is the largest single predictor of overall pass/fail outcome.

### 3.2 Domain Sub-Topics (ISC² Outline)

| # | Sub-topic | Anchor note |
|---|-----------|-------------|
| 3.1 | Security design principles (least privilege, defense in depth, fail secure, economy of mechanism, complete mediation, open design, separation of privilege, least common mechanism, psychological acceptability) | [[security-architecture-patterns]] |
| 3.2 | Security model concepts, architectures, and evaluation criteria (TCSEC, ITSEC, Common Criteria, EAL) | [[security-models]] |
| 3.3 | System security evaluation models (formal models: BLP, Biba, Clark-Wilson, Brewer-Nash, Graham-Denning, HRU) | [[security-models]] |
| 3.4 | Cryptographic fundamentals: symmetric / asymmetric, hashing, MAC, digital signature | [[cryptography-fundamentals-and-math]] |
| 3.5 | Symmetric algorithms: DES, 3DES, AES, Blowfish/Twofish; modes of operation (ECB, CBC, CTR, GCM) | [[symmetric-encryption-algorithms]] |
| 3.6 | Asymmetric algorithms: RSA, DH, ECC, ElGamal, DSA; PKI, X.509, OCSP, CRL | [[asymmetric-encryption-and-pki]] |
| 3.7 | Hashing & message authentication: MD5, SHA-1/2/3, HMAC, birthday paradox | [[hash-functions-and-hmac]] |
| 3.8 | TLS 1.3 handshake, cipher suites, forward secrecy, 0-RTT | [[tls-1-3-deep-dive]] |
| 3.9 | Site and facility design (physical): CPTED, zones, media storage, fire classes | (see [[security-architecture-patterns]]) |
| 3.10 | Post-quantum cryptography: NIST PQC (Kyber, Dilithium, SPHINCS+) | [[post-quantum-cryptography]] |
| 3.11 | FIPS 140-3 validated cryptographic modules | [[fips-140-3]] |
| 3.12 | Secure boot, TPM, hardware roots of trust, attestation | [[secure-boot-and-trusted-platform-module-tpm]] |

### 3.3 Architectural Lenses

Three lenses appear repeatedly across this domain; every note ties back to at least one of them.

- **Confidentiality** — preventing unauthorized *read* (BLP, MLS, encryption in transit/at rest).
- **Integrity** — preventing unauthorized *write* (Biba, Clark-Wilson, MAC, digital signature).
- **Availability / Provenance** — ensuring the system is *reachable* and *trustworthy* from silicon up (secure boot, TPM attestation, DoS resistance, defense in depth).

### 3.4 Evaluation Frameworks

| Framework | Origin | Levels | Status |
|-----------|--------|--------|--------|
| TCSEC (Orange Book) | US DoD 1983 | A1 → C2 | Historic; influenced all successors |
| ITSEC | EU 1991 | E0–E6 (functional) × correctness | Historic (Europe) |
| Common Criteria (CC) | ISO/IEC 15408, 1996+ | EAL1–EAL7 | Global standard today |
| FIPS 140-3 | NIST 2019 | 4 levels (1–4) | US/Canada mandatory for federal crypto modules |
| NIAP Protection Profiles | US | PP-defined | Drives US federal procurement |

### 3.5 Reference Monitor (TCSEC "A1" requirement)

Anderson's 1972 report specified the three properties every trusted computing base (TCB) must satisfy; a TCB that violates any of these cannot be evaluated above Class B:

1. **Complete mediation** — every access to every object by every subject is checked.
2. **Tamper-proof** — the monitor itself cannot be modified.
3. **Verifiable** — the monitor is small enough to be analyzed and proven correct.

The implementation is the *security kernel*; the surrounding trusted OS is the *trusted computing base (TCB)*.

### 3.6 Crypto Agility — Strategic Posture

Modern architectures must be **crypto-agile**: designed so the algorithm, key length, and provider can be swapped without rewriting applications. This is the single most important defensive posture against:
- Quantum threat (Shor's algorithm against RSA/ECC) — see [[post-quantum-cryptography]]
- Newly broken primitives (e.g., SHA-1 collision 2017, MD5 collision 2004)
- Regulatory change (CNSA 2.0 timelines, PCI DSS 4.0, FIPS 140-3 transition)

<!-- EXPANDED -->

## Foundations

**Security is a property of the system, not a feature bolted on.** The reference monitor (Anderson 1972) is the abstract machine that enforces this — it must be *always invoked* (complete mediation), *tamperproof*, and *small enough to verify*. The concrete implementation is the **security kernel**; the surrounding trusted OS + hardware is the **Trusted Computing Base (TCB)**. A TCB that violates any reference monitor property cannot be evaluated above TCSEC Class B (or CC EAL4).

**The CIA triad as architectural constraints**: confidentiality → BLP (no read up, no write down); integrity → Biba (no read down, no write up); availability → defense in depth + redundancy. You cannot simultaneously satisfy strict BLP *and* strict Biba on the same system without a *trusted subject* that breaks at least one rule — this is the **Lipner integration** (1975): a privileged "auditor" ring writes up/down under controlled conditions, layering Clark-Wilson well-formed transactions on top. In practice, Lipner-style models are what banking, defense, and commercial MLS systems implement.

| Property | Model family | Core rule | Breaks if |
|----------|-------------|-----------|-----------|
| Confidentiality | Bell-LaPadula | NRU + NWD | Trusted subject leaks down |
| Integrity | Biba (strict) | No read down + no write up | Low-integrity data poisons high |
| Commercial integrity | Clark-Wilson | Well-formed TP + SoD | Dual-control bypass |
| Conflict-of-interest | Brewer-Nash | History-based access wall | Cross-tenant read without revocation |
| Safety guarantee | HRU (mono-operational) | Polynomial-time analysis | General HRU is undecidable |

**Why Brewer-Nash is unique**: it's the only *dynamic* model — access rights change as the subject reads data. A subject who reads Bank A's dataset is *irrevocably* locked out of Bank B's (same conflict class). This maps directly to multi-tenant SaaS: "company" = tenant, "conflict class" = industry vertical, implemented as row-level security with access-history tracking.

## Architecture

**Security modes of operation** (TCSEC/DoD 5200.28):
| Mode | Description | Modern analogue |
|------|-------------|-----------------|
| Dedicated | All subjects cleared to system-high, all need-to-know | Single-purpose admin console |
| System-high | All subjects cleared to system-high, *not* all need-to-know | Single-tenancy with RBAC |
| Compartmented | Subjects cleared to different compartments on same system | Multi-tenant SaaS with per-tenant crypto |
| Multilevel (MLS) | Full lattice — TS/S/C/U with compartments | SELinux MLS, Trusted Solaris |

**The layered security architecture** (bottom-up):
```
[7] Application  ← authZ, input validation, output encoding, WAF
[6] Middleware    ← API gateway, message bus, service mesh (mTLS)
[5] OS            ← MAC/DAC, SELinux, ASLR, NX bit, sandboxing
[4] Kernel        ← security kernel, system calls, I/O mediation
[3] Hypervisor    ← VM isolation, vTPM, AMD SEV-SNP / Intel TDX
[2] Firmware      ← UEFI secure boot, measured boot, option ROM
[1] Hardware      ← TPM/HSM/Pluton/Secure Enclave, AES-NI, SGX
```
Defense in depth means each layer *independently* resists a breach of the one below: a kernel exploit should not yield application data (because the app layer encrypts), a firmware rootkit should be detectable (because the TPM attests PCR drift).

**TPM chain of trust** (CRTM → OS loader):
1. CRTM (immutable firmware) hashes BIOS/UEFI → extends PCR 0.
2. UEFI hashes option ROMs → extends PCRs 1-3.
3. Boot loader is hashed → extends PCR 4.
4. OS kernel + initramfs are hashed → extends PCRs 5-7.
5. IMA (Linux) extends PCR 10 on every exec().
6. Remote attestation: `TPM2_Quote(AIK, {0,1,2,3,4,5,7,10}, nonce)` → signed quote compared against expected PCR policy (the *golden measurement* baseline).

**HSM placement in key management infrastructure**: HSMs are *not* general-purpose crypto accelerators — they exist at specific trust boundaries:
- Root CA key (offline, FIPS 140-3 Level 3, air-gapped) — signs issuing CA certs.
- Issuing CA key (online HSM cluster, Level 3) — signs leaf certs at high volume.
- Code-signing key (Level 3 HSM) — signs build artifacts; compromise = arbitrary malware signed.
- Key-wrapping master key (Level 3 HSM) — wraps all data-encrypting keys.
- TLS session key generation (Level 1 software module or Level 2 HSM) — ephemeral; volume matters.

**Crypto boundaries and why FIPS 140-3 cares**: every FIPS-validated module has a *defined boundary*. Crypto calls that cross the boundary must use the FIPS provider; non-FIPS algorithms (even ChaCha20) are illegal inside the boundary. The most common compliance failure: OpenSSL loads *both* the FIPS provider and the default provider, and an application calls `EVP_aes_256_gcm()` (default) instead of the FIPS provider path. The boundary test is: can the auditor trace every key bit through approved primitives?

## Execution

**Selecting cipher suites**: match the threat model to (algorithm, mode, key size, MAC):

| Threat | Confidentiality | Integrity/Auth | KEM/Signature |
|--------|----------------|----------------|---------------|
| Web (TLS 1.3) | AES-256-GCM | GHASH (GCM) | X25519 + Ed25519 |
| Mobile / IoT | ChaCha20-Poly1305 | Poly1305 | X25519 + Ed25519 |
| Disk at rest | AES-256-XTS | Merkle tree (dm-verity) | TPM-sealed key |
| Long-term archive (20+ yr) | AES-256-GCM | HMAC-SHA-384 | ML-DSA-65 or SLH-DSA |
| PQC-ready TLS | AES-256-GCM | GHASH | **X25519 + ML-KEM-768 hybrid** |
| In-memory session | AES-128-GCM | GHASH | ECDHE P-256 (perf) |

**Certificate lifecycle** (PKI operational playbook):
```
CSR generation (RSA-3072 or Ed25519 key on HSM)
  → Identity vetting (RA: domain control, org validation, EV)
  → CA signs (SHA-384 over TBS certificate)
  → CT log submission (3 SCTs from independent logs)
  → Deployment (ACME automated, e.g., certbot / lego)
  → Continuous monitoring (CT monitors, alerts on mis-issuance)
  → Renewal (~60 days before expiry, automated)
  → Revocation (CRL + OCSP stapling + Must-Staple flag)
  → Archival (for non-repudiation evidence)
```

**Building a PKI hierarchy**:
```
Root CA (self-signed, offline, FIPS 140-3 L3 HSM, 20-year cert)
  └── Policy CA (signs cross-certificates, offline)
        ├── Issuing CA 1 (TLS server certs, L3 HSM, 2-5 year cert)
        ├── Issuing CA 2 (code signing, L3 HSM, 2-5 year cert)
        └── Issuing CA 3 (client/S-MIME, L3 HSM, 2-5 year cert)
              └── OCSP responder (signed by issuing CA, stapled to EE cert)
```
Key constraints: Root CA offline always; issuing CAs online but HSMs never export private keys; OCSP responses signed by a delegated key (not the issuing CA's key); Certificate Transparency logs provide public audit; CRL distribution points published for legacy clients.

**Running a side-channel analysis** (practical methodology):
1. **Threat model**: who has physical access? (smart card vs. cloud tenant vs. remote timing).
2. **Attack surface**: cache-timing (AES T-tables → mitigated by AES-NI), power analysis (DPA on smart cards → mitigated by masking), EM emanation (TEMPEST → mitigated by shielding), acoustic (RSA key extraction from CPU sound — mitigated by constant-time RSA blinding).
3. **Tooling**: ChipWhisperer (power/EM), `dudect` / `ctgrind` (constant-time verification), `libflush` (cache eviction).
4. **Remediation**: constant-time code paths, hardware crypto (AES-NI, TPM, HSM), masking/blinding, key isolation.

**Implementing secure boot with measured boot and attestation**:
1. Platform Key (PK) enrolled → KEK enrolled → DB populated with OS vendor keys.
2. UEFI Secure Boot enabled in *enforced* mode (not audit).
3. TPM 2.0 present; SHA-256 PCR bank active.
4. BitLocker / LUKS2 volume key sealed to PCRs 0,2,4,7 (firmware + option ROMs + boot loader + Secure Boot state).
5. IMA appraisal mode enabled on Linux (deny execution of unmeasured binaries).
6. MDM / conditional access policy: device must produce valid `TPM2_Quote` matching golden PCR values within 24h.
7. Quarantine on attestation failure: network isolation + incident response.

## Mastery

**Cryptographic agility** — the ability to swap algorithms without rewriting systems: every cryptographic call site accepts `(alg, key, ...)`, never calls a named primitive directly. `RSA_sign(SHA256, msg, key)` is brittle; `crypto.sign(alg="RSASSA-PSS-3072", hash="SHA-256", key=key_ref)` is agile. Agility is the *single most important architectural property* for surviving: (a) a quantum break of RSA/ECC, (b) a practical collision in SHA-2, (c) regulatory mandates like CNSA 2.0.

**Homomorphic encryption** (FHE / SHE / PHE):
| Type | Operation | Compute overhead | Use case |
|------|-----------|-----------------|----------|
| PHE (Paillier) | Add *or* multiply (one) | ~10³× | Encrypted sums, voting |
| SHE (BGV/BFV/CKKS) | Add + multiply (bounded depth) | ~10⁴× | Encrypted ML inference |
| FHE (TFHE/FHEW) | Arbitrary circuits | ~10⁶–10⁸× | Encrypted search, database |

What FHE enables: a cloud provider can process encrypted data without ever seeing plaintext. What it costs: 6–8 orders of magnitude slower than plaintext computation. In 2025, FHE is practical for *low-volume, high-value* workloads (genomic matching, credit scoring on encrypted data). Not yet viable for general-purpose encrypted databases.

**Post-quantum migration timeline** (CNSA 2.0):
| Year | Milestone |
|------|-----------|
| 2025 | PQC algorithms available (FIPS 203/204/205); hybrid KEM in TLS 1.3 |
| 2027 | Software/firmware signing *must* use PQC (ML-DSA-65) |
| 2030 | Web/TLS, all new acquisitions: hybrid KEM mandatory |
| 2030 | RSA/ECC *off* for national-security use |
| 2033 | Full PQC: classical asymmetric algorithms disallowed |
CISO action: start hybrid KEM on public TLS endpoints in 2025; inventory all crypto assets; identify HNDL exposure.

**Zero-knowledge proofs** (ZKPs): a prover convinces a verifier that a statement is true *without revealing why*. SNARKs (succinct, non-interactive) enable zk-rollups on Ethereum (prove that 10,000 transactions are valid in a 200-byte proof). STARKs (scalable, transparent, post-quantum) remove the trusted setup. ZKPs apply to: anonymous credentials (prove you're over 18 without revealing your birthdate), verifiable compute (prove an ML model was trained on a specific dataset), private DIDs.

**Secure multi-party computation (SMPC)**: $n$ parties jointly compute a function $f(x_1, ..., x_n)$ without revealing their private inputs $x_i$. Applications: private set intersection (two companies discover shared customers without revealing non-shared), distributed key generation (DKG for threshold signing), sealed-bid auctions. SMPC protocols (GMW, BGW, SPDZ) trade rounds × communication × computational overhead. Typically 10³–10⁶× slower than cleartext.

## Resilience

**Top architecture mistakes** in order of prevalence:
1. **Security as a perimeter** — "inside the firewall = trusted." Solved by zero trust (NIST SP 800-207): every request authenticates, every session is authorized, the network is hostile.
2. **No defense in depth** — one breach = total compromise. Solved by layered controls with independent failure modes.
3. **Trusting the network** — plaintext east-west traffic. Solved by mTLS everywhere (service mesh, SPIFFE identities).
4. **Hardcoded keys** — in source, config, environment variables. Solved by secret manager + short-lived tokens + pre-commit scanning.
5. **Missing crypto-agility** — unable to migrate when a primitive breaks. Solved by versioned `crypto.sign(alg=..., key=...)` APIs.
6. **Credentials in CI/CD pipelines** — build systems have production credentials. Solved by OIDC federation + workload identity + just-in-time elevation.
7. **No attestation** — servers trusted at face value. Solved by TPM attestation + confidential computing (SEV-SNP/TDX).

**Bad crypto** — if you see any of these in production, stop and fix:

| Primitive | Why bad | When to ban | Replacement |
|-----------|---------|-------------|-------------|
| ECB mode | Identical plaintext → identical ciphertext (pattern leak) | 2000 (always was) | GCM, CBC+HMAC |
| Single DES | 56-bit key, exhaustive in hours | 2005 | AES-128 minimum |
| 3DES (2-key) | Meet-in-the-middle, effective ~2^80 | 2023 (NIST SP 800-131A) | AES-128-GCM |
| MD5 | Collisions in seconds | 2010 | SHA-256 minimum |
| SHA-1 | Practical collision $2^{63.1}$ (SHAttered 2017) | 2017 (sig), 2030 (hash) | SHA-256 or SHA-384 |
| RC4 | Stream cipher biases, plaintext recovery | 2015 (RFC 7465) | ChaCha20-Poly1305 |
| RSA PKCS#1 v1.5 | Bleichenbacher padding oracle (1998) | 2024 (NIST) | RSA-OAEP, RSA-PSS |
| RSA-1024 | Factored on commodity GPUs | 2009 | RSA-3072 minimum |
| Static DH/RSA key transport | No forward secrecy | Always was (TLS 1.3 removed) | ECDHE (X25519) |
| RSA-2048 (long-life data) | Harvest-now-decrypt-later via Shor | 2025 (HNDL window) | Hybrid PQC + X25519 |

**When defense-in-depth becomes defense-in-waste**: each additional layer has a *marginal benefit* and a *marginal cost*. A WAF in front of a server that validates all input at the application layer is redundant — unless the WAF catches *different* attack classes (e.g., DDoS volumetric, IP reputation). The test: "If this layer were bypassed, does the next layer independently prevent the breach?" If two layers check the same thing the same way, one is waste. If they check different things or at different abstraction levels, both earn their keep.

## Context

**Building for FedRAMP vs building for commercial**:
| Dimension | FedRAMP (US Gov Cloud) | Commercial (non-regulated) |
|-----------|------------------------|---------------------------|
| Crypto | FIPS 140-3 validated *mandatory* | FIPS recommended, not mandatory |
| Audit logging | NIST SP 800-53 AU controls, tamper-proof, append-only | Application-level logging sufficient |
| Key management | HSM for root keys (Level 3 minimum) | KMS or HSM depending on data class |
| Identity | PIV/CAC (FIPS 201), SAML federation | OIDC/SAML, password + MFA |
| Continuous monitoring | FedRAMP ConMon program, monthly POA&M | Self-defined cadence |
| Authorization boundary | Formal system security plan (SSP) diagram | Architecture diagram, ad hoc |
| Breach notification | US-CERT within 1 hour | State/contract-defined (often 72h) |

**Military grading: Common Criteria EAL1-EAL7** — the evaluation rigor ladder:
- EAL1-2: functionally tested; suitable for low-risk commercial. Self-asserted.
- EAL3-4: methodically tested and designed; requires a Security Target (ST). *Practical commercial ceiling.*
- EAL5-6: semiformally designed and verified; requires formal model + covert channel analysis. Smart cards, high-grade OS.
- EAL7: formally verified design; the TOE's formal spec is proven correct. Rare. Requires formal methods (Isabelle/HOL, Coq).

Important: EAL measures *assurance* (how well was it tested/reviewed?), not *strength* (how strong are the mechanisms?). An EAL7 evaluated firewall could protect against exactly the same threats as an EAL4 firewall — it's just been proven correct, not stronger.

**When to use hardware security vs. software security**:
- **Hardware (HSM, TPM, Secure Enclave)**: any key that *signs what others trust* (CA keys, code-signing keys), any key whose theft enables impersonation (FIDO2 private keys), any key that wraps a class of lower keys (master key). Tamper-resistant boundary, non-exportable keys.
- **Software (FIPS-validated library)**: per-session keys (TLS), data-encrypting keys with short cryptoperiods, keys that are routinely rotated. Speed and volume justify software at Level 1.
- **Hybrid**: the HSM holds the *master key*; software KMS derives per-use keys from it. The master key never leaves the HSM.

**The cost of crypto acceleration** (AES-NI, CLMUL, SHA extensions):
- AES-NI (Intel Westmere 2010): AES-128-GCM drops from ~30 cycles/byte (software) to ~1.3 cycles/byte. **23× speedup.**
- CLMUL (Carry-Less Multiplication, Intel 2010): enables GHASH in hardware; GCM tag generation at line rate.
- ARMv8 Crypto Extensions (2013): AES + SHA-1/SHA-256 hardware instructions on mobile.
- Cost: transistors. On a modern server SoC, crypto acceleration is ~1% of die area. There is no scenario in 2025 where software-only AES is justified on x86 — the hardware is always present. On ARM/IoT, ChaCha20 (constant-time in software) is preferred where crypto extensions are absent.

## Nuance

**"Trusted" vs. "Trustworthy"** — the most important terminological split in security engineering:
- **Trusted**: the system *is permitted* to violate security policy (it sits inside the TCB). A *trusted subject* in Lipner is trusted to write down. A *trusted boot* component is trusted to load the next layer.
- **Trustworthy**: the system *has been evaluated* and found to not violate security policy (it has been proven, audited, or tested). A TPM is *trustworthy* because its design and implementation have been analyzed.
- The gap: a trusted component that is not trustworthy is a **vulnerability**. A trustworthy component that is not trusted is a **wasted evaluation**. The architect's job is to minimize the first and justify the second.

**Reference monitor vs. security kernel** — they're not the same:
- Reference monitor = *abstract concept*: always invoked, tamperproof, verifiable.
- Security kernel = *concrete implementation*: the hardware + software that realizes the reference monitor on a specific platform.
- TCB = security kernel + all other trusted components (the auth system, the audit system, the TPM chain).
- Analogy: reference monitor = "the constitution requires checks and balances"; security kernel = "the physical vault and the guard rotation"; TCB = "the whole government building."
- CC EAL5+ requires the security kernel to be formally specified; EAL7 requires formal verification of correspondence between specification and implementation.

**Why RBAC + DAC + MAC coexist** — they address different threat models:
| Model | Who sets policy? | Threat model | Implementation |
|-------|-----------------|--------------|----------------|
| DAC (Discretionary) | Object owner (chmod) | User error, casual abuse | File permissions, ACLs |
| MAC (Mandatory) | System security officer (label) | Confinement, Trojan, insider | SELinux, SMACK, BLP/Biba labels |
| RBAC (Role-Based) | Administrator (role/permission) | Privilege escalation, admin abuse | IAM roles, RBAC matrix |
- DAC is flexible but fragile — owners can `chmod 777`.
- MAC is rigid but confinement-guaranteed — labels prevent even the root user from leaking.
- RBAC maps to business structure (manager, analyst, auditor) — the most maintainable at scale.
- In a modern enterprise: SELinux provides MAC (process labels), file ACLs provide DAC, IAM roles provide RBAC. All three layers are present; they enforce *different* invariants.

**Brewer-Nash and insider trading** — how it dynamically adjusts access:
A consultant at a law firm reads Microsoft's merger documents (Company A, conflict class "technology"). The Chinese Wall model *automatically* revokes her access to all other technology-company documents — Apple, Google, Intel. She can never read both sides of a potential deal within the same conflict class. The revocation is *irreversible* for that session. In cloud SaaS, this maps to *industry-vertical tenant isolation*: a support engineer who accesses Bank X's data in the "financial services" cluster cannot access Bank Y's data. Implemented via access-history tracking + dynamic row-level security. Brewer-Nash is the only formal model that solves the *transitive information flow* problem: "I learned something from Company A, and now I'm reading Company B's documents — I might trade on what I learned."

**Clark-Wilson and business integrity** — two mechanisms that make it work:
1. **Well-formed transactions (TPs)**: every state change goes through a certified procedure that preserves a consistency invariant. A bank transfer TP ensures $\sum(\text{balances after}) = \sum(\text{balances before})$. No ad-hoc SQL `UPDATE` is permitted on the CDI.
2. **Separation of duty**: the *same* subject cannot perform *both halves* of a sensitive operation. The TP registry enforces that `issue_check` and `approve_check` cannot be performed by the same user. This is the *four-eyes principle* codified as an access-control rule: $E3$ enforces the SoD relation, $C3$ certifies it.
- The combination means: even the most privileged user cannot *both* modify the ledger and approve the modification. This is why Clark-Wilson, not Biba, is the model for banking — Biba prevents write-up, but Clark-Wilson prevents *fraud*.

## Resources

**Cipher suite selection matrix** (quick reference for greenfield design):

| Scenario | TLS Cipher Suite | Key Exchange | Signature | Notes |
|----------|-----------------|--------------|-----------|-------|
| Public HTTPS (2025) | TLS_AES_256_GCM_SHA384 | X25519 | Ed25519 | Default for all new endpoints |
| Mobile / IoT HTTPS | TLS_CHACHA20_POLY1305_SHA256 | X25519 | Ed25519 | No AES-NI on ARM |
| Public HTTPS (PQC-ready) | TLS_AES_256_GCM_SHA384 | X25519 + ML-KEM-768 **hybrid** | Ed25519 | CNSA 2.0 compliant |
| Internal mTLS (service mesh) | TLS_AES_128_GCM_SHA256 | X25519 | Ed25519 | Perf over security; short-lived |
| IPsec IKEv2 | AES-256-GCM | ECDH-P-384 | ECDSA-P-384 | PQC hybrid available (RFC 9370) |
| SSH | chacha20-poly1305@openssh.com | X25519 | Ed25519 | Default modern SSH |
| WireGuard | ChaCha20-Poly1305 | X25519 | (implicit in DH) | Fixed cipher suite by design |

**Certificate template checklist** (leaf TLS cert, X.509 v3):

- [ ] Subject: CN = domain (legacy), SAN = all required DNS names (CA/B Forum mandatory)
- [ ] Validity: ≤ 398 days (CA/B Forum), ≤ 90 days preferred (automated ACME)
- [ ] Public key: RSA-3072 or Ed25519 (P-256 minimum for ECDSA)
- [ ] Signature algorithm: SHA-384 with RSA-PSS or SHA-512 with Ed25519
- [ ] `keyUsage`: `digitalSignature`, `keyEncipherment` (RSA) / `keyAgreement` (ECC)
- [ ] `extendedKeyUsage`: `serverAuth` (and `clientAuth` if mTLS)
- [ ] `basicConstraints`: `CA:FALSE`
- [ ] `authorityInformationAccess`: OCSP responder URL + CA issuer URL
- [ ] `CRLDistributionPoints`: CRL URL
- [ ] `ct_precert_scts`: ≥ 2 SCTs from independent CT logs (embedded or stapled)
- [ ] Must-Staple (`id-pe-tlsfeature` with `status_request`)

**Secure architecture review checklist** (the 10 questions every design review must answer):

1. What is the reference monitor for this system? (Name it, show the boundary.)
2. What is the threat model? (STRIDE, attack trees, or formal adversary model.)
3. Which security model applies? (BLP, Biba, Clark-Wilson, Brewer-Nash, or hybrid.)
4. Is every inter-component call authenticated? (mTLS, signed tokens, or API keys with replay protection.)
5. Where are the keys? (HSM level, rotation policy, who has access, what is the compromise radius?)
6. Is the system crypto-agile? (Can algorithms be swapped without code changes?)
7. What is the attestation path? (How does a verifier know this node ran the right code?)
8. Is there separation of duty on sensitive operations? (Clark-Wilson E3/C3 enforced?)
9. Are failure modes fail-secure? (Default deny. No fail-open auth. No fallback to plaintext.)
10. Can the system survive 5 years without a redesign? (HNDL, PQC, CNSA 2.0, EAL reevaluation.)

**FIPS validation query URL**: `https://csrc.nist.gov/projects/cryptographic-module-validation-program/validated-modules/search` — search by vendor, module name, certificate number, or algorithm. Every FIPS-validated module has a certificate number; the auditor will ask for it. A module *in process* has no number; a module *validated* has a current (not expired) number in the CMVP database. The CISSP must know the difference.

## CISO / Risk Manager View

**Board framing**: "If a nation-state walks in, can we still prove who we are and what ran?" Domain 3 answers that question. The CISO's job is to translate cryptographic and architectural decisions into *business outcomes* — regulatory posture, M&A trust, breach liability limit, and quantum-readiness timeline.

**Key management strategy** (board-level requirements):
- Centralized KMS / HSM (HSM = Hardware Security Module, FIPS 140-3 Level 3+ for root keys).
- Documented key lifecycle: generation → distribution → storage → rotation → escrow → destruction.
- Separation of duties between key custodian, key user, and auditor (Clark-Wilson principle applied to keys).
- Quantum-readiness: hybrid PQC + classical (e.g., X25519+ML-KEM-768) phased in over a 3–5 year roadmap; inventory all crypto assets; identify "harvest-now-decrypt-later" exposure on data with confidentiality life > 5 years.

**Crypto-agility mandate**:
- No hard-coded algorithm identifiers in product code.
- All crypto behind a versioned internal API.
- Annual algorithm inventory and rotation drill.
- Vendor contracts must include algorithm disclosure and migration commitments.

**Metrics the board will recognize**:
- % of systems on FIPS 140-3 validated crypto
- % of TLS endpoints on TLS 1.3
- Mean-time-to-rotate keys (KRI — Key Risk Indicator)
- Cryptographic asset inventory completeness
- PQC migration roadmap milestone burn-down

## Related Connections
- Security models: [[security-models]]
- Cryptography fundamentals: [[cryptography-fundamentals-and-math]]
- Symmetric algorithms: [[symmetric-encryption-algorithms]]
- Asymmetric and PKI: [[asymmetric-encryption-and-pki]]
- Architecture patterns: [[security-architecture-patterns]]
- TLS 1.3: [[tls-1-3-deep-dive]]
- Hashes & HMAC: [[hash-functions-and-hmac]]
- Post-quantum: [[post-quantum-cryptography]]
- FIPS 140-3: [[fips-140-3]]
- Secure boot / TPM: [[secure-boot-and-trusted-platform-module-tpm]]

Related domains:
- [[../Domain-01-Security-and-Risk-Management/index|Domain 1]] — risk framing, governance
- [[../Domain-02-Asset-Security/index|Domain 2]] — data classification drives crypto choice
- [[../Domain-04-Communication-and-Network-Security/index|Domain 4]] — TLS, IPsec, network crypto
- [[../Domain-05-Identity-and-Access-Management/index|Domain 5]] — IAM uses the primitives built here
- [[../Domain-06-Security-Assessment-and-Testing/index|Domain 6]] — validates the architecture
- [[../Domain-07-Security-Operations/index|Domain 7]] — runs the keys and the SOC
- [[../Domain-08-Software-Development-Security/index|Domain 8]] — applies the models in code

## References
- ISC² CISSP CBK, 6th/7th/8th editions — Domain 3 chapters
- NIST SP 800-175B — Guideline for Using Cryptographic Standards in the Federal Government
- NIST FIPS 140-3 — Security Requirements for Cryptographic Modules
- NIST SP 800-208 — Recommendation for Stateful Hash-Based Signature Schemes
- NIST SP 800-227 — Migration to Post-Quantum Cryptography
- Anderson, *Security Engineering*, 2nd ed., Ch. 5 (security models) and Ch. 7 (TLS)
- Common Criteria portal: https://www.commoncriteriaportal.org
- NIAP Protection Profiles: https://www.niap-ccevs.org
