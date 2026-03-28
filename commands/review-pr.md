---
description: Review a pull request with structured multi-pass analysis covering correctness, security, performance, and test coverage. Usage: /ai-powerhub:review-pr [PR number or URL]
---

# Review PR

Invoke the PR reviewer agent to conduct a thorough review.

```
Agent("ai-powerhub:pr-reviewer")
```

The PR to review is: `$ARGUMENTS`

- If `$ARGUMENTS` is a PR number (e.g. `123`), fetch it via `gh pr diff 123`
- If `$ARGUMENTS` is a GitHub URL, extract the PR number and fetch it
- If `$ARGUMENTS` is empty, review the current branch's changes against `main`
