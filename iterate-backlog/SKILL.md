---
name: iterate-backlog
description: Use when working through a backlog of implementation issues from a local markdown tracker, implementing them one at a time via subagent-driven TDD with full review cycle.
---

# Iterate Backlog

## Overview

Thin dispatcher. Reads one issue from a local markdown tracker, dispatches subagents for implementation, review, and fixes, then marks the issue done. One issue per invocation. Does no implementation or review work inline — all work happens in subagents to keep the main context clean.

## When to Use

- Working through issues in `.scratch/<feature>/issues/`
- Road-testing implementation and review skills in isolation (each phase runs in its own subagent)
- Before a full orchestrator is built — manual equivalent

NOT for: multiple issues, GitHub Issues, planning/breakdown work.

## Workflow

### 1. Discover and select issue

Scan `<dir>/issues/` for files with `Status: ready-for-agent`. Pick lowest `NN`. If none, stop. If target dir ambiguous, ask. Skip issues with non-terminal blockers.

Read the issue: acceptance criteria, parent PRD/requirements reference, blocked-by.

### 2. Mark in-progress

Change `Status:` to `in-progress`.

### 3. Dispatch implementation subagent

Launch a subagent (general type) with:
- The full issue body (acceptance criteria, what to build)
- Instruction to load the `tdd` skill
- Target: write code that satisfies every acceptance criterion
- Commit as they go (pre-commit hooks enforce lint + unit tests)
- No review step — that's the next phase

After subagent exits, verify: commits were made, `uv run ruff check` passes, `uv run pytest tests/unit` passes. If subagent failed or no changes, escalate.

### 4. Dispatch reviewer subagent

Load `requesting-code-review` skill. Dispatch reviewer subagent with:
- The issue body
- The diff from the implementation subagent's work
- No implementation context leaked

### 5. Handle review verdict

Reviewer returns one of: approved, changes-requested, or escalates.

**Changes requested:** Dispatch a fix subagent with the review feedback and `receiving-code-review` skill. After fix subagent exits, go to step 4 (re-review loop).

**Approved:** Proceed to step 6.

### 6. Verify

Load `verification-before-completion` skill. Run a fresh test pass. Do NOT use test results from implementation or fix subagents — those are stale.

### 7. Mark done

Change `Status:` to `done`. Append `## Outcome` section to the issue with a one-line summary. Stop.

### Escalation

If any subagent hits serious doubt (ambiguous ACs, unfixable failure, conflicting constraints) or if the review loop exceeds 3 rounds:

- Change `Status: ready-for-human`
- Append `## Outcome` with explanation
- **Stop execution.** Do not continue to next issue.

## Local Tracker Conventions

```
Status: ready-for-agent  →  in-progress  →  done
```

Issue files: `.scratch/<feature>/issues/NN-slug.md`. Status line near top. Append outcomes and discussion under `## Outcome` or `## Comments`.

Blocked issues: skip if `Blocked by` references a non-terminal issue.

## Common Mistakes

- Doing implementation inline instead of dispatching a subagent — pollutes main context.
- Reusing test results from implementation phase — verification must be a fresh run.
- Proceeding to next issue — one issue per invocation.
- Letting the review loop exceed 3 rounds — escalate.
- Duplicating other skills' workflows — this skill orchestrates handoffs, nothing more.