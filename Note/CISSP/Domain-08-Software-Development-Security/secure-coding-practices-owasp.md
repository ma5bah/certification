---
title: Secure Coding Practices and OWASP
layer: 2
domain: 8
tags: [cissp, secure-coding, owasp, asvs, input-validation, output-encoding, parameterized-queries]
related_domains: [3, 5, 6]
parent: domain-08-software-development-security
---

# Secure Coding Practices and OWASP

## Feynman Explanation

A car is built from rules, not talent. Every weld has a spec, every bolt has a torque, every wire has a colour. Software is the opposite — there is no natural spec; every team writes its own. **Secure coding is the rulebook that turns "it works" into "it works and cannot be tricked."** It is not about being a brilliant programmer; it is about memorizing five things (validate input, encode output, parameterize queries, handle errors, log safely) and applying them 100% of the time. OWASP is the international body that writes the rulebook — the **Top 10** is the awareness list, the **ASVS** is the verifiable checklist.

## Technical Details

### The Five Universal Secure-Coding Rules

These rules are language-agnostic and apply to every application. Failing any one of them creates a vulnerability that the OWASP Top 10 catalogues.

| # | Rule | Defeats | OWASP 2021 Reference |
|---|---|---|---|
| 1 | **Validate all input** at the trust boundary | Injection, XSS, SSRF, path traversal | A03 — Injection |
| 2 | **Encode all output** in the destination context | XSS (reflected, stored, DOM) | A03 — Injection |
| 3 | **Parameterize all queries** (no string concatenation) | SQLi, NoSQLi, LDAPi, command injection | A03 — Injection |
| 4 | **Handle errors without leaking secrets** | Information disclosure, stack-trace leaks | A05 — Misconfiguration |
| 5 | **Log and monitor security events** | Delayed detection, log-injection | A09 — Logging Failures |

### Rule 1 — Input Validation (Allow-list, Not Deny-list)

**Vulnerable — blocklist (deny-list) of characters:**

```python
# NEVER DO THIS
def bad_query(username):
    blacklist = ["'", '"', "--", ";", "/*", "*/"]
    safe = username
    for c in blacklist:
        safe = safe.replace(c, "")
    return db.execute(f"SELECT * FROM users WHERE name = '{safe}'")
```

A blocklist is **always bypassable**: a `\` at the end of `'admin\'-- ` escapes the quote; the trailing quote in the original query terminates the string and `--` comments out the rest.

**Safe — allow-list (whitelist) of permitted characters:**

```python
import re
def good_query(username):
    if not re.fullmatch(r"[a-zA-Z0-9_]{3,32}", username):
        raise ValueError("invalid username")
    # still parameterize, even after validation
    return db.execute(
        "SELECT * FROM users WHERE name = ?", (username,)
    )
```

> **Principle.** Validate against a **strict allow-list** of permitted characters and length. Reject everything else with a generic error. Never echo the offending input back to the user (reflection XSS).

### Rule 2 — Output Encoding (Context-Dependent)

The same string is interpreted differently in HTML, JavaScript, CSS, URL, SQL, and shell. Encode for the **destination context**, not the source.

| Context | Encoding | Example |
|---|---|---|
| HTML body | HTML entity encode | `<` → `&lt;` |
| HTML attribute | Attribute encode (incl. quotes) | `"` → `&quot;` |
| JavaScript string | `\xHH` or `\uHHHH` | `</script>` → `<\u002Fscript>` |
| URL | Percent-encode | ` ` → `%20` |
| SQL | Parameterized (NEVER concat) | see Rule 3 |
| Shell | Argument list, not string concat | `["ls", user_dir]` not `f"ls {user_dir}"` |
| LDAP | LDAP filter escape | `*` → `\\2a` |

**Vulnerable — JS string from unvalidated input:**

```javascript
// NEVER
element.innerHTML = "Hello, " + username;
```

If `username` is `<img src=x onerror=alert(1)>`, this executes. (Stored XSS.)

**Safe — textContent or framework auto-escaping:**

```javascript
// Safe — textContent never parses HTML
element.textContent = "Hello, " + username;
```

Modern frameworks (React, Vue, Angular) auto-escape interpolation `{{ }}` — the trap is using `v-html`, `dangerouslySetInnerHTML`, or `.innerHTML`.

### Rule 3 — Parameterized Queries (SQL)

**Vulnerable — string concatenation:**

```python
# NEVER — SQL injection
def bad_login(user, pwd):
    return db.execute(
        f"SELECT id FROM users WHERE name='{user}' AND pwd='{pwd}'"
    )
```

If `user` is `' OR 1=1-- `, the query becomes `WHERE name='' OR 1=1-- '` — true for every row, password bypassed.

**Safe — parameterized (prepared statement):**

```python
def good_login(user, pwd):
    return db.execute(
        "SELECT id FROM users WHERE name = ? AND pwd = ?",
        (user, pwd)
    ).fetchone()
```

The database driver sends the SQL template and parameters **separately**; the parameter is never interpreted as SQL syntax. The driver takes care of escaping.

> **Even better — use an ORM with bound parameters:**

```python
User.objects.filter(name=user, pwd_hash=hash(pwd))
```

ORMs compose queries safely; **but** raw SQL fragments (`.raw()`, `.extra()`, `RawSQL`) re-introduce the risk. The L3 file [[database-security-and-sql-injection]] covers the failure modes.

### Rule 4 — Error Handling (Fail Closed, No Leakage)

```python
# Bad — leak internals
try:
    db.execute(query)
except Exception as e:
    return {"error": str(e), "trace": traceback.format_exc()}

# Good — generic message, log internally, alert on pattern
try:
    db.execute(query)
except DatabaseError:
    log.error("db failure", exc_info=True, extra={"user_id": uid})
    metrics.increment("db.errors")
    return {"error": "Internal error"}, 500
```

Production responses must:
- Return a **generic error** to the user
- **Log the stack trace** internally with correlation ID
- **Trigger alerting** on rate spikes (D6/D7 territory)
- **Never** echo SQL, file paths, internal hostnames, or framework versions

### Rule 5 — Logging (Security-Relevant, Injection-Safe)

Log security events; never log secrets; sanitize before logging.

```python
# Bad — log injection: a username with \n erases the audit trail
log.info(f"login failed for {username}")

# Good — structured, sanitized
log.warning("login.failed", extra={
    "user_id_hash": sha256(username).hexdigest()[:16],
    "ip": request.remote_addr,
    "ua": request.headers.get("User-Agent", "")[:200]
})
```

**Never log:** passwords, API keys, session tokens, full PAN/SSN, full credit-card numbers, OAuth client secrets.

### OWASP ASVS 4.0 — Verifiable Requirements

ASVS organizes 286 requirements into 14 chapters and 3 verification levels:

| Level | Target | Example Requirement |
|---|---|---|
| **L1 — Opportunistic** | Any app that handles low-sensitivity data | "Verify that input is validated against an allow-list" |
| **L2 — Standard** | Apps handling B2B data, regulated industries, PII | "Verify that all authentication is server-side" |
| **L3 — Advanced** | Critical apps — healthcare, financial, military | "Verify that all cryptographic operations use FIPS-approved primitives" |

Chapters:

| § | Chapter | Anchors |
|---|---|---|
| V1 | Architecture | Threat modeling, secure design |
| V2 | Authentication | MFA, session mgmt, credential storage (D5 link) |
| V3 | Session Management | Cookies, tokens, idle timeout |
| V4 | Access Control | RBAC/ABAC, tenant isolation (D5 link) |
| V5 | Validation, Sanitization, Encoding | Rules 1, 2, 3 above |
| V6 | Stored Cryptography | Algorithms, key management (D3 link) |
| V7 | Error Handling & Logging | Rules 4, 5 above |
| V8 | Data Protection | At rest, in transit, in use (D2 link) |
| V9 | Communication | TLS, cert validation (D4 link) |
| V10 | Malicious Code | Backdoors, time bombs, logic bombs |
| V11 | Business Logic | Workflow abuse, race conditions |
| V12 | Files & Resources | Upload validation, path traversal |
| V13 | API & Web Service | CORS, rate limiting, JWT (D5 link) |
| V14 | Configuration | Hardening, headers, dependency hygiene |

### OWASP Top 10 (Web 2021) — Quick Reference

| ID | Vulnerability | Primary Defense |
|---|---|---|
| A01 | Broken Access Control | Server-side authorization on every request |
| A02 | Cryptographic Failures | TLS, strong ciphers, key management (D3) |
| A03 | Injection (SQLi, XSS, NoSQLi, LDAPi) | Rules 1, 2, 3 |
| A04 | Insecure Design | Threat modeling, secure design patterns (D8) |
| A05 | Security Misconfiguration | Hardened baselines, IaC scanning |
| A06 | Vulnerable & Outdated Components | SCA, SBOM, patch SLAs |
| A07 | Identification & Auth Failures | MFA, session hygiene (D5) |
| A08 | Software & Data Integrity Failures | Signed artifacts, SLSA, integrity checks |
| A09 | Logging & Monitoring Failures | SIEM coverage, alerting |
| A10 | SSRF | Egress filtering, URL allow-lists |

> **Mapping reminder.** A01 and A07 are D5 concerns; A02, A08, A09 are D7; A04 is D8's own design phase; A05, A06 are pipeline/DevSecOps concerns (see [[devsecops-and-cicd-security]]).

### Common Anti-Patterns

1. **Client-side validation only** — JavaScript validation is UX, not security. Always re-validate server-side.
2. **Trusting the framework to "handle it"** — React, Django, Rails escape by default in safe APIs, but every framework has escape hatches (`.innerHTML`, `mark_safe`, `html_safe`).
3. **Rolling custom crypto** — `Math.random` for tokens, hand-rolled hash functions, MD5 for passwords. (D3 concern; see [[../Domain-03-Security-Architecture-and-Engineering/Domain-03-Index]].)
4. **Permissive CORS** — `Access-Control-Allow-Origin: *` with credentials is a privilege-escalation primitive.
5. **Comments / dead code in production** — credentials, internal URLs, and TODO-Security bugs are gold for attackers.

## CISO / Risk Manager View

**Board framing.** Secure coding is **the cheapest dollar in cybersecurity**. A single hour of developer training can prevent a defect that would cost 100 hours of incident response. The board does not need to know what parameterized queries are; the board needs to know **what percentage of the codebase meets the ASVS level we have committed to**, and what the trend is.

**Strategic priorities.**

1. **Pick an ASVS level as the policy bar.** Most enterprises commit to ASVS L2 for all internal apps, L3 for customer-facing and regulated. The choice is auditable.
2. **Train every developer, annually, on the OWASP Top 10 + your stack's escape hatches.** A 1-day workshop per engineer per year is a 0.1% salary cost; it pays back in the first prevented incident.
3. **Integrate SAST/SCA into the IDE and the PR.** Finding a vuln at code-review time is 30× cheaper than finding it in production.
4. **Maintain a secure-coding standards doc per language.** A one-pager per stack (Python, Go, Java, JavaScript) listing the 5 rules + framework escape hatches + banned functions.
5. **Measure, don't exhort.** Track SAST findings per 1,000 LoC, time-to-remediate, % of code at ASVS L2.

**Metrics the board cares about.**

- % of code covered by SAST (target: 100% of trunk)
- High/critical findings per release (target: $\leq 2$)
- Mean time to remediate critical (target: $\leq 15$ days)
- % of developers with current secure-coding training (target: $\geq 90\%$)
- ASVS verification status by app tier (L2 / L3)

## Related Connections

### Sibling L2
- [[sdlc-models-and-security-integration]] — secure coding lives in the PW practice group
- [[threat-modeling-stride-pasta]] — identifies the threats these rules defend against
- [[api-security-and-microservices]] — Rule 1–5 apply at API edges with additional controls

### L3 Standards
- [[database-security-and-sql-injection]] — Rule 3 deep-dive: SQLi types, ORM pitfalls, WAF bypass
- [[devsecops-and-cicd-security]] — automated enforcement of these rules in CI

### Cross-Domain
- [[../Domain-02-Asset-Security/data-classification-and-handling]] — input-validation rigor is set by data tier
- [[../Domain-03-Security-Architecture-and-Engineering/Domain-03-Index]] — cryptographic primitives and key management
- [[../Domain-05-Identity-and-Access-Management/Domain-05-Index]] — authN/AuthZ rules in ASVS V2/V4
- [[../Domain-06-Security-Assessment-and-Testing/Domain-06-Index]] — DAST/IAST validate the output of these rules

## References

- OWASP Top 10 — Web Application Security Risks (2021)
- OWASP ASVS v4.0.3 — Application Security Verification Standard
- OWASP Cheat Sheet Series (input validation, XSS prevention, SQL injection prevention, logging)
- NIST SP 800-53 Rev. 5 — SI-10 (Information Input Validation), SA-11 (Developer Testing)
- CERT Secure Coding Standards (per language)
- (ISC)² CISSP CBK — Domain 8: Software Development Security
- CWE/SANS Top 25 Most Dangerous Software Errors
- PCI-DSS v4.0 — Requirement 6 (Develop and maintain secure systems and software)
