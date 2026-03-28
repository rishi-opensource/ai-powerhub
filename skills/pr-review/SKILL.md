---
name: pr-review
description: Use when reviewing a pull request, diff, or set of code changes. Trigger on "review this PR", "review these changes", "review my diff", "code review", "look at this PR". Produces a structured review with severity-rated findings across correctness, security, performance, and test coverage.
---

# PR Review

## Severity Levels

- 🔴 **Blocker** — Must fix before merge. Correctness bugs, security vulnerabilities, data loss risks, broken contracts.
- 🟡 **Needs Work** — Should fix. Performance issues, error handling gaps, missing test coverage, unclear ownership.
- 🔵 **Suggestion** — Nice to have. Style improvements, naming clarity, minor simplifications.
- ✅ **Praise** — Call out genuinely good patterns worth reinforcing and reusing.

---

## Step 0: Understand Intent

Before reviewing any line of code:

1. Read the PR title and description
2. Identify the stated goal and acceptance criteria
3. Check the linked ticket or issue if present

Ask: **"Does the overall approach make sense for the stated goal?"**

If the approach is wrong at a high level, that's a 🔴 Blocker — raise it before doing line-level review. Reviewing the implementation of a bad approach wastes everyone's time.

---

## Step 1: Correctness

- Does the logic handle all cases, including edge cases and error paths?
- Are there off-by-one errors, nil/null handling gaps, or incorrect conditionals?
- Does the implementation match the stated acceptance criteria?
- Are there race conditions or incorrect concurrency patterns?
- Are errors handled, propagated, and surfaced correctly — not swallowed?
- Are external inputs (user data, API responses) validated before use?

---

## Step 2: Security

- Is user input validated and sanitized before use?
- Are there injection vectors — SQL, command injection, XSS?
- Are secrets, credentials, or PII ever logged or exposed in responses?
- Are authentication and authorization checks present, correct, and complete?
- Are new dependencies introduced? If so — known CVEs? Supply chain risk?
- Are cryptographic operations using standard library functions, not custom implementations?

---

## Step 3: Performance

- Are there N+1 query patterns? (loop containing a DB/API call)
- Are expensive operations — sorting, serialisation, network calls — inside hot loops?
- Is caching used correctly? (invalidation strategy, key collision risk)
- Are database queries using appropriate indexes?
- Will this scale to 10x the expected load without architectural changes?

---

## Step 4: Test Coverage

- Are tests present for the new behaviour?
- Do tests cover failure paths and edge cases, not just the happy path?
- Are tests testing behaviour (what the code does) rather than implementation (how it does it)?
- Would a future refactor silently break these tests without catching a real regression?
- Are any critical paths completely untested?

---

## Step 5: Code Quality

- Is the intent clear from the code without requiring comments to explain it?
- Is there unnecessary complexity, over-abstraction, or premature generalisation?
- Are function and variable names accurate, specific, and unambiguous?
- Is there duplicated logic that should be extracted into a shared function?
- Are there dead code paths, unused imports, or commented-out blocks?

---

## Step 6: Produce the Review

```
## PR Review: <title>

### Summary
<1–2 sentence overall verdict — approve / request changes / needs discussion>

### Findings

🔴 Blocker: <short title>
   File: `path/to/file.go:42`
   Issue: <what is wrong and why it matters>
   Fix: <specific, actionable suggestion>

🟡 Needs Work: <short title>
   File: `path/to/file.go:87`
   Issue: <what is wrong>
   Suggestion: <what to do instead>

🔵 Suggestion: <short title>
   File: `path/to/file.go:12`
   Note: <brief suggestion>

✅ Good: <what was done well and why it's worth noting>

### Verdict
[ ] Approve
[x] Request Changes

### Top 3 Priorities
1. <most important finding>
2. <second most important>
3. <third most important>
```

**Rules for every finding:**
- Always include the file path and line number
- For Blockers, always include a concrete fix, not just the problem
- Suggestions should be brief — one sentence
- Praise at least one thing if it genuinely exists
