---
name: minimizing-code
description: Use when reviewing code, auditing a codebase for quality, or evaluating declarative-over-procedural compliance — the systematic line-reduction audit process that counts eliminable lines per module against three questions (stdlib already solved? library could solve? code unnecessary?)
---

# Minimizing Code

## Overview

For each module, ask three questions from `.agents/skills/tdd/declarative-over-procedural.md`:

1. **Already solved?** Does stdlib or an existing dependency cover this?
2. **Solvable by adding a dependency?** Would a library express this in fewer lines?
3. **Unnecessary?** Is this dead code or a pass-through that adds no value?


## Audit Process

### Step 1: List modules with line counts

```bash
wc -l src/mypackage/*.py tests/**/*.py
```

### Step 2: For each module, answer the three questions

**Question 1 — Already solved by stdlib/dep?**
- Check `dataclasses.asdict()`, `functools`, `pathlib`, `os.path.expandvars`
- Read the code looking for manual field-by-field mapping, isinstance chains, custom serialization

**Question 2 — Solvable by adding a dep?**
- Search for libraries that handle the pattern: use `context7` to query docs, web search for "python serialize dataclasses to JSON" etc.
- For each candidate library, estimate: current lines vs. lines with library, what code it absorbs
- Weigh: one dep replacing manual code in 3+ modules is usually worth it

**Question 3 — Unnecessary?**
- For every function: grep for callers. If only called from tests or not called at all (and not an entrypoint) → dead code candidate.
- Look for duplicated logic across modules/functions (same pattern appearing twice)

### Step 3: Produce the reduction outcome

Output a reduction outcome document conforming to this schema:

```json
{
  "modules_audited": <int>,
  "total_lines": <int>,
  "lines_eliminable": <int>,
  "reduction_table": [
    {
      "module": "<path>",
      "lines": <int>,
      "what_changes": "<concrete description>",
      "reduction": <int>,
      "dep_cost": "<none | library name>"
    }
  ],
  "violations": [
    {
      "file_line": "<file:line>",
      "lines": <int>,
      "replacement": "<what would replace it>",
      "question": <1 | 2 | 3>
    }
  ],
  "review_notes": "<summary>",
  "action": "approved | changes-requested | escalate"
}
```

`reduction_table` rows capture the table from the audit — exact line numbers, exact library names, exact deltas. `violations` are the file violations (file:line, line count, replacement, which question applies). `action` drives the iterate-backlog workflow. Keep it concrete — no hand-waving.

## Library Discovery Workflow

When Question 2 is active:
1. Identify the procedural pattern (e.g. "manual isinstance validation", "hand-rolled JSON serialization")
2. Use `context7_resolve-library-id` to find relevant libraries
3. Use `context7_query-docs` to confirm the library handles the exact pattern
4. Check `pyproject.toml` — is it already a dependency? If yes, audit moves to Question 1.


## Critical Rules

**DO:**
- Trace callers (grep for function name) to find dead code
- Search for libraries (context7, web) before concluding Question 2 is "no"

**DON'T:**
- Conclude "no library exists" without searching

