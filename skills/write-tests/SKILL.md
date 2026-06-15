---
name: write-tests
description: Add, improve, debug, and validate unit tests, integration tests, end-to-end tests, regression tests, contract tests, and test fixtures for code changes. Use when the user asks Codex to write tests, increase coverage, protect a bug fix, test an API/component/module/workflow, add regression coverage, repair flaky tests, design a testing strategy, or verify code behavior with automated tests.
---

# Write Tests

## Overview

Write tests that protect behavior users and maintainers care about. Prefer the repository's existing test framework, conventions, helpers, and command structure over introducing new tools.

## Optional GStack and GBrain Compatibility

Use GStack and gbrain as optional memory, never as a required dependency.

Before writing tests, if `gbrain` is on PATH:

- Extract 2-4 concrete keywords from the code under test, bug name, feature area, or test framework.
- Run `gbrain search "<keywords>"`.
- Read at most the top 3 clearly relevant pages with `gbrain get_page "<slug>"`.
- Use memory only for known test commands, previous flakes, fixture conventions, bug regressions, or coverage gaps.
- If `gbrain` is unavailable, returns an error, or has no useful hits, continue from local code and tests.

After adding or improving meaningful tests, save a compact testing summary when `gbrain` is available:

```bash
gbrain put "safe-agentic/reviews/<test-slug>" --content "<markdown summary>"
```

The saved summary should include behaviors covered, files changed, commands run, results, and residual risk. Do not save secrets, credentials, raw user payloads, private keys, sensitive PII, or large test logs. The local tests and review artifacts remain the source of truth.

## Operating Rules

- Inspect the code under test and nearby tests before writing new tests.
- Test observable behavior, contracts, edge cases, and regressions instead of private implementation details.
- Choose the cheapest test level that proves the behavior: unit before integration, integration before end-to-end, unless the risk requires a higher level.
- Add a regression test for every bug fix when practical. Make the test fail on the old behavior before trusting it when the setup cost is reasonable.
- Keep tests deterministic. Control time, randomness, network, filesystem, concurrency, and external services.
- Use real dependencies when they are cheap and stable. Mock only boundaries that are slow, nondeterministic, expensive, or outside the unit of responsibility.
- Do not weaken production code or assertions to make tests pass.
- Do not skip, quarantine, or delete failing tests unless the user explicitly asks or the test is demonstrably invalid and replaced with stronger coverage.
- Run the narrowest relevant test first, then a broader command when the risk or repo size warrants it.

## Workflow

### Step 1: Discover the Test Surface

Read the smallest useful set of artifacts:

- Code under test and public entry points.
- Existing tests near the target code.
- Test helpers, fixtures, factories, mocks, and setup files.
- Package/build configuration for test commands.
- Recent bug report, issue, diff, or stack trace when available.

Use `rg --files` and `rg` to find tests quickly. Read `references/framework-commands.md` when the framework or command is not obvious.

### Step 2: Define the Behavior Contract

Before editing, write down the behavior to protect:

- Normal path
- Boundary values
- Error handling
- State transitions
- Side effects
- Authorization, validation, or persistence rules
- Regression condition, if this is a bug fix

If the expected behavior is ambiguous and multiple outcomes are plausible, ask one concise question. Otherwise state the assumption and proceed.

### Step 3: Select Test Types

Use this selection order:

1. Unit test for pure logic, branching, formatting, validation, reducers, utilities, and local component behavior.
2. Integration test for database behavior, service coordination, API handlers, framework routing, serialization, or multi-module workflows.
3. Contract test for boundaries between services, packages, public APIs, schemas, CLIs, events, or generated clients.
4. End-to-end test for critical user journeys where confidence requires the full stack.
5. Snapshot or golden test only when the output is intentionally stable and reviewable.
6. Property or fuzz test when the input space is large and invariants are clearer than examples.

Read `references/test-quality-patterns.md` for examples of strong assertions, fixtures, mocking, and flake control.

### Step 4: Write the Tests

Match existing style:

- Naming: follow local `describe`/`it`, `test_`, class, module, or file conventions.
- Placement: use the repo's established test directories and suffixes.
- Fixtures: reuse local factories/builders before inventing new setup.
- Assertions: assert specific outcomes and meaningful error messages.
- Cleanup: isolate state between tests.

Prefer Arrange-Act-Assert shape even if the framework does not name it. Keep each test focused on one behavior, but avoid tiny duplicate tests when a parameterized test is clearer.

### Step 5: Validate

Run tests in this order when possible:

1. The single new or changed test.
2. The containing test file or suite.
3. The package/module test command.
4. Broader project tests only if the change is shared, risky, or requested.

If a test fails:

- Read the failure, not just the summary.
- Decide whether the failure is a test bug, product bug, bad assumption, missing fixture, or environment issue.
- Fix the smallest correct thing.
- Rerun the relevant command.

If validation cannot run because dependencies or services are missing, report the exact blocker and the command that should be run once the environment is available.

### Step 6: Review Coverage Quality

Before finalizing, check:

- The new tests fail for the bug or behavior they are meant to protect.
- Assertions would fail if the important behavior regressed.
- Happy paths and meaningful failure paths are covered.
- Mocks do not duplicate the implementation or hide integration risk.
- Fixtures are minimal, named, and maintainable.
- Async tests await the right work and do not rely on arbitrary sleeps.
- The tests fit the repo's existing style.

## Output Format

When reporting back, include:

- Tests added or changed.
- Behaviors covered.
- Commands run and results.
- Any tests not run and why.
- Residual risk or follow-up coverage that would be useful.

## Anti-Patterns

- Do not add coverage-only tests that merely execute code without assertions.
- Do not assert against private call order unless call order is the contract.
- Do not overuse snapshots for complex UI or API output that changes often.
- Do not mock the exact function you are trying to test.
- Do not use sleeps for async readiness when the framework has polling, fake timers, or wait helpers.
- Do not add a new test framework unless the repository has no viable existing test path and the user accepts the dependency.
