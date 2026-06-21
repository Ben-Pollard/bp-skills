---
name: tdd
description: Test-driven development with red-green-refactor loop. Use when user wants to build features or fix bugs using TDD, mentions "red-green-refactor", wants integration tests, or asks for test-first development.
---

# Test-Driven Development

## Philosophy

**Core principle**: Tests should verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't.

**Good tests** are integration-style: they exercise real code paths through public APIs. They describe _what_ the system does, not _how_ it does it. A good test reads like a specification - "user can checkout with valid cart" tells you exactly what capability exists. These tests survive refactors because they don't care about internal structure.

**Bad tests** are coupled to implementation. They mock internal collaborators, test private methods, or verify through external means (like querying a database directly instead of using the interface). The warning sign: your test breaks when you refactor, but behavior hasn't changed. If you rename an internal function and tests fail, those tests were testing implementation, not behavior.

See [tests.md](tests.md) for examples and [mocking.md](mocking.md) for mocking guidelines.

## Anti-Pattern: Horizontal Slices

**DO NOT write all tests first, then all implementation.** This is "horizontal slicing" - treating RED as "write all tests" and GREEN as "write all code."

This produces **crap tests**:

- Tests written in bulk test _imagined_ behavior, not _actual_ behavior
- You end up testing the _shape_ of things (data structures, function signatures) rather than user-facing behavior
- Tests become insensitive to real changes - they pass when behavior breaks, fail when behavior is fine
- You outrun your headlights, committing to test structure before understanding the implementation

**Correct approach**: Vertical slices via tracer bullets. One test → one implementation → repeat. Each test responds to what you learned from the previous cycle. Because you just wrote the code, you know exactly what behavior matters and how to verify it.

```
WRONG (horizontal):
  RED:   test1, test2, test3, test4, test5
  GREEN: impl1, impl2, impl3, impl4, impl5

RIGHT (vertical):
  RED→GREEN: test1→impl1
  RED→GREEN: test2→impl2
  RED→GREEN: test3→impl3
  ...
```

# Software Engineering Best Practices
You implement via test-driven development. But don't forget best practices: SOLID, DRY, YAGNI.

## Workflow

### 1. Planning

When exploring the codebase, refer to CONTEXT.md so that test names and interface vocabulary match the project's language, and respect ADRs in the area you're touching (docs/adr).

Before writing any code:

HITL mode:
- [ ] Confirm with user what interface changes are needed
- [ ] Confirm with user which behaviors to test (prioritize)
- [ ] Identify opportunities for [deep modules](deep-modules.md) (small interface, deep implementation)
- [ ] Design interfaces for [testability](interface-design.md)
- [ ] List the behaviors to test (not implementation steps)
- [ ] Get user approval on the plan

Ask: "What should the public interface look like? Which behaviors are most important to test?"

**You can't test everything.** Confirm with the user exactly which behaviors matter most. Focus testing effort on critical paths and complex logic, not every possible edge case.

AFK mode:
- [ ] Identify what interface changes are needed
- [ ] Identify what behaviors to test (prioritize)
- [ ] Identify opportunities for [deep modules](deep-modules.md) (small interface, deep implementation)
- [ ] Design interfaces for [testability](interface-design.md)

Focus testing effort on critical paths and complex logic, not every possible edge case.



### 2. Tracer Bullet

Write ONE test that confirms ONE thing about the system:

```
RED:   Write test for first behavior → test fails
GREEN: Write minimal code to pass → test passes
```

This is your tracer bullet - proves the path works end-to-end.

### 3. Incremental Loop

For each remaining behavior:

```
RED:   Write next test → fails
GREEN: Minimal code to pass → passes
```

Rules:

- One test at a time
- Only enough code to pass current test
- Don't anticipate future tests
- Keep tests focused on observable behavior

### 4. Refactor

After all tests pass, look for [refactor candidates](refactoring.md):

- [ ] Minimise procedural code — see [declarative-over-procedural.md](declarative-over-procedural.md)
- [ ] Extract duplication
- [ ] Deepen modules (move complexity behind simple interfaces)
- [ ] SRP: every class has both data and behavior — no empty shells. DRY: no identical mutation sequence repeated across functions.
- [ ] Consider what new code reveals about existing code
- [ ] Run tests after each refactor step

- Unfamiliar with a library? -> use context7
- Don't know the cleanest way to approach a problem? -> use web search to inform yourself

**Never refactor while RED.** Get to GREEN first.

## Checklist Per Cycle

```
[ ] Test describes behavior, not implementation
[ ] Test uses public interface only
[ ] Test would survive internal refactor
[ ] Code is minimal for this test
[ ] No speculative features added
```

### 5. Commit and Report

Commit all changes:

```bash
git add -A && git commit -m "<brief summary of what was built>"
```

Get the commit SHA:

```bash
git rev-parse HEAD
```

When the checklist is complete, write the outcome file at the path provided by the orchestrator (`outcome_path`). Create parent directories if they don't exist.

```json
{
  "status": "DONE",
  "summary": "Implemented user authentication with JWT flow",
  "test_results": {
    "passed": 14,
    "failed": 0,
    "skipped": 0
  },
  "commit_sha": "abc123def456...",
  "concerns": []
}
```

Use `DONE_WITH_CONCERNS` if you completed but have doubts. Use `BLOCKED` if you cannot proceed. Use `DONE` only if all ACs are met.


## Critical Rules

**DO:**
- Respect testing levels explicitly defined in the issue. If it calls for an integration test against a real service then write an integration test.

**DON'T:**
- Reason away the one test at a time rule because you are planning on creating many tests: maybe that means you are not testing public interfaces or not parameterising effectively?
- Minimise procedural code — see [declarative-over-procedural.md](declarative-over-procedural.md)