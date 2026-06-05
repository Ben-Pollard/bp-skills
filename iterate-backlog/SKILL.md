---
name: iterate-backlog
description: Use when working through a backlog of implementation issues from a local markdown tracker, implementing them one at a time via subagent-driven TDD with full review cycle. Requires implementer and reviewer subagent profiles in opencode.json.
---

# Iterate Backlog

## Overview

Thin dispatcher. Reads one issue from a local markdown tracker, dispatches `implementer` and `reviewer` subagents, then marks the issue done. One issue per invocation. Does no implementation or review work inline when subagents are available.

## When to Use

- Working through issues in `.scratch/<feature>/issues/`
- Road-testing implementation and review skills in isolation (each phase runs in its own subagent)
- Before a full orchestrator is built — manual equivalent

NOT for: multiple issues, GitHub Issues, planning/breakdown work.

## Workflow

### 1. Discover and select issue

Scan `<dir>/issues/` for files with `Status: ready-for-agent`. Pick lowest `NN`. If none, stop. If multiple feature dirs exist, ask which one. Skip issues with non-terminal blockers.

Read the issue: acceptance criteria, parent PRD/requirements reference, blocked-by.

### 2. Mark in-progress

Change `Status:` to `in-progress`.

### 3. Dispatch implementer subagent

Extract `<feature>` from the issue's path (the directory under `.scratch/`). Derive outcome paths:

```
OUTCOMES_DIR = .scratch/<feature>/outcomes/
IMPLEMENT_OUTCOME = .scratch/<feature>/outcomes/implement-outcome.json
REVIEW_OUTCOME = .scratch/<feature>/outcomes/review-outcome.json
```

Dispatch `implementer` subagent via Task tool with:
- Full issue body (acceptance criteria, what to build)
- Instruction: load the `tdd` skill, implement issue
- Dispatch context:
  - `outcome_path`: `IMPLEMENT_OUTCOME`
  - `commit`: `true`

### 4. Dispatch reviewer subagent

After implementer exits, read `IMPLEMENT_OUTCOME` for the `commit_sha`. Use `HEAD~1` as base since the implementer prepared a single commit.

Dispatch `reviewer` subagent via Task tool with:
- Issue body (acceptance criteria)
- Instruction: load `requesting-code-review` skill, inspect the diff, produce verdict
- Dispatch context:
  - `outcome_path`: `REVIEW_OUTCOME`
  - `base_sha`: `HEAD~1`
  - `head_sha`: from `implement-outcome.json` `commit_sha`

### 5. Handle review verdict

Read `REVIEW_OUTCOME`. Parse the `action` field: `approved`, `changes-requested`, or `escalate`.

**Changes requested:** Dispatch `implementer` subagent with review feedback and `receiving-code-review` skill. Provide `outcome_path` so it overwrites `IMPLEMENT_OUTCOME`. After fix subagent exits, go to step 4 (re-review).

**Approved:** Proceed to step 6.

### 6. Verify

Load `verification-before-completion` skill. Run fresh test pass. Do NOT reuse results from implementation or fix phases — stale.

### 7. Mark done

Change `Status:` to `done`. Append `## Outcome` section with one-line summary. Reference `IMPLEMENT_OUTCOME` and `REVIEW_OUTCOME` paths. Stop.

### Subagent Failure

If any dispatch returns empty output, truncated output (ends mid-sentence, no closing JSON), or a tool-error result:
- Retry once with identical instructions
- If the retry also fails: change `Status: ready-for-human`, append `## Outcome` with the failure details, **stop execution**

### Escalation

If any subagent hits serious doubt, review loop exceeds 3 rounds, or subagent retry fails:
- Change `Status: ready-for-human`
- Append `## Outcome` with explanation
- **Stop execution.** Do not continue.

### Outcome Artefacts

```
.scratch/<feature>/outcomes/
├── implement-outcome.json    # from implementer (overwritten on fix cycles)
└── review-outcome.json       # from reviewer (overwritten on re-review)
```

Implement schema: `{status, summary, test_results, commit_sha, concerns}`
Review schema: `{spec_compliance, principles_aligned, violations, review_notes, action}`

## Local Tracker Conventions

```
Status: ready-for-agent  →  in-progress  →  done
```

Issue files: `.scratch/<feature>/issues/NN-slug.md`. Status line near top. Append outcomes under `## Outcome` or `## Comments`.

Blocked issues: skip if `Blocked by` references a non-terminal issue.

## Common Mistakes

- Doing implementation inline — always dispatch via Task tool when subagents are configured.
- Reusing test results from implementation phase — verification must be fresh.
- Proceeding to next issue — one per invocation.
- Review loop exceeding 3 rounds — escalate.
- Duplicating other skills' workflows — this skill orchestrates handoffs, nothing more.