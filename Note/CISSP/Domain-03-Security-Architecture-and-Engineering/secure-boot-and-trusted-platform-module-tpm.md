---
title: Secure Boot and Trusted Platform Module (TPM)
layer: 3
domain: 3
tags: [cissp, secure-boot, tpm, tpm-2-0, measured-boot, trusted-boot, attestation, uefi, pcr, hcr, quote, integrity-measurement-architecture, ima, tpm2-commands, root-of-trust, hardware-root-of-trust, fido2, sgma]
related_domains: [1, 5, 7, 8]
parent_note: security-architecture-patterns
---

# Secure Boot and Trusted Platform Module (TPM)

## Feynman Explanation
Secure boot is the *chain of trust* that ensures a computer can only run software the manufacturer approved, starting from a tiny read-only program in the chip itself — a single-purpose "root of trust" that checks the next layer, which checks the next, all the way up to the operating system. A **TPM** is a separate tamper-resistant chip that *records* what was loaded (via cryptographic hashes stored in Platform Configuration Registers) and can later *attest* to a remote server: "I booted exactly this code, signed by these parties, in this order." Together they turn "trust me, my laptop is clean" into "here is a cryptographic proof signed by hardware that cannot be forged."

## Technical Details

### 1. The Boot Threat Model

The traditional boot sequence is a *chain of trust* with a *root of trust*:

```
    CRTM (Core Root of Trust for Measurement, in firmware)
       │   hashes the next layer
       ▼
    BIOS / UEFI firmware
       │   hashes the next
       ▼
    Boot loader (e.g., GRUB, systemd-boot)
       │   hashes the next
       ▼
    OS kernel + initramfs
       │   hashes the next
       ▼
    User space, services, applications
```

The threat model: an attacker has physical access and modifies the boot loader / kernel to plant a *bootkit* — code that runs *before* the OS, has unrestricted memory access, and is invisible to OS-level defenses.

Defense: every step in the chain **measures** (hashes) the next step into a TPM register *before* executing it, and the firmware **verifies** a digital signature on each step.

### 2. The Three Boot Models

| Model | What it does | Detection? | Refuse to boot? |
|-------|--------------|------------|------------------|
| **Measured boot** (TPM-based) | Records hashes in TPM PCRs; *attests* to a remote verifier | Yes (via attestation) | No (informational) |
| **Trusted boot** | Verifies signatures of each loader; refuses to load an unverified one | Yes | Yes (refuse) |
| **Secure boot** (UEFI) | Refuses to load any executable whose signature is not in the OEM's DB | Yes | Yes (refuse) |

The three are *complementary*: secure boot verifies the *signatures* of boot components; measured boot *records* the actual hashes (regardless of signature); trusted boot *requires* a signature to load. A typical modern laptop uses **all three**:
- Secure boot verifies signed boot components.
- Measured boot hashes each into the TPM.
- The OS can later *attest* the PCR values.

### 3. UEFI Secure Boot

Defined in the UEFI Specification (§ 27 "Secure Boot").

#### 3.1 Key Databases

| Database | Contents |
|----------|----------|
| **DB** (allowed signatures) | Public keys / X.509 certs that *can* be used to sign boot components |
| **DBX** (forbidden signatures / hashes) | Revoked keys and explicit hash denials |
| **KEK** (Key Exchange Key) | Keys authorized to *update* the DB and DBX |
| **PK** (Platform Key) | The *root* key; only key that can update KEK |

```
PK (Platform Key)    — one per platform
  └── KEK            — multiple; OEM-managed
        ├── DB       — allowed signers
        └── DBX      — revoked signers
              └── Secure boot verifies each executable against DB
```

#### 3.2 The Boot Decision

For each executable (firmware module, boot loader, OS loader, driver):

1. The firmware *verifies the executable's signature* against the DB.
2. If the signature is in DB → execute.
3. If the signature is in DBX → refuse (and log).
4. If unsigned and not in DB → refuse (or *execute and log* depending on policy).

The DB is a list of public keys (not a single key). Microsoft Windows, RHEL, Ubuntu, Fedora, and SUSE all have keys in the standard OEM DB.

#### 3.3 Custom DB (Enterprise Use)

Enterprise IT can replace the OEM DB with a custom DB. To do this, the *enrolled* OEM DB keys are removed and replaced with the enterprise's own PK and DB. This locks the laptop to *enterprise-approved* boot components. Common in high-security environments (DoD, defense, finance).

#### 3.4 Shim, PreLoader, and the Microsoft 2011 Signer

The *shim* is a small first-stage bootloader signed by Microsoft (in the standard DB). It contains a separate *MOKList* (Machine Owner Key) — keys enrolled by the OS to extend the DB with custom keys (e.g., for Linux distributions and custom boot loaders).

### 4. Measured Boot and TPM

#### 4.1 The TPM 2.0 Chip

A **TPM** is a tamper-resistant crypto-processor (typically a discrete chip soldered to the motherboard, or firmware TPM in newer CPUs — `fTPM` AMD, `PTT` Intel). It provides:

- **Endorsement Key (EK)** — a 2048-bit RSA or P-256 ECDSA key burned at manufacture. *Never* exposed off-chip; only used to derive Attestation Identity Keys (AIKs).
- **Attestation Identity Key (AIK)** — signs attestation quotes; can be backed by the EK via a Privacy CA.
- **Platform Configuration Registers (PCRs)** — 24+ registers holding 256-bit (SHA-256) or 384-bit (SHA-384) hashes.
- **HMAC and cryptographic primitives** — HMAC-SHA-256, RSA, ECC, AES (TPM 2.0).
- **PCR extend** — a one-way update:

$$
\text{PCR}_n \gets \text{SHA-256}(\text{PCR}_n \,\|\, \text{new\_measurement})
$$

PCRs can only be *extended* (the previous value is the input). They cannot be set to a chosen value.

#### 4.2 PCR Bank (TPM 2.0)

The TPM 2.0 spec allows multiple hash algorithms simultaneously. Common banks:

| Bank | Hash | PCRs used | Use |
|------|------|----------|-----|
| SHA-1 | SHA-1 | 0–23 | Legacy (deprecated) |
| SHA-256 | SHA-256 | 0–23 + 24+ (up to 32) | Modern |
| SHA-384 | SHA-384 | Optional | High-assurance |

#### 4.3 PCR Assignments (TCG Standard, x86)

| PCR | Measured by |
|-----|---------------|
| 0 | CRTM, BIOS, platform firmware |
| 1 | Platform configuration (option ROMs, SMM) |
| 2 | Option ROM code (network cards, storage controllers) |
| 3 | Option ROM configuration |
| 4 | MBR (legacy boot), boot loader (UEFI) |
| 5 | Boot loader configuration |
| 6 | Wake events, handoff to OS |
| 7 | Secure Boot state (vendor-specific) |
| 8–9 | Reserved (NTFS fast boot, etc.) |
| 10 | IMA (Integrity Measurement Architecture — Linux) |
| 11–15 | Reserved / vendor |
| 16 | Debug |
| 17 | Boot services applications |
| 18–22 | Reserved |
| 23 | Application support |

A typical Linux/Windows system measures the firmware, option ROMs, boot loader, kernel, and initramfs into PCRs 0–7, with the IMA (Integrity Measurement Architecture) extending into PCR 10 as the OS runs.

#### 4.4 The PCR Extend Operation

PCRs are *register* objects that can only be extended:

$$
\text{PCR}[i] \leftarrow \text{Hash}(\text{PCR}[i] \,\|\, \text{data})
$$

Consequence: a TPM cannot be reset to a chosen value; the only state change is "more measurements."

### 5. Attestation

A TPM *attests* by signing a structure that quotes a set of PCR values:

```
TPM2_Quote(AIK_handle, PCR_selection, qualifying_data, signature_scheme)
  → quote: { PCR_values, nonce, signature_over(PCR_values || nonce) }
```

The verifier:
1. Knows the *expected* PCR values for a *known-good* boot (an *attestation policy*).
2. Verifies the signature on the quote with the AIK's public key.
3. Checks the nonce to prevent replay.
4. Compares the PCR values to the expected values; rejects if mismatched.

#### 5.1 Remote Attestation Flow

```
Laptop                                  Verifier
  -----                                   --------
  boot; PCRs are measured
  TPM2_Quote(AIK, {0,1,2,...,7}, nonce)
  → signed quote
              ----------->   quote + nonce
                                            1. Verify AIK signature
                                            2. Compare to expected PCRs
                                            3. Policy decision: allow, deny, or quarantine
              <----------   verdict
```

#### 5.2 TPM Identity

The AIK is a 2048-bit RSA or P-256 ECDSA key. To verify a quote, the verifier needs to know *which* AIK to trust. The **Privacy CA** model: a CA certifies that the AIK belongs to a *genuine* TPM (via the EK signed by the TPM vendor's root). **Direct Anonymous Attestation (DAA)** is the alternative — the TPM proves its identity to the verifier without revealing a long-term key (used in FIDO2 / WebAuthn).

#### 5.3 TPM2_Certify vs TPM2_Quote

- `TPM2_Quote` — signs a *set of PCR values* with a chosen signing key. Most common attestation primitive.
- `TPM2_Certify` — signs a *TPM object* (key, NVRAM) with another key, proving it is TPM-resident and has a specific structure.

### 6. Sealing and Unsealing

TPMs can seal a key to a *PCR policy*:

```
TPM2_Seal(key_handle, policy { PCR[0] == X, PCR[4] == Y, ... }, data)
  → sealed_object: { encrypted_key }
TPM2_Unseal(sealed_object)
  → returns data, but only if current PCRs match the sealed policy
```

This is *binding*: a key is only usable when the boot state is correct. Example: a BitLocker volume key is sealed to PCRs 0, 2, 4, 11 — a *non-Microsoft* boot fails the PCR check, the key cannot be unsealed, the disk is unreadable.

### 7. Linux IMA (Integrity Measurement Architecture)

A Linux kernel subsystem that *measures* every executable before launch and extends PCR 10:

```
execve("ls", ...)  →  hash(binary, command line)  →  TPM2_PCR_Extend(10, hash)
```

The IMA measurement list is also written to a kernel-internal log (and optionally to TPM 2.0's *measurement log* via `ima-buf` template).

A remote verifier can replay the IMA log and reconstruct PCR 10's value; if the values match, the system is *known* to have run the claimed programs.

**Appraisal mode**: IMA can *deny* execution of binaries whose hash is not in a signed *IMA policy* (akin to a local secure boot for userspace).

### 8. Attested TLS (RFC 8446 + TPM)

Mutual TLS with a TPM-held key:
1. The TLS server presents a certificate with a TPM-protected private key.
2. The private key is non-exportable from the TPM.
3. A verifier (e.g., a Kubernetes admission controller) can require the server to *attest* that the key was generated inside a TPM, via a DAA signature or a vendor-specific extension.
4. This proves "the server is a genuine TPM-protected machine" — defeating credential theft from a software-only server.

### 9. TPM 2.0 Commands (Selected)

| Command | Use |
|---------|-----|
| `TPM2_Startup` | Initialize the TPM (STATE = SU or STATE = SU_CLEAR) |
| `TPM2_Shutdown` | Save state (for S3 sleep, hibernate) |
| `TPM2_PCR_Extend` | Extend a PCR with a new measurement |
| `TPM2_PCR_Read` | Read a PCR (with the public part of the bank) |
| `TPM2_Quote` | Sign a set of PCRs |
| `TPM2_Seal` / `TPM2_Unseal` | Bind data to a PCR policy |
| `TPM2_NV_DefineSpace` / `TPM2_NV_Write` / `TPM2_NV_Read` | Non-volatile storage |
| `TPM2_Certify` | Sign a TPM-resident object |
| `TPM2_GetRandom` | Read entropy from the TPM's RNG |
| `TPM2_HMAC` / `TPM2_Hash` | Use TPM's hash / HMAC engine |
| `TPM2_Create` / `TPM2_Load` | Create / load a key |
| `TPM2_Sign` | Sign with a TPM-loaded key |
| `TPM2_VerifySignature` | Verify with a TPM-loaded key |
| `TPM2_DictionaryAttackLockReset` | Reset the dictionary-attack counter |
| `TPM2_Clear` | Zeroize the TPM (full reset) |
| `TPM2_ChangeAuth` | Change the TPM owner / endorsement password |

### 10. TPM 2.0 in Practice (2024)

#### 10.1 BitLocker (Windows)
- Full-volume encryption key (FVEK) is sealed to a PCR set (typically PCR 7 for the Secure Boot state plus PCR 11 for BitLocker metadata).
- Boot sequence: UEFI → Secure Boot → measured boot → TPM unseals FVEK → OS loads.

#### 10.2 FileVault 2 (macOS)
- Uses a software-only "fTPM" in the Apple Silicon Secure Enclave.
- Recovery key + user password + escrow; no TPM 2.0 in the strict sense.

#### 10.3 Linux
- `tpm2-tss` (TPM Software Stack from Intel) provides userspace API.
- `tpm2-tools` (CLI).
- LUKS2 supports TPM2 unsealing: `systemd-cryptenroll --tpm2-device=auto`.
- IMA + EVM (Extended Verification Module) provide full-file integrity.

#### 10.4 FIDO2 / WebAuthn
- The "authenticator" in FIDO2 can be a TPM 2.0 with the *ctap2-hidmac* profile.
- The TPM holds a *credential private key* per user+site; the key is *non-exportable*.
- The browser (or a phishing-resistant client) signs a challenge with the TPM; the server verifies the signature against the public key registered at enrollment.
- Phishing-resistant because the *origin* (RP ID) is part of the signed challenge — a phishing site cannot reuse the response.

#### 10.5 Microsoft Pluton
A *processor-integrated* TPM-class security processor on modern Windows PCs. Holds BitLocker keys, Windows Hello credentials, attestation identity. Replaces the discrete TPM on many OEMs; *behaves* like a TPM 2.0 to the OS.

#### 10.6 Apple Secure Enclave
A separate processor in Apple devices that holds biometric data (Touch ID, Face ID), encryption keys, and attestation identities. *Not* a TPM, but a hardware root of trust with a similar role.

### 11. The TPM Endorsement Key and Privacy

The EK is a *permanent* identity — the TPM vendor burns a unique RSA or ECC key at manufacture. If exposed, the EK identifies *the chip*, which identifies *the device*, which identifies *the user* (via the device's other identities).

**Privacy mitigations**:
- **AIK** — derived from EK; the verifier trusts the AIK via the Privacy CA.
- **DAA** — Direct Anonymous Attestation; the TPM proves it is genuine without revealing its EK.
- **EK certificate** — kept offline; the vendor's root signs the EK cert; the cert is rarely transmitted.

### 12. Microsoft Measured Boot, UEFI PCR Mapping

Microsoft's attestation spec (Microsoft "PCR Mapping"):

| PCR | Mapped to (Windows) |
|-----|---------------------|
| 0 | Core System Firmware (BIOS) |
| 1 | Platform Firmware Extensions |
| 2 | Option ROMs |
| 3 | Option ROM Configuration |
| 4 | Boot Manager (MBR / EFI System Partition) |
| 5 | Boot Manager Configuration |
| 7 | Secure Boot state |

Windows 11 attestation (TPM 2.0 mandatory since 2021) requires PCRs 0, 1, 2, 3, 4, 5, 7, 11, 14.

### 13. TPM Failure Modes and Defenses

| Failure | Defense |
|---------|---------|
| TPM 2.0 absent | OS fails to start (Windows 11, certain Linux modes) |
| TPM tampered with (physical) | ZTC (zeroization) on tamper; EK is destroyed |
| TPM stolen (chip lifted) | EK certificate from chip; serial number matched to a stolen-device list |
| Brute-force PIN attack | TPM's *dictionary-attack lockout* (TPM 2.0 enforces exponential backoff) |
| Rollback (old PCR values replayed) | TPM2_Quote includes a *nonce* from the verifier |
| TOCTOU on measurement | PCR extend happens *before* control is passed to the measured code |
| Sneak attack on the firmware itself | Secure Boot's signature verification |
| Weak PCR policy | Policy should include all relevant PCRs (do not seal a key to PCR 0 alone — a BIOS that doesn't measure 4 leaves the OS uncontrolled) |

### 14. Attestation Use Cases (Enterprise)

1. **Conditional access** — Intune / Jamf / Okta refuses to grant VPN or SaaS access to devices whose PCRs do not match the corporate *known-good* profile.
2. **Kubernetes admission** — a webhook validates that the worker node's attestation is current.
3. **Supply chain** — verify that the firmware on a server is the OEM-signed version.
4. **Disk encryption** — BitLocker, LUKS, FileVault — sealed to PCRs.
5. **FIDO2** — phishing-resistant login.
6. **Public cloud** — Azure Confidential Computing (Intel TDX, AMD SEV-SNP, NVIDIA H100 CC) uses attestation to prove an enclave is running the expected code.

### 15. TPM and Quantum

- The EK is a 2048-bit RSA or P-256 ECDSA key. *Quantum-vulnerable*.
- TPM 2.0 spec (TCG) added *hybrid* primary keys: ECC + ML-DSA hybrid signatures for AIKs and quotes (TPM 2.0 Rev. 1.59+, 2023).
- Sealing is not affected (sealing is a symmetric primitive).

## CISO / Risk Manager View

**Board framing**: "Can we prove that the laptops and servers connecting to our network are running *our* approved software, signed by *our* approved keys, and have not been tampered with?" The answer is attestation. A device that *cannot* produce a valid attestation is treated as untrusted — no network, no data, no access.

**Hardware-root-of-trust program (CISO charter)**:
1. **Inventory** every endpoint (laptop, desktop, server, mobile, IoT) and its hardware root of trust (TPM 2.0, Pluton, Secure Enclave, Android StrongBox, iPhone SEP).
2. **Mandate TPM 2.0** for every new endpoint procurement; require attestation in the RFP.
3. **Enable Secure Boot** in *enforced* mode (not "audit") for every endpoint.
4. **Deploy attestation-aware conditional access** — Okta, Intune, Jamf, or an MDM that uses TPM attestation as an access decision input.
5. **Deploy BitLocker / LUKS2 / FileVault with TPM unsealing** for all data-bearing devices.
6. **Server attestation** — for any server running sensitive workloads, attest the boot PCRs and verify them against a known-good list.
7. **PQC roadmap** — track TPM 2.0 firmware updates that add hybrid PQC signatures; require them on procurement.
8. **Supply chain attestation** — for firmware, require SBOMs (Software Bill of Materials) and SLSA Level 3+ build provenance.

**Common audit findings**:
- Secure Boot disabled in BIOS ("it broke the dual-boot").
- TPM 1.2 (incompatible with Windows 11, attestation in modern Linux).
- TPM not used (BitLocker running on a USB key instead of sealing to TPM).
- Pin / owner password absent (no anti-dictionary-attack lockout; loss of anti-rollback).
- Firmware not measured (PCR 0 = 0 — the firmware was bypassed).
- Attestation policy too lenient (PCR 0 alone; ignores OS-level state).

**CISO-level callout**:
- *Hardware root of trust is the only defense against physical-access bootkit attacks*. A determined attacker with physical access *can* defeat software-only secure boot. A TPM with measured boot + remote attestation makes the attacker's success *visible* — the laptop can be silently quarantined by an MDM the moment its PCRs drift.
- *BitLocker + TPM is the single highest-ROI endpoint control* for laptop data-at-rest. Recovery key escrow to Active Directory / Entra ID is mandatory.
- *FIDO2 + TPM is the single highest-ROI authentication control* — phishing-resistant, no shared secrets, no replay, no SIM-swap. The board's "MFA fatigue" / "MFA bypass" problem is solved.
- *Server attestation (TPM + measured boot) is becoming the standard for confidential computing* — Intel TDX, AMD SEV-SNP, and NVIDIA H100 CC all rely on the same root-of-trust principles. A CISO's PQC-aware confidential computing roadmap includes attestation.

**Board question to prepare for**: "What percentage of our endpoints are TPM-attested? What is the conditional access policy for non-attested devices? How do we recover a device that fails attestation?" The CISO's three-quarter answer is the program.

## Related Connections
- Security architecture patterns: [[security-architecture-patterns]]
- Cryptography fundamentals: [[cryptography-fundamentals-and-math]]
- Asymmetric and PKI: [[asymmetric-encryption-and-pki]]
- FIPS 140-3: [[fips-140-3]]
- Post-quantum: [[post-quantum-cryptography]]
- Domain 5 (IAM) — uses TPM-backed keys (FIDO2)
- Domain 7 (operations) — attestation monitoring and incident response
- Domain hub: [[domain-03-security-architecture-and-engineering]]

## References
- TCG (Trusted Computing Group). *TPM Library Specification 2.0* (TCG Published, 2019).
- TCG. *PC Client Platform Firmware Profile Specification* (2021).
- UEFI Forum. *UEFI Specification 2.10*, § 27 — Secure Boot (2022).
- NIST SP 800-155 (BIOS Integrity Measurement Guidelines draft).
- NIST SP 800-193 (Platform Resiliency Guidelines, 2018).
- Microsoft. *Measured Boot and PCR Mappings* (Windows Hardware Compatibility Spec).
- Apple. *Apple Platform Security — Secure Enclave* (2024).
- Microsoft. *Pluton Security Processor* documentation.
- IBM. *Software Guard Extensions (SGX)* — enclave attestation (legacy reference).
- Intel. *Trust Domain Extensions (TDX)* and *Software Guard Extensions (SGX)* — confidential computing.
- AMD. *Secure Encrypted Virtualization — Secure Nested Paging (SEV-SNP)* — confidential computing.
- FIDO Alliance. *FIDO2 / WebAuthn Specification* (W3C + FIDO Alliance, 2018+).
- Linux Foundation. *tpm2-tss*, *tpm2-tools*, *ima-evm-utils*.
- J. Hendricks, L. van Doorn. *Secure Bootstrap for Linux* (MIT, 2004).
- Sailer, R. et al. (2004). *Design and Implementation of a TCG-based Integrity Measurement Architecture* (IBM).
- RFC 8446 — TLS 1.3 (attested TLS profile).
- NIST IR 8320 (Hardware-Enabled Security for Server Platforms).
- NSA. *Mitigations for Recently Disclosed Microsoft Netlogon and Secure Boot Vulnerabilities* (advisories).
- Cloudflare (2024). *Confidential Computing at Cloudflare*.
- Google (2024). *Titan — Hardware Root of Trust at Google*.
