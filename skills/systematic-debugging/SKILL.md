---
name: systematic-debugging
description: >-
  Use when investigating a bug, unexpected behavior, error, or test failure. Trigger on "debug", "investigate this error", "why is X failing", "figure out why", "help me debug", or when given a stack trace. Enforces evidence-based root cause analysis — never guessing.
---

# Systematic Debugging

## Core Principle

Never guess. Every hypothesis must be tested with evidence before implementing a fix.
Debugging by intuition wastes more time than it saves.

## Step 1: Understand the Problem Completely

Before touching any code:

**Reproduce it** — can you make it happen reliably?
- If not, establish the conditions under which it occurs
- Intermittent bugs require a logging strategy before debugging

**Capture full context:**
- Exact error message and stack trace
- Environment (OS, runtime version, dependency versions)
- Last known working state and what changed since then

**Define the failure in one sentence:**
> "When [X happens], I expect [Y] but instead get [Z]."

If you can't write this sentence clearly, stop and ask the user for clarification before proceeding.

---

## Step 2: Locate the Blast Radius

1. **Find the failure boundary** — at what layer does correct become incorrect?
   - User input → validation → business logic → persistence → external service
   - Identify the earliest point of failure

2. **Search for recent changes near the failure:**
   ```bash
   git log --oneline -20 -- <affected-file>
   git diff HEAD~5 -- <affected-file>
   ```

3. **Search the codebase for the exact error string:**
   ```bash
   grep -r "exact error message" --include="*.go" -l
   ```

4. **Check for existing tests covering this code path** — if they exist and pass, the bug is likely environmental or in untested branches.

---

## Step 3: Form Hypotheses

List 2–3 candidate causes ranked by likelihood. For each:
- What would make this hypothesis true?
- What single test would confirm or definitively rule it out?

**Do not implement fixes yet.** A fix before confirmed root cause is a guess.

---

## Step 4: Test Hypotheses Systematically

For each hypothesis (in likelihood order):

1. Write a **minimal reproduction** — a unit test or short script that isolates the behavior
2. Run it — observe the actual outcome
3. Mark the hypothesis as confirmed or ruled out

If confirmed → move to Step 5.
If all ruled out → return to Step 3 with new hypotheses informed by what you learned.

**Do not skip ahead.** Testing hypotheses out of order wastes time.

---

## Step 5: Fix and Verify

1. Implement the **minimal fix** — change only what is necessary to correct the root cause
2. Run the reproduction test from Step 4 — it must now pass
3. Run the full test suite — no regressions allowed
4. If tests don't exist for this path, write one now (see Step 6)

---

## Step 6: Regression Prevention

Write a test that:
- Would have caught this bug before the fix
- Is named to describe the scenario, not the code path

```
# Good: describes the failure scenario
test "returns error when payment service is unavailable, not a 500"

# Bad: describes the code path
test "PaymentService.call handles nil response"
```

---

## Step 7: Document the Investigation

Produce a brief summary:

```
Bug:            <one-line description>
Root Cause:     <what actually caused it — be precise>
Fix:            <what was changed and why it fixes the root cause>
Regression Test: <test name / file>
Prevented By:   <what would have caught this earlier — missing test, lint rule, type check>
```
