---
title: API Security and Microservices
layer: 2
domain: 8
tags: [cissp, api-security, owasp-api, oauth2, jwt, microservices, service-mesh, zero-trust]
related_domains: [3, 5]
parent: domain-08-software-development-security
---

# API Security and Microservices

## Feynman Explanation

A traditional app is a castle with one gate — the login form. A modern microservices app is a city of castles, each with its own gate, and the roads between them are public. **API security is the design of those gates and roads: who can enter, what they can do, who vouched for them, and how every trip is recorded.** The OWASP API Top 10 is the pattern catalog of how attackers break those gates; OAuth 2.0 + JWT is the standard way to issue the "voucher" (token); a service mesh is the road-patrol system that polices east-west traffic inside the city.

## Technical Details

### The API Attack Surface Shift

A 2020-era monolith has $\sim 1$ entry point. A 2024-era microservice mesh has $10^2$–$10^3$ services, each exposing $5$–$50$ endpoints, often across multiple protocols (REST, gRPC, GraphQL, async). The attack surface scales **multiplicatively**, not linearly. Traditional WAF + perimeter security does not address east-west traffic between services; this is the gap API security and service meshes fill.

### OWASP API Security Top 10 (2023)

| # | Vulnerability | Description | One-line Defense |
|---|---|---|---|
| **API1** | **Broken Object Level Authorization (BOLA)** | User A reads/modifies User B's objects by ID | Server-side: enforce `object.owner_id == session.user_id` on every request |
| **API2** | **Broken Authentication** | Auth tokens forged, replayed, or not rotated | Use a vetted library; validate JWT signature, exp, iss, aud; rotate keys |
| **API3** | **Broken Object Property Level Authorization** | Mass assignment, excessive data exposure | Whitelist response fields; separate `User` DTOs from `UserEntity` |
| **API4** | **Unrestricted Resource Consumption** | No rate limits; expensive endpoints exploited | Rate limit per-IP, per-user, per-API-key; cap payload size; pagination |
| **API5** | **Broken Function Level Authorization** | Regular user hits admin endpoints | Role checks on every endpoint; not just UI hiding |
| **API6** | **Unrestricted Access to Sensitive Business Flows** | Automation of signup, ticket-buying, scraping | CAPTCHA, device fingerprinting, behavior analysis |
| **API7** | **Server Side Request Forgery (SSRF)** | Server fetches attacker-controlled URL | Egress filter; URL allow-list; resolve + check IP before fetch |
| **API8** | **Security Misconfiguration** | CORS `*`, debug endpoints, default creds | Hardened baseline; IaC scanning; env separation |
| **API9** | **Improper Inventory Management** | Old `/v1` API still exposed; shadow APIs | API gateway; version sunset policy; continuous discovery |
| **API10** | **Unsafe Consumption of APIs** | Trust in upstream API responses | Validate, schema-check, size-cap every upstream response |

> **BOLA is the #1 API vulnerability** (per OWASP and every API pentest report). It is a logic flaw, not a tech-stack flaw — scanners miss it; only design review + pentest + abuse-case testing find it.

### Authentication vs Authorization

| Concern | Question | Examples |
|---|---|---|
| **Authentication (AuthN)** | *Who is this caller?* | OAuth 2.0, OIDC, API keys, mTLS, JWT |
| **Authorization (AuthZ)** | *What may this caller do?* | RBAC, ABAC, OAuth scopes, BOLA/BFLA enforcement |

A common failure: **AuthN passes, AuthZ is skipped** → BOLA. Every endpoint must re-check AuthZ, even for "obviously" the same user.

### OAuth 2.0 — The Authorization Framework

OAuth 2.0 (RFC 6749) is **delegated authorization**: a user grants a client limited access to a resource on a third-party server, without sharing credentials.

#### Roles

| Role | Definition |
|---|---|
| **Resource Owner** | The user |
| **Client** | The app requesting access |
| **Authorization Server (AS)** | Issues tokens (e.g., Auth0, Keycloak, Okta) |
| **Resource Server (RS)** | Hosts the protected resource (your API) |

#### Grant Types (Flows)

| Grant | Use Case | Security Notes |
|---|---|---|
| **Authorization Code + PKCE** | First-party and third-party web/mobile apps | **Required** for public clients; defends against code interception |
| **Client Credentials** | Service-to-service, no user | Client secret / mTLS; never use for user-context APIs |
| **Implicit (deprecated)** | Legacy SPAs | Replaced by Auth Code + PKCE in OAuth 2.1 |
| **Resource Owner Password (deprecated)** | Trusted first-party legacy | Replaced by Auth Code + PKCE |
| **Device Code** | Smart TVs, IoT | Requires user authorization on second device |
| **Refresh Token** | Long-lived sessions | Rotate; revoke on logout; bind to client (RFC 8252) |

> **CISSP exam focus.** Know the **four roles**, the **grant types**, and that **OAuth 2.0 is authorization, NOT authentication** (that's OpenID Connect, OIDC).

### JWT (JSON Web Token) — Structure

A JWT is a signed JSON object. Three base64url segments separated by `.`:

```
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6InByb2Qta2V5In0
.
eyJzdWIiOiJ1c2VyXzEyMyIsImlzcyI6Imh0dHBzOi8vYXMuc2l0ZS5jb20iLCJhdWQiOiJhcGkuc2l0ZS5jb20iLCJleHAiOjE3MTk5OTk5OTksImlhdCI6MTcxOTkxMzU5OSwic2NvcGUiOiJyZWFkOm9yZGVycyB3cml0ZTpvcmRlcnMifQ
.
<signature>
```

**Decoded header:**

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "prod-key"
}
```

**Decoded payload (claims):**

```json
{
  "sub": "user_123",
  "iss": "https://as.site.com",
  "aud": "api.site.com",
  "exp": 1719999999,
  "iat": 1719913599,
  "scope": "read:orders write:orders",
  "role": "customer"
}
```

| Claim | Purpose | Validation Rule |
|---|---|---|
| `iss` | Issuer | Must match expected AS |
| `aud` | Audience | Must match this RS |
| `exp` | Expiration | Must be in the future |
| `nbf` | Not before | Must be in the past |
| `iat` | Issued at | Informational; check for replay |
| `sub` | Subject (user) | Used for BOLA checks |
| `jti` | JWT ID | Optional; used for revocation / replay defense |
| `kid` | Key ID (in header) | Use to pick the right verification key |

#### Critical JWT Defenses

1. **Always verify the signature** — never trust the payload without `alg` check.
2. **Pin the algorithm** — reject `alg: none`; reject `alg: HS256` if you issued with RS256 (key confusion).
3. **Validate `iss`, `aud`, `exp`** on every request.
4. **Use short-lived access tokens** (≤ 15 min) + refresh tokens.
5. **Never put secrets in the JWT payload** — it is base64, not encrypted. (Use JWE if needed.)
6. **Store JWTs securely** on the client — httpOnly cookie or secure storage; never localStorage for high-value tokens.

### Service Mesh — East-West Security

A service mesh (Istio, Linkerd, Consul Connect, AWS App Mesh) inserts a **sidecar proxy** (Envoy) next to every microservice. All traffic flows through the proxy, enabling:

| Capability | How |
|---|---|
| **mTLS by default** | Sidecars issue and rotate certs; traffic encrypted in-cluster |
| **Authorization policies** | "Service A may call Service B on path /v1/orders" — declared, not coded |
| **Observability** | Every request has a trace ID, latency, status |
| **Retries / circuit breaking** | Policy-based resilience |
| **L7 routing** | Canary, A/B, traffic shifting |

**Zero-trust posture:** with a mesh, no service trusts another by network location — every call is authenticated and authorized, even inside the cluster perimeter.

### API Gateway Functions

| Function | Description |
|---|---|
| **Routing** | URL → upstream service mapping |
| **AuthN termination** | Validate JWT/OAuth at the edge; pass identity to services |
| **Rate limiting** | Per-key, per-IP, per-tenant |
| **Schema validation** | Reject malformed JSON before it reaches the service |
| **Caching** | For read-heavy endpoints |
| **Logging / metrics** | Edge-level request logs |
| **WAF** | Some gateways integrate WAF rules |

### gRPC and GraphQL Notes

| Protocol | Security Considerations |
|---|---|
| **gRPC** | Binary; mTLS recommended; reflection API should be disabled in prod; schema required for input validation |
| **GraphQL** | Single endpoint → query cost attacks; depth/complexity limits mandatory; field-level authZ; persisted queries |

### Common API Security Anti-Patterns

1. **API keys as AuthZ** — API keys identify the *client*, not the *user*. A stolen key is total compromise.
2. **Long-lived JWTs** (e.g., 30 days) — stolen token = 30-day access. Use refresh tokens + rotation.
3. **No BOLA check on `/users/{id}`** — the canonical OWASP API #1.
4. **CORS `*` with credentials** — privilege escalation primitive.
5. **Internal APIs exposed to internet** — API #9 (improper inventory).
6. **JWT in localStorage** — XSS exfiltrates the token. Use httpOnly secure cookies.
7. **Trusting upstream APIs without validation** — API #10.

## CISO / Risk Manager View

**Board framing.** APIs are the **dominant attack surface of every modern enterprise**. Public APIs, partner APIs, internal APIs, and microservice east-west traffic all carry the same risk profile with different trust levels. The board does not need to know JWT internals; the board needs to know **what percentage of APIs are inventoried, authenticated, authorized, rate-limited, and monitored** — and whether the program keeps pace as the API count grows.

**Strategic priorities.**

1. **Inventory every API.** Shadow APIs (forgotten `/v1`, partner endpoints, deprecated mobile backends) are the #1 risk. Continuous API discovery (e.g., 42Crunch, Akamai API Security) is a board-level control.
2. **Standardize on OAuth 2.0 + OIDC.** No bespoke auth. Centralized Identity Provider (IdP) for all user-facing APIs.
3. **Enforce AuthZ at every endpoint.** Code review check: no endpoint reads/writes an object without verifying ownership. Static analysis rules for BOLA are emerging.
4. **Adopt a service mesh for east-west.** Even small meshes pay back in observability and mTLS coverage.
5. **Threat-model the top 10 APIs by business value.** The OWASP API Top 10 is the checklist.
6. **Treat API keys like passwords** — short-lived, scoped, rotated, audited.

**Metrics the board cares about.**

- # of APIs in inventory (and discovery coverage %)
- % of APIs behind OAuth 2.0 (target: 100% user-facing)
- % of APIs with rate limiting (target: 100% public)
- Mean age of a high-severity API finding
- # of mTLS-protected service-to-service connections
- # of shadow/deprecated APIs discovered in last 90 days (should be $\geq 1$ — discovery works)

## Related Connections

### Sibling L2
- [[threat-modeling-stride-pasta]] — STRIDE per API endpoint; BOLA = Elevation of Privilege
- [[secure-coding-practices-owasp]] — Rules 1–5 apply at API trust boundaries
- [[sdlc-models-and-security-integration]] — API design phase must include threat model + auth design

### L3 Standards
- [[devsecops-and-cicd-security]] — API contract testing in CI; schema validation
- [[software-supply-chain-attacks-slsa]] — third-party API risks = upstream supply chain

### Cross-Domain
- [[../Domain-03-Security-Architecture-and-Engineering/Domain-03-Index]] — TLS / mTLS / JWT signature algorithms
- [[../Domain-05-Identity-and-Access-Management/Domain-05-Index]] — OAuth/OIDC/SAML are IAM primitives used at the API edge
- [[../Domain-06-Security-Assessment-and-Testing/Domain-06-Index]] — API pentest focuses on BOLA and broken auth

## References

- OWASP API Security Top 10 (2023)
- IETF RFC 6749 — OAuth 2.0 Authorization Framework
- IETF RFC 6750 — OAuth 2.0 Bearer Token Usage
- IETF RFC 7519 — JSON Web Token (JWT)
- IETF RFC 7636 — PKCE (Proof Key for Code Exchange)
- OpenID Connect Core 1.0
- NIST SP 800-204 — Security Strategies for Microservices-based Application Systems
- NIST SP 800-204A — Building Secure Microservices-based Applications
- (ISC)² CISSP CBK — Domain 8: Software Development Security
- IETF draft — OAuth 2.1 (consolidation of OAuth 2.0 + security BCP)
