---
allowed-tools: Read, Glob, Grep, Bash(ls:*), Bash(find:*), Bash(cp:*), Bash(mkdir:*), Bash(date:*), Write, Edit
description: Comprehensive review of Claude Code architecture with security audit, best-practice analysis, health score, and guided interactive redesign.
---

# Review Claude Setup

You are acting as an expert Claude Code developer and system architect. Perform a comprehensive review of the user's Claude Code setup and guide them through a safe, interactive redesign.

---

## Phase 1 — Discovery

Read all files in parallel. Use Glob and Read tools directly.

### Files to read

**Core config:**
- `~/.claude/CLAUDE.md`
- `~/.claude/settings.json`
- `~/.claude/settings.local.json`
- `~/.mcp.json`

**Rules:** Glob `~/.claude/rules/**/*.md` — read every file found

**Agents:** Glob `~/.claude/agents/**/*.md` — read every file found

**Commands:** Glob `~/.claude/commands/**/*.md` — read every file found

**Hooks:** `~/.claude/hooks/` — list contents if directory exists

**Plugins:** Glob `~/.claude/plugins/**/*.json` — read `marketplace.json` and `plugin.json` files

**Stale forks:** Check `~/Desktop/CLAUDE.md` and `~/CLAUDE.md`

**Shell profile:** Read `~/.zshrc` and `~/.bashrc` (if they exist) — extract all `export` lines containing: `TOKEN`, `SECRET`, `KEY`, `PASSWORD`, `API`, `CLAUDE`, `ANTHROPIC`, `ATLASSIAN`. Redact values, keep var names only.

**Memory files:** Glob `~/.claude/projects/*/memory/*.md` — note file paths and sizes. Read contents briefly to check for sensitive data (account IDs, tokens, internal URLs).

---

## Phase 2 — Analysis

Analyse every finding with a specific file path and line number. Be critical and precise.

### 2.1 CLAUDE.md Quality

- **Size bloat**: Over 200 lines → must be split into rule modules via `@`-imports
- **Monolith pattern**: Inline rules that should live in `rules/*.md`
- **Stale forks**: Multiple copies of CLAUDE.md on Desktop, home dir, etc.
- **@-import syntax**: Must be `@rules/filename.md` — not `@~/.claude/rules/filename.md`
- **Broken @-imports** ← NEW: For every `@rules/filename.md` found in CLAUDE.md, verify the file actually exists at `~/.claude/rules/filename.md`. A missing file silently loads nothing — no error is shown.

### 2.2 Security Audit — HIGH PRIORITY

- **Hardcoded secrets**: Scan all config files for `Bearer`, `token`, `api_key`, `password`, `secret`, or long alphanumeric strings that look like tokens
- **Credential-scanning commands**: `Bash(find ...)`, `Bash(cat ...)`, `Bash(grep ...)` in allow-list that scan for OAuth tokens or secrets
- **Shell loop artifacts**: `Bash(while read:*)`, `Bash(do echo:*)`, `Bash(done)` in allow-list
- **Overly broad Read permissions**: `Read(//**)` or `Read(~/*)`
- **`enableAllProjectMcpServers: true`**: Auto-activates all project MCP servers — must be `false`
- **Hardcoded tokens in `.mcp.json`**: Must use `${ENV_VAR}` references
- **`Bash(env)` in allow-list**: Dumps all env vars including secrets — must be removed
- **Session artifacts in allow-list**: `Skill(...)` entries auto-added during sessions that were never deliberately approved
- **ENV_VAR cross-check** ← NEW: For every `${VAR_NAME}` reference in `.mcp.json`, verify `VAR_NAME` is actually exported in `~/.zshrc` or `~/.bashrc`. A missing export means the MCP server will fail silently at runtime with no clear error.

### 2.3 Rules Architecture

- **No rules directory**: All rules embedded in CLAUDE.md is an anti-pattern
- **Conflicting rules**: E.g. different commit formats across files
- **Missing coverage**: Are there rules for AWS, Git, and any CLI tools in use?
- **Clear activation triggers**: Do rules define when they apply?

### 2.4 Agents / Subagents

- **Missing frontmatter**: Agent files must have `name:`, `description:`, `tools:`, `model:`
- **Unrestricted `Bash`**: A bare `Bash` in `tools:` is over-privileged — scope it to `Bash(aws:*)` etc.
- **Persona and methodology clarity**: Does the agent have a clear workflow?
- **Agent tools vs allow-list alignment** ← NEW: For every agent, extract all tools listed in its `tools:` frontmatter. Cross-reference each tool against `settings.local.json` allow-list. Flag any tool that is NOT pre-approved — it will prompt the user on every invocation, breaking workflows that depend on parallel tool calls.

### 2.5 Commands

- **Duplicate logic**: Command re-implementing rules already in a rule file
- **Missing `allowed-tools` frontmatter**: Every command must restrict its tool access
- **Hardcoded absolute paths**: `/Users/username/...` embedded in commands
- **Version-pinned binary paths**: Fragile paths like `~/.devbox/ai/claude/bun x --bun ccusage@17.1.6`

### 2.6 MCP Configuration

- **Hardcoded credentials in `.mcp.json`**: Must use `${ENV_VAR}` — never literal values
- **ENV_VAR exported in shell**: Covered in 2.2 cross-check
- **Unused MCP servers**: Configured but not referenced anywhere

### 2.7 Permissions Allow-list

- **Principle of least privilege**: Narrow scoping (`Bash(aws lambda list-functions:*)`) vs broad (`Bash(aws:*)`)
- **Orphaned entries** ← NEW: For every entry in the allow-list, check whether it is referenced in any rule file, command, agent, or hook. Entries for tools that no longer exist anywhere are dead weight and should be removed.
- **Missing useful permissions**: Agent tools not pre-approved (covered in 2.4)

### 2.8 Memory Files ← NEW

- Do memory files exist at `~/.claude/projects/*/memory/`?
- Do any contain sensitive data: account IDs, API tokens, internal hostnames, passwords?
- Note: memory files persist across all sessions and accumulate over time. Sensitive context written during an investigation can linger indefinitely.

### 2.9 Project CLAUDE.md Files ← NEW (Optional)

After the main report, ask the user:
> "Would you also like me to audit project-level CLAUDE.md files in a directory (e.g. `~/work/`)? I'll check each for duplication with global rules, conflicts, and best practice violations."

If yes, ask for the directory path, then:
- Glob `<dir>/*/CLAUDE.md`
- For each file found, check:
  - Duplicates rules already covered globally (commit format, AWS governance, IaC enforcement)
  - Conflicting rules vs global (especially commit formats, branch names)
  - Re-states AWS account IDs — are they the SAME as global or DIFFERENT? Different = legitimate project-specific info; same = unnecessary duplication
  - Uses `@`-imports vs inline rules
  - Has project-specific content that justifies its existence (commands, architecture, data models)

---

## Phase 3 — Generate Report

### Health Score Calculation

Start at 100. Apply deductions:

| Issue | Deduction |
|---|---|
| Hardcoded credential in any config file | −20 |
| `${ENV_VAR}` not exported in shell profile | −15 |
| `enableAllProjectMcpServers: true` | −15 |
| `Bash(env)` in allow-list | −10 |
| Stale CLAUDE.md fork (Desktop/home) | −10 |
| CLAUDE.md over 200 lines (monolith) | −10 |
| No `rules/` directory | −10 |
| Broken `@-import` (referenced file missing) | −10 |
| Credential-scanning command in allow-list | −10 |
| Session artifact in allow-list | −5 per entry |
| Bare `Bash` in agent tools | −5 |
| Agent tool not pre-approved in allow-list | −5 per tool |
| Command missing `allowed-tools` frontmatter | −5 |
| Orphaned allow-list entry | −3 per entry |
| Memory file with sensitive content | −5 |

Cap score at 0. Classify:
- 90–100: **EXCELLENT**
- 75–89: **GOOD**
- 50–74: **NEEDS IMPROVEMENT**
- Below 50: **CRITICAL ISSUES**

### Output this exact report structure

```
## Claude Setup Review Report

**Date:** [today]
**Health Score:** [N/100] — [EXCELLENT / GOOD / NEEDS IMPROVEMENT / CRITICAL ISSUES]

### Score Breakdown
  Security:      [n pts deducted] — [issues count]
  Architecture:  [n pts deducted] — [issues count]
  Permissions:   [n pts deducted] — [issues count]

---

### Security Issues ⚠️
[Each issue: File + line, exact problem, why dangerous, recommended fix]
If none: "No security issues found."

---

### Architecture Issues
[Grouped by: CLAUDE.md | Rules | Agents | Commands | MCP | Permissions | Memory]
For each:
- **File:** path + line
- **Problem:** what's wrong
- **Impact:** why it matters
- **Fix:** exact change needed

---

### What's Working Well
[Specific, accurate positives]

---

### Recommended File Structure
[Show the ideal structure tailored to THIS user's actual setup]

---

### Issues Summary
[Numbered list of all issues found — used for interactive fix mode]
1. [issue title] — [file] — SECURITY / ARCHITECTURE / PERMISSIONS
2. ...
```

---

## Phase 4 — Interactive Options

After printing the report, ask:

> "Found [N] issues ([S] security, [A] architecture, [P] permissions).
>
> How would you like to proceed?
> 1. **Fix one by one** (recommended) — I'll show each change before applying it
> 2. **Fix all at once** — apply everything in one pass
> 3. **Security fixes only** — only fix the security issues now
> 4. **Just the report** — no changes
>
> Also: would you like me to **take a backup** before making any changes?
> I'll copy all affected files to `~/claude-backup-[timestamp]/`."

Wait for explicit user response before proceeding.

---

## Phase 5 — Backup (if requested)

Create: `~/claude-backup-$(date +%Y%m%d-%H%M%S)/`

Copy into it:
- `~/.claude/CLAUDE.md`
- `~/.claude/settings.json`
- `~/.claude/settings.local.json`
- `~/.mcp.json`
- All files from `~/.claude/rules/`
- All files from `~/.claude/agents/`
- All files from `~/.claude/commands/`

Confirm: "Backup created at `~/claude-backup-[timestamp]/`"

---

## Phase 6 — Apply Fixes

### If "one by one" mode

For each issue in the numbered Issues Summary:
1. Print: `Fixing [N/total]: [issue title]`
2. Show exactly what will change (old → new)
3. Ask: `Apply this fix? (yes / no / skip all remaining)`
4. If yes: apply using Edit or Write tool, confirm applied
5. Move to next issue

### If "all at once" mode

Work through all issues in this order: security first, then architecture, then permissions.
For each change: state what you are doing and why, apply it, confirm.

### If "security only" mode

Apply only issues classified as SECURITY in the Issues Summary. Skip the rest.

---

## Implementation Rules (apply to all modes)

- `settings.local.json` → `enableAllProjectMcpServers: false`
- `.mcp.json` → `${ENV_VAR}` for all credentials, never literals
- CLAUDE.md → ≤60 lines, `@`-imports for all rule modules
- Each rule module → ≤100 lines, single concern
- Commands → must have `allowed-tools` frontmatter
- Agents → must have `name`, `description`, `tools`, `model` frontmatter
- Never delete user content without confirmation
- If a fix is ambiguous, ask before applying

After all changes: print a final summary table of every file modified and what changed.
