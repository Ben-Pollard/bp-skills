# bp-skills

Agent skills for the vibe engineering workflow: requirements → architecture → implementation.

## Install

Clone into your agent's skills directory:

```bash
# OpenCode
git clone https://github.com/your-org/bp-skills.git ~/.agents/skills

# Claude Code
git clone https://github.com/your-org/bp-skills.git ~/.claude/skills
```

Skills are discovered automatically from `SKILL.md` files. Run `/setup-matt-pocock-skills` in each project to configure the issue tracker and domain docs.

## Workflow

```
grill-requirements ──→ docs/requirements/<aspect>.md ──┐
                                                        ├──→ to-prd ──→ to-issues ──→ triage ──→ tdd
grill-architecture ──→ docs/architecture/gap-analysis.md ┘
```

**Requirements** describe what the system must do. **Architecture** describes how to build it. Both are crystallised through relentless one-question-at-a-time grilling sessions before any code is written.

Side channels: `prototype` (throwaway design exploration), `diagnose` (bug loop), `zoom-out` (code map), `grill-me` (generic grilling).

## Skills

### Engineering

| Skill | Does |
|-------|------|
| `grill-requirements` | Crystallise what a system should do. Outputs structured requirements docs. |
| `grill-architecture` | Align architecture with requirements. 30-dimension scan, gap analysis + ADRs. |
| `to-prd` | Compile requirements + architecture into a PRD on the issue tracker. |
| `to-issues` | Break a PRD into vertical-slice issues for autonomous agents. |
| `triage` | Move issues through a state machine (needs-triage → ready-for-agent). |
| `tdd` | Red-green-refactor loop. Integration tests through public interfaces. |
| `diagnose` | 6-phase debug loop: feedback loop → reproduce → hypothesise → fix. |
| `zoom-out` | Map modules and callers using domain vocabulary. |
| `prototype` | Throwaway terminal app or UI variations for design exploration. |
| `setup-matt-pocock-skills` | Scaffold per-project issue tracker and domain doc config. |

### Productivity

| Skill | Does |
|-------|------|
| `grill-me` | Relentless interview about any plan or design. |
| `handoff` | Compact conversation for handoff to another agent. |
| `caveman` | 75% token reduction communication mode. |
| `writing-skills` | Create new skills with TDD-for-skills methodology. |

### Misc

| Skill | Does |
|-------|------|
| `git-guardrails-claude-code` | Block dangerous git commands. |
| `setup-pre-commit` | Husky + lint-staged + typecheck hooks. |
| `scaffold-exercises` | Create exercise directory structures. |
| `migrate-to-shoehorn` | Migrate `as` assertions to @total-typescript/shoehorn. |

## License

MIT — see [LICENSE](LICENSE).
