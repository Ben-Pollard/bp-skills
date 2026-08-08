---
name: grill-requirements
description: Interview the user relentlessly about what a system should do until requirements are crystallised. Codebase-aware — reads CONTEXT.md for terminology, proposes new glossary entries. Outputs structured requirements docs with EARS-form acceptance criteria and scenario traceability. Also handles respec: if an existing aspect doc is provided via @-reference, grills only the implicated criteria and appends a changelog entry rather than re-running the full interview. Use when user wants to define requirements, spec out a feature, crystallise what to build before any architecture work, or amend an existing requirements doc.
---

# Grill Requirements

Interview me relentlessly about every aspect of what the system should do until we reach shared understanding. Walk down each branch of the requirements tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time.

## What This Skill Produces, and What It Doesn't

This session crystallises requirements into `docs/requirements/<aspect>.md`, using [REQUIREMENTS-TEMPLATE.md](REQUIREMENTS-TEMPLATE.md) as the output structure.

Multiple aspects can be covered in one session. When the current aspect is fully resolved, ask: "Another aspect to cover, or are we done?"

## Requirements vs Architecture

Requirements describe WHAT the system must do. Architecture describes HOW we'll build it. This session stays firmly on the WHAT side.

**In scope:** Problem statements, definition of done, user stories, behavioral scenarios, acceptance criteria, domain descriptions, non-functional concerns, priority, scope boundaries.

**Out of scope:** Module design, interface contracts, data schemas, technology choices, deployment models, seam placement. Those live in `/grill-architecture`.

## Before Grilling

- [ ] **Check what's in context, not what's on disk.** If an existing aspect doc has been provided (e.g. the user `@`-referenced it), this is a **respec** — go to "Respec Mode" below. If no aspect doc is in context, this is a **fresh aspect** — proceed with the full interview.
- [ ] Read `CONTEXT.md` at the repo root (or `CONTEXT-MAP.md` if multi-context). Use its vocabulary when naming concepts.
- [ ] Explore the codebase at a high level — what already exists? Don't trace call graphs or analyse modules. Understand what features are built, what capabilities the system has. This prevents impossible requirements.

## During Grilling (Fresh Aspect)

- [ ] **One question at a time.** Walk each branch of the requirements tree to completion before moving laterally.
- [ ] **Provide a recommended answer** with every question. Don't ask open-ended — propose and let me correct.
- [ ] **Distinguish intent from verification.** For each requirement, first surface the user story ("what do you want to achieve and why?"), then push for the acceptance criterion ("how will you know it's done? what observable outcome tells you it works?"). Every AC must describe something observable from outside the system — a log line, a file created, a process exit, a container state change, a network response. If it can't be observed without reading source code, it's architecture, not a requirement.
- [ ] **Converge on a Definition of Done.** Throughout the conversation, probe for what success looks like from the user's perspective: "When this system is done, what does a user actually do and see?" Help the human crystallize a short, end-to-end description — "I do X, the system does Y, I see Z." Don't demand it as a fixed step; let it emerge as the conversation sharpens understanding. By the time scenarios and ACs are written, the DoD should be clear. Write it as a short paragraph in the output doc.
- [ ] **Push back on vague language.** "The system should handle it" → "What does 'handle' mean? What are the specific outcomes?"
- [ ] **Test with concrete scenarios.** Invent edge cases. "What happens when the user has no items? What about concurrent access?" Write these up as Behavioral Scenarios before extracting ACs from them — the scenario comes first, ACs are extracted from it, not the other way round.
- [ ] **Use CONTEXT.md vocabulary.** If I use a term that conflicts with the glossary, surface it. If a new domain concept emerges, propose adding it to CONTEXT.md using [CONTEXT-FORMAT.md](../grill-architecture/CONTEXT-FORMAT.md).
- [ ] **Resolve dependencies first.** If decision B depends on decision A, ask about A before B.

## Writing Acceptance Criteria (EARS)

Every AC gets a stable ID and one of these five forms. Pick the form that matches the criterion's actual shape — don't force everything into "WHEN... SHALL."

| Form | Pattern | Use for |
|---|---|---|
| Ubiquitous | `THE SYSTEM SHALL {response}` | Always-true properties, no trigger |
| Event-driven | `WHEN {trigger}, THE SYSTEM SHALL {response}` | Something happens, system reacts |
| State-driven | `WHILE {state}, THE SYSTEM SHALL {response}` | Behavior conditional on an ongoing state |
| Unwanted behavior | `IF {trigger}, THEN THE SYSTEM SHALL {response}` | Error handling, failure modes |
| Optional feature | `WHERE {feature present}, THE SYSTEM SHALL {response}` | Behavior that only applies if some capability is enabled |

Format:

```
- [AC-07] (Scenario: {scenario name}) {EARS statement}
```

- **ID**: sequential per aspect doc, zero-padded two digits, never reused once retired.
- **Scenario tag**: which Behavioral Scenario this AC was extracted from. Every scenario should have at least one AC citing it back — if a scenario has zero ACs, that scenario has a hole in it, not just the AC list.
- **Response clause**: must be externally observable. If you can't describe what an outside observer would see, it isn't an AC yet — go back to the scenario and find the observable consequence.

## Updating CONTEXT.md

When a new domain term crystallises during grilling, propose adding it to `CONTEXT.md` immediately:

1. State the term and your proposed one-sentence definition.
2. List any synonyms to explicitly avoid.
3. Ask: "Add this to CONTEXT.md?"

Write to `CONTEXT.md` inline — don't batch. If `CONTEXT.md` doesn't exist, create it lazily using [CONTEXT-FORMAT.md](../grill-architecture/CONTEXT-FORMAT.md).

## Respec Mode

Triggered when an aspect doc is already in context. Do not re-run the full interview.

- [ ] Read the doc's current Acceptance Criteria and its Changelog.
- [ ] Ask what's changing. Grill **only** the ACs implicated by the change — same rigor as fresh grilling (recommended answer per question, push on vagueness, one at a time), but scoped to the affected criteria and any scenario they cite.
- [ ] For each change, determine: ADDED (new AC, next available ID), MODIFIED (existing ID, content changes, old wording preserved in the changelog), or REMOVED (ID retired permanently, reason recorded).
- [ ] Update the AC section in place so it always reflects current state. Append one line per change to the Changelog section — never rewrite changelog history.
- [ ] If a MODIFIED or REMOVED criterion invalidates part of a Behavioral Scenario, update the scenario too, and note in the changelog which scenario was touched.
- [ ] Update the doc's `Last updated` date.

Changelog line format:

```
- {date} — {ADDED|MODIFIED|REMOVED} {AC-ID}: {what changed, one line}
```

## Writing the Requirements Doc

After grilling an aspect (fresh) or resolving a change (respec), write/update `docs/requirements/<aspect>.md` per [REQUIREMENTS-TEMPLATE.md](REQUIREMENTS-TEMPLATE.md):

## After Writing — Two Separate Checks

Run these as distinct passes. They catch different failure modes and one does not substitute for the other.

**1. Form check (mechanical — did we write ACs correctly):**
- [ ] Every AC has a stable ID and matches exactly one of the five EARS shapes.
- [ ] Every AC's response clause is externally observable without reading source code.
- [ ] No EARS phrasing, AC-style IDs, or bracketed criteria appear anywhere outside the Acceptance Criteria section.
- [ ] No architectural decisions have leaked in (no tech choices, no interface designs, no module names) anywhere in the doc.

**2. Coverage check (semantic — did we capture everything, which no grammar can verify for you):**
- [ ] A Definition of Done exists — a short paragraph describing what the user does and sees when the system works end-to-end.
- [ ] Every Behavioral Scenario has at least one AC citing it. Zero-citation scenarios are holes — go back and grill them.
- [ ] Re-reading only the cited ACs for a given scenario reconstructs that scenario's important steps. If it doesn't, the atomization has lost something the prose scenario had — add an AC or revise the scenario.
- [ ] Every question resolved during grilling has its answer reflected somewhere in the doc.
- [ ] Domain terms match CONTEXT.md.
- [ ] Out of scope is explicit.
- [ ] **Actors surfaced.** Propose the set of actors that emerged during grilling (human roles, non-human actors like agents or daemons). Ask the user to confirm the list and flag any missing actor whose perspective was not covered. If a confirmed actor has no user stories, either add them or explicitly note why they don't need requirements.

Then ask: "Requirements crystallised for `<aspect>`. Another aspect to cover, or proceed to `/grill-architecture`?"

## Never

- Propose architecture or technology choices.
- Define formal schemas or data models.
- Design interfaces or module boundaries.
- Skip the grilling interview — don't write or amend the doc without the conversation.
- Batch questions — one at a time, follow each branch to completion.
- Reuse a retired AC ID.
- Let a downstream skill (architecture, gap analysis, tickets) restate or invent AC content — they cite, they don't originate.
- Go hunting for existing requirements files on disk. Only treat a doc as "existing" if it's actually in context.