---
name: requesting-code-review
description: Use when completing tasks, implementing major features, or before merging to verify work meets requirements
---
# What we're trying to achieve
Code that:
- Works
- Is maintainable
- Is understandable


# What to Check

  **Spec compliance:**
  - Does the implementation match the intent of the ticket? The intent is never that code should exist but that the system should behave a certain way when a human new to the system starts using it. 
  - **Trace the call chain end-to-end.** If the behavioural scenario says "poll discovers, dispatches, advances," verify the code contains every link. A log line at the entry point does not prove the chain completes. Proximity is not connectivity — presence of a call at step 1 is not evidence step 2 runs.
  - All planned functionality present? No extra features not requested?
  - **Full test suite passes** Run every integration test file. Not just the module you changed. Not just the directory you edited. A change in module A can break assumptions in module B. The full suite protects all downstream consumers.
  - **Testing levels met?** Read the Testing Decisions section. Count the test levels specified (e.g. node → pipeline → E2E = 3 levels). Verify every level exists in the test suite. A missing level is a spec violation — not a test quality niggle. `pytest` collecting 0 integration tests when the issue calls for them is a hard FAIL. 


  **Test quality** — read `.agents/skills/tdd/tests.md`, `.agents/skills/tdd/deep-modules.md`, and `.agents/skills/tdd/mocking.md`. Do the tests adhere to these principles?

  **Code quality**
  - SOLID
  - KISS
  - DRY
  - YAGNI
  - `.agents/skills/tdd/deep-modules.md`
  - `.agents/skills/tdd/interface-design.md`
  - `.agents/skills/tdd/refactoring.md`
  - `.agents/skills/tdd/declarative-over-procedural.md`

  **Operational:**
  - Security risks?
  - Docs match what was built?

  ## Output Format

  ### Issues
  [Flat list. Each issue is something that must be fixed before approval.
  File:line reference, what's wrong, why it matters.]

  ### Assessment
  **Verdict:** [approved | changes-requested]
  **Reasoning:** [1-2 sentences]

  ## Critical Rules

  **DO:**
  - Read all the .md files referred to above (use Read tool on the full `.agents/skills/tdd/*.md` paths)
  - Be specific (file:line, not vague)
  - Explain WHY each issue matters
  - Give a clear verdict
  - **Review the full system, not a set of patches.** Read the full diff from main (or all affected files) every time — especially on re-review. Checking that listed fixes were applied is not a review.
  - **Count test levels.** If the Testing Decisions section lists N test levels, the implementation must have N test levels. Missing a level is a spec violation. Report it.

  **DON'T:**
  - Ignore criteria defined in skills because they are less familiar: deep modules, mocking at system boundaries, sdk-style interfaces, refactoring opportunities, testing public interfaces, declarative over procedural, etc. as referenced in `.agents/skills/tdd/deep-modules.md`, `.agents/skills/tdd/mocking.md`, `.agents/skills/tdd/interface-design.md`, `.agents/skills/tdd/refactoring.md`, `.agents/skills/tdd/tests.md`, and `.agents/skills/tdd/declarative-over-procedural.md`.
  - Assume this code will be inspected thoroughly in the future - "It's fine for now", "pre-existing issue", "not relevant to this change"
  - Ignore spec criteria: "that's not needed yet" - that's not your decision.
  - Avoid giving a clear verdict
  - Accept a list of fixes as a review scope. You are reviewing the system the fix is part of, not the correctness of the fix in isolation.
  - Treat testing levels as someone else's problem. If the issue says E2E tests must exist and they don't, that is your finding to report.

  ## Red Flags — STOP and Report Changes Requested

  If you find yourself thinking any of these, you are rationalising:

  - "The fixes were applied — approved" (you're reviewing patches, not the system)
  - "Testing strategy is someone else's concern" (test level gaps are spec violations)
  - "The code structure looks right, the wire will work" (proximity is not connectivity)
  - "No integration tests but the unit tests are solid" (if the issue calls for them, missing = FAIL)
  - "They said the previous 7 violations were resolved" (verify the full system, not the claim)

  **All of these mean: you are reviewing surface fixes, not the system.**



# Verdict

## Format

  ```json
  {
    "spec_compliance": true,
    "code_quality": {
      "deep-modules.md": false,
      "interface-design.md": true,
      "refactoring.md": false,
      "declarative-over-procedural.md": false,
      "SOLID": true,
      "DRY": true,
      "YAGNI": true
    },
    "test_quality": {
      "deep-modules.md": false,
      "interface-design.md": true,
      "mocking.md": false
    },
    "operational": true,
    "violations": [
      {
        "principle": "Survivable Tests",
        "file": "src/payment.ts:55",
        "issue": "Internal collaborator mocked — tests will break on refactor"
      }
    ],
    "review_notes": [
      "All acceptance criteria met with passing tests",
      "Mock at internal seam (payment.ts:55) violates Survivable Tests principle"
    ],
    "action": "changes_requested"
  }
  ```

  ## Output
  
  Write the verdict JSON to the `outcome_path` if provided. Create parent directories if needed.