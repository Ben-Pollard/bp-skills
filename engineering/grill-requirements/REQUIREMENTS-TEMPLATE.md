# Requirements Template

```markdown
# {Aspect Name}

> Last updated: {date}

## Problem Statement

{What problem does this solve? Why does it matter? Who experiences it?}

## Solution

{What does the system need to do, from the user's perspective? Describe the desired outcome and behavior. No architecture here.}

## User Stories

- As a **{actor}**, I want **{feature}**, so that **{benefit}**.
- As a **{actor}**, I want **{feature}**, so that **{benefit}**.

## Domain Context

{Prose description of the domain. What entities, concepts, and relationships exist? No formal schemas or data models — those are architectural. Use the project's CONTEXT.md vocabulary where applicable.}

## Behavioral Scenarios

{Walk through key workflows step by step. What happens in each case? Include edge cases and error conditions.}

### Scenario: {Name}

1. {Step}
2. {Step}

### Scenario: {Name}

1. {Step}
2. {Step}

## Acceptance Criteria

- [ ] {Observable, testable condition}
- [ ] {Observable, testable condition}

## Non-Functional Requirements

{Performance, security, scalability, reliability, usability concerns. Use natural language, not service-level targets unless known.}

## Out of Scope

- {What are we explicitly NOT building?}
- {Prevents scope creep and clarifies boundaries}

## Priority

{Relative importance: critical / high / medium / low. What depends on this? What can wait?}
```
