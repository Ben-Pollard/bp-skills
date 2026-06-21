---
name: iterate-backlog
description: Use when working through a backlog of implementation issues from a local markdown tracker, implementing them one at a time via subagent-driven TDD with full review cycle. Requires implementer and reviewer subagent profiles in opencode.json.
---

# Iterate Backlog

## Overview

Thin dispatcher. Reads one issue from a local markdown tracker, dispatches `implementer` and `reviewer` subagents, then marks the issue done. One issue per invocation. Does no implementation or review work inline when subagents are available.


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
REDUCTION_OUTCOME = .scratch/<feature>/outcomes/reduction-outcome.json
```

Dispatch `implementer` subagent via Task tool with:
- Full issue body (acceptance criteria, what to build)
- Relevant sections from the parent PRD and/or docs/architecture/gap-analysis.md. ONLY if they add valuable context to the issue. DO NOT include context that will confuse the implementer about the scope of its work.
- Instruction: load the `tdd` skill, implement issue
- Dispatch context:
  - `outcome_path`: `IMPLEMENT_OUTCOME`
  - `commit`: `true` (create new commit on top of HEAD, never --amend)

### 4. Dispatch reviewer subagent

After implementer exits, read `IMPLEMENT_OUTCOME`.

Dispatch `reviewer` subagent via Task tool with:
- Issue body (acceptance criteria)
- Instruction: load `requesting-code-review` skill
- Dispatch context:
  - `outcome_path`: `REVIEW_OUTCOME`


### 5. Handle review verdict

Read `REVIEW_OUTCOME`. Parse the `action` field: `approved`, `changes-requested`, or `escalate`.

**Changes requested:** Dispatch `implementer` subagent with review feedback and `receiving-code-review` skill. Provide `outcome_path` so it overwrites `IMPLEMENT_OUTCOME`. After fix subagent exits, go to step 4 (re-review — both code reviewer and reduction auditor re-run).

**Approved:** Proceed to step 5.5.

### 5.5 Dispatch code reduction auditor

After reviewer approves, dispatch a fresh `reviewer` subagent for a code reduction audit:

Dispatch `reviewer` subagent via Task tool with:
- Instruction: load `minimizing-code` skill
- Dispatch context:
  - `outcome_path`: `REDUCTION_OUTCOME`

### 5.6 Handle reduction verdict

Read `REDUCTION_OUTCOME`. Parse the `action` field.

**Changes requested:** Dispatch `implementer` subagent with reduction feedback and `receiving-code-review` skill. Provide `outcome_path` so it overwrites `IMPLEMENT_OUTCOME`. After fix subagent exits, go to step 4 (re-review: both reviewer and reduction auditor must pass).

**Approved:** Proceed to step 6.

### 6. Verify

Load `verification-before-completion` skill. Run fresh test pass. Do NOT reuse results from implementation or fix phases — stale.

### 7. Mark done

Change `Status:` to `done`. Append `## Outcome` section with one-line summary. Reference `IMPLEMENT_OUTCOME`, `REVIEW_OUTCOME`, and `REDUCTION_OUTCOME` paths. Stop.

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
├── review-outcome.json       # from reviewer (overwritten on re-review)
└── reduction-outcome.json    # from reduction auditor (overwritten on re-audit)
```


## Local Tracker Conventions

```
Status: ready-for-agent  →  in-progress  →  done
```

Issue files: `.scratch/<feature>/issues/NN-slug.md`. Status line near top. Append outcomes under `## Outcome` or `## Comments`.

Blocked issues: skip if `Blocked by` references a non-terminal issue.

# Critical Rules

**DO:**
- Ensure the intent expressed in the docs is clear and well scoped

**DON'T:**
- Implement code
- Get a full understanding of the codebase
- Pass the agents any code in their context: they will explore the codebase.
- Reuse test results from implementation phase — verification must be fresh.
- Procede to next issue — one per invocation.
- Review loop exceeding 3 rounds — escalate.
- Define return schemas - they are defined in the skills.