# Refactor Candidates

After TDD cycle, look for:

- **Duplication** → Extract function/class
- **Long methods** → Break into private helpers (keep tests on public interface)
- **Shallow modules** → Combine or deepen
- **Feature envy** → Move logic to where data lives
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
