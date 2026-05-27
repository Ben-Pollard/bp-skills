---
name: to-prd
description: Compile requirements docs and architectural gap analysis into a PRD and publish it to the project issue tracker. Use after grill-requirements and grill-architecture are complete.
---

# To PRD

Compile the structured outputs of `/grill-requirements` and `/grill-architecture` into a single PRD on the project issue tracker. This is a compilation step — the decisions are already made.

Do NOT interview the user. Read the inputs, synthesise, publish.

The issue tracker and triage label vocabulary should have been provided to you — run `/setup-matt-pocock-skills` if not.

## Inputs

- [ ] `docs/requirements/<aspect>.md` — one or more requirements files
- [ ] `docs/architecture/gap-analysis.md` — the canonical gap analysis
- [ ] `docs/adr/` — ADRs referenced by the gap analysis
- [ ] `CONTEXT.md` — domain glossary (for consistent terminology)

If any of these don't exist, stop and tell the user which are missing. The pipeline expects `grill-requirements` → `grill-architecture` to have run first.

## Process

1. Read all inputs. Use the project's domain glossary vocabulary throughout.

2. Map requirements to architecture:
   - User stories from the requirements doc become the "User Stories" section
   - Behavioral scenarios and acceptance criteria inform the user stories
   - Module decisions, interface changes, technology choices from the gap analysis become the "Implementation Decisions" section
   - Test strategy from the gap analysis (dimension 28) and testability principles become the "Testing Decisions" section
   - Out of scope from the requirements doc becomes the "Out of Scope" section

3. Write the PRD using the template below, then publish it to the project issue tracker. Apply the `ready-for-agent` triage label — no need for additional triage.

<prd-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

This list of user stories should be extremely extensive and cover all aspects of the feature.

## Implementation Decisions

A list of implementation decisions from the architecture alignment. This can include:

- The modules that will be built/modified (using CONTEXT.md vocabulary and [LANGUAGE.md](../grill-architecture/LANGUAGE.md) terminology)
- The interfaces of those modules
- Architectural decisions (reference relevant ADRs by number)
- Schema changes
- API contracts
- Technology choices
- Seam placement and dependency strategy

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts.

## Testing Decisions

A list of testing decisions from the architecture alignment. Include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested and at what level
- What's mocked vs real (system boundaries only)
- Prior art for the tests (similar types of tests in the codebase)
- Which modules are deep enough to warrant focused test coverage

## Out of Scope

A description of the things that are out of scope for this PRD.

## Source Documents

- Requirements: `docs/requirements/<aspect>.md`
- Architecture: `docs/architecture/gap-analysis.md`
- ADRs: {list referenced ADR numbers}

</prd-template>

## After Publishing

Confirm the PRD is live on the issue tracker. Offer: "PRD published. Run `/to-issues` to break this into vertical slices?"
