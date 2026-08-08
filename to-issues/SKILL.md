---
name: to-issues
description: Break requirements docs and architecture gap analysis into independently-grabbable vertical-slice tickets on the project issue tracker. Use when converting plans into issues, creating implementation tickets, or breaking down work from grill-requirements + grill-architecture output. Replaces the old to-prd + to-issues pipeline.
---

# To Issues

Break requirements and architecture docs into independently-grabbable tickets using tracer-bullet vertical slices. No intermediate PRD — source content flows verbatim from source docs into tickets.

The issue tracker and triage label vocabulary should have been provided to you — run `/setup-matt-pocock-skills` if not.

## Process

### 1. Read source documents

Read ALL of:

- `docs/requirements/<aspect>.md` — one or more requirements files (definition of done, user stories, domain context, behavioral scenarios, acceptance criteria, non-functional requirements, out of scope)
- `docs/architecture/gap-analysis.md` — module interfaces, ADR decisions, testing strategy, required changes
- `docs/adr/` — ADRs referenced by the gap analysis
- `CONTEXT.md` — domain glossary (for consistent terminology)

If any of these don't exist, stop and tell the user which are missing. The pipeline expects `grill-requirements` → `grill-architecture` to have run first.

DO NOT summarise, compile, or re-synthesise the source material. Carry it forward verbatim.

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so to understand the current state of the code. Issue titles and descriptions should use the project's domain glossary vocabulary, and respect ADRs in the area you're touching.

### 3. Draft vertical slices

Break the plan into **tracer bullet** issues. Each issue is a thin vertical slice that cuts through ALL integration layers end-to-end, NOT a horizontal slice of one layer.

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every layer
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
</vertical-slice-rules>

### 4. Converge on ticket-level done

For each draft slice, derive a ticket-level "What Done Means" from the feature Definition of Done. This is the subset of the end-to-end human path that this slice enables. Present each slice with:

- **Title**: short descriptive name
- **What Done Means**: one sentence — "After this ticket, a user following the README can [do X, observe Y]"
- **Blocked by**: which other slices (if any) must complete first

Quiz the user:

- Does the ticket-level done accurately capture what this slice enables?
- Do any slices need re-drafting because the DoD alignment revealed a gap?
- Does any slice lack a clear "done means" (no demoable outcome)?

Iterate — re-draft slices if necessary — until the user approves the DoD alignment.

### 5. Assign source content to slices

For each slice, identify which sections of the source docs it implements. The union of all slices MUST cover the union of all source docs — no material may be dropped. Every user story, AC, architectural decision, module interface, testing decision, domain constraint, and the Definition of Done must be assigned to at least one ticket.

This is a distribution step, not a compilation step. Source content is assigned verbatim.

Slices may be 'HITL' or 'AFK'. HITL slices require human interaction, such as an architectural decision or a design review. AFK slices can be implemented and merged without human interaction. Prefer AFK over HITL where possible.

### 6. Quiz the user on granularity and dependencies

Present the content-assigned breakdown. For each slice, show:

- **Title**: short descriptive name
- **Type**: HITL / AFK
- **Blocked by**: which other slices (if any) must complete first
- **Source sections assigned**: which user stories, ACs, architectural decisions, and module interfaces this slice covers

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the dependency relationships correct?
- Should any slices be merged or split further?
- Are the correct slices marked as HITL and AFK?
- Is any source material missing from the breakdown?

Iterate until the user approves the breakdown.

### 7. Verify coverage

Before publishing, verify that no source material was dropped. Walk through each source doc section-by-section and confirm it is assigned to at least one ticket. Specifically:

- Every user story from the requirements doc
- Every behavioral scenario
- Every acceptance criterion (AC-xx)
- The Definition of Done (assigned to all tickets — any slice whose ticket-level done doesn't advance the feature DoD is misaligned)
- Every module interface definition from the gap analysis
- Every architectural decision (ADR and inline) from the gap analysis
- Every testing decision from the gap analysis
- Every required change from the gap analysis
- Every non-functional requirement
- Out of scope and priority sections (assigned to a ticket or confirmed as reference-only)

Report any unassigned material to the user and ask for placement. Do not publish until all source material is accounted for.

### 8. Publish the issues to the issue tracker

For each approved slice, publish a new issue to the issue tracker. Publish issues in dependency order (blockers first) so you can reference real issue identifiers in the "Blocked by" field. Apply the `ready-for-agent` triage label unless instructed otherwise.

Use the issue body template below. Do NOT close or modify any parent issue.

<issue-template>
## Source Documents

- Requirements: `docs/requirements/<aspect>.md`
- Architecture: `docs/architecture/gap-analysis.md`
- ADRs: {list referenced ADR numbers}
- Glossary: `CONTEXT.md`

## What Done Means

**Feature-level** (verbatim from the requirements doc):

{Copy the Definition of Done section verbatim from the requirements doc.}

**This ticket:** After this ticket, a user following the README can {do X, observe Y}. This is the subset of the feature DoD that this slice enables.

## What to Build

A concise description of this vertical slice. Describe the end-to-end behavior, not layer-by-layer implementation.

Avoid specific file paths or code snippets — they go stale fast. Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it here and note briefly that it came from a prototype. Trim to the decision-rich parts — not a working demo, just the important bits.

## Requirements

The relevant material from the source documents, carried forward verbatim. The implementing agent MUST address every section below. Placeholders show what to include — copy the actual content from the source docs, not these descriptions.

### User Stories
Copy the user stories this slice implements, verbatim from the requirements doc. If none apply to this slice, write "None."

### Domain Context
Copy relevant domain context sections verbatim from the requirements doc. If none apply, write "None."

### Behavioral Scenarios
Copy the behavioral scenarios this slice participates in, verbatim from the requirements doc. If none apply, write "None."

### Acceptance Criteria
Copy the ACs this slice must satisfy, verbatim from the requirements doc. If none apply, write "None."

### Architectural Constraints
Copy relevant module interfaces, ADR decisions, dependency direction rules, seam placement, and technology choices verbatim from the gap analysis. DO NOT summarise — copy the exact interface definitions, code blocks, and decision text. If none apply, write "None."

### Testing Decisions
Copy relevant testing strategy, mock/real boundaries, and test levels verbatim from the gap analysis. If none apply, write "None."

### Non-Functional Requirements
Copy NFRs that apply to this slice, verbatim from the requirements doc. If none apply, write "None."

## This Ticket's Acceptance Criteria

Issue-specific acceptance criteria not already captured in the Requirements section above. These may include internal constraints or verification steps specific to this slice.

- [ ] Criterion 1
- [ ] Criterion 2

## Blocked by

- A reference to the blocking ticket (if any)

Or "None — can start immediately" if no blockers.

</issue-template>