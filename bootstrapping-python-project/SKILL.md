---
name: bootstrapping-python-project
description: Use when starting a new Python project or repo, before writing implementation code, to scaffold a standardised project structure with uv, ruff, pytest, pre-commit, and VS Code settings.
---

# Bootstrapping Python Project

## Overview

Scaffold a Python project with standard tooling in one pass. Creates `pyproject.toml`, `.pre-commit-config.yaml`, `.vscode/settings.json`, `.gitignore`, `README.md`, `CONTEXT.md`, `AGENTS.md`, empty `src/<pkg>`, `tests/unit`, `tests/integration`.

## When to Use

- Starting a new Python project or repo
- Before any implementation code
- When the project needs uv, ruff, pytest, pre-commit, and VS Code defaults

NOT for: adding tooling to an existing project, setting up CI/CD, Docker config.

## Quick Reference

| File | Contents |
|------|----------|
| `pyproject.toml` | [project], [dependency-groups] (dev), [build-system], [tool.ruff] (format + lint + isort), [tool.pytest.ini_options] |
| `.pre-commit-config.yaml` | ruff check, ruff format --check, pytest tests/unit/ |
| `.vscode/settings.json` | formatOnSave, ruff linter/formatter/organizeImports, pytest, extraPaths |
| `.gitignore` | Python std + state.json |
| `README.md` | Style guide + project name + one-liner |
| `CONTEXT.md` | Style guide only (populated by grilling) |
| `AGENTS.md` | uv instructions + CONTEXT pointer |

## Implementation

### 1. Gather project info

Ask the user these questions. ONE AT A TIME. Default options in brackets.

1. Project name? (kebab-case, e.g. `my-project`)
2. Package name? (underscore, e.g. `my_project`. defaults from project name.)
3. Python version? [3.11]
4. Package layout? [src] → `src/<pkg>/` or [flat] → `<pkg>/`
5. Short description? (one line for pyproject.toml + README)
6. Author name?

### 2. Write the files

Use the answers to generate each file. Templates below.

#### `pyproject.toml`

```toml
[project]
name = "{{ project_name }}"
version = "0.1.0"
description = "{{ description }}"
readme = "README.md"
requires-python = ">={{ python_version }}"
authors = [{ name = "{{ author }}" }]

[dependency-groups]
dev = [
    "ruff>=0.8.0",
    "pytest>=8.0",
    "pre-commit>=4.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["{% if layout == 'src' %}src/{% endif %}{{ package_name }}"]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
line-ending = "lf"

[tool.ruff.lint]
select = ["F", "E", "W", "I"]
ignore = ["E501", "F811", "F841"]

[tool.ruff.lint.isort]
known-first-party = ["{{ package_name }}"]

[tool.pytest.ini_options]
testpaths = ["tests"]
```

#### `.pre-commit-config.yaml`

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.8.0
    hooks:
      - id: ruff
        args: [check]
      - id: ruff-format
  - repo: local
    hooks:
      - id: pytest-unit
        name: unit tests
        entry: uv run pytest tests/unit
        language: system
        pass_filenames: false
        types: [python]
```

#### `.vscode/settings.json`

```json
{
  "[python]": {
    "editor.formatOnSave": true,
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit"
    }
  },
  "python.analysis.extraPaths": ["src"]{% if layout == "flat" %}<!-- omit for flat layout -->{% endif %},
  "python.testing.pytestEnabled": true,
  "python.defaultInterpreterPath": ".venv/bin/python"
}
```

#### `.gitignore`

```
__pycache__/
*.pyc
.venv/
.ruff_cache/
.pytest_cache/
dist/
*.egg-info/
state.json
```

#### `README.md`

```markdown
<!-- Style: headings tight, bullets terse, single lines. No multi-clause sentences. No filler words. -->

# {{ project_name }}

{{ description }}
```

#### `CONTEXT.md`

```markdown
<!-- Style: headings tight, bullets terse, single lines. No multi-clause sentences. No filler words. -->
```

#### `AGENTS.md`

```markdown
<!-- Style: headings tight, bullets terse, single lines. No multi-clause sentences. No filler words. -->

## Getting Started

- Use `uv sync` to install dependencies.
- Use `uv run <command>` to run Python commands (e.g. `uv run pytest`).
- Dev-only deps (ruff, pytest, pre-commit) live in the `dev` dependency group.

## Context

Read `CONTEXT.md` for the domain glossary and terminology conventions.
```

### 3. Create directory structure

```
mkdir
  src/{{ package_name }}/  (or {{ package_name }}/ for flat)
  tests/unit/
  tests/integration/
  .vscode/

touch
  src/{{ package_name }}/__init__.py
  tests/__init__.py
  tests/unit/__init__.py
  tests/integration/__init__.py
  tests/conftest.py          (empty)
```

### 4. Install and verify

```bash
uv sync
uv run ruff check .
uv run ruff format --check .
uv run pytest
```

If ruff `check` fails on empty src directory, confirm it's only the "no Python files" warning — that's expected.

### 5. Set up pre-commit

```bash
uv run pre-commit install
```

## Common Mistakes

- Forgetting `__init__.py` in test dirs → pytest discovery fails.
- FS/D rule (flake8-docstrings) — not included. Do NOT add unless user asks.
- `python.analysis.extraPaths` for flat layout — omit, don't point to root.
- `state.json` in gitignore needed for Symphony projects. Remove if not a Symphony project.
- `ruff format --check` vs `ruff format` — the hook checks, doesn't mutate. Users format manually or via editor on-save.
