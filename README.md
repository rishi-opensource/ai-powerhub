# AI Powerhub

Universal developer productivity plugin for AI coding agents — works with Claude Code, Cursor, and other AI IDEs.

## What It Does

- **Systematic debugging** — evidence-based root cause analysis, no guessing
- **PR review** — structured multi-pass review with severity-rated findings
- **Bug investigation agent** — multi-step workflow from error to confirmed fix
- **PR reviewer agent** — fetches diffs, reviews, optionally posts to GitHub

## Install

### Claude Code
```
/plugin marketplace add ./ai-powerhub
/plugin install ai-powerhub@ai-powerhub-dev
```

### Cursor
Add the plugin directory to your Cursor plugins or install from the Cursor marketplace.

### Local development
```bash
/plugin marketplace add /path/to/ai-powerhub
```

## Commands

| Command | What it does |
|---|---|
| `/ai-powerhub:review-pr [PR#]` | Review a pull request. Pass a PR number, URL, or leave empty to review the current branch |
| `/ai-powerhub:debug-issue [description]` | Investigate a bug. Paste an error message or describe the problem |

## Skills

Skills are triggered automatically when you describe a task. You can also invoke them explicitly.

| Skill | Triggers on |
|---|---|
| `systematic-debugging` | "debug", "why is X failing", "help me investigate", stack traces |
| `pr-review` | "review these changes", "review my diff", "code review" |

## Agents

| Agent | What it does |
|---|---|
| `pr-reviewer` | Fetches the PR diff via `gh`, reads changed files in full context, runs the pr-review skill, produces a severity-rated report, optionally posts to GitHub |
| `bug-investigator` | Gathers context, searches the codebase, runs the systematic-debugging skill, produces a full investigation report with root cause and fix |

## Requirements

- `gh` CLI — for PR review features (`brew install gh` then `gh auth login`)
- `git` — for debugging history features (checking recent changes near failures)

## Structure

```
ai-powerhub/
  .claude-plugin/
    plugin.json          # Claude Code manifest
    marketplace.json     # Dev marketplace for local install
  .cursor-plugin/
    plugin.json          # Cursor manifest (points at same skills/agents/commands)
  skills/
    systematic-debugging/SKILL.md
    pr-review/SKILL.md
  agents/
    pr-reviewer.md
    bug-investigator.md
  commands/
    review-pr.md
    debug-issue.md
  hooks/
    hooks.json           # Claude SessionStart hook
    hooks-cursor.json    # Cursor SessionStart hook
    session-start        # Shared shell script (checks gh + git on startup)
  package.json
```

## How Components Relate

```
/review-pr command
  └── invokes pr-reviewer agent
        ├── fetches diff via gh CLI
        ├── reads changed files
        └── runs pr-review skill → produces report

/debug-issue command
  └── invokes bug-investigator agent
        ├── searches codebase for error
        ├── checks git history
        └── runs systematic-debugging skill → produces investigation report

SessionStart hook
  └── checks gh + git availability
      └── injects warnings if missing (silent otherwise)
```

Skills can also be triggered independently by natural language — you don't need a command or agent to use them.
