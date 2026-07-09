---
title: "Access Control Attacks and Mitigations"
layer: 2
domain: 5
tags:
  - cissp
  - domain-05
  - attacks
  - credential-theft
  - mfa-fatigue
  - session-hijacking
  - privilege-escalation
  - lateral-movement
related_domains:
  - "[[domain-01-security-and-risk-management]]"
  - "[[domain-06-security-assessment-and-testing]]"
  - "[[domain-07-security-operations]]"
  - "[[authentication-factors-and-mechanisms]]"
  - "[[authorization-models]]"
  - "[[federation-sso-and-saml-oidc]]"
  - "[[multi-factor-authentication-mfa]]"
  - "[[privileged-access-management-pam]]"
  - "[[kerberos-protocol-deep-dive]]"
---

# Access Control Attacks and Mitigations

## Feynman Explanation

Authentication and authorization fail in recognizable patterns. The same handful of attacks — stolen passwords, intercepted MFA prompts, stolen session cookies, exploited misconfigurations, abused standing privilege — show up in breach after breach because they are the easy ways in. This note catalogues those patterns: how each one works at the protocol level, what it looks like in the SIEM, and the specific defensive control that breaks it. The CISO view is: you don't need to invent new defenses; you need to make sure the well-known ones are actually deployed.

## Technical Details

### 1. Threat Model — the Attacker's Mental Map

The attacker thinks in *stages*:

1. **Initial access** — phish a credential, brute force, buy creds on a forum, exploit a public app.
2. **Persistence** — register a second MFA factor, create an OAuth grant, plant a webhook.
3. **Privilege escalation** — find an over-privileged account, abuse a misconfigured role, run Kerberoasting.
4. **Lateral movement** — pivot through the IdP-federated app estate using the stolen session.
5. **Exfiltration / impact** — encrypt, destroy, or steal data.

Every control below is mapped to which stage it breaks.

### 2. Password Attacks

| Attack | Mechanism | Cost to attacker | Defense |
|---|---|---|---|
| **Online brute force** | Automated guessing at the login endpoint | High (rate-limited) | Account lockout, throttling, CAPTCHA, breached-password check, IP allow-list |
| **Offline brute force** | Steal the password hash, crack locally | Cheap if hash is weak (NTLM, MD5) | Strong KDF (Argon2id / bcrypt / scrypt), long unique salts, no LM hash, disable NTLM |
| **Credential stuffing** | Reuse credentials from other breaches | Very cheap (commodity lists) | MFA, breach-corp check on login, device fingerprint, risk-based step-up |
| **Dictionary** | Common passwords | Cheap | Block top 1 000 / 10 000 / 100 000 password lists, training (low ROI) |
| **Rainbow table** | Precomputed hash → password | Cheap if unsalted | Unique per-user salt |
| **Spray** | One password across many accounts to avoid lockout | Cheap | Detection: low-and-slow distribution; break-glass thresholds; breach-corp check |
| **Shoulder surfing / keylogger** | Watch or sniff input | Physical | Hardware tokens / FIDO2, screen filters, endpoint EDR |
| **Phishing** | Real-time proxy / fake site / AiTM | Cheap, scalable | Phishing-resistant MFA (FIDO2 / passkeys), training (low ROI), email auth (DMARC) |

#### Hashes the exam loves

| Hash | Status |
|---|---|
| **LM** | Trivially broken (14-char split, no salt) — disable everywhere |
| **NTLMv1** | Broken (MD4-based, replay) — disable |
| **NTLMv2** | Legacy, vulnerable to relay — restrict to NTLMv2 only, prefer Kerberos |
| **MD5 / SHA-1** | Broken for password storage |
| **bcrypt / scrypt / Argon2id** | Current acceptable KDFs |
| **PBKDF2-HMAC-SHA256/512** | Acceptable; tune iterations ≥ 600 000 |
| **Argon2id** | Current best practice (memory-hard, side-channel-resistant) |

> **Exam hook:** If the question is "which hash to use," the answer is **Argon2id** (or bcrypt at minimum). LM, NTLMv1, MD5, SHA-1, plain SHA-256 are all wrong.

### 3. MFA Bypass Attacks (the modern threat surface)

| Attack | Mechanism | Defense |
|---|---|---|
| **MFA fatigue / push bombing** | Spam push prompts until user approves | Number matching (approve the number on screen), 30-60 s push TTL, rate-limit prompts, FIDO2 |
| **AiTM (Adversary-in-the-Middle)** | Reverse proxy (Evilginx, Muraena, Modlishka) relays session and MFA | Phishing-resistant MFA (FIDO2 binds to origin, not session); short token TTL |
| **SIM swap** | Port victim number to attacker SIM; receive SMS OTP | Move from SMS to TOTP / FIDO2; carrier PIN; account-freeze |
| **SS7 intercept** | Read SMS over telecom signaling | Move from SMS to TOTP / FIDO2 |
| **MFA prompt theft via session cookie** | Steal cookie after MFA, replay | Bind tokens to device (DPoP / mTLS), short TTL, refresh-token rotation |
| **Real-time phishing kit** | Proxy in real time, capture TOTP and reuse | FIDO2 / WebAuthn (origin-bound) |
| **Helpdesk social engineering** | Call IT, claim lost phone, get MFA reset | Helpdesk verification with out-of-band callback, manager attestation, identity proofing |
| **OAuth consent phishing** | Trick user to grant malicious OAuth app access to mailbox / files | Restricted consent, admin approval for high-risk scopes, monthly OAuth grant review |
| **MFA factor reset attack** | Use "forgot MFA" flow to add a new factor | Recovery requires same-or-stronger factor; identity proofing before MFA reset |
| **Pass-the-cookie** | Replay a long-lived session cookie from malware / infostealer | Short session TTL, refresh-token rotation, device binding, EDR detection |

#### MFA fatigue economics (CISO framing)

- Push bombing succeeds ~30-50% of the time on un-trained users (Microsoft research, 2023).
- AiTM kits are commodity, < $200/month subscription.
- The only durable defense is **phishing-resistant MFA** (FIDO2 / passkeys / smart card).

### 4. Session and Token Attacks

| Attack | Mechanism | Defense |
|---|---|---|
| **Session hijack (XSS)** | Inject script to steal cookie | `HttpOnly` + `Secure` + `SameSite=Strict` cookies; CSP; output encoding |
| **Session fixation** | Force a known session ID, victim authenticates into it | Regenerate session ID on auth; bind session to other attributes |
| **Cookie theft by malware (infostealer)** | Read browser cookie DB | Device-bound tokens (DPoP / mTLS), short session TTL, EDR |
| **Bearer token replay** | Capture access token from logs / proxy | Short TTL, audience binding, sender-constrained tokens |
| **JWT `alg=none` forgery** | Strip signature, accept unsigned token | Whitelist algorithms on the server (`RS256`/`ES256`); never accept `none` |
| **JWT key confusion** | Server uses RSA public key as HMAC secret | Restrict alg to `RS*` / `ES*` for asymmetric keys |
| **JWT `kid` SQL injection / path traversal** | Use `kid` to read arbitrary file or build SQL | Whitelist `kid`; sanitize input |
| **Refresh-token theft** | Long-lived token stolen; attacker re-issues | Rotation with reuse detection; revoke the family on anomaly |
| **Mix-up attack (OAuth)** | Confuse which IdP issued the token | Issuer validation; per-IdP `state` encryption |
| **Bearer token in URL** | Token leaked in referrer / proxy logs | Tokens only in `Authorization` header; never in URL |
| **SAML XSW (signature wrapping)** | Swap signed assertion for forged one | Strict schema validation; unique `ID` reference; both Response and Assertion signed |
| **SAML replay** | Replay a captured assertion | `NotOnOrAfter`, `InResponseTo`, single-use `ID` |
| **Open redirect in OAuth** | Steal auth code via crafted `redirect_uri` | Exact-match `redirect_uri` allow-list |
| **CSRF on session-authenticated app** | Browser sends session cookie cross-site | `SameSite=Strict` cookie; CSRF token |

### 5. Privilege Escalation Patterns

| Pattern | Mechanism | Defense |
|---|---|---|
| **Vertical privesc** | Low-priv user gains admin (e.g., local exploit, misconfigured sudo, SUID binary) | Patch, disable unnecessary SUID, sudoers policy with no `NOPASSWD: ALL` |
| **Horizontal privesc** | User A accesses User B's data (e.g., IDOR — insecure direct object reference) | Authorization checks server-side on every object; ABAC; ReBAC |
| **Confused deputy** | Trick a high-priv service (e.g., SSRF → cloud metadata → IAM creds) | IMDSv2 (session-token required); no metadata reachable from app |
| **OAuth scope creep** | App requests more scopes than needed | Least-privilege scopes, scope review at consent, periodic scope audit |
| **AWS IAM wildcards** | `iam:PassRole` with `*` → privesc to any service | Restrict `iam:PassRole` to specific roles, not `*` |
| **GitHub branch protection bypass** | Push to a branch that triggers privileged CI | Enforce CODEOWNERS, branch protection, signed commits |
| **K8s RBAC abuse** | `pods/exec` on a privileged pod | Restrict `pods/exec` and `pods/portforward`; pod security standards |
| **Sudo / NOPASSWD** | `user ALL=(ALL) NOPASSWD: ALL` | Restrict to specific commands; require TTY + MFA |
| **Token impersonation** | Steal a service-account token, impersonate | Short-lived tokens (1 h max); workload identity bound to pod identity |
| **DLL / library hijack** | Place a higher-precedence DLL in a path the privileged app loads | WDAC / AppLocker; signed binaries only |
| **Scheduled task / cron** | Add a job that runs as a privileged user | Audit scheduled tasks; restrict creation rights |
| **Kernel / driver exploit** | Exploit a local kernel vuln for SYSTEM | Patch; HVCI / Credential Guard; secure boot |
| **Container escape** | Break out of container to host | Pod Security Standards; seccomp; non-root; read-only fs |

### 6. Lateral Movement Patterns (post-auth)

| Pattern | Mechanism | Defense |
|---|---|---|
| **Pass-the-Hash (PtH)** | Reuse a captured NTLM hash to authenticate | Disable NTLM, use Kerberos (AES), gMSA, LSA Protection (RunAsPPL) |
| **Pass-the-Ticket (PtT)** | Replay a Kerberos TGT or service ticket | TGT lifetime limits, monitor unusual TGT use, restrict ticket caching, LSA Protection |
| **Kerberoasting** | Request a service ticket for an SPN with a known username, crack offline | Long (≥ 25 char) service-account passwords or gMSA; AES only; smart card / certificate for service accounts |
| **AS-REP roasting** | Target accounts with "Do not require Kerberos preauthentication" | Disable "Do not require preauth" by default; require preauth for all |
| **Golden ticket** | Forge a TGT with the krbtgt hash | Rotate `krbtgt` twice, monitor PAC anomalies, LSA Protection |
| **Silver ticket** | Forge a service ticket with the service account hash | Strong service-account passwords, AES, monitor service-ticket use |
| **DCShadow** | Register a rogue DC to push changes | Monitor DC registration events; restrict DC replication rights |
| **DCSync** | Replicate directory data with `Replicating Directory Changes` rights | Restrict those rights to actual DCs only; alert on use by non-DCs |
| **Lateral SMB / WinRM** | Move via admin shares or WinRM | Restrict inbound SMB / WinRM; tiered admin model; PAW |
| **Lateral RDP** | Use RDP to move between hosts | Restricted Admin mode; Network Level Authentication; RDP gateway |
| **WMI / PsExec** | Remote execution via WMI / PsExec | Restrict remote WMI / service creation; alert on use |
| **OAuth lateral movement** | Use the same stolen session to call every federated SaaS API | Short token TTL; per-app audience binding; monitor cross-app token use |
| **Cloud lateral movement** | Use stolen cloud creds to enumerate every service the role can reach | SCPs (Service Control Policies), CloudTrail, IAM Access Analyzer |

### 7. Account / Identity Attacks

| Attack | Mechanism | Defense |
|---|---|---|
| **Account enumeration** | Login endpoint reveals valid usernames | Generic error messages; rate-limit both user-exists and bad-password |
| **Password reset poisoning** | Inject host header / manipulate reset email | Use known email channel; signed reset links; time-bound token |
| **Pre-account takeover** | Register with victim's email before they do | Identity proofing; email verification before activation |
| **OAuth account squatting** | Register a 3rd-party login provider with a victim's email | Domain verification of IdP; require matching email |
| **JWT jti reuse** | Replay a `jti` value | Track `jti`; reject duplicates within TTL |
| **OIDC prompt=none abuse** | Try to silently get a token for a different RP | `prompt=login` for sensitive; `max_age` parameter |
| **Session donation / chaining** | Login attacker, then chain into victim's session | Bind session to user agent + IP (loosely) + device fingerprint |

### 8. Authorization Bypass Patterns

| Pattern | Mechanism | Defense |
|---|---|---|
| **IDOR** (Insecure Direct Object Reference) | Change `id=123` in URL to `id=124` | Server-side authorization check on every object access |
| **Forced browsing** | Direct GET to `/admin/panel` | Default-deny routing; authorization middleware on every route |
| **Parameter tampering** | Change `role=user` to `role=admin` in hidden field | Never trust client-supplied authorization inputs; server-side check |
| **JWT claim manipulation** | Decode JWT, change `role` claim, re-sign (if weak) | Signature integrity; claims as policy inputs, not policy itself |
| **Path traversal** | `../../etc/passwd` | Canonicalize path; allow-list path prefixes |
| **Race condition (TOCTOU)** | Two requests at the same time bypass a check | Idempotency keys; transaction isolation; database constraints |
| **Privilege creep** | Old entitlements never removed | Quarterly recertification, role mining, JIT for privileged |
| **SOD violation** | Same person can create + approve a payment | Static and dynamic SoD enforcement |

### 9. Insider Threats (the human factor)

| Type | Description | Detection |
|---|---|---|
| **Malicious insider** | Authorized user with intent to harm | UEBA, DLPs, peer-baseline analytics, just-culture review |
| **Negligent insider** | Authorized user who makes a mistake | Training, least privilege, friction-at-the-moment-of-risk |
| **Compromised insider** | Authorized user whose account is taken over | UEBA, impossible-travel, MFA anomaly, endpoint EDR |
| **Departed insider** | Authorized user whose access was not removed | Join/Move/Leave lifecycle SLA; daily orphan hunt |

#### Insider threat indicators (UEBA signals)

- Bulk download outside of normal pattern
- Access at unusual times
- Access from unusual geo / IP / device
- Sudden interest in confidential projects (M&A, layoff lists)
- Email forwarding rules to external addresses
- Use of personal cloud storage from corporate device
- Privilege escalation shortly before resignation
- Data hoarding (large file copies to USB / personal drive)

### 10. Detection Patterns (the SIEM / SOC view)

| Detection | Telemetry source | Notable rule |
|---|---|---|
| Impossible travel | IdP, VPN | 2 logins from geographically distant IPs in < 1 h |
| MFA push spam | IdP, push provider | > 5 push prompts in 10 min to same user |
| OAuth consent grant | IdP, CASB | New OAuth app with high-risk scopes (`Mail.Read`, `Files.ReadWrite.All`) |
| Anomalous data download | DLP, CASB | Download volume > 5× user baseline |
| Service-ticket abuse | AD / Kerberos logs | RC4 ticket requests (0x17); many ticket requests in short window (Kerberoasting) |
| LSASS access | EDR | `procdump`, `Mimikatz`, `comsvcs.dll` reading LSASS |
| New admin added | IdP / AD | Privilege change outside change window |
| New service principal | Cloud audit | New IAM role / SP outside IaC pipeline |
| Token issuance from unusual location | IdP | Token from data-center IP for a user who has never logged in from there |
| Cross-app token reuse | SaaS audit, CASB | Same bearer token used across multiple SaaS apps in short time |
| Failed MFA spike | IdP | > 10 MFA failures in 5 min per user |
| Dormant account reactivation | IdP | Account inactive > 90 days, then active session |

### 11. Mitigations — the Defensive Map

| Layer | Defense | Breaks which attack |
|---|---|---|
| **Network** | mTLS, IP allow-list, network segmentation | Pass-the-Hash, lateral SMB/RDP |
| **Identity** | Phishing-resistant MFA, FIDO2/passkeys, hardware token | Phishing, AiTM, MFA fatigue |
| **Session** | Short TTL, refresh-token rotation, DPoP, `HttpOnly` `Secure` `SameSite=Strict` | Session hijack, replay |
| **Authorization** | ABAC, server-side check on every object, SoD | IDOR, privesc, role explosion |
| **Endpoint** | EDR, Credential Guard, LSA Protection, WDAC, application allowlist | Local privesc, token theft |
| **Data** | DLP, classification, encryption at rest | Insider exfiltration |
| **Detection** | SIEM + UEBA + SOAR | Catch the rest |
| **Response** | Playbook for identity incidents (rotate creds, revoke sessions, freeze account) | Contain blast radius |

### 12. Exam Pattern Recap

- "**Hash type**" → Argon2id (or bcrypt).
- "**Best MFA**" → phishing-resistant (FIDO2 / passkey / smart card).
- "**OAuth + public client**" → PKCE required.
- "**Federation**" → SAML for enterprise B2B; OIDC for modern.
- "**SAML assertion replay**" → `NotOnOrAfter` + `InResponseTo` + one-time `ID`.
- "**Lateral movement with hash**" → disable NTLM, use Kerberos AES, gMSA, LSA Protection.
- "**Service account exploitation**" → gMSA, ≥ 25 char password, AES only.
- "**Insider data theft**" → UEBA + DLP + least privilege.
- "**Stolen session cookie**" → short TTL, `HttpOnly` + `Secure` + `SameSite`, device binding.
- "**Orphaned account**" → Join/Move/Leave SLA + recertification.

## CISO / Risk Manager View

Access control attacks are the **leading root cause of breaches** in nearly every annual report (Verizon DBIR, IBM Cost of a Data Breach, Microsoft Digital Defense). The CISO's job is to translate this catalogue into a prioritized investment program.

| Investment | Defends against | Estimated reduction in breach probability |
|---|---|---|
| Phishing-resistant MFA (FIDO2 / passkeys) for all users | Phishing, AiTM, MFA fatigue, push bombing | 90%+ reduction in account takeover (Microsoft, Google data) |
| Short token TTL + refresh-token rotation + device binding | Token replay, cookie theft | 60-80% reduction in session-driven incidents |
| Just-in-Time admin + standing-priv removal | Insider threat, lateral movement, ransomware | Major reduction in admin-takeover incidents |
| Strong KDF (Argon2id) + breach-corp check on login | Credential stuffing, offline cracking | ~50% reduction in credential-driven takeover |
| LSA Protection, Credential Guard, disable NTLM | Pass-the-Hash, LSASS dumping | Near-elimination of the most common AD attack chain |
| UEBA + impossible-travel + MFA-fatigue detection | Compromised-insider | Catches what MFA cannot |
| Join/Move/Leave SLA + recertification | Orphaned accounts, ex-employee access | 80%+ reduction in dormant-account incidents |
| DLP + classification + ABAC | Insider data theft | Significant; depends on coverage |

**Maturity ladder (access-control-attacks):**

| Level | Name | Defining characteristic |
|---|---|---|
| 1 | Reactive | After-the-breach response; no telemetry |
| 2 | Hardened | MFA, basic SIEM, patch discipline |
| 3 | Defended | Phishing-resistant MFA for admins, FIDO2 rollout, UEBA, JIT for privileged |
| 4 | Instrumented | UEBA-driven IR, device-bound tokens, microsegmentation, identity-aware DLP |
| 5 | Antifragile | Continuous verification, passkey-default, ABAC everywhere, red-team-tested quarterly |

**The board narrative:** "Every dollar we spend on identity hygiene returns more than every dollar we spend on any other control. Phishing-resistant MFA alone would have prevented the majority of the largest public breaches of the last five years."

**Privileged risk concentration:** the **5% of accounts that own 80% of entitlements** are the single most consequential attack surface. Privileged Access Management (PAM) is the highest-ROI identity security investment a CISO can make. See L3 [[privileged-access-management-pam]].

**Compliance hooks:** PCI-DSS 8 (Identify users and authenticate access), PCI-DSS 10 (logging), SOX 404, HIPAA 164.312 (technical safeguards), ISO 27001 A.5.17 (authentication information), A.8.5 (secure authentication), NIST 800-53 AC-2, IA-2, IA-5 (account management, auth, password management).

## Related Connections

### Sibling L2
- [[authentication-factors-and-mechanisms]] - The factors under attack and how to choose
- [[authorization-models]] - The models that fail when bypassed
- [[identity-lifecycle-and-provisioning]] - Orphaned-account attack surface
- [[federation-sso-and-saml-oidc]] - Token / assertion attack surface

### L3
- [[kerberos-protocol-deep-dive]] - Kerberoasting, PtH, PtT, golden ticket
- [[multi-factor-authentication-mfa]] - MFA bypass attacks and phishing-resistant defenses
- [[privileged-access-management-pam]] - The highest-risk slice of the identity estate
- [[zero-trust-architecture-nist-800-207]] - Continuous verification minimizes the blast radius of any of these attacks

### Cross-Domain
- [[domain-01-security-and-risk-management]] - Threat model and risk treatment prioritization
- [[domain-06-security-assessment-and-testing]] - Red-team / pen-test of these attack paths
- [[domain-07-security-operations]] - SIEM, UEBA, IR playbooks
- [[domain-08-software-development-security]] - IDOR, broken auth in code

## Sources / References

- Verizon 2024 Data Breach Investigations Report (DBIR)
- IBM Cost of a Data Breach Report 2024
- Microsoft Digital Defense Report 2024
- MITRE ATT&CK - Enterprise Matrix (TA0001 Initial Access, TA0004 Privilege Escalation, TA0008 Lateral Movement)
- MITRE D3FEND - Defensive countermeasures
- NIST SP 800-63B - Authentication and Lifecycle Management
- NIST SP 800-53 Rev. 5 - AC, IA, SC controls
- ANSSI (France) - "State of the Art of Kerberos" (2023)
- The Hacker Recipes - Kerberos attack path catalogue (Orange Cyberdefense)
- Evilginx, Muraena, Modlishka documentation
- (ISC)² CISSP CBK 2024 - Domain 5.8 / 5.9
- OWASP Top 10 2021 - A01 Broken Access Control, A07 Identification and Authentication Failures
- OWASP API Security Top 10 2023
- Have I Been Pwned (HIBP) - breach corpus for password checks
