---
title: "Authentication Factors and Mechanisms"
layer: 2
domain: 5
tags:
  - cissp
  - domain-05
  - authentication
  - factors
  - passwordless
  - fido2
  - webauthn
  - biometrics
related_domains:
  - "[[domain-01-security-and-risk-management]]"
  - "[[domain-04-communication-and-network-security]]"
  - "[[domain-07-security-operations]]"
  - "[[multi-factor-authentication-mfa]]"
  - "[[kerberos-protocol-deep-dive]]"
  - "[[access-control-attacks-and-mitigations]]"
---

# Authentication Factors and Mechanisms

## Feynman Explanation

Authentication is the proof of "I am who I say I am." The strength of that proof is classified into three categories — **something you know** (a password), **something you have** (a phone or token), and **something you are** (a fingerprint). Each category is one "factor." The whole art of authentication is to choose factors that an attacker cannot simultaneously steal, copy, and replay. A password alone is one factor; a password plus a phone code is two factors; a FIDO2 security key that *also* checks the URL is the strongest single factor we have today because it cannot be phished at all.

## Technical Details

### 1. The Three (now four) Factor Categories

| Factor | "Something you ..." | Examples | Properties |
|---|---|---|---|
| **Type 1 — Knowledge** | know | Password, PIN, security question, passphrase | Cheapest; revocable; replayable / phishable |
| **Type 2 — Possession** | have | Smart card, OTP token, mobile phone, FIDO2 security key, TOTP app | Hard to copy remotely; can be lost / stolen / SIM-swapped |
| **Type 3 — Inherence** | are | Fingerprint, face, iris, palm, voice, gait | Always with the user; **not revocable**; requires liveness |
| **Type 4 — Context / Behavior** (modern) | do / are at | IP, geo, device posture, time-of-day, typing rhythm, mouse pattern | Implicit, risk-scored, never a primary factor alone |

> **Exam rule:** MFA means *two or more factors from different categories*. Two passwords (Type 1 + Type 1) is **not** MFA. Password + SMS code (Type 1 + Type 2) is MFA. Fingerprint + PIN on the same phone (Type 3 + Type 1, *different* categories) is MFA.

### 2. Passwords — Still Here, Still Weak

| Mechanism | Strength | Weakness | Exam hook |
|---|---|---|---|
| Plain password | Trivially phishable | Replay, brute force, credential stuffing | "Strongest single factor" → **never** |
| Salted hash (bcrypt, scrypt, Argon2) | Resists precomputation | Still subject to phishing and reuse | Storage control, not transmission control |
| PBKDF2 / bcrypt / Argon2id | Tunable cost | CPU work, no defense against phishing | Modern default; Argon2id = current best practice |
| Password manager-generated | High entropy | Single point of compromise (vault) | Better than human-chosen |
| Passphrase (4-7 random words) | ~60-90 bits effective entropy | Harder to type, dictionary if poor word choice | NIST SP 800-63B endorses length over complexity |

#### NIST SP 800-63B password rules (now the de-facto standard)

- **No periodic forced rotation** unless compromise is suspected (rolled back from earlier 60- or 90-day policy).
- **No composition rules** (no "must contain !@#") — length ≥ 8 (≥ 12 recommended for higher assurance).
- **Check passwords against breach corpuses** (HIBP-style) on creation.
- **Rate-limit and lockout** online attacks; throttle / use CAPTCHAs.
- **Allow paste** in password fields (password managers).
- **Show** password-strength meter; do not silently reject.

### 3. Possession Factors — The Practical Spectrum

| Type | Cryptographic model | Phishable? | Replayable? | Notes |
|---|---|---|---|---|
| **SMS OTP** | Out-of-band code | Yes (SS7, SIM swap) | Yes | Deprecated by NIST for new systems; still ubiquitous |
| **Email OTP** | Out-of-band code | Yes (mailbox takeover) | Yes | Better than nothing, worse than SMS |
| **TOTP (RFC 6238)** | Shared secret, HMAC of time | Yes (real-time phishing kit) | Yes (within 30-60 s) | Standard for "good enough" MFA |
| **HOTP (RFC 4226)** | Shared secret, counter | Yes | Yes | Event-based; less common than TOTP |
| **Push notification** | Server-pushed approval | Yes (MFA fatigue, push spam) | Yes | Highly vulnerable to "prompt bombing" |
| **Smart card (PKI)** | Challenge-response with private key | No (private key never leaves card) | No (signature includes challenge) | Strong, but needs PKI |
| **FIDO2 / WebAuthn (passkey)** | Public-key, origin-bound, attestation | **No** (origin check is in the spec) | No (challenge is unique per request) | The current "gold standard" |

> See L3 [[multi-factor-authentication-mfa]] for the deep dive on TOTP / FIDO2 / push / risk-based.

### 4. Biometrics — Math of False Matches and False Rejects

A biometric system produces two errors. Their relationship is the **ROC curve**, and tuning one always worsens the other.

| Metric | Definition | Why it matters |
|---|---|---|
| **FAR** (False Acceptance Rate) | Impostor accepted as genuine | Security; lower is safer |
| **FRR** (False Rejection Rate) | Genuine user rejected | Usability; lower is friendlier |
| **CER / EER** (Crossover / Equal-Error Rate) | Threshold where FAR = FRR | Headline "accuracy" number; lower is better |
| **Failure-to-Enroll (FTE)** | % of users who can't register | Operational headache (lotion on fingers, gloves) |

For fingerprinted devices, modern sensors hit **FAR < 0.001 % (1 in 100 000)** and **FRR < 1 %** — but the FAR figure is *only* meaningful when paired with **liveness detection** (anti-spoof against a printed photo, a 3D mask, or a severed finger).

#### Biometric comparison table

| Modality | FAR (typical) | FRR (typical) | Spoof risk | Best use |
|---|---|---|---|---|
| Fingerprint (capacitive) | 0.001 % | 0.6 % | Latent prints, fake fingers | Phone unlock, KYC |
| Face (2D IR + dot projector) | 0.001 % | 0.3 % | 2D photo, identical twin | Phone unlock, border |
| Iris | 0.0001 % | 0.5 % | Contact lens with printed iris | High-security physical |
| Voice | 0.5 % | 2-5 % | Recording, synthesis (AI) | Call-center auth |
| Vein (palm / finger) | 0.0001 % | 0.1 % | Low (internal pattern) | ATMs, healthcare |

> **Cancellable biometrics:** store a *transformed* template (e.g., bio-hash) so the original can be "revoked" by re-issuing a new transformation. Solves the "you can't un-share a fingerprint" problem.

### 5. Single Sign-On (SSO) and Federated Authentication

| Protocol | Purpose | Token format | Used by |
|---|---|---|---|
| **Kerberos** | Ticket-based SSO inside a realm | Service ticket (TGT-based) | Active Directory, MIT Kerberos |
| **SAML 2.0** | Browser SSO between IdP and SP | XML assertion | Enterprise B2B / B2E |
| **OAuth 2.0** | Delegated authorization (scopes) | Bearer access token | API / mobile / SaaS |
| **OIDC** | Identity on top of OAuth 2.0 | JWT `id_token` | Modern consumer / SaaS |
| **WS-Federation** | Older Microsoft federation | SAML token | Legacy SharePoint / ADFS |

See L2 [[federation-sso-and-saml-oidc]] for the full picture.

### 6. Passwordless and FIDO2 / WebAuthn — the 2024+ default

**FIDO2** is the W3C / FIDO Alliance pair:

- **WebAuthn (W3C)** — browser API. `navigator.credentials.create()` and `.get()`.
- **CTAP2 (FIDO)** — the wire protocol from authenticator (security key, phone) to client.

Authentication flow:

```
User clicks "Sign in"
        │
        ▼
RP (Relying Party) → server: /login → returns challenge (random nonce), rpId
        │
        ▼
Browser → Authenticator: navigator.credentials.get({publicKey, challenge, rpId})
        │
        ▼
Authenticator verifies user (PIN, biometric, touch)
        │
        ▼
Authenticator signs challenge with private key (origin-bound)
        │
        ▼
Browser → RP: signed assertion {id, rawId, response: {clientDataJSON, authenticatorData, signature}}
        │
        ▼
RP verifies signature using stored public key + checks origin in clientData
```

Key properties:

- **Origin-bound** — the authenticator embeds the RP ID (e.g., `login.example.com`) in the signed assertion. A phishing site at `login-examp1e.com` produces a different origin → signature is invalid. **Phishing resistance is the killer feature.**
- **Private key never leaves the authenticator** — no shared secret to steal.
- **Counter / signature counter** — increments on each use; cloning detection.
- **Attestation** — proves *which model* of authenticator produced the key (helps enterprise policy: "must be YubiKey 5, not phone-based passkey").
- **Discoverable credentials (passkeys)** — the credential lives on the authenticator with a discoverable ID; no need to provide the username first.

### 7. Mutual Authentication and Channel Binding

| Term | Definition |
|---|---|
| **Unilateral auth** | Only client proves identity to server (most common) |
| **Mutual auth (mTLS)** | Both sides present certificates; the *channel* is bound to identity |
| **Channel binding** (e.g., `tls-server-end-point` for SASL) | Auth proof is bound to the TLS channel so it cannot be relayed |
| **Channel binding for OAuth** (RFC 8705, `token_binding`, `dpop`) | Bind access tokens to a key on the device; mitigates token theft |

### 8. Account, Session, and Registration Management

- **Identity proofing** (IAL1/2/3 in NIST 800-63A) — establishing the *real-world* identity behind the digital one.
- **Session management** — server-issued session ID, secure cookie (`HttpOnly`, `Secure`, `SameSite`), idle timeout, absolute timeout.
- **Re-authentication on sensitive actions** — step-up auth, time-based re-prompt, or step-up to a stronger factor.
- **Account recovery** — the most-attacked flow in any IdP. Recovery should require at least as strong a factor as the original enrollment.

### 9. Exam-Relevant Attack Surfaces

| Attack | Vector | Mitigation |
|---|---|---|
| Brute force / dictionary | Online guessing | Rate-limit, lockout, CAPTCHAs, breached-password check |
| Credential stuffing | Reused creds from another breach | MFA, breach check on login, risk-based step-up |
| Phishing | Real-time proxy or fake site | Phishing-resistant MFA (FIDO2), training (low ROI) |
| MFA fatigue / push bombing | Repeated push prompts until user approves | Number matching, MFA rate limit, FIDO2 |
| SIM swap | Port victim's number to attacker SIM | Move from SMS to TOTP / FIDO2 |
| Session hijack | Steal cookie / bearer token | Short TTL, refresh tokens, `HttpOnly` + `Secure` cookies, `SameSite=Strict`, device binding |
| Token replay | Replay a captured token | Short TTL, one-time use, audience binding, DPoP / mTLS |
| Kerberoasting | Request SPN ticket, crack offline | Long strong service passwords, AES, Managed Service Accounts (gMSA) |
| Golden ticket | Forge Kerberos TGT | Rotate `krbtgt` twice, monitor PAC anomalies |

See L2 [[access-control-attacks-and-mitigations]] for the full table.

## CISO / Risk Manager View

Authentication is the single highest-ROI control in any security program — but the *type* of authentication matters more than the *existence* of MFA. From the boardroom:

| Conversation | What to push for | Why |
|---|---|---|
| "Do we have MFA?" | **Phishing-resistant MFA** (FIDO2 / passkeys / PKI smart card) for admins, then for all users | SMS / push / TOTP are *better than nothing* but all have credible bypass attacks |
| "Is our password policy strong?" | Length ≥ 12, breach check on login, no forced rotation, allow managers / passphrases | NIST 800-63B has retired most of the 2010-era complexity rules |
| "Are we ready for passwordless?" | Pilot passkeys on the highest-value apps, retire SMS first | Phishing resistance is the procurement argument |
| "How exposed are we to credential theft?" | Track credential-stuffing, MFA-fatigue, session-hijack incidents per 1 000 users | Move the metric from "MFA coverage %" to "phish-resistant MFA coverage %" |
| "What's our recovery story?" | Recovery should require the same or stronger factor as enrollment | Recovery is the back door; make it as strong as the front door |

**Identity assurance ladder for executives:** "We know who you are" goes from **self-asserted** (low) → **verified by one factor** (low) → **multi-factor** (medium) → **phishing-resistant multi-factor** (high) → **cryptographic identity bound to a vetted device** (highest).

**Privileged accounts get the strictest policy first.** A FIDO2-only rule for Domain Admin / root / cloud-break-glass is the cheapest, highest-impact upgrade a CISO can ship in a quarter.

## Related Connections

### Sibling L2
- [[authorization-models]] - Once authenticated, what may the subject do?
- [[identity-lifecycle-and-provisioning]] - How the account was created matters for trust in the auth
- [[federation-sso-and-saml-oidc]] - Federated auth reduces per-app secrets
- [[access-control-attacks-and-mitigations]] - The threat model against these factors

### L3
- [[multi-factor-authentication-mfa]] - TOTP / HOTP / FIDO2 / push / risk-based in detail
- [[kerberos-protocol-deep-dive]] - Ticket-based authentication for AD / Windows
- [[privileged-access-management-pam]] - Highest-risk accounts need the strongest factors

### Cross-Domain
- [[domain-01-security-and-risk-management]] - Authentication strength is a risk-treatment decision
- [[domain-04-communication-and-network-security]] - mTLS, 802.1X, RADIUS / TACACS+ for network access
- [[domain-07-security-operations]] - UEBA, impossible-travel, brute-force detection in the SIEM
- [[domain-08-software-development-security]] - OIDC / OAuth in apps; secrets in code; JWT pitfalls

## Sources / References

- NIST SP 800-63 Rev. 3 - Digital Identity Guidelines (Overview + 63A / 63B / 63C)
- NIST SP 800-63B - Authentication and Lifecycle Management
- NIST SP 800-63C - Federation and Assertions
- W3C Web Authentication (WebAuthn) - Level 2 Recommendation
- FIDO Alliance CTAP 2.1 Specification
- IETF RFC 4226 - HOTP: An HMAC-Based One-Time Password Algorithm
- IETF RFC 6238 - TOTP: Time-Based One-Time Password Algorithm
- IETF RFC 8705 - OAuth 2.0 Mutual-TLS Client Authentication and Certificate-Bound Access Tokens
- OWASP Authentication Cheat Sheet
- OWASP ASVS v4.0 - Section V2 (Authentication)
- (ISC)² CISSP CBK 2024 - Domain 5
