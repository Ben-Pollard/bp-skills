# Refactor Candidates

After TDD cycle, review the entire module you changed — not just your diff.
The code you touched reveals friction in the code around it; ignore that friction and it compounds.

Look for:

- **Duplication** → Extract function/class. Same 2+ field assignments across 3+ sites is duplication.
- **Long methods** → Break into private helpers (keep tests on public interface)
- **Shallow modules** → Dataclasses with zero methods, only external mutation. Data and its behavior should co-locate.
- **Feature envy** → Logic that mutates another module's data belongs on that data's class. If >2 functions outside a dataclass's owning module mutate its fields, extract those mutations as methods on the dataclass.
- **SRP: data without behavior** → A class with only public fields and no methods has no single responsibility — it exposes its internals to everyone. Ask: what transitions are legal? What invariants must hold?
- **DRY: repeated mutation** → `obj.field = value` appearing identically across 3+ functions means the mutation should be a named method.
- **Primitive obsession** → Introduce value objects
- **Existing code** the new code reveals as problematic
- **Repeated test variants** → Parametrise. Multiple test functions that differ only by input value are a smell.

```python
# BAD: three functions, one per status value
def test_parses_ready_for_agent():
def test_parses_in_progress():
def test_parses_done():

# GOOD: one parametrised test
@pytest.mark.parametrize("status", ["ready-for-agent", "in-progress", "done"])
def test_parses_known_status(status):
```
