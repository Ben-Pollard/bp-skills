---
name: verification-before-completion
description: Use when verifying that implemented work satisfies behavioral acceptance criteria before merging, claiming completion, or handing off for human approval. Use when the system is ready to be exercised against its documented requirements.
---

# Verification Before Completion

Verify behavioral acceptance criteria against the live system. You are the last line of defence before human approval — the TDD agent scoped to this ticket, the reviewer scoped to this feature. You protect main from regressions and misalignment.

**Core principle:** The system must be exercised, not read. Code is not behaviour. A running system is the only valid evidence.

## Fast-Fail Chain

Do not skip stages. Each stage gates the next. If a stage fails, stop and report — do not proceed to later stages.

```
1. LINT / TYPE-CHECK → fail? STOP, report FAIL
2. UNIT TESTS → fail? STOP, report FAIL
3. INTEGRATION TESTS (full suite) → fail? STOP, report FAIL
4. AC VERIFICATION → exercise each AC against the live system
```

Integration tests are not redundant. They test real infrastructure, configuration, and wiring that unit tests mock away. Skipping integration tests because "the unit tests cover the logic" is skipping the only tests that catch infrastructure regressions.

### Full suite means the whole project

Run every integration test in the test suite. Not just the module you changed. Not just the tests in the same directory as your changes.

A change in module A can break assumptions in module B. The integration test for module B may call module A's API — if you broke the contract, that test catches it. Running only the tests for the module you changed leaves every downstream consumer untested.

"Full suite" means every integration test file in the project. No exceptions for "unrelated modules." If a module imports from your module or calls its API, it is related.

## Live System Verification

### Start the system

Read the project's README and any docs linked from it. Follow the startup instructions exactly as a new user would. Start every service the system depends on. If the README is insufficient to start the system, every AC that requires a running system is CANT_VERIFY — that is a FAIL.

### Exercise each AC

For each acceptance criterion in the ticket's Requirements section:

1. Determine what event must be triggered and what behaviour must be observed — use the README, linked docs, and the AC's own description (log messages, HTTP endpoints, CLI commands, state transitions)
2. Trigger the event against the live system
3. Observe the result — `docker logs`, `curl`, `git log`, CLI output, HTTP responses
4. Report PASS (with captured evidence) or FAIL (expected vs actual)

If the README and linked docs do not contain enough information to trigger an event or observe its result, report CANT_VERIFY. Insufficient documentation is a system failure.

**Never verify an AC by reading source code.** A `logger.info()` call is not evidence that the log appears at runtime. A `return 200` line is not evidence that `/health` responds. Configuration, wiring, environment, and startup order all affect what actually runs. Only the live system tells the truth.

## Output

Write output as JSON. Report only failures and cannot-verify — passed ACs are implicit.

```json
{
  "status": "PASS" | "FAIL",
  "stage_results": {
    "lint": {"passed": true},
    "unit_tests": {"passed": 42, "failed": 0},
    "integration_tests": {"passed": 18, "failed": 0}
  },
  "failed_acs": [
    {
      "id": "AC-01",
      "text": "WHEN a ticket is in Ready state...",
      "reason": "expected 'dispatching tdd' in docker logs, found nothing"
    },
    {
      "id": "AC-31",
      "text": "Given a ticket ID, the front end SHALL return...",
      "reason": "CANT_VERIFY: front-end URL not documented in README"
    }
  ]
}
```

`status` is PASS only if every stage passes and `failed_acs` is empty. If any stage fails or any AC fails or cannot be verified, `status` is FAIL.
