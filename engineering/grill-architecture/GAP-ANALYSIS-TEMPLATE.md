# Gap Analysis Template

```markdown
# Architecture Gap Analysis

> Last updated: {date}
> Triggered by: {requirements doc reference or "health check"}

## Current State

{What does the architecture look like right now? Summarise key decisions, module boundaries, technology stack, patterns in use. Reference relevant ADRs.}

## Required Changes

{What must change to accommodate the requirements? What new modules, seams, interfaces, technologies, patterns are needed? What existing ones are affected?}

## Decisions Made

{List each architectural decision that crystallised during this pass. For decisions meeting the ADR three-part test, reference the ADR number.}

- Decision 1
- Decision 2

## Decisions Deferred

{Decisions raised but not yet resolved. Why deferred? When will they need to be revisited?}

## Affected Dimensions

{Which of the 30 scan dimensions are impacted? Brief note on each.}

- **Module decomposition**: {what changes}
- **Data models**: {what changes}

## Open Risks

{What could go wrong? What assumptions are we making? What don't we know yet?}
```
