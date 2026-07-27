---
name: verification-before-completion
description: Use when about to claim work is complete, before committing or creating PRs. Also use to verify system behavior against a PRD's Behavioral Acceptance Criteria after all issues for that PRD are done. Requires running verification commands and confirming output before making any success claims; evidence before assertions always.
---

# Verification Before Completion

## Overview

Claiming work is complete without verification is dishonesty, not efficiency.

**Core principle:** Evidence before claims, always.

**Violating the letter of this rule is violating the spirit of this rule.**

## The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

If you haven't run the verification command in this message, you cannot claim it passes.

## The Gate Function

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

## Common Failures

| Claim | Requires | Not Sufficient |
|-------|----------|----------------|
| Tests pass | Full suite output: 0 failures | Previous run, selective re-run, "not my ticket" |
| Linter clean | Linter output: 0 errors | Partial check, extrapolation |
| Build succeeds | Build command: exit 0 | Linter passing, logs look good |
| Bug fixed | Test original symptom: passes | Code changed, assumed fixed |
| Regression test works | Red-green cycle verified | Test passes once |
| Selective verification | Full suite: every test green | Running only your subset, dismissing failures as "not my ticket" |
| Agent completed | VCS diff shows changes | Agent reports "success" |
| Requirements met | Line-by-line checklist | Tests passing |

## Red Flags - STOP

- Using "should", "probably", "seems to"
- Expressing satisfaction before verification ("Great!", "Perfect!", "Done!", etc.)
- About to commit/push/PR without verification
- Trusting agent success reports
- Relying on partial verification
- Thinking "just this once"
- Tired and wanting work over
- **ANY wording implying success without having run verification**

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "Should work now" | RUN the verification |
| "I'm confident" | Confidence ≠ evidence |
| "Just this once" | No exceptions |
| "Linter passed" | Linter ≠ compiler |
| "Agent said success" | Verify independently |
| "I'm tired" | Exhaustion ≠ excuse |
| "Partial check is enough" | Partial proves nothing |
| "Different words so rule doesn't apply" | Spirit over letter |

## Key Patterns

**Tests:**
```
✅ [Run test command] [See: 34/34 pass] "All tests pass"
❌ "Should pass now" / "Looks correct"
```

**Regression tests (TDD Red-Green):**
```
✅ Write → Run (pass) → Revert fix → Run (MUST FAIL) → Restore → Run (pass)
❌ "I've written a regression test" (without red-green verification)
```

**Build:**
```
✅ [Run build] [See: exit 0] "Build passes"
❌ "Linter passed" (linter doesn't check compilation)
```

**Requirements:**
```
✅ Re-read PRD → Read Behavioral Acceptance Criteria → For each AC, determine verification command → Run it → Capture output → Report pass/fail with evidence
❌ "Tests pass, phase complete" (tests ≠ behavioral verification)
```

**PRD-level verification (all issues done):**
```
✅ Load PRD → Read Behavioral Acceptance Criteria section → For each AC:
       1. Determine command/check needed (e.g., `config-check --schema x.yaml` → expect exit 0)
       2. Run against the running system
       3. Capture output and exit status
       4. Report PASS (evidence) or FAIL (actual vs expected)
   Write verification report alongside PRD (e.g., PRD.md.verification.md)
❌ Assume from reading code. ❌ Infer from "it should work." ❌ Trust agent reports.
```

**Agent delegation:**
```
✅ Agent reports success → Check VCS diff → Verify changes → Report actual state
❌ Trust agent report
```

## PRD-Level Behavioral Verification

When all issues for a PRD reach `done`, verify system behavior against the PRD's specification.

### Process

1. **Load the PRD** — find the PRD file (`.scratch/<slug>/PRD.md` or as provided)
2. **Read Behavioral Acceptance Criteria** — the section with verbatim ACs from requirements
3. **For each AC:**
   1. **Determine** the command or check needed to verify it (e.g., `config-check --schema x.yaml` and expect exit 0)
   2. **Run** the verification against the running system
   3. **Capture** full output and exit status
   4. **Report** PASS (with evidence) or FAIL (actual output vs expected)
4. **Write verification report** alongside the PRD as `PRD.md.verification.md`

### Report Format

```
# Verification Report: <PRD Title>

## Summary
- Passed: N
- Failed: N
- Skipped: N

## Per-AC Results

### AC-1: {text}
**Status:** PASS | FAIL
**Command:** `{verification command}`
**Expected:** {expected behavior}
**Actual:** {actual output / exit code}
**Evidence:** {command output, log excerpts, file state}

### AC-2: {text}
...
```

### Edge Cases

| Situation | Handling |
|-----------|----------|
| AC passes | Report PASS with evidence |
| AC fails | Report FAIL with actual output vs expected |
| System cannot start | Report all ACs as FAIL with startup error |
| No behavioral ACs found in PRD | Report warning, skip verification |

### The Gate applies

PRD-level verification is subject to the same Iron Law and Gate Function above. You must run the actual commands and report actual output. Inferring pass/fail is lying.

## Why This Matters

From 24 failure memories:
- your human partner said "I don't believe you" - trust broken
- Undefined functions shipped - would crash
- Missing requirements shipped - incomplete features
- Time wasted on false completion → redirect → rework
- Violates: "Honesty is a core value. If you lie, you'll be replaced."

## When To Apply

**ALWAYS before:**
- ANY variation of success/completion claims
- ANY expression of satisfaction
- ANY positive statement about work state
- Committing, PR creation, task completion
- Moving to next task
- Delegating to agents

**Apply PRD-level verification when:**
- All issues for a PRD reach `done`
- Before marking a PRD as complete
- Before a review or demo of the integrated system

**Rule applies to:**
- Exact phrases
- Paraphrases and synonyms
- Implications of success
- ANY communication suggesting completion/correctness

## The Bottom Line

**No shortcuts for verification.**

Run the command. Read the output. THEN claim the result.

This is non-negotiable.
