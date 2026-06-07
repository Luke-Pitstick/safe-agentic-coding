# Complexity Smells

Use this checklist to identify accidental complexity. A smell is a prompt to investigate, not proof that the code is wrong.

## Control Flow

- Deeply nested `if`/`else`, `switch`, `match`, or exception handling.
- Boolean flags that create many behavior combinations.
- Repeated guard logic across multiple call sites.
- Hidden state transitions spread across unrelated functions.
- Branches that differ only by a small value or callback.

## Abstractions

- Interface or base class with one implementation.
- Generic helper whose callers all use the same concrete type.
- Wrapper functions that only rename another function without adding meaning.
- Layers that pass data through unchanged.
- Dependency injection where no test or runtime variation exists.

## Data Flow

- Repeated conversion between shapes or types.
- Large mutable objects passed through many functions.
- Optional fields used as implicit states.
- Validation repeated after data could have been normalized once.
- Parsing, authorization, business logic, and presentation mixed together.

## Module Shape

- Large files containing unrelated concerns.
- Utility modules with vague names and unrelated exports.
- Public exports that are only used internally.
- Circular dependencies or imports that force awkward initialization.
- Code that exists only to support hypothetical future cases.

## Tests and Error Handling

- Test setup larger than the behavior under test.
- Mock-heavy tests that mirror implementation details.
- Error branches that cannot be triggered.
- Catch-all error handling that hides simpler control flow.
- Retries, caches, queues, or concurrency primitives without a measured need.
