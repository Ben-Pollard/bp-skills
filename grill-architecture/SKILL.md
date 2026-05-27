---
name: grill-architecture
description: Interview the user relentlessly about architectural alignment, scanning 30 dimensions silently and surfacing only what the gap disturbs. Merges top-down gap analysis with bottom-up friction discovery. Outputs gap analysis and ADRs. Use when user wants to align architecture with new requirements, run an architecture health check, or re-align mid-implementation.
---

# Grill Architecture

Interview me relentlessly about architectural alignment until every disturbed dimension is resolved. Walk down each branch of the decision tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time.

## Two Entry Modes

### Gap-Driven Mode
Follows a `/grill-requirements` session. Input: one or more `docs/requirements/<aspect>.md` files. Goal: align current architecture with new requirements.

### Health-Check Mode
Invoked standalone ("let's check the architecture", "where's the friction", "audit the codebase"). Input: current codebase only. Goal: find friction, shallow modules, architectural debt — same as the old `improve-codebase-architecture` but top-down.

## Architecture vs Requirements

Requirements describe WHAT. Architecture describes HOW. This session handles the HOW.

**In scope:** Module decomposition, interface design, seam placement, data models, technology choices, protocol decisions, deployment models, test strategy, build/CI/CD, code organisation, friction discovery, and every other dimension in the scan.

## Before Grilling

- [ ] Read `CONTEXT.md` (or `CONTEXT-MAP.md` for multi-context repos). Use its vocabulary.
- [ ] Read all ADRs in `docs/adr/`. Know what's already decided.
- [ ] **Gap-driven mode:** Read the `docs/requirements/<aspect>.md` file(s). Understand what the system must do.
- [ ] **Health-check mode:** No requirements doc. Proceed directly to architecture exploration.
- [ ] Explore the codebase. In gap-driven mode, focus on modules touched by the requirements. In health-check mode, walk organically looking for friction (shallow modules, tight coupling, untestable surfaces) using [DEEPENING.md](DEEPENING.md).
- [ ] Read the existing `docs/architecture/gap-analysis.md` if it exists. Know the current architectural snapshot.

## The Dimension Scan

Internally, scan all dimensions below. **Do not enumerate them to the user.** Silently check each against the gap (or current architecture in health-check mode). Surface only dimensions the gap disturbs.

**Structure & Seams:**
1. Module decomposition & boundaries
2. Interface design & contracts (invariants, error modes, config)
3. Seam placement (where behavior changes without editing in-place)
4. Dependency direction (in-process → substitutable → remote → external)
5. Domain boundaries / bounded contexts
6. Code health — friction discovery (deletion test, shallow modules, tight coupling)

**Data:**
7. Data models & schemas
8. Data flow (push/pull, sync/async, event streams)
9. State ownership & consistency guarantees
10. Storage technology (relational, document, key-value, event store)
11. Data lifecycle (retention, archival, deletion)

**Communication:**
12. Protocol choices (REST, GraphQL, gRPC, async messaging)
13. Message/event architecture (pub/sub, queues, ordering, idempotency)
14. API versioning & compatibility
15. Integration patterns with external systems

**Quality Attributes:**
16. Performance & latency budgets
17. Scalability model (vertical, horizontal, data volume)
18. Reliability & fault tolerance (retries, circuit breakers, degradation)
19. Consistency model (strong, eventual, causal)
20. Availability & disaster recovery
21. Security & threat model

**Cross-cutting:**
22. AuthN/AuthZ model
23. Error handling strategy
24. Observability (logging, metrics, tracing, alerting)
25. Configuration & feature flags

**Operations:**
26. Build, dependency management, CI/CD
27. Deployment model & infrastructure
28. Test strategy (levels, tools, what's mocked, what's real)
29. Migration strategy (brownfield, data migration, backwards compat)

**Governance:**
30. Repository structure & code sharing

### Relevance Filter

For each dimension, ask three questions silently:

1. Does the gap disturb this dimension? (In health-check mode: is there observable friction?)
2. Is the project mature enough for this dimension to matter? (No auth questions on a bare repo. No migration strategy on a greenfield project. Use judgment — the list is complete, the filtering is contextual.)
3. Is this dimension already settled by an ADR that the gap doesn't challenge?

Only surface the dimension if all three checks pass.

### Settled Decisions

When an ADR covers a dimension and the gap doesn't disturb it, acknowledge it silently and move on. If the gap does disturb it, surface it: "ADR-0003 chose Postgres, but the new requirements need graph traversal. Reopen?"

## During Grilling

- [ ] **One disturbed dimension at a time.** Present what's changed, why it matters, and your recommended answer.
- [ ] **Use precise vocabulary.** Module, interface, seam, adapter, depth, leverage, locality — per [LANGUAGE.md](LANGUAGE.md). Never "component", "service", "API", or "boundary."
- [ ] **Deletion test for module decisions.** If proposing a new module, ask: if we deleted it, would complexity vanish or spread? If vanish, it's shallow — don't create it.
- [ ] **One adapter = hypothetical seam. Two = real.** Don't introduce ports unless something varies across them.
- [ ] **Dependency injection at system boundaries.** Modules take their dependencies, don't create them. Mock only external boundaries — never internal collaborators.
- [ ] **The interface is the test surface.** Tests cross the same seam callers cross.
- [ ] **Push back on shallow proposals.** "That module has a one-line body behind a three-method interface. What's it hiding?"
- [ ] **Offer ADRs sparingly.** Only when the 3-part test passes per [ADR-FORMAT.md](ADR-FORMAT.md): hard to reverse, surprising without context, real trade-off.
- [ ] **Update CONTEXT.md.** When a new domain term emerges, propose adding it per [CONTEXT-FORMAT.md](CONTEXT-FORMAT.md).
- [ ] **Push for concrete interfaces.** When interface design or data models are disturbed, pin down exact method signatures, field types, and schemas before moving to the next dimension. Abstract descriptions like "a Tracker with fetch methods" are insufficient for downstream skills — write the Python-typed protocol, the JSON schema, and the YAML schema inline during grilling.
- [ ] **Distill principles as they emerge.** When a high-level intent emerges that reflects the trade-offs the developer is willing to make or the direction of the project or that are highly generally applicable (e.g., "tests shouldn't care about internals"), note it. At the end of grilling, crystallise only valid candidates into `docs/architecture/principles.md`. Each principle is a short label and a single intent sentence — no expanded rationale. Principles outlive the gap analysis and are not overwritten on subsequent passes.
- [ ] **Resolve dependencies first.** If seam placement depends on technology choice, ask about technology first.

## Interface Design (Optional Sub-Process)

When interface design needs deeper exploration, delegate to [INTERFACE-DESIGN.md](INTERFACE-DESIGN.md):

1. Frame the problem space
2. Spawn 3+ sub-agents with different design constraints in parallel
3. Present and compare designs
4. Make a recommendation

Only invoke this when the interface is genuinely contested — routine interfaces can be decided inline during grilling.

## Writing the Gap Analysis

After grilling, overwrite `docs/architecture/gap-analysis.md` using [GAP-ANALYSIS-TEMPLATE.md](GAP-ANALYSIS-TEMPLATE.md):

- [ ] **Current State** — architecture snapshot, key decisions, ADR references
- [ ] **Required Changes** — what modules, seams, technologies, patterns must change
- [ ] **Module Interfaces** — concrete contracts at every module boundary (method signatures, data model fields, config schema, persisted file schemas). Python-typed dataclasses and ABCs where applicable. This section is the primary input for `to-prd` and TDD — without it, implementation agents cannot write tests against concrete contracts.
- [ ] **Decisions Made** — architectural decisions crystallised in this pass, with ADR numbers
- [ ] **Decisions Deferred** — unresolved decisions, why deferred, when revisited
- [ ] **Affected Dimensions** — which scan dimensions are impacted, brief notes
- [ ] **Open Risks** — assumptions, unknowns, what could go wrong

This is a single canonical file, overwritten per alignment pass. Do not create per-feature gap analyses — they will contradict on cross-cutting concerns.

## Writing ADRs

For decisions meeting the ADR three-part test ([ADR-FORMAT.md](ADR-FORMAT.md)):

1. Scan `docs/adr/` for the highest existing number.
2. Create `docs/adr/NNNN-slug.md` with the one-paragraph format.
3. Reference the ADR from the gap analysis's "Decisions Made" section.

Create the `docs/adr/` directory lazily if it doesn't exist.

## After Writing

Re-read the gap analysis against the grilling conversation. Check:

- [ ] Every decision crystallised during grilling is reflected
- [ ] ADRs created for qualifying decisions
- [ ] No dimension was left unresolved — each is either decided, deferred, or confirmed unchanged
- [ ] Domain terms match CONTEXT.md
- [ ] Module Interfaces section is concrete enough for an implementation agent to write code without re-interviewing the architect — method signatures have types, data models have fields, persisted files have schemas
- [ ] **Principles written** — `docs/architecture/principles.md` created or updated with one-line principles distilled from this session
- [ ] The gap analysis is self-contained — a new agent reading only this file and the requirements docs should understand what to build

Then ask: "Architecture aligned. Another pass needed, or proceed to `/to-prd`?"

## Never

- Enumerate the full dimension list to the user — silent scan, selective surfacing
- Grill dimensions that aren't disturbed by the gap (or show no friction in health-check mode)
- Grill dimensions the project isn't mature enough for
- Re-litigate settled ADRs unless the gap explicitly challenges them
- Create per-feature gap analysis files — overwrite the single canonical file
- Batch questions — one dimension at a time, follow each branch to completion
- Propose shallow modules — apply the deletion test before proposing new structure
