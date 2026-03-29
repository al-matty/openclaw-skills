---
name: spring-clean
description: >
  Workspace file audit: check injected file token budgets, cross-file conflicts,
  redundancy, stale info, misplaced content, and unreferenced non-injected files.
  Produces a diff-preview for human approval before any writes.
  Use when: (1) cron triggers a periodic audit, (2) user says "spring clean" or
  "workspace audit" or "check workspace files", (3) after major config/infra changes
  to verify file consistency.
---

# Spring Clean — Workspace File Audit

Audit all workspace files for bloat, conflicts, redundancy, and misplacement.
Output a diff-preview for human approval. Never write without approval.

## Token Budgets (injected files)

These files are auto-injected into every turn's context window. Keep them lean.

| File | Target | Hard Max | Purpose |
|---|---|---|---|
| AGENTS.md | ~700 tok (~2,800B) | 1,000 tok (4,000B) | Behavioral rules only |
| SOUL.md | ~450 tok (~1,800B) | 600 tok (2,400B) | Persona and tone |
| USER.md | ~220 tok (~900B) | 400 tok (1,600B) | Factual profile |
| TOOLS.md | ~670 tok (~2,700B) | 1,000 tok (4,000B) | Environment-specific tool notes |
| IDENTITY.md | ~80 tok (~330B) | 150 tok (600B) | Name, emoji, signature |
| HEARTBEAT.md | ~260 tok (~1,050B) | 400 tok (1,600B) | Heartbeat checklist |
| MEMORY.md | ~600 tok (~2,400B) | 1,500 tok (6,000B) | Curated long-term memory |
| **Total** | **~3,000 tok** | **5,000 tok** | — |

MEMORY.md has a higher max because it grows naturally with new projects/lessons.

> **Customizing budgets:** These are sensible defaults for a 200k-token context window.
> If your workspace uses different files or you have a larger/smaller context window,
> adjust the targets proportionally. The principle: injected files should use <3% of
> your total context window.

## Non-Injected Files (read on demand)

These are NOT auto-injected. They're accessed via `read` or `memory_search`.
They MUST be referenced from at least one injected file so the agent knows when to consult them.

Common examples:

| File | Purpose | Should be referenced from |
|---|---|---|
| memory/*.md | Daily logs, standing instructions | MEMORY.md or AGENTS.md |
| skills/*/SKILL.md | Skill docs | System prompt (automatic) |

> **Your workspace may have additional non-injected files** (setup guides, goal docs,
> reference material). Add them to this table if you want the audit to track them.

## Protected Sections (optional)

Some sections exist for identity, tone, or onboarding reasons and should NOT be
removed or compressed, even if they seem redundant or "obvious."

To protect specific sections from compaction, add a comment block at the top of
any workspace file:

```markdown
<!-- spring-clean: protect
- "Section heading or unique phrase"
- "Another section to preserve"
-->
```

The audit will skip these sections during compaction proposals.

**Default protection (always applied):**
- **SOUL.md in its entirety** — persona files are never trimmed for token savings.

## Audit Checklist

Run all checks against ALL workspace files (injected AND non-injected). In order:

### 1. Size Check
For each file, measure bytes and estimate tokens (~4 chars/token).

**Injected files:** compare against target and hard max in the budget table above.
**Non-injected files:** no hard token limit, but shorter is always better. Flag anything
that seems bloated relative to its purpose.

### 2. Cross-File Conflict Check
Same fact stated differently in two files. Check ALL file pairs, not just injected ones.
Common conflict zones:
- Config values (settings, ports, paths) across files
- Behavioral rules appearing in multiple files with different wording
- Personal info duplicated with different details
- Tool instructions split across files with contradictions

### 3. Redundancy Check
Content appearing in multiple files. Check ALL files. Rules:
- Each fact has ONE canonical location. Cross-references are fine; duplicated content is not.
- Decide canonical locations based on the file's purpose (see the budget table).

### 4. Misplacement Check
Content in the wrong file based on each file's stated purpose.

### 5. Staleness Check
ALL files. Look for: completed tasks listed as active, old version numbers, resolved
issues, expired reminders, done TODO items, outdated status markers.

### 6. Reference Check (non-injected files)
For each non-injected file:
- Verify it's referenced from at least one injected file
- If unreferenced: flag as "orphaned — agent won't know to read this"

### 7. Rule Quality Check
For each behavioral rule found in any file:
- **Default behavior?** Model already does this without being told → candidate for removal
- **Vague?** Would be interpreted differently each time → rewrite or remove
- **One-off fix?** Reads like it was added after one bad output → consider removing
- **Verbose?** Can it be said in fewer words → rewrite

## Output Format

Produce a structured report:

```
🧹 Spring Clean Report — YYYY-MM-DD

📏 SIZE CHECK
  AGENTS.md:    XXX tok (target 700, max 1000) ✅/⚠️/🔴
  MEMORY.md:    XXX tok (target 600, max 1500) ✅/⚠️/🔴
  ...
  Total:        XXX tok (target 3000, max 5000)

⚔️ CONFLICTS (N found)
  - [file1 line X] vs [file2 line Y]: description

🔁 REDUNDANCY (N found)
  - "fact X" in file1 AND file2. Canonical: file1. Remove from: file2.

📦 MISPLACED (N found)
  - "content X" in file1 → should be in file2

🕰️ STALE (N found)
  - "item X" in file1: reason it's stale

🔗 ORPHANED FILES (N found)
  - file1: not referenced from any injected file

✂️ PROPOSED CHANGES
  ━━ file1 ━━
  - line X: [old] → [new]
  + line Y: [addition]

  ━━ file2 ━━
  ...

  ━━ No changes: file3, file4
```

## Execution Rules

1. **Never write without approval.** Always present the report and wait.
2. **Show diffs**, not just descriptions. The human should see exactly what changes.
3. If no issues found, say so briefly. Don't invent problems.
4. Track audit history in `memory/spring-clean-state.json`:
   ```json
   {
     "lastAudit": "YYYY-MM-DD",
     "totalTokens": 0,
     "issuesFound": 0,
     "issuesFixed": 0
   }
   ```
