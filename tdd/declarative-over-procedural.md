# Declarative Over Procedural

**Procedural code** tells the machine *how* — step by step. It is the
agent's natural output, but it is also a cost: it must be read, tested,
maintained, and debugged. Reducing code improves a codebase more than
adding to it.

**Declarative code** expresses *what* you want. It reads like its intent.
It is often already written for you — in stdlib, in this project's
dependencies, or elsewhere in this codebase. Where it isn't, adding a
dependency that makes the problem declarative may be cheaper than writing
procedural code.

## The principle

Minimise procedural code. Whether writing or reviewing, check each piece
against three questions:

1. **Already solved.** Does stdlib, an installed dependency, or another
   module in this project already express this intent? Read the docs to
   find out. Documentation reveals solved problems.

2. **Solvable by adding or changing a dependency.** Would a library express this in
   a fraction of the lines? Would switching to a different libary cover more use cases? Weigh the dependency cost against the
   maintenance cost of custom code. Many problems are cheaper to depend on
   than to implement.

3. **Unnecessary.** Is this code adding value, or is it a thin wrapper that
   just renames an existing call? Pass-through abstractions are cost
   without benefit. Prefer removing code over keeping it.

## Why

Procedural glue is error-prone, brittle, and duplicates work. The best line
of code is the one you delete or don't write in the first place. The second best is the one someone else
already wrote, tested, and documented.
