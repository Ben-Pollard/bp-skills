---
name: receiving-code-review
description: Use when the implementer subagent receives review, reduction, or qa feedback in the iterate-backlog fix loop — understands the violations, fixes code, runs tests, and reports the outcome.
---

# Receiving Code Review

## Overview

Dispatched by `iterate-backlog` after a reviewer or qa subagent requests changes. One invocation handles all violations from a single review pass. Does not push back — the reviewer is a peer subagent with the same capabilities.

## Workflow

```
1. UNDERSTAND — Read the feedback. Identify what's wrong and why.
2. FIX — One violation at a time. Blocking issues first, then simple, then complex.
3. TEST — Run the test suite. Fix doesn't count until tests pass.
4. REPORT — Write outcome to outcome_path.
```

## Review Types

The `review_type` tag tells you which feedback format you're receiving.

### Standard Review (`review_type: "standard"`)

Source: `requesting-code-review` outcome.

Each violation has `{principle, file, issue}`. The outcome also has `code_quality` and `test_quality` fields with `true|false` per criterion — the failing criteria tell you which TDD skill docs to read.

Before fixing, read the TDD skill docs where criteria are `false`:

| Failing criterion | Doc to read |
|---|---|
| `deep-modules.md` | `.agents/skills/tdd/deep-modules.md` |
| `interface-design.md` | `.agents/skills/tdd/interface-design.md` |
| `refactoring.md` | `.agents/skills/tdd/refactoring.md` |
| `declarative-over-procedural.md` | `.agents/skills/tdd/declarative-over-procedural.md` |
| `mocking.md` | `.agents/skills/tdd/mocking.md` |
| `tests.md` | `.agents/skills/tdd/tests.md` |
| `SOLID`, `DRY`, `KISS`, `YAGNI` | (general principles — no doc needed) |

For each violation, read the referenced file, understand the issue, and fix it.

### Reduction (`review_type: "reduction"`)

Source: `minimizing-code` outcome.

Each violation has `{file_line, lines, replacement, question}`. The `replacement` field tells you exactly what to do. Apply the replacement, remove the eliminated code, run tests.

### QA (`review_type: "qa"`)

Source: `qa` outcome.

Each failure is in `failed_acs`: `{id, text, reason}`. The ticket (issue body) provides the full acceptance criteria. The `reason` field says what was expected vs observed. Trace the code, find the behavioral mismatch, fix it.

These are behavioral failures — the system doesn't do what the AC says. Code quality or reduction changes won't fix them. Fix the behavior, then re-test.

**Green tests + broken live system = insufficient test coverage.** When the QA reports FAIL with every `stage_result` passing (lint, unit tests, integration tests all green) but the live system is visibly broken, your job is not just to fix the code — it is to write the test that would have caught the gap. A QA failure that slipped past all existing tests means the test suite is missing a behavioural wire. Add an integration or E2E test that exercises the exact path the QA found broken before you fix the code. Then fix the code. Then run both the new test and the full suite.

## Output

Write outcome JSON to `outcome_path`. Create parent directories if needed.

```json
{
  "status": "DONE" | "BLOCKED",
  "summary": "<one-line description of what was fixed, or reason blocked>"
}
```

- `DONE` — all violations fixed, tests pass
- `BLOCKED` — cannot proceed (can't find file, fix breaks tests and can't resolve, can't find satisfactory solution, permissions issues, etc.)

## Critical Rules

**DO:**
- Fix one violation at a time, test each
- Fix ordering: blocking issues first (breaks, security), then simple fixes, then complex refactoring
- Run the full test suite after all fixes
- Report BLOCKED if the fix is impossible

**DON'T:**
- Push back on reviewer feedback — reviewer is a peer subagent
- Skip tests after fixing
- Implement unrelated changes
