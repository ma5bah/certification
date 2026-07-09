---
title: Database Security and SQL Injection
layer: 3
domain: 8
tags: [cissp, sql-injection, sqli, nosqli, parameterized-queries, orm, waf, database-security]
related_domains: [2, 3]
parent: secure-coding-practices-owasp
---

# Database Security and SQL Injection

## Feynman Explanation

Imagine a restaurant where waiters write orders directly into the kitchen's ledger by shouting them across the counter. A clever customer who yells "and also give me the manager's house key" gets the key written in. **SQL injection is the same trick played on a database: the user types something the application does not expect into a form, and the database — which only knows how to follow orders — happily runs the malicious instruction along with the real one.** The fix is structural: never let user input touch the database query directly. Treat the query as a *template* and the user input as a *parameter* that the database driver will safely stitch in.

## Technical Details

### How SQL Injection Works

A normal query:
```sql
SELECT id, name FROM users WHERE name = 'alice'
```

A malicious input `alice' OR '1'='1` produces:
```sql
SELECT id, name FROM users WHERE name = 'alice' OR '1'='1'
```

The condition `'1'='1'` is always true → the query returns every row in the table. Worse, with stacked queries, the attacker can append `; DROP TABLE users; --` to delete the table.

### The Four Primary SQLi Types

| Type | Mechanism | Detection Difficulty | Example |
|---|---|---|---|
| **Union-based** | Attacker uses `UNION SELECT` to extract data in the response | Easy — visible in output | `' UNION SELECT username, password FROM users-- ` |
| **Boolean-based (blind)** | Attacker infers data via true/false response differences | Medium — many requests | `' AND SUBSTRING(version(),1,1)='P` (true if Postgres) |
| **Time-based (blind)** | Attacker infers data via response delay | Hard — no output | `'; IF (1=1) WAITFOR DELAY '0:0:5'-- ` (5s delay if true) |
| **Error-based** | Attacker triggers DB errors that leak schema | Easy — verbose errors only | `' AND 1=CONVERT(int, version())-- ` (MSSQL leaks version) |

### Out-of-Band SQLi

A fifth variant: extract data via DNS or HTTP callbacks to a server the attacker controls. Used when time-based is too slow and boolean-based is unreliable. Defenses: egress filtering, WAF, IDS for unusual outbound traffic from the DB.

### Vulnerable vs Safe — Side-by-Side

```python
# VULNERABLE — string concatenation
def login_vuln(username, password):
    sql = f"SELECT id FROM users WHERE name='{username}' AND pwd='{password}'"
    return db.execute(sql).fetchone()
# input: username="' OR 1=1-- " → returns first user; auth bypassed

# SAFE — parameterized (prepared statement)
def login_safe(username, password):
    sql = "SELECT id FROM users WHERE name=? AND pwd=?"
    return db.execute(sql, (username, password)).fetchone()
# driver sends SQL and parameters separately; safe regardless of input

# SAFER — use an ORM with bound params, plus hashed password
def login_orm(username, password):
    user = User.objects.filter(name=username).first()
    if user and bcrypt.verify(password, user.pwd_hash):
        return user.id
    return None
```

### Parameterized Query Language Matrix

| Language / Driver | Syntax | Notes |
|---|---|---|
| Python `sqlite3` / `psycopg2` | `db.execute("... ? ...", (param,))` | `?` for sqlite, `%s` for psycopg2 |
| Python SQLAlchemy | `db.execute(text("... :name ..."), {"name": val})` | Bound parameters via `text()` |
| Java JDBC | `PreparedStatement ps = conn.prepareStatement("... ? ..."); ps.setString(1, val);` | Always use `PreparedStatement`, not `Statement` |
| Node.js `pg` | `client.query("... $1 ...", [val])` | `$1`, `$2` placeholders |
| Node.js `mysql2` | `conn.query("... ? ...", [val])` | `?` placeholders |
| C# ADO.NET | `cmd.Parameters.AddWithValue("@name", val)` | Named or positional `@name` |
| PHP PDO | `$stmt = $pdo->prepare("... :name ..."); $stmt->execute([":name" => $val]);` | Emulated prepares off for MySQL |
| Go `database/sql` | `db.Query("... $1 ...", val)` | Placeholders are `$1`, `$2` |

### Why ORMs Are (Mostly) Safe

ORMs (Django ORM, SQLAlchemy, Hibernate, ActiveRecord, Entity Framework) compose queries as **expression trees**, not strings. The driver translates the tree to SQL with bound parameters. This eliminates the most common SQLi class.

| Safe ORM Pattern | Unsafe ORM Pattern |
|---|---|
| `User.objects.filter(name=user)` | `User.objects.raw(f"SELECT * FROM users WHERE name='{user}'")` |
| `db.session.query(User).filter(User.id == uid)` | `db.session.execute(text(f"SELECT * FROM users WHERE id={uid}"))` |
| `User.where(email: email)` | `User.where("email = '#{email}'")` |

**Rule of thumb:** If the ORM call accepts a string and a variable, it is concatenated and vulnerable. If it accepts keyword arguments or a structured expression, it is safe.

### WAF Bypass Techniques (and Why Defense-in-Depth Matters)

A WAF (Web Application Firewall) is a band-aid, not a fix. Attackers routinely bypass them. Knowing the bypass patterns helps defenders understand why **parameterized queries are the only true fix**.

| Bypass Technique | Example |
|---|---|
| **Comment injection** | `'/**/OR/**/1=1-- ` (spaces replaced with comment blocks) |
| **Case variation** | `' oR 1=1-- ` (most WAFs are case-sensitive on rules) |
| **URL encoding** | `%27%20OR%201%3D1` (decoded by app server after WAF) |
| **Double encoding** | `%2527OR%25201%253D1` (decoded twice) |
| **HPP (HTTP Parameter Pollution)** | `?id=1&id=' OR 1=1-- ` (server picks one, WAF sees other) |
| **Unicode normalization** | `ʻ OR 1=1-- ` using U+02BC modifier letter apostrophe |
| **Hex encoding** | `0x27204f5220313d31` (string `'OR 1=1`) |
| **Time-based delay in benign endpoint** | Time-based blind SQLi that mimics normal latency |
| **Second-order SQLi** | Payload stored in DB, executed later by another query — WAF sees the second query, not the first |
| **JSON / XML body** | Payload in `{"name":"' OR 1=1-- "}` body, not URL — some WAFs don't inspect bodies |

> **Lesson.** Treat WAFs as *defense in depth* — they buy time, not safety. The actual fix is **parameterized queries + least privilege + input validation**.

### Database Hardening Beyond SQLi

| Control | Purpose |
|---|---|
| **Least-privilege DB users** | App user has `SELECT/INSERT/UPDATE/DELETE` only — no `DROP`, no `GRANT`, no access to admin tables |
| **No DDL in app queries** | Schema changes are migrations, not runtime SQL |
| **Read-only replicas for analytics** | Reduce blast radius of compromised app server |
| **Encryption at rest (TDE / cloud-native)** | Disk theft → no plaintext |
| **Encryption in transit (TLS to DB)** | Wire-tap → no plaintext (still common failure!) |
| **Audit logging of all queries** | Detect exfiltration in real time |
| **Row-level security (RLS)** | Tenant isolation at the DB layer |
| **Dynamic data masking** | Hide sensitive columns from non-privileged roles |
| **Query timeouts / resource governors** | Limit blast radius of expensive/recursive queries |
| **Backup encryption + tested restore** | Ransomware resilience |

### Credential and Connection Security

```python
# Bad — cleartext, embedded
conn = psycopg2.connect(
    "host=db.prod port=5432 dbname=orders user=app password=secret"
)

# Good — TLS enforced, secret from vault, no password in command line
import os, hvac
vault = hvac.Client(url=os.environ["VAULT_ADDR"],
                    token=open(os.environ["VAULT_TOKEN"]).read())
creds = vault.secrets.database.generate_credentials("app-db", ttl="15m")
conn = psycopg2.connect(
    host=os.environ["DB_HOST"],
    port=5432,
    dbname="orders",
    user=creds["username"],
    password=creds["password"],
    sslmode="verify-full",
    sslrootcert="/etc/ssl/certs/db-ca.pem",
)
```

> **Mandatory settings.** `sslmode=verify-full` (Postgres) or equivalent — never `disable` or `allow` or `prefer`. The CA bundle verifies the server cert, defeating MITM.

### NoSQL Injection

NoSQL does not use SQL, but it has the *same vulnerability class* — unvalidated user input concatenated into a query.

```javascript
// VULNERABLE — MongoDB
db.users.findOne({ name: req.body.name, pwd: req.body.pwd });
// input: { "name": "admin", "pwd": { "$ne": null } }
// → matches first user where pwd is not null → auth bypass

// SAFE — type validation + parameterized query
const { name, pwd } = req.body;
if (typeof name !== 'string' || typeof pwd !== 'string') {
  return res.status(400).end();
}
const user = await User.findOne({ name, pwd_hash: await bcrypt.hash(pwd, 10) });
```

Other NoSQLi patterns:

| Pattern | Payload | Defense |
|---|---|---|
| **Operator injection** | `{$ne: null}`, `{$gt: ""}` | Type-check all inputs; reject objects where strings are expected |
| **Regex DoS** | `{$regex: "(a+)+$"}` | Reject regex input from user; cap query time |
| **JavaScript injection (older MongoDB)** | `{$where: "this.x == userInput"}` | Disable `$where` server-side; never pass user input to it |
| **GraphQL injection** | Crafted nested query that bypasses resolver auth | Field-level authZ; query depth/complexity limits |

### Detection and Response

| Signal | Tool | Meaning |
|---|---|---|
| `UNION SELECT` in URL/body | WAF / ModSecurity | Union-based SQLi attempt |
| Slow queries with `SLEEP()` / `WAITFOR` | DB query log + APM | Time-based blind SQLi |
| Errors containing `SQL`, `syntax`, `ODBC` | App error log / WAF | Error-based SQLi |
| Sudden DB egress traffic | NDR / DB firewall | Data exfiltration in progress |
| DB user escalating privileges | DB audit log | Post-exploitation |

### Common Anti-Patterns

1. **`f-string` / template interpolation into SQL** — the #1 cause of SQLi in Python and Node tutorials.
2. **Trusting ORM `raw()` / `extra()`** — they re-introduce concatenation.
3. **"We're safe because we have a WAF"** — WAF is bypass-able; only parameterization is reliable.
4. **DB user with `DBA` privileges** — every SQLi is a potential `DROP DATABASE`.
5. **Storing passwords in plaintext or MD5** — SQLi becomes a credential breach.
6. **No TLS to the DB** — wire-tap on the same VPC.

## CISO / Risk Manager View

**Board framing.** SQL injection is the **oldest, most-known, and still most-prevalent** application vulnerability. OWASP has listed injection as #1 (or tied) for 20+ years. The reason it persists is that **string concatenation is a habit, not a bug** — it requires cultural and tooling change to eliminate. The board does not need to know the four SQLi types; the board needs to know **what percentage of database queries in our codebase are parameterized, and whether our DB credentials are least-privilege and rotated**.

**Strategic priorities.**

1. **Set parameterized queries as a non-negotiable gate.** A PR that contains string-concatenated SQL fails review. Linting rules (e.g., `semgrep` `python-flask.security.audit.formatted-sql-query`) catch the pattern in CI.
2. **Eliminate DB `root` / `sa` / `DBA` credentials from app servers.** Service-account DB users with least-privilege; quarterly privilege review.
3. **Enforce TLS to every database.** "Prefer" is not a setting — `require` / `verify-full` is the only acceptable posture.
4. **Adopt an ORM and audit the unsafe escape hatches.** `raw()`, `extra()`, `text()`, `RawSQL`, `to_sql()` — review every use.
5. **Run continuous DAST + IAST in staging.** ZAP, Burp, Contrast, DataDog ASM. Findings in production cost 10×.
6. **Encrypt sensitive columns at the application layer** (envelope encryption / Vault transit) — DB compromise does not leak plaintext.

**Metrics the board cares about.**

- % of queries parameterized (target: 100%; anything else is a P0 finding)
- # of SQL injection findings in last DAST scan (target: 0 high/critical)
- % of DB connections enforcing TLS (target: 100%)
- # of DB accounts with `DBA`/`superuser` privileges (target: 0 in app tier)
- Mean time to patch a new SQLi finding (target: critical $\leq 7$ days)

## Related Connections

### Parent L2
- [[secure-coding-practices-owasp]] — Rule 3 (parameterize queries) and ASVS V5 origin

### Sibling L2
- [[api-security-and-microservices]] — API #1 (BOLA) and API #7 (SSRF) often chain with SQLi
- [[sdlc-models-and-security-integration]] — SQLi prevention is a PW practice

### Sibling L3
- [[devsecops-and-cicd-security]] — SAST/SCA gates catch the vulnerable code in PR
- [[software-supply-chain-attacks-slsa]] — ORM package itself can be a supply-chain vector

### Cross-Domain
- [[../Domain-02-Asset-Security/data-classification-and-handling]] — encryption strength driven by data tier
- [[../Domain-03-Security-Architecture-and-Engineering/Domain-03-Index]] — DB encryption (TDE) and TLS primitives

## References

- OWASP — SQL Injection Prevention Cheat Sheet
- OWASP Top 10 — A03: Injection (2021)
- OWASP ASVS v4.0.3 — V5 (Validation, Sanitization, Encoding); V12 (Files and Resources)
- CWE-89 — Improper Neutralization of Special Elements used in an SQL Command
- PortSwigger Web Security Academy — SQL Injection (free labs)
- NIST SP 800-53 Rev. 5 — AC-3, AC-6, SI-10, SC-8, SC-13, SC-28
- PCI-DSS v4.0 — Requirement 6.2.4 (Injection prevention)
- (ISC)² CISSP CBK — Domain 8: Software Development Security
