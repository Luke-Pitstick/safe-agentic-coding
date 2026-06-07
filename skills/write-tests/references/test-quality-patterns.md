# Test Quality Patterns

Use this reference when deciding what to test and how to avoid brittle or low-value tests.

## Strong Test Shape

Good tests usually have:

- A behavior-focused name.
- Minimal setup that describes the scenario.
- One meaningful action.
- Assertions on externally visible results.
- Cleanup or isolation for shared state.

Prefer:

```text
given expired token, refresh returns unauthorized and does not write a session
```

Avoid:

```text
calls validateToken
```

unless calling `validateToken` is the public contract.

## Assertion Quality

Strong assertions check:

- Return values
- Rendered text or accessible roles
- HTTP status, headers, body, and persisted side effects
- Database rows or emitted events
- Error type and message
- Files written or CLI output
- Metrics/logs only when they are part of the contract

Weak assertions:

- Only check that no exception was thrown.
- Only check that a mock was called.
- Assert broad snapshots no human will review.
- Mirror the implementation line by line.

## Mocking Guidance

Mock:

- Network calls to external services.
- Time, randomness, and IDs.
- Slow queues, email/SMS providers, payment providers, and analytics.
- Framework boundaries when testing pure domain logic.

Prefer real:

- Pure helpers and validators.
- In-memory stores or test databases when persistence behavior matters.
- Serialization and schema parsing.
- Component children when their behavior is relevant to the user flow.

Do not mock the subject under test.

## Regression Tests

For bug fixes:

1. Encode the failing scenario as narrowly as possible.
2. Include the input that triggered the bug.
3. Assert the corrected behavior and the absence of the bad side effect.
4. Keep the test name tied to behavior, not the issue number only.

## Integration Tests

Use integration tests when confidence depends on:

- Wiring between modules.
- Database constraints, transactions, or migrations.
- API request/response behavior.
- Auth, permissions, or middleware.
- Serialization, schema compatibility, or generated clients.

Keep integration tests fewer but higher value. Reuse fixtures and reset state reliably.

## UI and End-to-End Tests

Assert user-visible behavior:

- Accessible roles and names.
- Navigation, form submission, validation, and persistence.
- Loading and error states.
- Critical path journeys.

Avoid tests that depend on exact CSS class names, animation timing, or incidental DOM nesting.

## Flake Control

Control:

- Time with fake timers or injectable clocks.
- Randomness with seeded generators or fixed IDs.
- Async work with framework wait helpers.
- Network with explicit mocks or test servers.
- Filesystem with temp directories.
- Database with transactions, truncation, or isolated schemas.

Avoid arbitrary sleeps. If a wait is unavoidable, explain why and keep it bounded.

## Coverage Balance

High-value coverage includes:

- Important branches and edge cases.
- Failure paths users can hit.
- Regressions for recently fixed bugs.
- Public APIs and compatibility promises.
- Risky refactors and shared utilities.

Low-value coverage includes:

- Trivial getters/setters.
- Framework boilerplate.
- Private helpers already covered through public behavior.
- Tests that require more maintenance than the behavior is worth.
