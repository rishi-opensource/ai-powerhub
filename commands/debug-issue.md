---
description: Investigate a bug, error, or unexpected behaviour using systematic root cause analysis. Usage: /ai-powerhub:debug-issue [error message or description]
---

# Debug Issue

Invoke the bug investigator agent to systematically find the root cause.

```
Agent("ai-powerhub:bug-investigator")
```

The issue to investigate is: `$ARGUMENTS`

- If `$ARGUMENTS` contains an error message or stack trace, begin investigation immediately
- If `$ARGUMENTS` is empty, ask the user to describe the bug, paste the error, or share a stack trace
