---
title: Security Models — BLP, Biba, Clark-Wilson, Brewer-Nash, Graham-Denning, HRU, Lipner
layer: 2
domain: 3
tags: [cissp, security-models, bell-lapadula, biba, clark-wilson, brewer-nash, graham-denning, hru, lipner, reference-monitor, tcb, confidentiality, integrity]
related_domains: [1, 5, 8]
---

# Security Models

## Feynman Explanation
A security model is a *mathematical statement* of what "secure" means for a computer system, written before any code exists so designers can prove properties instead of hoping for them. Some models (Bell-LaPadula, Biba) sort subjects and objects into a lattice of labels and forbid certain information flows; others (Clark-Wilson) protect *commercial integrity* by certifying transactions; Brewer-Nash prevents conflicts of interest like an investment banker seeing two bidders; Graham-Denning and HRU are frameworks for reasoning about who can grant/revoke *which* permissions. Knowing these models is what lets a CISO point to a published theorem instead of arguing from intuition.

## Technical Details

### 1. Foundational Vocabulary

- **Subject** — active entity (user, process, device) that requests access.
- **Object** — passive container (file, record, table, memory segment).
- **Access mode** — read (r), write (w), append (a), execute (e).
- **Label / level** — security attribute assigned to subject or object.
- **Lattice** — partial order $(\mathcal{L}, \leq)$ on labels such that every two elements have a least upper bound (join) and greatest lower bound (meet). In BLP the lattice is a chain (total order); in compartmented MLS it is a product of chains.
- **Reference monitor** — abstract machine mediating every subject–object access.
- **TCB (Trusted Computing Base)** — totality of protection mechanisms enforcing the policy (hardware + software).
- **State** — $(b, M, f)$ where $b$ is current subject, $M$ is access matrix, $f$ is a function mapping subjects and objects to their security levels.

---

### 2. Bell–LaPadula (BLP) — Confidentiality for Military MLS

**Original paper**: Bell & LaPadula, MITRE Technical Report MTR-2997, 1973 (the formal model that drove the Orange Book / TCSEC).

#### 2.1 Sensitivity Lattice

Labels are pairs $(L, C)$ where:
- $L$ is a hierarchical classification: $\text{TS} > \text{S} > \text{C} > \text{U}$.
- $C$ is a set of *non-hierarchical compartments* (e.g., $\{Crypto, NATO, Nuclear\}$).

Dominance is defined:

$$
(L_1, C_1) \geq (L_2, C_2) \iff L_1 \geq L_2 \;\land\; C_1 \supseteq C_2
$$

Strict dominance $\succ$ is defined analogously. The least upper bound (join) of two labels is:

$$
\text{lub}\big((L_1, C_1),\, (L_2, C_2)\big) = (\max(L_1, L_2),\, C_1 \cup C_2)
$$

#### 2.2 Three Core Rules

Let $S$ be a subject with clearance $(L_s, C_s)$ and clearance-level function $f_s(S)$; let $O$ be an object with classification $(L_o, C_o)$.

| Rule | Form | Plain reading |
|------|------|---------------|
| **Simple Security (no read up — NRU)** | $S$ may read $O$ only if $f_s(S) \geq f_o(O)$ | A Secret subject cannot read Top Secret. |
| ***-property (no write down — NWD)** | $S$ may write $O$ only if $f_s(S) \leq f_o(O)$ | A Top Secret subject cannot write a Secret file. |
| **Discretionary Security (ds-property)** | $S$ may access $O$ only if the access matrix $M$ grants it | Application of need-to-know within the lattice. |

The *-property prevents *information leakage downward* (e.g., a TS user could embed a secret into a U file via "Trojan" logic; BLP forbids the channel). Combined with NRU, BLP enforces *peaceful coexistence* of multiple classification levels on one machine.

#### 2.3 Tranquility

- **Strong tranquility** — security labels of subjects and objects never change at runtime. Maximally restrictive but impractical for MLS systems.
- **Weak tranquility** — labels may change only in ways that do not violate the *-property (e.g., raising a user's clearance is allowed if no downgrading of objects happens). Most operating MLS systems (e.g., MULTICS, SELinux MLS) implement weak tranquility.
- **Dynamic tranquility** — labels may change freely (no safety guarantee; not used in certified systems).

#### 2.4 The "BLP is silent" result

BLP says nothing about *covert channels* (storage and timing) and nothing about integrity. The *-property combined with NRU yields a *total order* of allowed reads, which makes the lattice read-only effectively for lower levels — this is what enables confidentiality but cripples commercial usability.

#### 2.5 Lipner's Integrity / Commercial Use

Lipner (1975) layered Biba's *low-water-mark* and Clark-Wilson-style commercial controls on top of BLP to produce the first commercially usable model that combined confidentiality, integrity, and an *auditor* role.

---

### 3. Biba — Integrity (the "Biba Model" 1977)

Biba addresses the dual of BLP: how to keep *untrusted* data from corrupting *trusted* decisions. Subjects and objects are assigned an *integrity level* $(I, R)$ where $I$ is hierarchical (Crucial > Important > ...) and $R$ is a set of integrity *categories*.

#### 3.1 Three Variants

| Variant | Rule | Effect |
|---------|------|--------|
| **Strict integrity** (dual of BLP) | (a) No read down: $S$ may read $O$ only if $I_s \leq I_o$. (b) No write up: $S$ may write $O$ only if $I_s \geq I_o$. | Confines data inside integrity tiers; extremely restrictive. |
| **Low-water-mark (LWM)** | Subject integrity is *demoted* on read of a lower-integrity object. No write-up. | Tracks taintedness through the system. |
| **Ring (Biba-Clark-Wilson hybrid)** | A privileged *trusted* ring may write up and read down; everything else obeys strict integrity. | Practical middle ground; forms the basis of Trusted Solaris mandatory integrity controls. |

The strict integrity lattice is the strict BLP dual — together they form the *complete lattice* of confidentiality × integrity.

---

### 4. Clark–Wilson — Commercial Integrity (1987)

Clark and Wilson recognized that *commercial* integrity is not "no write up" but rather "every state change is a *well-formed transaction* that preserves a *consistency invariant* and is *separated* across duties."

#### 4.1 Core Constructs

- **CDI** — Constrained Data Item (data whose integrity must be preserved; e.g., a bank ledger).
- **UDI** — Unconstrained Data Item (input data of unknown integrity).
- **TP** — Transformation Procedure (the only programs allowed to modify CDIs).
- **IVP** — Integrity Verification Procedure (asserts that all CDIs satisfy the consistency invariant).

#### 4.2 Nine Certification / Enforcement Rules

**Certification rules (C1–C5)** — assessed by the *auditor*:
1. **C1** — All IVPs are valid and execute without help.
2. **C2** — For each CDI, all TPs that manipulate it preserve the invariant.
3. **C3** — The relation "TP/CDI list" is certified to enforce separation of duty.
4. **C4** — All TPs log to a tamper-proof *append-only* CDI.
5. **C5** — Any TP that takes a UDI as input either rejects it or transforms it into a CDI.

**Enforcement rules (E1–E5)** — assessed by the *system* (the TCB):
1. **E1** — Only TPs certified for a CDI may manipulate it.
2. **E2** — Subjects may only access CDIs through certified TPs.
3. **E3** — Subject–TP–CDI binding must satisfy the certified separation-of-duty relation.
4. **E4** — Only the *security officer* may change the certified TP list.
5. **E5** — Subjects must be *authenticated*; TP/CDI bindings are auditable.

#### 4.3 Well-Formed Transaction

A TP is *well-formed* if it takes a system from one state in which all CDIs satisfy the invariant to another such state. The composition of well-formed transactions is itself well-formed, which is the basis for the *application-level* integrity guarantee Clark-Wilson delivers.

#### 4.4 Two-Person Rule / Separation of Duty

Implemented via E3/C3: a single subject cannot perform both halves of a sensitive operation (e.g., the clerk who issues a check is not the clerk who reconciles the account). Modern analogues: four-eyes principle, dual-control key release, Maker-Checker in banking.

---

### 5. Brewer–Nash (Chinese Wall) — Conflict-of-Interest (1989)

A *non-lattice* discretionary model for commercial data with conflict-of-interest classes.

#### 5.1 Hierarchy

- **Objects** are organized into *company datasets* (e.g., Bank A records, Bank B records, Oil Co X records).
- **Datasets** are grouped into *conflict-of-interest classes* (e.g., "all banks", "all oil companies").
- **Subjects** are *user identities*.

#### 5.2 Access Rules (initially a subject has no history)

1. **Simple security** — $S$ may read $O$ iff $O$ is in the same dataset as some $O'$ already read by $S$, **or** $O$ is in a *conflict class* that $S$ has not yet accessed.
2. ***-property (write)** — $S$ may write $O$ only if $S$ can read $O$ **and** no object in a different dataset in the same conflict class is readable by $S$.

The *-property in Chinese Wall is the *secrecy-preserving write rule* — it prevents a subject from writing information inferred from a Bank A dataset into a Bank B dataset, even if the subject has read access to both (which is forbidden anyway by the *Sanitization* corollary).

#### 5.3 Implementation Pattern

- Tracks *access history per subject*; revocation on company switch.
- In modern systems, "company" maps to *tenant*, "conflict class" maps to *industry vertical*; re-architected as *tenant isolation* in cloud.

---

### 6. Graham–Denning (1972) — Secure Capability Operations

A *primitive* model for capability-based protection systems, defined as the secure state-transition rules of a system of subjects, objects, and rights.

#### 6.1 Primitives

Eight primitive operations on the access matrix $M$ and authority state:

$$
\begin{aligned}
\text{create\_object}(S, O) &\quad \text{— add row } O \\
\text{delete\_object}(S, O) &\quad \text{— remove row } O \\
\text{create\_subject}(S, S') &\quad \text{— add column } S' \\
\text{delete\_subject}(S, S') &\quad \text{— remove column } S' \\
\text{grant\_right}(S, O, R) &\quad \text{— add } R \text{ to } M[S, O] \\
\text{revoke\_right}(S, O, R) &\quad \text{— remove } R \text{ from } M[S, O] \\
\text{grant\_right\_to\_subject}(S, S', R) &\quad \text{— add } R \text{ to } M[S, S'] \text{ (transfer)} \\
\text{revoke\_right\_from\_subject}(S, S', R) &\quad \text{— remove } R \text{ from } M[S, S']
\end{aligned}
$$

#### 6.2 Eight Conditions for a *Secure State*

The state $(S_i, O_j, M_{ij})$ is "secure" iff:
1. Every subject $S$ has a unique identifier not in the active set.
2. Every object $O$ has a unique identifier not in the active set.
3. The access matrix $M$ is *type-consistent* (entries are rights from a closed set).
4. The *copy flag* for transferable rights is honored.
5. The *owner* of each object can grant/revoke rights on it.
6. The *controller* of each object can delegate.
7. *Conferring* rights is permitted only by the *owner*.
8. The system maintains an *audit log* of every primitive.

The *transferable right* and the *copy flag* together prevent the "Trojan horse" capability leak. Graham-Denning is the foundation of capability architectures (KeyKOS, E, Capsicum, AWS Nitro-style isolation).

---

### 7. Harrison–Ruzzo–Ullman (HRU, 1976) — Safety Analysis

HRU extends Graham-Denning with *commands* (procedures) that conditionally perform primitive operations. The *safety problem* is:

> *Given an access matrix $M$ and a set of commands, is there a sequence of commands that can grant a subject $S$ a right $R$ on an object $O$ that $S$ does not currently have?*

#### 7.1 The Result (Undecidability Theorem)

$$
\text{HRU safety is undecidable in the general case.}
$$

The proof reduces from the *halting problem*: a Turing machine $T$ on input $w$ halts iff a particular right $R$ can be leaked through a particular sequence of HRU commands. Consequence: there is no algorithm that, given *any* HRU protection system, can determine with certainty whether a future state can leak a right.

#### 7.2 Practical Decidable Sub-Cases

HRU is decidable (polynomial-time) when one of the following holds:
- **Mono-operational** systems (each command is a single primitive).
- **Mono-conditional** systems (each command has at most one condition).
- **Acyclic** subject–object relations.

These results drive real-world authorization engines: HRU-style analysis is run on simplified schemas; full safety is achieved by *domain-specific invariants* (e.g., Bell-LaPadula) layered on top.

---

### 8. Comparison Table

| Model | Year | Goal | Lattice? | Trojan defense? | Modern use |
|-------|------|------|----------|-----------------|------------|
| Bell-LaPadula | 1973 | Confidentiality (MLS) | Yes (chain) | Yes (via *-property) | SELinux MLS, MULTICS heritage |
| Biba | 1977 | Integrity | Yes (chain) | Yes (via no write up) | Trusted Solaris, Biba ring |
| Clark-Wilson | 1987 | Commercial integrity | No (uses CDIs) | Yes (well-formed TP) | Banking, ERP, SOX apps |
| Brewer-Nash | 1989 | Conflict-of-interest | No (uses COI classes) | Yes (history-based) | Multitenant SaaS, brokerage |
| Graham-Denning | 1972 | Capability primitives | n/a | Via copy flag | Capability OS, HSMs |
| HRU | 1976 | Safety analysis | n/a | Undecidable in general | Authorization engines |
| Lipner | 1975 | BLP + Biba commercial | Yes | Combined | Early banking MLS |

---

### 9. TCSEC / Common Criteria Mapping

| Class (TCSEC) | EAL (CC) | Required model |
|---------------|----------|----------------|
| C2 — Controlled Access Protection | EAL2 | DAC |
| B1 — Labeled Security | EAL3 | BLP (with labels) |
| B2 — Structured Protection | EAL4 | BLP + covert channel analysis |
| B3 — Security Domains | EAL5 | Reference monitor + TCB structure |
| A1 — Verified Design | EAL6+ | Formal top-level specification, BLP + covert channels |

## CISO / Risk Manager View

**Board framing**: "Can we *prove* to the regulator that our data is protected, not just claim it?" The answer is the model. Each of these models is a *regulatory shortcut*: PCI DSS, HIPAA, FedRAMP, SOX, and GLBA auditors recognize BLP (for data classification), Clark-Wilson (for transactional integrity), and Brewer-Nash (for Chinese-wall separation).

**Strategic deployment pattern**:
- **Customer data segregation** (multi-tenant SaaS) → Brewer-Nash + row-level security.
- **Financial transaction systems** → Clark-Wilson well-formed transactions + dual control.
- **National-security / government contracts** → BLP (confidentiality) + Biba (integrity) → Lipner.
- **Internal capability-based microservices** → Graham-Denning-style capability tokens, no ambient authority.
- **Authorization-system correctness reviews** → HRU safety analysis on simplified schemas.

**Common control gaps** the board will ask about:
1. Is the *reference monitor* verifiable (small, isolated, formally specified)?
2. Is there a *named, published model* the system is evaluated against?
3. Is the *audit log* tamper-proof (Clark-Wilson C4)?
4. Are *compartments* enforced by the kernel, not by application code?
5. Is *separation of duty* (E3) actually implemented at the identity layer?

**Vendor selection heuristic**: a vendor claiming Common Criteria EAL4+ evaluation must publish the *Protection Profile* and the *Security Target* documents. If they cannot, the claim is "in evaluation" — meaning nothing yet.

## Related Connections
- Architecture patterns: [[security-architecture-patterns]]
- Cryptography fundamentals: [[cryptography-fundamentals-and-math]]
- Domain 1 (risk framing) — risk drives the model choice
- Domain 5 (IAM) — implements the model's enforcement rules
- Domain 8 (secure development) — applies the model in code
- Domain 6 (assessment) — verifies the model holds
- Domain hub: [[domain-03-security-architecture-and-engineering]]

## References
- Bell, D. E.; LaPadula, L. J. (1973). *Secure Computer Systems: Mathematical Foundations*. MITRE MTR-2997.
- Biba, K. J. (1977). *Integrity Considerations for Secure Computer Systems*. MITRE MTR-3153.
- Clark, D. D.; Wilson, D. R. (1987). *A Comparison of Commercial and Military Computer Security Policies*. IEEE S&P.
- Brewer, D. F. C.; Nash, M. J. (1989). *The Chinese Wall Security Policy*. IEEE S&P.
- Graham, G. S.; Denning, P. J. (1972). *Protection — Principles and Practice*. AFIPS SJCC.
- Harrison, M. A.; Ruzzo, W. L.; Ullman, J. D. (1976). *Protection in Operating Systems*. CACM 19(8).
- Lipner, S. B. (1975). *A Comment on the Confinement Problem*. MITRE.
- DoD 5200.28-STD (1983, 1985). *Trusted Computer System Evaluation Criteria* (Orange Book / TCSEC).
- ISO/IEC 15408 (2022). *Common Criteria for Information Technology Security Evaluation*.
- Pfleeger, C. P.; Pfleeger, S. L. (2015). *Security in Computing*, 5th ed., Ch. 5–6.
