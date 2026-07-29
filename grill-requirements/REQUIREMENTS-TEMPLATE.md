# {Aspect Name}

> Last updated: {date}

## Problem Statement

> Form: free prose. No IDs, no EARS phrasing (`WHEN... SHALL...`), no bullet-per-clause structure.

{What problem does this solve? Why does it matter? Who experiences it? Exists so anyone can recover motivation.}

## Solution

> Form: free prose, from the user's perspective. No architecture, no IDs.

{What does the system need to do? Describe the desired outcome and behavior. }

## User Stories

> Form: `As a **{actor}**, I want **{feature}**, so that **{benefit}**.` One per bullet, no nesting, no IDs. Not independently checkable — these are read, not queried, and nothing downstream cites a story by number. 

- As a **{actor}**, I want **{feature}**, so that **{benefit}**.
- As a **{actor}**, I want **{feature}**, so that **{benefit}**.

## Domain Context

> Form: free prose. No formal schemas, data models, or entity-relationship diagrams — those are architectural. Use the project's CONTEXT.md vocabulary where applicable.

{What entities, concepts, and relationships exist?}

## Behavioral Scenarios

> Form: the shape of the intended experience, as numbered step-by-step walkthroughs, prose. No EARS phrasing here — this section carries relational/sequential information (order, cross-step constraints) that atomized acceptance criteria below cannot express on their own. Every scenario should end up cited by at least one Acceptance Criterion; a scenario with zero citations is a coverage gap.

### Scenario: {Name}

1. {Step}
2. {Step}

### Scenario: {Name}

1. {Step}
2. {Step}

## Acceptance Criteria

> Form: EARS. Every line has a stable ID (`AC-NN`, zero-padded, sequential per doc, never reused once retired) and a scenario tag. Pick whichever of the five EARS shapes actually matches the criterion — don't force everything into event-driven form.
>
> - Ubiquitous: `THE SYSTEM SHALL {response}`
> - Event-driven: `WHEN {trigger}, THE SYSTEM SHALL {response}`
> - State-driven: `WHILE {state}, THE SYSTEM SHALL {response}`
> - Unwanted behavior: `IF {trigger}, THEN THE SYSTEM SHALL {response}`
> - Optional feature: `WHERE {feature present}, THE SYSTEM SHALL {response}`
>
> The response clause must be externally observable — a log line, a file created, a process exit, a state change, a network response — without reading source code.

- [AC-01] (Scenario: {Name}) WHEN {trigger}, THE SYSTEM SHALL {observable response}
- [AC-02] (Scenario: {Name}) THE SYSTEM SHALL {observable response}

{ACs are the unit that gets validated. Their job: one claim, with some external observation that would prove it false if it weren't true}

## Non-Functional Requirements

> Form: free prose. Natural language, not service-level targets unless already known.

{Performance, security, scalability, reliability, usability concerns.}

## Out of Scope

> Form: free prose bullets.

- {What are we explicitly NOT building?}
- {Prevents scope creep and clarifies boundaries}

## Priority

> Form: free prose.

{Relative importance: critical / high / medium / low. What depends on this? What can wait?}

## Changelog

> Form: append-only. Never rewrite or delete a prior line — this is the history that a redefinition-tracking metric reads. One line per change, oldest first.
>
> `- {date} — {ADDED|MODIFIED|REMOVED} {AC-ID}: {what changed, one line}`

{Empty on first authoring. Grows on every respec.}