# Note System Structure — Offensive Security

## Knowledge Layers

```
Layer 0: Master Checklist / System Entry
  Purpose:
    Central checklist, system rules, taxonomy, templates, drafts, archive, and reading queue.

  Valid paths:
    Note/offsecChecklist.md
    Note/_note-system-structure.md
    Note/_tag-taxonomy.md
    Note/StandardNote.md
    Note/Reading-Queue.md
    Note/draft/
    Note/_archive/

Layer 1: Category / Reference / Organization
  Purpose:
    Broad offensive categories aligned to the attack lifecycle, category hubs, general resources, framework references, and tool references.

  Valid paths:
    Note/Categories/<Tactic>.md
    Note/Categories/<Tactic>/<subtopic>.md
    Note/References/
    Note/Resources/
    Note/framework/
    Note/Tools/

  Tactic categories (MITRE ATT&CK aligned):
    Note/Categories/Reconnaissance.md
    Note/Categories/Initial-Access.md
    Note/Categories/Execution.md
    Note/Categories/Persistence.md
    Note/Categories/Privilege-Escalation.md
    Note/Categories/Defense-Evasion.md
    Note/Categories/Credential-Access.md
    Note/Categories/Discovery.md
    Note/Categories/Lateral-Movement.md
    Note/Categories/Collection.md
    Note/Categories/Command-and-Control.md
    Note/Categories/Exfiltration.md
    Note/Categories/Impact.md

  Cross-cutting categories:
    Note/Categories/Exploit-Development.md
    Note/Categories/Malware-Development.md
    Note/Categories/Active-Directory.md
    Note/Categories/Social-Engineering.md
    Note/Categories/Physical-Security.md
    Note/Categories/OSINT.md
    Note/Categories/C2-Frameworks.md
    Note/Categories/Evasion.md

Layer 1.5: Expert Handbooks / All-in-One Guides
  Purpose:
    Massive, comprehensive syntheses of multiple Layer 2 and Layer 3 notes into definitive expert-level master guides.

  Valid paths:
    Note/Handbooks/<topic>-expert-handbook.md
    Note/Handbooks/<topic>-comprehensive-guide.md

  Rule:
    Use Layer 1.5 for notes that act as the ultimate overarching authority on a major offensive domain.

  Examples:
    Note/Handbooks/red-team-ops-expert-handbook.md
    Note/Handbooks/adversary-simulation-expert-handbook.md
    Note/Handbooks/malware-dev-expert-handbook.md
    Note/Handbooks/c2-framework-deep-dive.md
    Note/Handbooks/active-directory-attack-handbook.md
    Note/Handbooks/osint-operations-handbook.md

Layer 2: Specific Technique / Methodology / Playbook
  Purpose:
    Broad reusable executable techniques, structured testing workflows, methodology notes, playbooks, and how-to notes aligned to offensive tactics.

  Valid paths:
    Note/Techniques/<Tactic>/<specific-technique>.md
    Note/Techniques/<Tactic>/<TechniqueCluster>/<specific-technique>.md
    Note/Methodology/
    Note/HowTo/
    Note/Playbooks/
    Note/Recon/
    Note/Payloads/

  Rule:
    Use Layer 2 when the note is executable and broadly reusable across engagements.

  Examples:
    Note/Techniques/Initial-Access/phishing-with-custom-templates.md
    Note/Techniques/Persistence/scheduled-task-triggers.md
    Note/Techniques/Privilege-Escalation/KrbRelay-service-for-dcsync.md
    Note/Techniques/Defense-Evasion/etw-pat-memory-bypass.md
    Note/Techniques/Lateral-Movement/dcom-lateral-execution.md
    Note/Techniques/C2/domain-fronting-via-cdn.md
    Note/Methodology/adversary-emulation-planning.md
    Note/Methodology/internal-pentest-methodology.md
    Note/HowTo/bypass-defender-with-custom-loader.md
    Note/Playbooks/stealth-kerberos-attacks.md
    Note/Payloads/dll-hijacking-scaffold.md

Layer 3: Niche Variant / Applied / Source-Driven / Reusable Learning
  Purpose:
    Narrow technique variants, tool-specific implementations, platform-specific edge cases, source-driven writeups, labs, findings, talks, target notes, lessons, research notes, and applied reusable knowledge.

  Valid paths:
    Note/Techniques/<Tactic>/<TechniqueCluster>/<niche-variant>.md
    Note/Labs/
    Note/Lessons/
    Note/Talks/
    Note/Targets/
    Note/RedTeam/
    Note/C2-Profiles/
    Note/Exploit-Development/
    Note/Physical/
    Note/OSINT/
    Note/Development/
    Note/Supply-Chain/

  Rule:
    Use Layer 3 when the note is narrow, tool-specific, platform-specific, source-driven, or a lesson heuristic.

  Examples:
    Note/Techniques/C2/sliver-https-profile-hardening.md
    Note/Techniques/Defense-Evasion/amsi-patch-via-veh-hook.md
    Note/Techniques/Privilege-Escalation/printnightmare-msfvenom-payload.md
    Note/Labs/sliver-lab-deployment.md
    Note/Lessons/evasion/etw-bypass-choice-of-api.md
    Note/Talks/black-hat-use-after-free-trends.md
    Note/Targets/conti-leak-playbook.md
    Note/C2-Profiles/malleable-c2-profiles.md
    Note/Exploit-Development/windows-x64-kernel-exploit-primer.md
    Note/Physical/wireless-deauth-attack-setup.md
```

## Directory Structure

```
Note/
├── offsecChecklist.md
├── _archive/
├── _tag-taxonomy.md
├── _note-system-structure.md
│
├── Handbooks/          <- L1.5 massive all-in-one expert guides
│
├── Categories/         <- L1 tactic hubs and category references
│   ├── Reconnaissance.md
│   ├── Initial-Access.md
│   ├── Execution.md
│   ├── Persistence.md
│   ├── Privilege-Escalation.md
│   ├── Defense-Evasion.md
│   ├── Credential-Access.md
│   ├── Discovery.md
│   ├── Lateral-Movement.md
│   ├── Collection.md
│   ├── Command-and-Control.md
│   ├── Exfiltration.md
│   ├── Impact.md
│   ├── Exploit-Development.md
│   ├── Malware-Development.md
│   ├── Active-Directory.md
│   ├── Social-Engineering.md
│   ├── Physical-Security.md
│   ├── OSINT.md
│   ├── C2-Frameworks.md
│   ├── Evasion.md
│   └── <Tactic>/
│       └── <subtopic>.md
│
├── Techniques/         <- specific executable technique notes by tactic
│   ├── Reconnaissance/
│   ├── Initial-Access/
│   ├── Execution/
│   ├── Persistence/
│   ├── Privilege-Escalation/
│   ├── Defense-Evasion/
│   ├── Credential-Access/
│   ├── Discovery/
│   ├── Lateral-Movement/
│   ├── Collection/
│   ├── Command-and-Control/
│   ├── Exfiltration/
│   ├── Impact/
│   └── <Tactic>/<TechniqueCluster>/
│
├── Lessons/            <- heuristics and reusable offensive lessons
│   ├── evasion/
│   ├── exploitation/
│   ├── methodology/
│   ├── mindset/
│   ├── reconnaissance/
│   ├── tooling/
│   ├── chaining/
│   ├── c2/
│   └── security/
│
├── Labs/               <- lab setup and practice notes
├── Payloads/           <- payload templates and patterns
├── C2-Profiles/        <- C2 profile configs and patterns
├── Exploit-Development/
├── Physical/
├── OSINT/
│
├── RedTeam/
├── Targets/
│
├── References/
├── Resources/
├── framework/
├── Recon/
├── Tools/
├── Methodology/
├── HowTo/
├── Playbooks/
├── Talks/
├── Development/
├── Supply-Chain/
│
├── StandardNote.md
├── Reading-Queue.md
└── draft/
```

## Hierarchy System

- **Layer 0** = master checklist and system entry files; use for actionable checklist links, taxonomy, drafts, archive, and queue.
- **Layer 1** = general hub; broad static tactic page like `Categories/Reconnaissance.md`, `Categories/Defense-Evasion.md`.
- **Layer 1.5** = expert handbook; the ultimate synthesis of a major offensive domain (e.g., `adversary-simulation-expert-handbook.md`).
- **Layer 2** = specific deep-dive; standalone technique, workflow, methodology, playbook, or complex reusable offensive method. Always placed under the relevant tactic.
- **Layer 3** = niche variant; tool-specific quirk, platform-specific edge case, chain detail, target-specific finding, or source-driven applied note.

- General note name = broad tactic only: `reconnaissance.md`, `defense-evasion.md`, `c2-frameworks.md`
- Specific note name = what makes it unique: `kerberos-s4u2self-abuse-for-privesc.md`, `etw-patching-via-ntdll-hook.md`
- Niche note name = context plus technique: `sliver-dns-c2-profile-hardening.md`, `windows-defender-bypass-via-msbuild.md`
- Specific + general split = specific note preserves concrete context; general note captures transferable pattern.
- Promote to Layer 2 when content has a multi-step execution flow, unique tooling, custom payload, full methodology, or broad reuse across engagements.
- Promote to Layer 3 when content is a narrow variant, tool-specific behavior, rare edge case, or chain detail.
- If an existing note already covers the topic, add a delta instead of creating a new sibling.
- Every Layer 2+ note must link up to a parent; every parent should reference its children.

**Routing Invariant (Evaluation Order)**:
1. **Decompose**: For multi-entity sources, plan a parent hub before routing children.
2. **Delta / `section_add`**: Add net-new content to an existing note if it already covers the topic.
3. **Lesson**: Extract to `Lessons/` if the value is a mental-model/heuristic pivot.
4. **Delta + Lesson**: Use both if there is a technical add-on and a separable heuristic.
5. **New Note**: Last resort only. Always search first to confirm no merge target.

## Naming

- Use lowercase kebab-case for new note files: `kerberos-s4u2self-abuse.md`.
- Use acronyms only when they are the stable topic name: `DCOM`, `RPC`, `ETW`, `AMSI`, `RDP`, `SMB`, `LDAP`, `AD`, `C2`.
- Use `_` prefix for system/index files: `_tag-taxonomy.md`, `_note-system-structure.md`.
- **Generic vs Specific Rule**: Generic names are ONLY allowed for Layer 1 hubs (`Categories/Reconnaissance.md`). Specific notes (Layer 2+) MUST have specific, descriptive filenames.
- **General naming formula**: `<tactic>.md`; never name a general note after a vendor.

**Specific naming styles**:
- **Tactic-first**: `<tactic>-<technique>` (`privesc-via-unquoted-service-paths.md`)
- **Mechanism-focused**: `<mechanism>-<technique>` (`etw-pat-memory-scan-bypass.md`)
- **Tool-focused**: `<tool>-<technique>` (`sliver-dns-c2-multiplexing.md`)
- **Platform-focused**: `<platform>-<technique>` (`linux-persistence-vs-systemd-service.md`)
- **Target-focused**: `<target>-<vuln>` (`exchange-proxylogon-chain.md`)
- **Chain-description**: `<technique>-to-<impact>-via-<mechanism>` (`krbrelay-to-dcsync-via-shadow-copy.md`)

### Placement Rules

1. Bare tactic names belong in `Categories/` (Generic Names).
   - Use `Categories/Reconnaissance.md`, not `Techniques/reconnaissance.md`.
   - `Techniques/` must use specific technique names, not duplicate category names.

2. `Categories/` is for organization and reference.
   - Use `Categories/<Tactic>.md` for the primary tactic hub.
   - Use `Categories/<Tactic>/<subtopic>.md` only for specific category-level reference material.
   - Do not place step-by-step exploitation techniques in `Categories/`.

3. `Techniques/` is for executable technique content (Specific Names Only).
   - Use `Techniques/<Tactic>/<specific-technique>.md` when the technique stands alone.
   - Use `Techniques/<Tactic>/<TechniqueCluster>/<specific-technique>.md` when a tactic has enough related technique notes to justify a cluster.
   - Use `Techniques/<Tactic>/<TechniqueCluster>/<niche-variant>.md` for narrow variants inside that cluster.
   - The folder path does not determine L2 vs L3 by itself; the note scope does. Broad reusable technique = L2. Narrow tool/platform/edge-case variant = L3.
   - `Techniques/Privilege-Escalation/printnightmare-exploitation.md` is L2 when it is a general reusable technique.
   - `Techniques/Privilege-Escalation/printnightmare-msfvenom-payload.md` is L3 because it is tool-specific (msfvenom).

4. Create a cluster directory only when it prevents clutter.
   - A cluster is justified when there are at least three related notes, or one parent topic with expected child variants.
   - Do not create a subfolder for a single note.

5. `Labs/` stores environment setups, practice notes, and training reflections.
   - Extract reusable techniques from `Labs/` into `Techniques/<Tactic>/` when they become standalone testing methods.

6. Legacy duplicates are not naming patterns.
   - Existing files that deviate from these rules are legacy.
   - New notes should follow the specific naming rules above.
