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
  - Does the implementation match the requirements?
  - Testing requirements met? Integration tests included where called for in the issue?
  - All planned functionality present?
  - No extra features not requested?
  - Will the code work?

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

  **DON'T:**
  - Ignore criteria defined in skills because they are less familiar: deep modules, mocking at system boundaries, sdk-style interfaces, refactoring opportunities, testing public interfaces, declarative over procedural, etc. as referenced in `.agents/skills/tdd/deep-modules.md`, `.agents/skills/tdd/mocking.md`, `.agents/skills/tdd/interface-design.md`, `.agents/skills/tdd/refactoring.md`, `.agents/skills/tdd/tests.md`, and `.agents/skills/tdd/declarative-over-procedural.md`.
  - Assume this code will be inspected thoroughly in the future - "It's fine for now", "pre-existing issue", "not relevant to this change"
  - Ignore spec criteria: "that's not needed yet" - that's not your decision.
  - Avoid giving a clear verdict

## Critical Rules

**DO:**
- Respect testing levels explicitly defined in the issue. If it calls for an integration test against a real service then write an integration test.



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