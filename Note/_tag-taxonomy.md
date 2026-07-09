# Tag Taxonomy — Offensive Security Controlled Conventions

> **Purpose:** Keep tags consistent across the vault so filtering, graph view, and
> Dataview queries remain reliable. Add new tags sparingly and only when they
> carve out a genuinely new dimension.

---

## Required Tags (every reusable note)

Every non-template note **must** carry at least one tag from the **Domain**
category, one from the **Tactic** category (if applicable to offensive ops),
and optionally from **Vuln-Class** when the note addresses a specific
vulnerability type. These go under `tags:` in the YAML frontmatter.

### Domain — *what area does this belong to?*

| Tag | Scope |
|-----|-------|
| `offsec` | General offensive security (fallback when no specific domain fits) |
| `pentest` | Internal/external penetration testing |
| `red-team` | Adversary simulation, covert operations, stealth tradecraft |
| `exploit-dev` | Exploit development, fuzzing, binary exploitation |
| `malware` | Malware development, payloads, droppers, loaders |
| `c2` | Command and control frameworks, profiles, protocols |
| `recon` | Asset discovery, enumeration, OSINT |
| `phishing` | Social engineering, phishing operations |
| `physical` | Physical security testing, lockpicking, badge cloning |
| `methodology` | Workflow, strategy, decision frameworks |
| `tool` | Tool usage, configuration, automation |
| `handbook` | Comprehensive all-in-one expert guide / synthesis |
| `reference` | Reference material, cheatsheet, lookup table |
| `playbook` | End-to-end walkthrough or runbook |
| `lab` | Lab setup, training environment, practice notes |
| `development` | General software development supporting offsec tooling |

### Tactic (MITRE ATT&CK Aligned) — *where in the attack lifecycle?*

| Tag | ATT&CK Tactic ID | Scope |
|-----|-------------------|-------|
| `reconnaissance` | TA0043 | Active/passive information gathering |
| `initial-access` | TA0001 | Gaining initial foothold |
| `execution` | TA0002 | Running malicious code |
| `persistence` | TA0003 | Maintaining access across reboots/credential changes |
| `privilege-escalation` | TA0004 | Gaining higher-level permissions |
| `defense-evasion` | TA0005 | Avoiding detection (AV, EDR, logging) |
| `credential-access` | TA0006 | Stealing credentials |
| `discovery` | TA0007 | Enumerating the environment |
| `lateral-movement` | TA0008 | Moving through the network |
| `collection` | TA0009 | Gathering targeted data |
| `c2` | TA0011 | Command and control communication |
| `exfiltration` | TA0010 | Data exfiltration |
| `impact` | TA0040 | Manipulate, interrupt, or destroy systems |

### Vuln-Class — *what kind of vulnerability?*

| Tag | Scope |
|-----|-------|
| `xss` | Cross-Site Scripting (all flavours) |
| `sqli` | SQL Injection |
| `ssrf` | Server-Side Request Forgery |
| `idor` | Insecure Direct Object Reference |
| `ato` | Account Takeover |
| `rce` | Remote Code Execution |
| `lfi` | Local File Inclusion |
| `csrf` | Cross-Site Request Forgery |
| `cors` | Cross-Origin Resource Sharing misconfig |
| `subtakeover` | Subdomain Takeover |
| `authz` | Authorization / Privilege Escalation |
| `info-disclosure` | Information Disclosure |
| `crypto` | Cryptographic weaknesses |
| `deser` | Deserialisation attacks |
| `injection` | Generic injection (template, LDAP, command, etc.) |
| `ssp` | Server-Side Prototype Pollution |
| `memory-corruption` | Buffer overflow, use-after-free, heap spray |
| `ad-exploit` | Active Directory specific exploits (Kerberos, NTLM, ACL abuse) |
| `kernel` | Kernel-level vulnerabilities and exploits |

---

## Optional Tags (use when relevant)

| Tag | Scope |
|-----|-------|
| `windows` | Windows / Active Directory specific |
| `linux` | Linux / Unix specific |
| `macos` | macOS specific |
| `mobile` | iOS / Android specific |
| `cloud` | AWS, GCP, Azure specific |
| `container` | Docker / Kubernetes specific |
| `network` | Network-level attacks, protocols |
| `wireless` | WiFi, Bluetooth, RF attacks |
| `api` | API / GraphQL testing |
| `web` | Web application attacks |
| `forensics` | Forensics / log analysis |
| `evasion` | AV/EDR bypass specific |
| `pivot` | Pivoting / tunnelling / proxying |
| `shellcode` | Shellcode development and loaders |
| `dll` | DLL hijacking, sideloading, injection |
| `driver` | Kernel driver exploitation, third-party driver abuse |
| `scheduled-task` | Scheduled task / cron based operations |
| `service` | Service abuse for persistence/lateral movement |
| `registry` | Windows Registry operations |
| `wmi` | WMI for execution/lateral movement |
| `winrm` | WinRM for remote execution |
| `smb` | SMB-based operations |
| `kerberos` | Kerberos specific attacks (golden, silver, delegation) |
| `ldap` | LDAP queries and abuse |
| `ntlm` | NTLM relay, capture, abuse |
| `dcom` | DCOM lateral movement |
| `ps-exec` | PsExec-style remote execution |
| `schtasks` | Scheduled tasks remote execution |
| `cobalt-strike` | Cobalt Strike specific |
| `sliver` | Sliver C2 specific |
| `metasploit` | Metasploit specific |
| `empire` | PowerShell Empire / Starkiller |
| `mythic` | Mythic C2 specific |
| `havoc` | Havoc C2 specific |
| `brute-ratel` | Brute Ratel C4 specific |
| `nishang` | Nishang PowerShell framework |
| `python` | Python tooling / scripts |
| `go` | Go tooling / implants |
| `c-sharp` | .NET / C# tooling |
| `rust` | Rust tooling / implants |
| `powershell` | PowerShell tradecraft |
| `bash` | Bash / shell scripting |
| `c` | C / native payload development |

---

## Convention Rules

1. **Lowercase + hyphen** — always. Use `defense-evasion` not `Defense_Evasion`.
2. **One concept per tag** — do not compound (`evasion-advanced` → `evasion` + `advanced`). Use separate tags for orthogonal dimensions.
3. **Prefer existing** — search before creating. A new tag must affect ≥ 3 notes or open a genuinely new category.
4. **Alias sparingly** — add `aliases:` in frontmatter only for common synonyms, not for every variation.
5. **Lifecycle** — tags in `Required Tags` are permanent. Optional tags may be deprecated; removal requires updating all affected notes.
6. **Tactic tags are primary** — every offensive technique note should carry its ATT&CK tactic tag. This enables `dataview` queries by lifecycle phase.
7. **Tool tags for C2** — use framework-specific tags (e.g., `cobalt-strike`, `sliver`) for notes that are specific to that C2. Generic C2 tradecraft uses `c2`.
8. **Domain distinguishes role** — `pentest` vs `red-team` vs `exploit-dev` reflect different operational constraints (time, stealth, scope). Choose the most restrictive applicable domain.

---

*Last reviewed: 2026-07-06*
