---
title: "Federation, SSO, and SAML / OAuth / OIDC"
layer: 2
domain: 5
tags:
  - cissp
  - domain-05
  - federation
  - sso
  - saml
  - oauth
  - oidc
  - jwt
  - claims
  - ws-fed
  - scim
related_domains:
  - "[[domain-04-communication-and-network-security]]"
  - "[[domain-08-software-development-security]]"
  - "[[authentication-factors-and-mechanisms]]"
  - "[[identity-lifecycle-and-provisioning]]"
  - "[[zero-trust-architecture-nist-800-207]]"
---

# Federation, SSO, and SAML / OAuth / OIDC

## Feynman Explanation

Single Sign-On is the idea that you log in **once** and then move between apps without re-typing your password — the apps *trust* the login system you used. Federation is SSO across **different organizations** (your company logs into a partner's portal without a new account). The three protocols that actually do this are SAML (the enterprise classic), OAuth 2.0 (the authorization plumbing for APIs), and OIDC (a thin identity layer on top of OAuth for modern apps). They look similar on the surface — they all hand out some kind of "token" — but each answers a different question. Knowing which protocol is the right tool for which job is one of the most-tested pieces of Domain 5.

## Technical Details

### 1. The Three Big Protocols — at a Glance

| Property | **SAML 2.0** | **OAuth 2.0** | **OIDC** |
|---|---|---|---|
| Purpose | Authentication (browser SSO) | Authorization (delegation) | Authentication (on top of OAuth) |
| Token type | SAML assertion (XML) | Access token (opaque or JWT) | `id_token` (JWT) + access token |
| Years / standard | OASIS, 2005 | IETF RFC 6749, 2012 | OpenID Foundation, 2014 (on top of OAuth) |
| Wire format | XML (SAMLPOST / Redirect) | JSON over HTTP | JSON over HTTP, JWT (JWS) |
| Typical use | Enterprise B2B / B2E browser SSO | API authorization, "Sign in with Google" | Modern consumer / SaaS SSO |
| Who owns the credential | IdP | Authorization server | IdP (still the OAuth AS) |
| Subject identifier | `NameID` | `sub` (resource owner) | `sub` (end user) |
| Audience | `Audience` element | `aud` claim | `aud` claim (set to client_id) |
| Who can be IdP | SAML IdP | n/a (AS) | OIDC OP (provider) |

> **Exam hooks:**
> - "OAuth 2.0 is an **authorization** framework, **not** an authentication protocol." (a famous exam trap)
> - "OIDC adds identity (id_token) on top of OAuth 2.0."
> - "SAML is XML-based; OIDC is JSON/JWT-based."

### 2. SAML 2.0 — the Enterprise Classic

**Vocabulary**

| Term | Definition |
|---|---|
| **IdP** (Identity Provider) | Authenticates the user, asserts identity to the SP |
| **SP** (Service Provider) | The app that consumes the assertion |
| **Principal** | The user (or service) |
| **Assertion** | Signed XML statement: "user X is authenticated at time T with method M, has attributes A₁, A₂" |
| **AuthnRequest** | SP → IdP request for an assertion |
| **Response** | IdP → SP, contains the assertion |
| **Binding** | How messages are transported — HTTP Redirect, HTTP POST, Artifact |
| **SLO** (Single Logout) | Logout propagated across all SPs |

**Browser SSO Profile (SP-initiated, POST binding)** — the most common flow:

```
User                Browser                  SP                    IdP
 │                     │                      │                      │
 │  1. GET /app ──────▶│ ────────▶            │                      │
 │                     │                      │                      │
 │                     │◀── 2. 302 + SAMLRequest (AuthnRequest) ──────│
 │                     │                                                  │
 │                     │──── 3. GET IdP?SAMLRequest=... ─────────────────▶
 │                     │                                                  │
 │                     │  4. login (if no SSO session) + MFA             │
 │                     │                                                  │
 │                     │◀── 5. 302 + SAMLResponse (signed assertion) ────│
 │                     │                                                  │
 │                     │──── 6. POST SAMLResponse ─────────────────────▶ │
 │                     │                                                  │
 │  7. logged in ────▶ │ ◀── 8. 200 OK /app ─────                       │
```

**The assertion** (simplified):

```xml
<saml:Assertion ID="_a1b2" IssueInstant="2026-07-08T10:00:00Z" Version="2.0"
                xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">
  <saml:Issuer>https://idp.example.com</saml:Issuer>
  <ds:Signature>...sign of canonicalized XML...</ds:Signature>
  <saml:Subject>
    <saml:NameID Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress">
      alice@example.com
    </saml:NameID>
    <saml:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
      <saml:SubjectConfirmationData NotOnOrAfter="2026-07-08T10:05:00Z"
                                    Recipient="https://sp.example.com/acs"
                                    InResponseTo="_req123"/>
    </saml:SubjectConfirmation>
  </saml:Subject>
  <saml:Conditions NotBefore="..." NotOnOrAfter="...">
    <saml:AudienceRestriction>
      <saml:Audience>https://sp.example.com</saml:Audience>
    </saml:AudienceRestriction>
  </saml:Conditions>
  <saml:AuthnStatement AuthnInstant="2026-07-08T10:00:00Z"
                       SessionIndex="_sess1">
    <saml:AuthnContext>
      <saml:AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport
      </saml:AuthnContextClassRef>
    </saml:AuthnContext>
  </saml:AuthnStatement>
  <saml:AttributeStatement>
    <saml:Attribute Name="role">
      <saml:AttributeValue>manager</saml:AttributeValue>
    </saml:Attribute>
  </saml:AttributeStatement>
</saml:Assertion>
```

**Security controls (SAML-specific exam hooks)**

- **XML Signature wrapping (XSW)** — attacker swaps the signed assertion for a forged one. Defend with strict schema validation and unique `ID` reference.
- **Replay** — assertions are bearer tokens. Defend with `NotOnOrAfter`, `InResponseTo`, and one-time `ID` tracking.
- **Signature must cover `Issuer` + assertion ID** — otherwise the signature can be moved to another assertion.
- **AudienceRestriction** — the SP must reject assertions not aimed at it.
- **SAMLResponse signing + Assertion signing** — both should be on; the SP verifies both.

### 3. OAuth 2.0 — Authorization Framework (RFC 6749)

OAuth 2.0 lets a user grant a **client** limited access to a **resource** owned by the user, held by a **resource server**, without sharing the user's password.

| Role | Definition |
|---|---|
| **Resource Owner** | The user |
| **Client** | The app that wants to act on the user's behalf |
| **Authorization Server (AS)** | Issues tokens; authenticates the user |
| **Resource Server (RS)** | Hosts the protected resource; accepts tokens |

#### Authorization Code Flow (the safe, default one)

```
User              Client (app)              Authorization Server (AS)        Resource Server (RS)
 │                    │                              │                              │
 │  1. click "log in" │                              │                              │
 │  ────────────────▶ │                              │                              │
 │                    │── 2. 302 to /authorize?response_type=code&client_id=…&redirect_uri=…&scope=…&state=…&code_challenge=…──▶
 │                    │                              │                              │
 │                    │  3. login + consent          │                              │
 │                    │◀──── 4. 302 with ?code=AUTH_CODE&state=… ──                │
 │                    │                              │                              │
 │                    │── 5. POST /token (code + code_verifier, client_secret) ──▶ │
 │                    │◀── 6. {access_token, refresh_token, id_token? (OIDC), expires_in} ──
 │                    │                              │                              │
 │                    │── 7. GET /api/me Authorization: Bearer ACCESS_TOKEN ──────▶│
 │                    │◀── 8. 200 OK {…} ────────────│                              │
```

**Key parameters (must know):**

| Parameter | Purpose | Exam hook |
|---|---|---|
| `response_type=code` | Use the auth code flow | The implicit flow (`response_type=token`) is **deprecated** |
| `client_id`, `client_secret` | Client authentication at /token | Public clients use PKCE instead of a secret |
| `redirect_uri` | Where the AS sends the response | Must be **exact match** — the #1 OAuth vuln |
| `scope` | What the token is allowed to do | `openid profile email` for OIDC; `read:files` for API |
| `state` | CSRF protection | **MUST** be present and validated |
| `PKCE` (`code_challenge`, `code_verifier`) | Proof-of-possession for public clients (mobile, SPA) | S256 = base64url(SHA256(verifier)) |

#### Tokens in OAuth 2.0

| Token | Format | Lifetime | Purpose |
|---|---|---|---|
| **Access token** | Opaque or JWT | Short (5 min - 1 h) | Sent to RS to access resource |
| **Refresh token** | Opaque | Long (days - 30 d) | Exchanged at /token for new access token |
| **Authorization code** | Opaque, single-use | ~30 s | One-time ticket to redeem for tokens |

#### Grant types (which one for which client)

| Grant | Client type | Use |
|---|---|---|
| **Authorization Code (+ PKCE)** | Confidential or public | **Default** for web, SPA, mobile, native |
| **Client Credentials** | Service-to-service | Machine identity, no user |
| **Device Code** | Input-limited (TV, CLI) | Device with browser elsewhere |
| **Refresh Token** | Any | Get a new access token without re-login |
| ~~**Implicit**~~ | ~~Browser SPA~~ | **Deprecated** — use auth code + PKCE |
| ~~**Resource Owner Password (ROPG)**~~ | ~~Trusted first-party~~ | **Discouraged** — only for legacy migration |

> **Exam hook:** "**OAuth 2.0 does NOT define how the user authenticates to the AS**." That's why OIDC exists.

### 4. OIDC — OpenID Connect (on top of OAuth 2.0)

OIDC adds an **identity layer** to OAuth 2.0: the AS returns an `id_token` (a signed JWT) that the client can verify locally.

#### Required claims in the `id_token` (a JWT)

| Claim | Meaning |
|---|---|
| `iss` | Issuer (the OP URL) |
| `sub` | Subject — unique, stable user ID at the OP |
| `aud` | Audience — must equal `client_id` |
| `exp` | Expiry (Unix seconds) |
| `iat` | Issued-at |
| `auth_time` | When the user last authenticated |
| `nonce` | One-time value tied to the request — defeats replay |

#### Optional standard claims

`name`, `email`, `email_verified`, `phone_number`, `address`, `groups`, `acr` (Authentication Context Class Reference — e.g., `urn:mace:incommon:iap:silver`), `amr` (Authentication Methods References — array: `["pwd","mfa","fido2"]`).

#### Flow additions vs pure OAuth

- `scope=openid` (mandatory) + optional `profile email address phone groups`
- AS returns `id_token` alongside `access_token` (and `refresh_token`)
- Client validates `id_token` signature with JWKS published at `iss/.well-known/jwks.json`
- Optional **UserInfo endpoint** (`/userinfo`) for fresh claims with the access token

#### Discovery: the `.well-known/openid-configuration` document

```json
{
  "issuer": "https://login.example.com",
  "authorization_endpoint": "https://login.example.com/oauth2/authorize",
  "token_endpoint": "https://login.example.com/oauth2/token",
  "userinfo_endpoint": "https://login.example.com/oauth2/userinfo",
  "jwks_uri": "https://login.example.com/.well-known/jwks.json",
  "scopes_supported": ["openid","profile","email","groups"],
  "response_types_supported": ["code"],
  "id_token_signing_alg_values_supported": ["RS256","ES256"],
  "token_endpoint_auth_methods_supported": ["client_secret_basic","private_key_jwt"]
}
```

### 5. JWT — JSON Web Token (RFC 7519)

A JWT is three base64url parts: `header.payload.signature` (a JWS).

```json
// Header
{ "alg": "RS256", "typ": "JWT", "kid": "key-2026-01" }
// Payload (claims)
{
  "iss": "https://login.example.com",
  "sub": "user-1234",
  "aud": "app-finance",
  "exp": 1752000000,
  "iat": 1751996400,
  "scope": "openid profile email",
  "amr": ["pwd","mfa"]
}
```

#### Critical JWT pitfalls (exam-relevant)

| Pitfall | Attack | Defense |
|---|---|---|
| `alg=none` accepted | Attacker forges unsigned token | Whitelist `RS256`/`ES256`, never accept `none` |
| `alg=HS256` with public key as secret | Server uses RSA pub key as HMAC secret → forgery | Restrict alg to `RS*`/`ES*` for asymmetric keys |
| Weak signing secret | Brute force | 256+ bit random secret for `HS*` |
| Missing `exp` / `aud` | Replay / confused-deputy | Validate all claims |
| `kid` SQL injection / path traversal | Read arbitrary file | Whitelist `kid` values |
| Long-lived tokens | Stolen token has long window of misuse | Short TTL (≤ 1 h), refresh tokens, sender-constrained (DPoP / mTLS) |

### 6. WS-Federation (WS-Fed) — the Legacy Path

- Microsoft / .NET heritage. Used by ADFS in older SharePoint / Office 365 stacks.
- Uses **SAML 2.0 tokens** but a SOAP / WS-Trust wire format.
- **Active** (SOAP) and **Passive** (browser redirect) profiles.
- Being phased out; OIDC is the Microsoft-recommended replacement (Entra ID supports both).

### 7. SCIM — provisioning, not auth

| Property | Value |
|---|---|
| Purpose | Push lifecycle events (create / update / disable) from IdP to SaaS |
| Wire | REST + JSON |
| Auth | Bearer token (usually OAuth client credential) |
| Schema | RFC 7643 (User, Group, EnterpriseUser) |
| Operations | POST, GET, PUT, PATCH (JSON Patch), DELETE on `/Users`, `/Groups` |
| Mapping | `userName` ↔ email; `active=false` ↔ disable |

> **Exam hook:** SCIM is **provisioning**, not authentication. SCIM does not log users in. The IdP uses SCIM to *push* the user account to the SaaS, then SAML or OIDC for the actual login.

### 8. Federation Trust Models

| Model | Definition | Used by |
|---|---|---|
| **Hub-and-spoke** | One central IdP, many SPs | Enterprise with central IdP (Okta, Entra) |
| **Mesh** | Direct trust between each pair | B2B federation, small partner network |
| **Federation proxy** | A broker (e.g., F5, Radiant Logic) terminates and re-asserts | M&A with many IdPs and SPs |
| **Chained federation** | IdP → proxy → IdP → SP (e.g., eduGAIN) | Academic federation, government |

### 9. Trust Frameworks and Levels of Assurance

| Framework | What it defines | Examples |
|---|---|---|
| **NIST 800-63 AAL** (1, 2, 3) | Authentication strength | AAL2 = MFA, AAL3 = phishing-resistant + hardware |
| **NIST 800-63 IAL** (1, 2, 3) | Identity-proofing strength | IAL2 = remote with one ID |
| **NIST 800-63 FAL** (1, 2, 3) | Federation assertion strength | FAL2 = signed, encrypted, audience-bound |
| **eIDAS** (EU) | Levels: Low / Substantial / High | eIDAS High = qualified cert + secure device |
| **Kantara** | SAML / OIDC profile mapping to NIST AAL | Interop testing |

#### Mapping exam: what FAL means in practice

| FAL | What's required |
|---|---|
| FAL1 | Signed assertion; audience = SP |
| FAL2 | FAL1 + signed by a cryptographically verified IdP (e.g., PKI cert) + audience + replay protection (`AuthnRequest` ID / `nonce`) |
| FAL3 | FAL2 + assertion encrypted to SP + holder-of-key (proof-of-possession) |

### 10. Just-in-Time (JIT) Provisioning via Federation

Many IdPs (Okta, Entra, Auth0) support **JIT provisioning** — when a user successfully authenticates via SAML / OIDC and the SP does not have an account, the SP creates one on the fly using the asserted attributes. This avoids the SCIM round-trip for first login.

Risks:
- Accounts created from assertions can drift from HR; ownership unclear.
- Terminated users' JIT-created accounts may persist if deprovisioning is not symmetric.

> **Exam hook:** JIT provisioning + deprovisioning-via-federation = still need a process for *delete* on termination.

### 11. Token Theft and Defenses (the modern attack surface)

| Attack | Defense |
|---|---|
| Stolen access token from logs | Short TTL, structured logging, secrets-redaction |
| Token replay | Short TTL, `nonce` (OIDC), `InResponseTo` (SAML), DPoP / mTLS binding |
| Phishing (consent screen) | Phishing-resistant MFA, restricted `redirect_uri`, `state`/`nonce` |
| Open redirect via `redirect_uri` | Exact-match `redirect_uri` allow-list |
| Mix-up attack (confused deputy between IdPs) | Issuer validation; per-IdP `state` encryption |
| Token leakage via JS | Keep tokens out of the browser; backend-for-frontend (BFF) pattern |
| Compromised refresh token | Refresh-token rotation with reuse detection; revoke family on anomaly |

### 12. WS-Security (for completeness)

- SOAP-level security: XML Signature + XML Encryption inside the SOAP envelope.
- Used in legacy enterprise SOA. Rarely seen in 2024 outside of banking / government.

## CISO / Risk Manager View

Federation is what turns IAM from "lots of local accounts" into "one identity, many apps." It is the **single biggest defensive multiplier** in modern enterprise IAM: fewer passwords to steal, fewer accounts to deprovision, one place to enforce MFA. It is also the **single biggest integration risk** — every federation link is a trust relationship that must be governed.

| Conversation | Board framing | What to push for |
|---|---|---|
| "Are we federated?" | "How many passwords does the average employee have?" | Target: 1 IdP, ≤ 2 SaaS exceptions, rest federated |
| "Which protocol for new apps?" | "Are we picking the right tool?" | Default to OIDC for new apps; SAML for legacy enterprise |
| "Is OAuth safe?" | "Doesn't OAuth give attackers tokens?" | OAuth is fine when used with PKCE, exact-match redirect, short TTL, refresh-token rotation |
| "How do we trust partners?" | "What's the B2B story?" | Federation proxy + trust framework (Kantara / eIDAS) |
| "What's the AAL of our most sensitive app?" | "Can a phisher get in?" | AAL3 (phishing-resistant MFA) for admin and high-value apps |
| "Do we have visibility into all federation links?" | "Can we list every IdP we trust?" | Maintain a federation registry; audit each link annually |

**Maturity ladder (federation-specific):**

| Level | Name | Defining characteristic |
|---|---|---|
| 1 | Siloed | Each app has its own user store |
| 2 | Centralized IdP | One IdP, SAML / OIDC, MFA enforced |
| 3 | Federated | B2B federation, JIT provisioning, claim-based authz |
| 4 | Trust-frameworked | AAL/FAL levels matched to app criticality; audited trust registry |
| 5 | Adaptive | Short-lived tokens, sender-constrained (DPoP/mTLS), phishing-resistant MFA, federation-as-policy |

**Privileged risk concentration:** the **AS** (Authorization Server) is the single most valuable crown jewel in the entire IAM estate. Compromise of the AS = compromise of every app that trusts it. The CISO must invest in the AS the way the company invests in the CA / HSM.

**Compliance hooks:** eIDAS (electronic ID in EU), FedRAMP (US gov federation profile), HIPAA (federation of healthcare identities via DirectTrust / Carequality), financial Open Banking (PSD2 with FAPI profile for OAuth).

## Related Connections

### Sibling L2
- [[authentication-factors-and-mechanisms]] - Federation delivers the auth *result*, not the auth *factor*
- [[authorization-models]] - Claims from federation are inputs to ABAC policies
- [[identity-lifecycle-and-provisioning]] - SCIM is the provisioning protocol that complements federation
- [[access-control-attacks-and-mitigations]] - OAuth / OIDC / SAML attack surface

### L3
- [[kerberos-protocol-deep-dive]] - Federation's inside-the-firewall cousin for AD
- [[multi-factor-authentication-mfa]] - FIDO2 + federation = the gold standard
- [[zero-trust-architecture-nist-800-207]] - Federation tokens are the identity signal ZT uses

### Cross-Domain
- [[domain-04-communication-and-network-security]] - mTLS, certificate trust chain underpins federation
- [[domain-08-software-development-security]] - OIDC / OAuth misimplementation is the top appsec IAM bug

## Sources / References

- OASIS Security Services (SAML) TC - SAML 2.0 Standard
- IETF RFC 6749 - The OAuth 2.0 Authorization Framework
- IETF RFC 6750 - The OAuth 2.0 Authorization Framework: Bearer Token Usage
- IETF RFC 7519 - JSON Web Token (JWT)
- IETF RFC 7521 - Assertion Framework for OAuth 2.0 Client Authentication and Authorization Grants
- IETF RFC 7523 - JWT Profile for OAuth 2.0 Client Authentication
- IETF RFC 7636 - PKCE (Proof Key for Code Exchange)
- IETF RFC 8414 - OAuth 2.0 Authorization Server Metadata
- IETF RFC 8628 - OAuth 2.0 Device Authorization Grant
- IETF RFC 8705 - OAuth 2.0 Mutual-TLS Client Authentication and Certificate-Bound Access Tokens
- IETF RFC 9068 - JWT Profile for OAuth 2.0 Access Tokens
- IETF RFC 9332 - DPoP (Demonstrating Proof-of-Possession)
- OpenID Connect Core 1.0
- OpenID Connect Discovery 1.0
- FAPI 2.0 Security Profile (OpenID Foundation)
- NIST SP 800-63C - Federation and Assertions
- (ISC)² CISSP CBK 2024 - Domain 5.5
- OWASP API Security Top 10 - API1 Broken Object Level Authorization, API2 Broken Authentication
- OWASP OAuth 2.0 Cheat Sheet
