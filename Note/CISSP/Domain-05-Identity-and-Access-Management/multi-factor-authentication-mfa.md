---
title: "Multi-Factor Authentication (MFA) — TOTP, FIDO2, Push, Risk-based"
layer: 3
domain: 5
tags:
  - cissp
  - domain-05
  - mfa
  - totp
  - hotp
  - fido2
  - webauthn
  - passkey
  - push
  - risk-based
  - step-up
  - sim-swap
related_domains:
  - "[[domain-01-security-and-risk-management]]"
  - "[[domain-04-communication-and-network-security]]"
  - "[[domain-07-security-operations]]"
  - "[[authentication-factors-and-mechanisms]]"
  - "[[access-control-attacks-and-mitigations]]"
  - "[[federation-sso-and-saml-oidc]]"
  - "[[kerberos-protocol-deep-dive]]"
  - "[[zero-trust-architecture-nist-800-207]]"
parent: authentication-factors-and-mechanisms
---

# Multi-Factor Authentication (MFA) — TOTP, FIDO2, Push, Risk-based

## Feynman Explanation

MFA is the rule that proving who you are requires two **different kinds** of evidence: something you *know* (a password) plus something you *have* (a phone or key) or something you *are* (a fingerprint). The reason it matters is that an attacker who steals your password still cannot get in without that second piece. The evolution of MFA over the last 15 years has been a slow retreat from "things the attacker can phish" (SMS codes, push prompts) toward "things the attacker cannot phish" (FIDO2 security keys and passkeys). For CISSP purposes, the actionable answer in 2024+ is **phishing-resistant MFA, default**.

## Technical Details

### 1. The Factor Model (canonical)

| Factor | Category | Examples | Properties |
|---|---|---|---|
| Type 1 | Knowledge | Password, PIN, passphrase, security Q | Cheap, revocable, **phishable** |
| Type 2 | Possession | OTP token, smart card, FIDO2 key, phone, TOTP app | Hard to copy, can be **stolen** / SIM-swapped |
| Type 3 | Inherence | Fingerprint, face, iris, voice | Always with user, **not revocable**, liveness required |
| Type 4 | Context / behavior | Geo, IP, device posture, time, typing rhythm | Implicit, **never a primary factor alone** |

> **Exam rule:** MFA = 2+ factors from **different** categories. Password + password ≠ MFA. Password + TOTP = MFA. Fingerprint + PIN on the same phone = MFA (different categories). Password + magic link emailed to the same mailbox = NOT MFA (Type 1 + Type 1).

### 2. MFA Mechanism Family Tree

```
MFA
├── Knowledge
│   └── Password + ???
├── Possession
│   ├── OOB channel
│   │   ├── SMS OTP  ────────── (deprecated by NIST 800-63B for new systems)
│   │   ├── Email OTP ───────── (acceptable, lower assurance)
│   │   └── Voice call OTP ──── (rare, social engineering risk)
│   ├── Local cryptographic
│   │   ├── HOTP (RFC 4226) ─── (event-based, counter)
│   │   ├── TOTP (RFC 6238) ─── (time-based, 30/60s window)
│   │   ├── OCRA (RFC 6287) ─── (challenge-response)
│   │   ├── Mobile push ─────── (server-pushed approval)
│   │   └── Smart card (PKI) ── (challenge-response with private key)
│   └── FIDO
│       ├── U2F (CTAP1) ─────── (legacy second factor only)
│       ├── FIDO2 / WebAuthn ── (first or second factor, phishing-resistant)
│       └── Passkey ─────────── (synced FIDO2 credential)
└── Inherence
    ├── Fingerprint
    ├── Face
    ├── Iris / palm / vein
    └── Voice
```

### 3. HOTP — HMAC-Based One-Time Password (RFC 4226)

- **Algorithm:** $\text{HOTP}(K, C) = \text{truncate}(\text{HMAC-SHA1}(K, C))$, where $K$ is the shared secret and $C$ is an 8-byte counter.
- **Counter** increments each use; out-of-sync requires resync.
- Default 6 digits, window of ±1 for tolerance.
- **Weakness:** the code is **phishable** (attacker captures it in transit and uses it within the validity window).

#### HOTP truncation (the bit-twiddling)

```python
import hmac, hashlib, struct

def hotp(K, C, digits=6):
    hmac_bytes = hmac.new(K, struct.pack(">Q", C), hashlib.sha1).digest()
    offset = hmac_bytes[-1] & 0x0F
    code = (struct.unpack(">I", hmac_bytes[offset:offset+4])[0] & 0x7FFFFFFF) % (10 ** digits)
    return str(code).zfill(digits)
```

### 4. TOTP — Time-Based One-Time Password (RFC 6238)

- **Algorithm:** $\text{TOTP}(K, T) = \text{HOTP}(K, \lfloor (T - T_0) / X \rfloor)$, where $T$ is the current Unix time, $T_0 = 0$, and $X$ is the time step (default 30 s).
- **Display:** the user reads a 6-digit code from an app (Google Authenticator, Authy, 1Password, Microsoft Authenticator).
- **Window:** typically ±1 (90 s of validity) for clock skew.
- **Weakness:** phishable in real time (proxy relay), but expires every 30 s.

#### TOTP QR provisioning URI

```
otpauth://totp/Example:alice@example.com?secret=JBSWY3DPEHPK3PXP&issuer=Example&algorithm=SHA1&digits=6&period=30
```

- `secret` is base32-encoded shared key (≥ 128 bits)
- `algorithm` is SHA-1, SHA-256, or SHA-512
- `digits` and `period` are configurable

> **Exam hook:** TOTP = time, HOTP = counter. TOTP needs a synchronized clock; HOTP needs no clock but suffers from counter drift.

### 5. Mobile Push (the modern default — and its problems)

| Step | Description |
|---|---|
| 1 | User enters password |
| 2 | Server pushes a "Did you just try to sign in?" prompt to the user's enrolled device |
| 3 | User taps Approve (or Deny) |
| 4 | App reports decision to server via APNs / FCM / vendor channel |
| 5 | Server issues the session |

**Weaknesses:**

- **MFA fatigue / push bombing** — attacker triggers dozens of pushes; user eventually approves.
- **Number-matching** (Microsoft / Google default) — user must type the number displayed on the screen into the push prompt. Defeats blind tap-to-approve.
- **No phishing resistance** — same problem as TOTP.
- **Account takeover via SIM swap** — if "MFA on a phone number" — different mechanism.
- **Push bombing bypass via social engineering of IT helpdesk** — "please reset my MFA."

### 6. Smart Card (PKI) — challenge-response with private key

- The card holds a private key; the corresponding certificate is published.
- The server issues a challenge; the card signs it with the private key; the server verifies the signature with the certificate.
- **Phishing-resistant:** the key never leaves the card; the challenge is bound to the server (and ideally to the channel — see mTLS).
- **Downsides:** needs PKI; reader hardware; user friction.

### 7. FIDO2 / WebAuthn — the modern gold standard

FIDO2 = **WebAuthn** (W3C browser API) + **CTAP2** (FIDO Alliance wire protocol from authenticator to client).

#### Key concepts

| Concept | Definition |
|---|---|
| **Authenticator** | Hardware key, phone, or built-in (TPM) that stores the private key and performs signing |
| **Relying Party (RP)** | The server / app that registered the credential |
| **rpId** | The RP's origin, e.g., `login.example.com` |
| **Attestation** | Proof of *which model* of authenticator produced the key |
| **User verification** | Local check: PIN, biometric, or touch; gates use of the key |
| **Discoverable credential (passkey)** | Credential lives on the authenticator with a discoverable ID; no username pre-input |
| **Non-discoverable** | Server must specify which credential to use (legacy) |

#### Registration (ceremony)

```
RP → browser: /attestation/options { challenge, rp: {id, name}, user: {id, name, displayName},
                                       pubKeyCredParams: [{type:"public-key", alg:-7}],
                                       authenticatorSelection: {residentKey:"preferred",
                                                                 userVerification:"preferred"},
                                       attestation: "direct" }

Browser → authenticator: navigator.credentials.create({publicKey})

Authenticator (on user touch / PIN / biometric):
    generates keypair (priv, pub)
    returns: { credentialId, clientDataJSON, attestationObject }

Browser → RP: POST /attestation/result {id, rawId, type:"public-key", response:{clientDataJSON, attestationObject}}

RP:
    verifies attestation certificate chain
    verifies clientDataJSON.challenge matches
    verifies clientDataJSON.origin == expected origin
    verifies rpIdHash in authenticatorData matches rpId
    stores public key indexed by credentialId
```

#### Authentication (assertion)

```
RP → browser: /assertion/options { challenge, rpId, allowCredentials: [{id, type}] }

Browser → authenticator: navigator.credentials.get({publicKey, mediation})

Authenticator (on user touch / PIN / biometric):
    looks up credential by rpId + credentialId
    signs challenge: signature = ECDSA(privKey, SHA256(authenticatorData || SHA256(clientDataJSON)))
    returns: { credentialId, clientDataJSON, authenticatorData, signature }

Browser → RP: POST /assertion/result {id, rawId, response:{clientDataJSON, authenticatorData, signature}}

RP verifies:
    challenge matches
    origin == expected
    rpIdHash matches
    user verified flag set if required
    signature valid under stored public key
    counter > previous counter (anti-clone)
```

#### Phishing resistance (why this is the answer)

The **rpId** is part of the signed data. A phishing site at `login-examp1e.com` has a **different** rpId; the authenticator will sign a different rpId hash; the legitimate RP will reject the assertion.

> **Exam hook:** FIDO2 is the only widely deployed authentication mechanism that is **inherently phishing-resistant** at the protocol level.

#### FIDO2 / WebAuthn modes

| Mode | Where private key lives | Synced? | Pros | Cons |
|---|---|---|---|---|
| **Roaming authenticator (security key)** | External (YubiKey, Titan) | No (typically) | Highest assurance; portable | Must carry; small UX cost |
| **Platform authenticator (TPM / Secure Enclave)** | On-device | Some (Apple iCloud Keychain, Google Password Manager, Windows Hello) | Convenient; no extra device | Less portable; tied to device OS |
| **Passkey (synced)** | Cloud-synced (iCloud, Google Password Manager, 1Password, Dashlane) | **Yes** | Convenience of passwords + phishing resistance | Trust the sync provider; revocable |

#### Counter, attestation, and clone detection

- A monotonic **signature counter** increments on each use. If the RP sees a counter that didn't increment, the key may have been cloned.
- **Attestation** is a chain back to the authenticator manufacturer (e.g., Yubico) — used in enterprise to enforce "must be YubiKey 5 series, not phone-based."
- **Enterprise attestation** can restrict which authenticators are accepted.

### 8. NIST SP 800-63B AAL Levels

| AAL | Requirements | Examples |
|---|---|---|
| **AAL1** | Single-factor (password OK) | Consumer apps |
| **AAL2** | MFA: ≥ 2 of {knowledge, possession, inherence} | Most enterprise apps |
| **AAL3** | MFA + phishing-resistant + hardware crypto + verifier impersonation resistance (channel binding) | Admin, high-value apps |

> **Exam hook:** AAL3 requires **phishing-resistant** authenticator + **hardware crypto** + **verifier impersonation resistance** (e.g., the authenticator binds to the channel so it cannot be tricked into authenticating to a proxy). Only FIDO2 with channel binding, smart card, or PKI meet AAL3.

### 9. Risk-based / Adaptive / Step-up Authentication

| Concept | Mechanism |
|---|---|
| **Risk-based** | Calculate a risk score from signals (geo, device, behavior); require stronger factor for higher risk |
| **Step-up** | A higher-risk action (wire transfer, MFA reset) demands a stronger factor than routine login |
| **Continuous** | Re-evaluate risk during the session; force re-auth on risk signal change |
| **Step-up to AAL3** | Admin task → require FIDO2 security key, not TOTP |

#### Risk signals

| Signal | Source | Higher-risk indicator |
|---|---|---|
| Geo (impossible travel) | IdP, VPN | Login from new country |
| Device posture | MDM | Jailbroken, out-of-date OS, no EDR |
| Network reputation | IP intel | Tor exit, known-proxy, residential ISP from unusual geo |
| Time of day | IdP | Login at 03:00 local |
| Behavior | UEBA | Atypical file access pattern |
| Risk signal change during session | UEBA | Same user, new device fingerprint → step-up |

### 10. SIM Swap — the SMS-OTP Killer

**Attack pattern:** attacker social-engineers the mobile carrier to port the victim's number to a new SIM. The SMS OTP now goes to the attacker's phone. The attacker resets passwords on accounts protected only by SMS.

**Defense:**

- Move from SMS to TOTP / FIDO2 / push (with number matching).
- Carrier-side: port-freeze PIN, in-person porting only, postpaid-account verification.
- Number-reuse monitoring; porting event detection (account-takeover surge after port).

### 11. Recovery and Reset (the back door)

| Control | Why |
|---|---|
| Recovery requires same-or-stronger factor as enrollment | The recovery flow is the back door — do not weaken it |
| Helpdesk identity proofing (out-of-band callback, video) | Prevents social engineering |
| Time-bound recovery codes | Self-service recovery with one-time codes |
| Manager attestation for high-risk resets | Audit + dual control |
| Account lockout after N recovery attempts | Slow down the attacker |
| Recovery is logged and alerted | Detection |

### 12. MFA in Federation (OIDC / SAML / Kerberos)

| Federation layer | MFA integration |
|---|---|
| **OIDC** | `acr_values=urn:mace:incommon:iap:silver` requests AAL2; `amr` claim reports methods used (`["pwd","mfa","fido2"]`) |
| **SAML 2.0** | `AuthnContextClassRef` carries the assurance class; SP requests higher via `<RequestedAuthnContext>` |
| **Kerberos / PKINIT** | Smart card signs the AS-REQ preauth (PKINIT) |
| **SAML + step-up** | SP re-issues AuthnRequest with higher AuthnContextClassRef |
| **OIDC + step-up** | Client re-requests with `prompt=login` or `max_age=0` |

### 13. MFA Failure Modes (the engineering list)

| Failure | Cause | Mitigation |
|---|---|---|
| Lockout | User loses phone / key | Recovery codes; backup factor; gMSA admin account for IT |
| Push prompt never arrives | APNs / FCM outage; user off-network | Fallback to TOTP; backup factor; voice call |
| Push prompt arrives but user can't approve | Device locked / dead battery | Recovery codes; passkey on multiple devices |
| TOTP out of sync | Clock drift | Allow wider resync window during enrollment only |
| FIDO2 key fails | USB port issue, driver | NFC; multiple keys per user; platform authenticator fallback |
| FIDO2 origin mismatch | User on subdomain / typo | Strict rpId; show user the rpId; CSP |
| Biometric false reject | Lotion, gloves, injury | Allow PIN fallback; multiple biometric modes |
| Helpdesk social engineering | "I lost my phone, please reset" | Mandatory out-of-band callback; ID proofing; manager approval |
| Recovery code leakage | Stored in password manager | Encrypt recovery codes; require step-up on first use |

### 14. MFA Program Metrics (the CISO dashboard)

| Metric | Target | Source |
|---|---|---|
| % of users on MFA | 100 % | IdP |
| % of users on **phishing-resistant** MFA | 100 % for admins, > 50 % for users in 18 months | IdP / hardware inventory |
| % of users with FIDO2 enrolled | > 80 % | IdP |
| # of MFA-fatigue attempts / 1 000 users / month | < 5 | SIEM |
| Push approval rate (average) | < 1 % of prompts approved | IdP |
| Recovery event rate | < 0.1 % of users / month | IdP |
| Mean time to MFA rollout per new app | < 30 days | IdP / project tracker |
| # of "SMS-only" accounts | 0 (target) | IdP |
| Helpdesk reset ticket volume | Down 50 % post-FIDO2 | Helpdesk |
| Sign-in success rate | > 99.5 % | IdP / app telemetry |

### 15. Exam Pattern Recap

- "MFA factors" — knowledge, possession, inherence (3 categories); context/behavior is a 4th, never primary.
- "TOTP" — RFC 6238, time-based, 30/60 s window, 6 digits, SHA-1/256/512, base32 secret.
- "HOTP" — RFC 4226, counter-based.
- "Push bombing" — defense: number matching, push rate limit, FIDO2.
- "SMS" — deprecated by NIST 800-63B for new systems; vulnerable to SIM swap, SS7.
- "Phishing-resistant MFA" — FIDO2 / passkey / PKI smart card.
- "AAL3" — phishing-resistant + hardware crypto + channel binding.
- "Recovery" — must require same-or-stronger factor as enrollment.
- "amr claim" — array of authentication methods used (OIDC).
- "acr" — Authentication Context Class Reference.
- "FIDO2 origin check" — the rpId binds the credential to the legitimate site.
- "Step-up" — higher risk → stronger factor; re-prompt on sensitive action.
- "SIM swap" — port the number, intercept SMS OTP; defense: leave SMS.

## CISO / Risk Manager View

MFA is the single most-invested security control of the 2010s and 2020s — and the **type** of MFA matters more than the **existence** of it. The CISO playbook is to **retire phishable MFA** (SMS, push, TOTP) and **default to phishing-resistant** MFA (FIDO2 / passkeys) over a 2-3 year plan.

| Investment | Defends against | ROI |
|---|---|---|
| FIDO2 security keys for **all admins** (Day 0) | Phishing, AiTM, push bombing, SIM swap | Highest; protect the keys to the kingdom first |
| FIDO2 platform authenticator (TPM / Windows Hello) for all staff | Phishing, AiTM | High; deploys with existing hardware refresh |
| Passkeys (synced) for consumer / customer | Phishing | High; Phishing-Protection-by-default |
| Retire SMS OTP for any production system | SIM swap, SS7 | High; rare legitimate pushback |
| Number matching on push | Push bombing | High; trivial to enable |
| Adaptive / risk-based step-up | Token theft, stolen session | Medium; requires UEBA investment |
| Backup factor + recovery policy | Lockout, helpdesk social engineering | Required to make the above work |
| Recovery-code MFA reset policy | Social engineering of IT | Required to close the back door |
| AAL3 for admin and high-value apps | Most advanced phishing | Highest; required for AAL3-eligible systems |

**Maturity ladder (MFA-specific):**

| Level | Name | Defining characteristic |
|---|---|---|
| 1 | None | Password only |
| 2 | Bolted-on | TOTP / push for some apps; SMS for others; legacy fallback everywhere |
| 3 | Standardized | FIDO2 for admins, TOTP / push for users, SMS removed from prod |
| 4 | Phishing-resistant | FIDO2 / passkeys for all staff; SMS removed; AAL3 for admin |
| 5 | Default + Adaptive | Passkey default for all new users; risk-based step-up; continuous evaluation; recovery requires same-or-stronger |

**The board narrative:** "Phishing-resistant MFA is the most important identity investment of the next three years. Every other identity control depends on the assumption that the second factor is not also stolen."

**Privileged risk concentration:** **admins first, customers second, general staff third**. The cost-benefit of phishing-resistant MFA is most favorable for the smallest population (admins) with the highest blast radius.

**Compliance hooks:** NIST 800-63B, NIST 800-63C (FAL), PSD2 SCA (Strong Customer Authentication requires ≥ 2 of {knowledge, possession, inherence}), PCI-DSS 8.4 (MFA for all access into the CDE), HIPAA 164.312(d), FedRAMP AC-2 / IA-2, ISO 27001 A.8.5.

## Related Connections

### Parent L2
- [[authentication-factors-and-mechanisms]] - The factor model and the strengths of each

### Sibling L3
- [[kerberos-protocol-deep-dive]] - PKINIT = smart-card preauth for Kerberos
- [[privileged-access-management-pam]] - The admin accounts that get FIDO2 first
- [[zero-trust-architecture-nist-800-207]] - Step-up + continuous re-evaluation is core to ZT

### Cross-Domain
- [[domain-01-security-and-risk-management]] - The risk-treatment decision that justifies MFA spend
- [[domain-04-communication-and-network-security]] - mTLS channel binding, 802.1X + cert for network MFA
- [[domain-07-security-operations]] - SIEM detects impossible-travel, MFA fatigue, recovery abuse
- [[domain-08-software-development-security]] - OIDC / OOB / WebAuthn in code; secrets in code

## Sources / References

- IETF RFC 4226 - HOTP: An HMAC-Based One-Time Password Algorithm
- IETF RFC 6238 - TOTP: Time-Based One-Time Password Algorithm
- IETF RFC 6287 - OCRA: OATH Challenge-Response Algorithm
- W3C Web Authentication (WebAuthn) - Level 2 / Level 3
- FIDO Alliance CTAP 2.1 Specification
- FIDO Alliance - FIDO2 / WebAuthn overview
- FIDO Alliance - Passkeys (synced credentials) white paper
- NIST SP 800-63B - Digital Identity Guidelines: Authentication and Lifecycle Management
- NIST SP 800-63C - Federation and Assertions
- NIST SP 800-63 Rev. 3 - Digital Identity Guidelines (Overview + 63A / 63B / 63C)
- Microsoft - "Number matching for MFA" (2022)
- Microsoft - "Authentication strength and AAL" (Entra ID)
- Google - "Security keys for employees" case study
- Yubico - "WebAuthn guide"
- OWASP ASVS v4.0 - Section V2 (Authentication)
- OWASP MFA Cheat Sheet
- (ISC)² CISSP CBK 2024 - Domain 5.4
- PSD2 - Strong Customer Authentication (RTS)
- PCI-DSS 4.0 - Requirement 8.4 (MFA for all access into the CDE)
