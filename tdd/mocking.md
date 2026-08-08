# When to Mock

This guide covers three test levels. Each level has different rules about what to mock, what to use instead, and what to do when a test won't pass. **Mocks exist to isolate unit tests — not to avoid fixing broken integrations.**

## Unit Tests

Scope: a single function or module, exercised through its public interface.

Mock at **system boundaries** only:

- External APIs (payment, email, etc.)
- Databases (sometimes — prefer a test DB)
- Time/randomness
- File system (sometimes)

Don't mock:

- Your own classes/modules
- Internal collaborators
- Anything you control

### Designing for Mockability

At system boundaries, design interfaces that are easy to mock:

**1. Use dependency injection**

Pass external dependencies in rather than creating them internally:

```typescript
// Easy to mock
function processPayment(order, paymentClient) {
  return paymentClient.charge(order.total);
}

// Hard to mock
function processPayment(order) {
  const client = new StripeClient(process.env.STRIPE_KEY);
  return client.charge(order.total);
}
```

**2. Prefer SDK-style interfaces over generic fetchers**

Create specific functions for each external operation instead of one generic function with conditional logic:

```typescript
// GOOD: Each function is independently mockable
const api = {
  getUser: (id) => fetch(`/users/${id}`),
  getOrders: (userId) => fetch(`/users/${userId}/orders`),
  createOrder: (data) => fetch('/orders', { method: 'POST', body: data }),
};

// BAD: Mocking requires conditional logic inside the mock
const api = {
  fetch: (endpoint, options) => fetch(endpoint, options),
};
```

The SDK approach means:
- Each mock returns one specific shape
- No conditional logic in test setup
- Easier to see which endpoints a test exercises
- Type safety per endpoint

### If a unit test won't pass

The test is testing something unreachable through the public interface. Redesign the interface — don't reach into internals or mock your own code.

## Integration Tests

Scope: multiple real modules wired together. Exercises the integration boundary — proves components communicate, not just that each works in isolation.

### What to mock

**No mocking of internal code.** Every module on the tested path runs for real.

For external dependencies:

| Dependency type | Approach |
|---|---|
| External HTTP APIs you don't own | Use VCR (record real HTTP once, replay cassettes deterministically). If a cassette doesn't exist, record it against the real service once, then commit the cassette. Use `new_episodes` mode to add new interactions. |
| Services you own (databases, message queues, caches) | Use Testcontainers. Use `docker compose` in test fixtures for multi-service setups. Real instances, spun up and torn down per test run. |
| Credentials / secrets | Use test-environment credentials. Never commit real credentials. Document prerequisites in the README. |

### If an integration test won't pass

**VCR cassette missing?** Record it against the real service. Don't mock the HTTP call — that turns it into a unit test that proves nothing about the integration.

**Test times out?** Investigate the root cause. Timeouts are often broken integrations, not slow tests. Check that the real service is reachable, that the request format is correct, that the response is what the code expects. A timeout means something isn't wired up — find it and fix it.

**Container won't start?** Fix the configuration. The service is broken or the test setup is wrong. Don't replace the container with a mock.

**Do not demote an integration test to a unit test by adding mocks.** That removes all confidence that the integration works.

Use VCR to reduce runtime only for genuinely long-running tests that already pass against the real service. VCR is a speed optimization, not an escape hatch for broken code.

## End-to-End Tests

Scope: the full system, all services, real everything. A human could perform the same steps and see the same results.

### What to mock

**No mocks, period.** Real services, real containers, real test-environment credentials. The system either works end-to-end or it doesn't.

### If an E2E test won't pass

The system is broken. Fix it. Skipping the test, adding a mock, or marking it xfail means the bug ships to users.

## Common Rules for All Levels

### Skips, timeouts, and xfails are bugs

- `pytest.skip` / `@pytest.mark.skip` → the test is not running. That code path has zero coverage.
- Timeouts → not a slow test. An integration that never completes. Investigate until you find the root cause.
- `@pytest.mark.xfail` → the test is known-broken. Quarantining a failure doesn't fix it. Un-xfail by fixing the code.
- `0 passed, 0 failed` → nothing was tested. A test suite that collects zero tests is a failing suite.

The only legitimate skip: a test that cannot run in the current environment (e.g., needs a physical device not available in CI). If you skip a test, document the reason and the environment it requires.

### Never mock to dodge a failure

If you find yourself adding a mock to an integration or E2E test, you are not fixing the test — you are demoting the test level. A mocked integration test is a unit test with a misleading name. Fix the integration instead.

### You will be evaluated by whether the live system works

The ultimate gate is whether a human following the README can observe the described behavior in the running system. Green tests that don't exercise real integrations are false confidence. If QA finds the live system broken despite a green test suite, you didn't test the right things at the right level.