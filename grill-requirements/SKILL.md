---
name: grill-requirements
description: Interview the user relentlessly about what a system should do until requirements are crystallised. Codebase-aware — reads CONTEXT.md for terminology, proposes new glossary entries. Outputs structured requirements docs. Use when user wants to define requirements, spec out a feature, or crystallise what to build before any architecture work.
---

# Grill Requirements

Interview me relentlessly about every aspect of what the system should do until we reach shared understanding. Walk down each branch of the requirements tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time.

## What Gets Produced

This session crystallises requirements into `docs/requirements/<aspect>.md`. Use [REQUIREMENTS-TEMPLATE.md](REQUIREMENTS-TEMPLATE.md) as the output structure.

Multiple aspects can be covered in one session. When the current aspect is fully resolved, ask: "Another aspect to cover, or are we done?"

## Requirements vs Architecture

Requirements describe WHAT the system must do. Architecture describes HOW we'll build it. This session stays firmly on the WHAT side.

**In scope:** Problem statements, user stories, behavioral scenarios, acceptance criteria, domain descriptions (prose, not schemas), non-functional concerns, priority, scope boundaries.

**Out of scope:** Module design, interface contracts, data schemas, technology choices, deployment models, seam placement. Those live in `/grill-architecture`.

## Before Grilling

- [ ] Read `CONTEXT.md` at the repo root (or `CONTEXT-MAP.md` if multi-context). Use its vocabulary when naming concepts.
- [ ] Explore the codebase at a high level — what already exists? Don't trace call graphs or analyse modules. Understand what features are built, what capabilities the system has. This prevents impossible requirements.
- [ ] Check for existing `docs/requirements/` files. Don't contradict settled requirements without flagging it.

## During Grilling

- [ ] **One question at a time.** Walk each branch of the requirements tree to completion before moving laterally.
- [ ] **Provide a recommended answer** with every question. Don't ask open-ended — propose and let me correct.
- [ ] **Distinguish intent from verification.** For each requirement, first surface the user story ("what do you want to achieve and why?"), then push for the acceptance criterion ("how will you know it's done? what observable outcome tells you it works?"). Every AC must describe something observable from outside the system — a log line, a file created, a process exit, a container state change, a network response. If it can't be observed without reading source code, it's architecture, not a requirement.
- [ ] **Push back on vague language.** "The system should handle it" → "What does 'handle' mean? What are the specific outcomes?"
- [ ] **Test with concrete scenarios.** Invent edge cases. "What happens when the user has no items? What about concurrent access?"
- [ ] **Use CONTEXT.md vocabulary.** If I use a term that conflicts with the glossary, surface it. If a new domain concept emerges, propose adding it to CONTEXT.md using [CONTEXT-FORMAT.md](../grill-architecture/CONTEXT-FORMAT.md).
- [ ] **Resolve dependencies first.** If decision B depends on decision A, ask about A before B.

## Updating CONTEXT.md

When a new domain term crystallises during grilling, propose adding it to `CONTEXT.md` immediately:

1. State the term and your proposed one-sentence definition.
2. List any synonyms to explicitly avoid.
3. Ask: "Add this to CONTEXT.md?"

Write to `CONTEXT.md` inline — don't batch. If `CONTEXT.md` doesn't exist, create it lazily using [CONTEXT-FORMAT.md](../grill-architecture/CONTEXT-FORMAT.md).

## Writing the Requirements Doc

After grilling an aspect, write the `docs/requirements/<aspect>.md` file using the template structure:

- [ ] **Problem Statement** — what and why, no how
- [ ] **Solution** — desired behavior from the user's perspective
- [ ] **User Stories** — "As a X, I want Y, so that Z" format
- [ ] **Domain Context** — prose descriptions of entities and relationships. No schemas.
- [ ] **Behavioral Scenarios** — step-by-step walkthroughs including edge cases
- [ ] **Acceptance Criteria** — observable, testable conditions
- [ ] **Non-Functional Requirements** — perf, security, scale, reliability concerns
- [ ] **Out of Scope** — explicit boundaries
- [ ] **Priority** — relative importance and dependencies
- [ ] **Last updated** date in the header

Keep domain descriptions in prose. Formal schemas, data models, and entity-relationship diagrams are architectural — save those for `/grill-architecture`.

## After Writing

Re-read the document against the grilling conversation. Check:

- [ ] Every question resolved during grilling has its answer reflected
- [ ] No architectural decisions have leaked in (no tech choices, no interface designs, no module names)
- [ ] Domain terms match CONTEXT.md
- [ ] **Acceptance criteria describe externally observable behavior** — visible from outside the system without reading source code
- [ ] Out of scope is explicit
- [ ] **Actors surfaced.** Propose the set of actors that emerged during grilling (human roles, non-human actors like agents or daemons). Ask the user to confirm the list and flag any missing actor whose perspective was not covered. If a confirmed actor has no user stories, either add them or explicitly note why they don't need requirements.

Then ask: "Requirements crystallised for `<aspect>`. Another aspect to cover, or proceed to `/grill-architecture`?"

## Never

- Propose architecture or technology choices
- Define formal schemas or data models
- Design interfaces or module boundaries
- Skip the grilling interview — don't write the doc without the conversation
- Batch questions — one at a time, follow each branch to completion
