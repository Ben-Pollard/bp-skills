# Gap Analysis Template

```markdown
# Architecture Gap Analysis

> Last updated: {date}
> Triggered by: {requirements doc reference or "health check"}

## Current State

{What does the architecture look like right now? Summarise key decisions, module boundaries, technology stack, patterns in use. Reference relevant ADRs.}

## Required Changes

{What must change to accommodate the requirements? What new modules, seams, interfaces, technologies, patterns are needed? What existing ones are affected?}

## Module Interfaces

{Concrete contracts at every module boundary. Define exact method signatures, data model fields, config schema keys, and persisted file schemas. Each downstream skill (to-prd, TDD) reads this section to write tests and decompose work without re-interviewing the architect.

Organise by module. Use Python-typed dataclass definitions for data models, ABC method signatures with error modes for interface seams, YAML/JSON schema keys for config and persisted files.}

### Example: `tracker/protocol.py`

```python
@dataclass
class Issue:
    id: str                          # tracker-internal stable ID
    identifier: str                  # human-readable key (e.g. "42", "symphony-01-poll-loop")
    title: str
    description: str | None
    state: str                       # normalized: "ready-for-agent", "done", etc.
    labels: list[str]
    blocked_by: list[BlockerRef]
    url: str | None
    priority: int | None             # lower = higher priority
    created_at: datetime | None
    updated_at: datetime | None

@dataclass
class BlockerRef:
    id: str
    identifier: str | None
    state: str | None

class Tracker(ABC):
    @abstractmethod
    async def fetch_candidates(self, active_states: set[str]) -> list[Issue]: ...
    @abstractmethod
    async def fetch_issue(self, issue_id: str) -> Issue | None: ...             # None = not found
    @abstractmethod
    async def fetch_terminal_ids(self, terminal_states: set[str]) -> set[str]: ...
```

### Example: `state.json` schema

```json
{
  "version": 1,
  "issues": {
    "<issue_id>": {
      "identifier": "42",
      "status": "running" | "retry_queued" | "unclaimed",
      "stage": "implement",
      "attempt": 0,
      "volume_name": "workspace-42",
      "container_name": "sandbox-42",
      "container_id": "abc123...",
      "started_at": "2026-05-17T10:00:00Z",
      "retry_due_at": null | "2026-05-17T10:05:00Z"
    }
  }
}
```

### Example: WORKFLOW.md YAML schema

```yaml
tracker:
  kind: github                     # "github" or "scratch"
  api_key: "$GITHUB_TOKEN"         # env var reference

sandbox:
  image: symphony-agent:latest
  runtime: ""                      # ""=default, "runsc"=gVisor
  cap_drop: all
  network: bridge                  # "bridge" or "none"

stages:
  <name>:
    trigger: [ready-for-agent]     # issue labels that activate this stage
    action: dispatch               # dispatch | hook
    image: symphony-agent          # only for action: dispatch
    prompt: prompts/implement.md   # Jinja2 template path
    max_turns: 30
    timeout_ms: 3600000
    outcomes:
      complete:
        actions: [merge_pr]        # merge_pr | set_label:<label> | add_comment:<text>
        transition: done           # next stage name
      escalate:
        actions:
          - set_label: ready-for-human
        transition: released
      fail:
        retry: true
        max_retries: 3
}
```

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
