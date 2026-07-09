---
title: "Authorization Models — DAC, MAC, RBAC, ABAC, Rule-based, ReBAC"
layer: 2
domain: 5
tags:
  - cissp
  - domain-05
  - authorization
  - dac
  - mac
  - rbac
  - abac
  - rubac
  - rebac
  - zanzibar
related_domains:
  - "[[domain-01-security-and-risk-management]]"
  - "[[domain-02-asset-security]]"
  - "[[domain-03-security-architecture-and-engineering]]"
  - "[[identity-lifecycle-and-provisioning]]"
  - "[[zero-trust-architecture-nist-800-207]]"
---

# Authorization Models — DAC, MAC, RBAC, ABAC, Rule-based, ReBAC

## Feynman Explanation

Once you know **who** someone is, you still have to decide **what** they are allowed to do — read this file, approve that invoice, delete that database. The rules for making that decision are called *authorization models*. The oldest ones let a file's owner decide (DAC) or a government classification decide (MAC). The newer ones organize users into roles (RBAC), evaluate policies against attributes (ABAC), apply firewall-style rules (Rule-based), or walk a graph of relationships (ReBAC). The exam rewards knowing which model solves which problem — and the formal definitions below are how you prove it.

## Technical Details

### 1. The Six Canonical Models — One-Line Mental Model

| Model | One-line mental model | Decision authority |
|---|---|---|
| **DAC** | "I made the file, I decide who opens it." | The data owner |
| **MAC** | "The system enforces classification." | The system / classification lattice |
| **RBAC** | "You are a Manager, Managers can approve." | The role catalog |
| **ABAC** | "Your role + resource sensitivity + time + location = policy decides." | A policy engine (PDP) |
| **Rule-based** | "If source = 10.0.0.0/8, allow." | A static rule set |
| **ReBAC** | "You can see a doc if you are its author's manager." | A graph of relationships |

---

### 2. DAC — Discretionary Access Control (formal)

**Definition (Lampson 1971 / TCSEC Orange Book):** Access to an object is restricted based on the **identity of the subject** and the **discretion** of the object's **owner** to grant or revoke access.

Formally, the access matrix $M$ is a 3-tuple $(S, O, A)$ where:

- $S = \{s_1, s_2, \ldots, s_n\}$ — set of subjects
- $O = \{o_1, o_2, \ldots, o_m\}$ — set of objects
- $A = \{r, w, x, d, \ldots\}$ — set of access rights (read, write, execute, delete, …)
- $M[s_i, o_j] \subseteq A$ — the rights of subject $s_i$ on object $o_j$
- The owner of $o_j$ may freely modify $M[\cdot, o_j]$.

In practice, two sparse representations:

- **ACL** (Access Control List) — column of $M$ stored with the object: $\text{ACL}(o_j) = \{(s_i, M[s_i, o_j])\}$
- **Capability list** — row of $M$ stored with the subject: $\text{Cap}(s_i) = \{(o_j, M[s_i, o_j])\}$

#### Examples
- UNIX permission bits `rwxr-xr-x` (owner / group / other)
- NTFS ACLs (DACL + SACL)
- Windows share permissions
- Google Drive "share with anyone with the link"

#### Pros and cons

| Pros | Cons |
|---|---|
| Simple, intuitive | Vulnerable to **Trojan horse** (a program the user runs can act on every file the user owns) |
| Owner can delegate | **No central policy** — auditors hate it |
| Granular per-object | Prone to "share sprawl" |

> **Exam hook:** DAC is the model where the **owner** decides. If the question says "the user can grant access to a file they own," the answer is DAC.

---

### 3. MAC — Mandatory Access Control (formal)

**Definition:** Access is determined by comparing the **classification label** of the subject (a *clearance*) with the **classification label** of the object (a *sensitivity*). The decision is enforced by the system; neither owner nor subject can override it.

Formally, every subject $s$ has a clearance $L(s) \in \mathcal{L}$ and every object $o$ has a label $L(o) \in \mathcal{L}$, where $(\mathcal{L}, \leq)$ is a **lattice** (a partially ordered set where every pair has a least upper bound and greatest lower bound).

Common labels: $\mathcal{L} = \{\text{Unclassified} < \text{Confidential} < \text{Secret} < \text{Top Secret}\}$.

Two foundational lattice policies:

#### 3.1 Bell-LaPadula (1973) — confidentiality

| Rule | Statement |
|---|---|
| **No Read Up (NRU)** | $s$ can read $o$ iff $L(s) \geq L(o)$ (Simple Security) |
| **No Write Down (NWD)** | $s$ can write $o$ iff $L(s) \leq L(o)$ (*-property, "star property") |
| **Tranquility** | Labels do not change during a session (or only in well-defined ways) |

Effect: a Secret clearance can read Confidential and Secret, but cannot write to Unclassified. Prevents leaking high data to low labels.

#### 3.2 Biba (1977) — integrity

| Rule | Statement |
|---|---|
| **No Read Down (NRD)** | $s$ can read $o$ iff $L(s) \leq L(o)$ (Low-water-mark on read) |
| **No Write Up (NWU)** | $s$ can write $o$ iff $L(s) \geq L(o)$ (Low-water-mark on write) |

Effect: a "Trusted" process can write to Untrusted, but cannot be corrupted by reading low-integrity data. The model for medical devices, financial systems, SCADA.

> **Exam hook:** "Bell-LaPadula = confidentiality / no read up. Biba = integrity / no write up." BLP is what DoD uses for classified data. Biba is what banks use to keep bad data out of the books.

#### Implementations
- **SELinux** (Type Enforcement + MLS labels)
- **Trusted Solaris**
- **AppArmor** (path-based, a hybrid)
- **Windows Integrity Levels** (Low / Medium / High / System)
- **Lattice-based** systems: HighWall, TrustedBSD

#### Pros and cons

| Pros | Cons |
|---|---|
| Enforced by the system — owner cannot leak | Rigid; bad UX (every file must be labelled) |
| Proven by formal methods | Expensive to label legacy data |
| Strong for confidentiality AND integrity | Does not scale to dynamic policy |

---

### 4. RBAC — Role-Based Access Control (formal)

**Definition (NIST INCITS 359-2012, ANSI):** Access decisions are based on the **roles** a user is assigned to; each role bundles a set of **permissions**; each permission is a tuple of (operation, object).

Formally, an RBAC reference model has:

| Component | Notation |
|---|---|
| Users | $U$ |
| Roles | $R$ |
| Permissions (operations × objects) | $P = OPS \times OBS$ |
| Sessions | $S$ |
| User-Role assignment | $UA \subseteq U \times R$ |
| Role-Permission assignment | $PA \subseteq R \times P$ |
| Session-User mapping | `user : S → U` |
| Session-Role activation (subset of UA) | `roles : S → 2^R` |

#### NIST RBAC levels

| Level | Name | Adds |
|---|---|---|
| **RBAC0** | Core | $U, R, P, UA, PA$, sessions, role activation |
| **RBAC1** | Hierarchical | Partial order $R \succeq R$ (Senior $\succeq$ Junior inherits all permissions) |
| **RBAC2** | Constrained | Adds **separation of duties (SoD)**: static SoD (no user has both $r_1$ and $r_2$) and dynamic SoD (no user activates both in same session) |
| **RBAC3** | Combines RBAC1 + RBAC2 | Symmetric — hierarchies + constraints |

**Constraints (the heart of RBAC2/3):**

- **Static SoD (SSOD):** $\forall u \in U : \neg(\exists (r_1, r_2) \in \text{mutually\_exclusive} : (u, r_1) \in UA \wedge (u, r_2) \in UA)$
- **Dynamic SoD (DSOD):** $\forall s \in S : \neg(\exists (r_1, r_2) \in \text{mutually\_exclusive} : r_1, r_2 \in \text{roles}(s))$
- **Cardinality:** at most $N$ users can hold role $r$.
- **Prerequisite:** to hold $r_1$ you must already hold $r_2$.

#### Examples
- AD groups (`Domain Admins`, `HR-Readers`, `Finance-AP`)
- AWS IAM roles with attached policies
- SAP role catalog
- Salesforce profiles + permission sets

#### Pros and cons

| Pros | Cons |
|---|---|
| Scales to thousands of users | **Role explosion** when policies are not orthogonal |
| Easy to audit ("who has role X?") | Cannot express "approve only if amount < $10k AND during business hours" cleanly |
| Easy SoD enforcement | Static; poor for dynamic, context-rich decisions |
| Maps to org structure | Role mining is a project unto itself |

> **Exam hook:** RBAC is **role-membership**-based. If the question is "what can the **role** do," the answer is RBAC. If it is "what can **this user, in this context, on this resource, at this time** do," the answer is ABAC.

---

### 5. ABAC — Attribute-Based Access Control (formal)

**Definition (NIST SP 800-162):** Access is granted or denied based on the evaluation of **attributes** of the subject, action, resource, and environment against a **policy** that expresses a Boolean combination of conditions.

Formally, a policy $P$ is a function:

$$\text{permit}(s, a, r, e) = \bigvee_{i} \bigwedge_{j} c_{ij}(a_s, a_a, a_r, a_e)$$

where:

- $s$ = subject, $a$ = action, $r$ = resource, $e$ = environment
- $a_s, a_a, a_r, a_e$ are the **attribute** tuples of each
- $c_{ij}$ is a Boolean condition over attribute values
- The output is a Boolean decision; the **default-deny** principle means absence of `permit` = `deny`

#### Reference architecture (NIST 800-162)

| Component | Role |
|---|---|
| **PEP** (Policy Enforcement Point) | Intercepts request, asks PDP, enforces decision |
| **PDP** (Policy Decision Point) | Evaluates the policy against attributes, returns permit/deny |
| **PAP** (Policy Administration Point) | Authors / stores policies |
| **PIP** (Policy Information Point) | Fetches attribute values (e.g., from HR, IdP, MDM) |
| **PRP** (Policy Retrieval Point) | Stores policies for PDP |

#### Policy language standards

- **XACML** (OASIS) — XML-based, request/response, rule + policy + policy set
- **OPA / Rego** (CNCF) — declarative, increasingly dominant in cloud-native
- **ALFA** (OASIS) — XACML-friendly authoring language
- **Cedar** (AWS) — policy language for AWS Verified Permissions
- **Open Policy Agent / Rego** + **AWS IAM** + **Azure ABAC** are the most common in 2024

#### Example policy (OPA / Rego)

```rego
package http.authz

default allow = false

allow {
    input.method == "GET"
    input.path == ["salary", employee_id]
    input.subject.role == "manager"
    input.subject.user_id == data.org_chart.manager_of[employee_id]
    input.env.business_hours == true
}
```

#### Pros and cons

| Pros | Cons |
|---|---|
| Expressive — context, time, geo, risk score | Harder to reason about ("who has access to X?") |
| Centralized policy → consistent decisions | Requires attribute infrastructure (PIPs) |
| Fine-grained down to row/column level | Performance — evaluation cost can be high |
| Easy to externalize (OPA sidecar) | Tooling maturity still varies |

> **Exam hook:** ABAC = policy-on-attributes. "A doctor can read a record only if the patient is on their ward and the record is not flagged as restricted" — that's ABAC. RBAC would need a "DoctorOnWard7ForPatientX" role and would explode.

---

### 6. Rule-based Access Control (RuBAC / RuleBAC)

**Definition:** Access is permitted or denied based on a set of **rules** evaluated in order; each rule is a Boolean expression of conditions.

Formally, the rule set $\mathcal{R} = \{r_1, r_2, \ldots, r_k\}$ is evaluated:

$$\text{decision} = \begin{cases}
\text{allow} & \text{if } r_1(s, a, o) \text{ matches} \\
\text{deny}  & \text{if } r_2(s, a, o) \text{ matches} \\
\vdots & \vdots \\
\text{default} & \text{otherwise}
\end{cases}$$

#### Examples
- **Firewall rules** — `if src=10.0.0.0/8 and dst port=22 then allow`
- **Router ACLs**
- **Time-of-day restriction** — login allowed only 08:00-20:00
- **Geofencing** — block access from sanctioned countries
- **Cisco IOS ACLs**, **iptables**, **Windows AppLocker**

#### Rule-based vs ABAC

| | Rule-based | ABAC |
|---|---|---|
| Rule source | Static configuration | Externalized policy + attributes |
| Attribute source | Inline literals | PIP-sourced, dynamic |
| Composition | Order-sensitive | Boolean / nested |
| Governance | Hard to audit at scale | Centralized PAP |

> **Exam hook:** When the answer choice is "firewall rules" or "ACL," that is **rule-based**, not ABAC. ABAC uses an externalized policy engine with dynamic attributes.

---

### 7. Relationship-Based Access Control (ReBAC)

**Definition (Gates 2007; Google Zanzibar 2019):** Access is granted based on the **relationship** between the subject and the object (and, in Zanzibar, intermediate objects) in a directed graph of objects and subjects.

Formally, subjects and objects are vertices $V$ in a graph; tuples $(s, r, o)$ represent "subject $s$ has relationship $r$ to object $o$." A policy is a tuple-rewrite rule:

$$\text{permit}(s, a, o) \iff \exists\, (s, \text{member\_of}, g),\ (g, \text{editor\_of}, o)$$

i.e., "s can write o if s is a member of a group that is an editor of o."

#### Examples
- **Google Zanzibar** — the consistency layer behind Google Docs, Drive, Maps, YouTube
- **Auth0 FGA** — managed ReBAC on top of Auth0
- **OpenFGA** — open-source Zanzibar implementation
- **AWS IAM** — when modeled as resource policies with `aws:ResourceTag` chains

#### Pros and cons

| Pros | Cons |
|---|---|
| Models hierarchies and "ownership" naturally | Authorization checks become graph queries |
| Scales to billions of objects (Zanzibar: trillions) | Consistency model (Zanzibar "snippets" / Zookies) is non-trivial |
| Easy to express "share with my team" | Caching and revocation require care |

---

### 8. Comparison Table (the one to memorize)

| Property | DAC | MAC | RBAC | ABAC | Rule-based | ReBAC |
|---|---|---|---|---|---|---|
| **Decision authority** | Owner | System (label) | Role catalog | Policy engine | Rule set | Relationship graph |
| **Granularity** | Per-object owner grants | Per-label lattice | Per-role permission | Per-decision policy | Per-rule condition | Per-graph edge |
| **Centralized policy** | No | Yes (label taxonomy) | Yes (role catalog) | Yes (PAP) | Partial | Yes (graph) |
| **Dynamic context (time, geo, risk)** | No | No | Limited | **Yes** | Limited | Yes |
| **Best fit** | Personal file sharing | Classified, regulated | Org-wide, stable roles | Cloud, ZT, fine-grained | Network, infra | Sharing, multi-tenant |
| **Compliance** | Weak (share sprawl) | Strong (DoD) | Strong (SoD) | Strong (attribute-based) | Medium | Strong (graph audit) |
| **Role explosion risk** | n/a | n/a | High | Low | Low | Low |
| **Auditability** | Per-owner | Per-label | Per-role | Per-policy | Per-rule | Per-tuple |
| **Examples** | NTFS, UNIX | SELinux, BLP | AD, AWS IAM, SAP | OPA, Cedar, XACML | Firewall, iptables | Zanzibar, OpenFGA |

### 9. Hybrid Models in Practice

- **RBAC + ABAC** — start with a role, refine with attributes ("Manager in Finance in EU during business hours")
- **MAC + DAC** — SELinux supports both Type Enforcement (TE) and user-based permissions
- **PBAC** (Policy-Based) — vendor umbrella term, often = ABAC
- **Risk-based** — adaptive: re-decide based on UEBA risk score

### 10. Exam Pattern Recap

- **"Owner can grant"** → DAC
- **"Clearance vs classification"** → MAC → BLP / Biba
- **"Group/role membership"** → RBAC (RBAC0/1/2/3)
- **"Attribute, context, dynamic"** → ABAC
- **"Firewall rule"** → Rule-based
- **"Can the user X see the doc if they are in a group that owns it?"** → ReBAC
- **"No two roles in same session"** → Dynamic SoD (RBAC2)
- **"Role inheritance"** → RBAC1

## CISO / Risk Manager View

The choice of authorization model is one of the highest-leverage architectural decisions in an enterprise — and the *worst* choice is to mix models by accident, which is what happens in most enterprises that grew through acquisition. A CISO's playbook:

| Question | Board-friendly framing | Recommendation |
|---|---|---|
| Are we doing RBAC, ABAC, or both? | "How do we decide who gets in?" | Start with RBAC (catalogue of ~50-200 roles). Layer ABAC for the 5-10 highest-risk, most-dynamic decisions. |
| Do we have role explosion? | "How many roles does a single department have?" | > 10 roles per person = smell. Consolidate. |
| Are we enforcing SoD? | "Could one person commit a fraud and audit it?" | Implement static + dynamic SoD on financial systems. |
| Do we have least privilege for admins? | "Could a single account take down the company?" | Standing privilege → JIT + PAM. |
| Do we have any MAC-style data classification? | "Could an intern read the M&A file?" | Tag data by sensitivity; ABAC on top of the tags. |
| Can we prove "who can access X" in 60 seconds? | "What is the audit story?" | Centralized IGA. |

**Maturity ladder (authorization-specific):**

| Level | Name | Defining characteristic |
|---|---|---|
| 1 | Ad hoc | Local groups, no SoD, shared accounts |
| 2 | Defined | Centralized IdP, role catalogue, baseline SoD |
| 3 | Managed | IGA with recertification, role mining, SoD enforcement |
| 4 | Measured | ABAC for high-risk decisions, privileged access via PAM |
| 5 | Adaptive | Zero Trust, continuous authorization, policy-as-code, graph-based (ReBAC) for sharing |

**Privileged risk concentration:** in most enterprises, **5% of accounts own 70-80% of entitlements**. The single biggest authorization risk is standing admin. Move to JIT + approval workflow + session recording before spending on anything else.

**Compliance anchor:** SOX 404, PCI-DSS 7 (least privilege), HIPAA 164.308(a)(3) (workforce security), GDPR Art. 32 (technical safeguards) all require demonstrable access control. Authorization models are the *evidence*.

## Related Connections

### Sibling L2
- [[authentication-factors-and-mechanisms]] - The auth half of the AAA pair
- [[identity-lifecycle-and-provisioning]] - Role assignments must be reviewed on join / move / leave
- [[federation-sso-and-saml-oidc]] - Claims from the IdP are inputs to ABAC policies
- [[access-control-attacks-and-mitigations]] - The threat model: privilege escalation, SoD bypass

### L3
- [[kerberos-protocol-deep-dive]] - Kerberos authorization is largely RBAC (PAC + SIDs) over tickets
- [[multi-factor-authentication-mfa]] - Step-up auth at authorization time
- [[privileged-access-management-pam]] - The highest-stakes RBAC slice
- [[zero-trust-architecture-nist-800-207]] - Continuous authorization is the ABAC / PEP / PDP pattern at scale

### Cross-Domain
- [[domain-01-security-and-risk-management]] - Authorization is the operational face of "least privilege"
- [[domain-02-asset-security]] - Data classification labels feed MAC and ABAC
- [[domain-03-security-architecture-and-engineering]] - Reference monitor, TCB, secure kernel
- [[domain-07-security-operations]] - Authorization logs feed SIEM / UEBA

## Sources / References

- NIST SP 800-162 - Guide to Attribute Based Access Control (ABAC)
- ANSI / INCITS 359-2012 - Information technology - Role Based Access Control (NIST RBAC ref model)
- NIST SP 800-178 - A Comparison of Attribute Based Access Control (ABAC) Standards
- Department of Defense - Trusted Computer System Evaluation Criteria (TCSEC, "Orange Book"), 1983/1985
- Bell, D.E. & LaPadula, L.J. (1973) - "Secure Computer Systems: Mathematical Foundations"
- Biba, K.J. (1977) - "Integrity Considerations for Secure Computer Systems"
- Lampson, B.W. (1971) - "Protection" (origin of the access matrix)
- OASIS XACML 3.0 - eXtensible Access Control Markup Language
- CNCF Open Policy Agent / Rego documentation
- AWS Cedar policy language
- Google Zanzibar paper (Pang et al., 2019) - "Spanner: Becoming a SQL System"
- OpenFGA / Auth0 FGA documentation
- (ISC)² CISSP CBK 2024 - Domain 5.2 / 5.6
