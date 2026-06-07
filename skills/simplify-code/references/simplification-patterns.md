# Simplification Patterns

Choose the smallest pattern that preserves behavior and improves readability or maintainability.

## Local Simplifications

- Replace nested conditionals with early returns when it clarifies the primary path.
- Merge branches that perform the same work with different constants.
- Extract a named helper only when it captures a real repeated concept.
- Inline trivial wrappers when the wrapper adds no contract, validation, logging, or domain meaning.
- Replace boolean flag arguments with explicit functions when callers represent distinct workflows.

## Boundary Simplifications

- Normalize input once at the boundary instead of repeatedly checking shape downstream.
- Convert implicit optional-state objects into explicit states when behavior depends on state.
- Keep validation close to the boundary and business rules close to the domain operation.
- Use a simple data structure before introducing classes, registries, factories, or plugin systems.

## Abstraction Simplifications

- Delete one-off interfaces or base classes when there is only one concrete implementation and no real extension point.
- Prefer direct calls over service locators or generic dispatch when the dependency is stable.
- Collapse pass-through modules if they do not enforce a boundary.
- Keep abstractions that protect a real external dependency, expensive setup, security boundary, or API contract.

## Library and Framework Adoption

- Use `$deep-dive` when custom code implements common infrastructure, such as date/time handling, schema validation, routing, state machines, auth helpers, parsing, serialization, retries, queues, charts, drag-and-drop, rich text, file uploads, search, or testing utilities.
- Prefer standard-library or existing framework APIs before adding a new package.
- Compare candidate libraries by maturity, maintenance activity, security history, license, API fit, dependency graph, bundle/runtime cost, docs quality, ecosystem fit, and ease of testing.
- Reject a library when the custom implementation is small and clear, the dependency surface is larger than the problem, the license is incompatible, or adoption would make behavior harder to reason about.
- For high-risk dependency changes, create a spike: install or prototype in the smallest area, preserve current behavior with characterization tests, and decide based on measured integration cost.

## Test Simplifications

- Prefer characterization tests before refactoring complex legacy behavior.
- Test observable behavior, not private helper choreography.
- Replace repeated setup with a small fixture builder only when it reduces noise without hiding important inputs.
- Remove test mocks that only assert implementation details, but keep mocks for true external boundaries.

## Reporting Tradeoffs

For each proposed simplification, include:

- What gets simpler.
- What risk or constraint remains.
- What test or check proves equivalence.
- What not to simplify because it is essential complexity.
