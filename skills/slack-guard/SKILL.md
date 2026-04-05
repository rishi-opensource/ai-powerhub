---
name: slack-guard
description: >
  Mandatory confirmation gate for all Slack-related operations. Triggers whenever
  Claude is about to perform any Slack action — including but not limited to:
  sending messages, reading channels or threads, searching, scheduling messages,
  creating or updating canvases, reading user profiles, or any other interaction
  with Slack via MCP tools (mcp__plugin_slack_slack__*), CLI commands, or direct
  API calls. Before executing any Slack operation, Claude must display a clear
  summary of the intended action and wait for explicit user confirmation.
---

# Slack Guard Protocol

Before executing **any** Slack operation, follow this protocol without exception.

## Step 1 — Display Intent

Show the user exactly what you are about to do. Include all of:

| Field | What to show |
|-------|-------------|
| **Action** | e.g. Send message / Read channel / Search / Schedule message |
| **Target** | Full name AND handle — e.g. `Sathvika Reddy (@sathvika.reddy)` for users or `#general` for channels. **Always resolve the target first** using `slack_search_users` or `slack_search_channels` before displaying this table, so the user sees both the display name and the handle before confirming. |
| **Details** | Message content (full text), search query, canvas content, etc. |

Keep the display clear and readable. Use a code block for message content if it is long.

## Step 2 — Ask for Confirmation

End with a single confirmation question:

> "I am about to [action] in [target]. Can I go ahead?"

Do **not** proceed until the user responds.

## Step 3 — Handle Feedback

- If the user says **yes** (or equivalent): execute the action.
- If the user provides **feedback or changes**: incorporate the changes, re-display the updated intent, and ask for confirmation again.
- If the user says **no** or cancels: stop and do not execute.

Repeat Steps 1–3 until the user explicitly approves.

## Scope

This protocol applies to **all** Slack interactions:

- MCP tools: all `mcp__plugin_slack_slack__*` tools
- CLI: any `slack` CLI commands
- API: any direct Slack API calls (webhooks, REST)

No exceptions for "read-only" operations — confirmation is always required.
