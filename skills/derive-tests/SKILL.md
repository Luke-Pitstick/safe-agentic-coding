---
name: derive-tests
description: Derive the first small, concrete, failing tests for test-driven development before implementation begins. Use when the user asks to implement a feature with TDD, asks what tests to write first, asks to decompose a feature into behavior tests, asks for Given/When/Then test cases, or gives prompts like "I want to implement [feature]. Help me break this into small testable behaviors. Do not write implementation code yet."
---

# Derive Tests

## Overview

Turn a feature idea into the first small tests the user can write before implementation exists. Focus on desired behavior, observable assertions, test placement, and the simplest red-test path; do not design implementation internals.

## Operating Rules

- Do not write implementation code unless the user explicitly changes scope after the plan.
- Prefer the first 5-10 concrete tests. Use fewer when the feature is tiny; use more only when the user asks for a broader test map.
- Make each test small enough to fail for one clear reason: one rule, state transition, validation, contract clause, or user-observable outcome.
- Convert every behavior idea into a concrete test case with Given, When, Then, and the key assertion.
- Define vague outcomes in observable terms. Replace "runs exactly one iteration" with checks such as "one progress update exists", "completed_iterations is 1", and "iteration 2 is not started".
- Phrase tests around what the system must guarantee, even before code exists. TDD tests desired behavior; they do not document the current implementation.
- Start with boring foundations before edge cases: initial state, simplest success path, one state transition, boundary or invariant, then modifiers or failures.
- Avoid compound behaviors joined by "and". Split them unless they truly form one observable rule.
- Include where tests should go and how they should be made, using the repository's existing test structure when available.
- If a repo is available, inspect relevant files before choosing test locations. Prefer codebase-memory graph tools when configured; fall back to `rg` for test directories, naming conventions, and framework config.
- If no repo context is available, state assumptions and give conventional test-location options by stack.

## Workflow

### Step 1: Frame the Feature

Summarize the feature in 1-3 bullets:

- The observable capability the user wants.
- The likely unit, component, integration, or end-to-end boundary to test.
- Any assumptions about domain rules, test framework, or project structure.

Ask one concise question only when the missing answer changes the first tests materially, such as unclear feature scope, unknown public API, or ambiguous domain rules.

### Step 2: Find the Test Home

When working inside a repository:

- Identify the likely production module, public API, route, command, component, or service boundary.
- Identify existing tests near that boundary and mirror their naming, fixture style, framework, and assertion style.
- Recommend exact test file paths. If the target file does not exist yet, say whether to create it.
- Prefer testing the public behavior boundary over private helper details.

When no repository is available:

- Recommend a conventional test location, such as `tests/<feature>.test.ts`, `tests/test_<feature>.py`, `<module>.test.tsx`, or `src/test/...`, and label it as assumption-based.

### Step 3: Derive Concrete Tests

Produce 5-10 test cases in this style:

```markdown
1. `[test_name]`
   - Given: [small setup]
   - When: [single trigger]
   - Then: [observable result]
   - Assert: [specific value, text, state, call count, file existence, stop reason, emitted event, rendered element, etc.]
```

Good test cases:

- `test_new_cell_starts_unburned`
  - Given: a newly created cell
  - When: its state is inspected
  - Then: the cell is unburned
  - Assert: `cell.state == "unburned"`
- `test_ignition_point_burns_on_tick_zero`
  - Given: a grid with one ignition point
  - When: tick 0 is applied
  - Then: the ignition cell is burning
  - Assert: the ignition coordinate has state `burning`
- `test_stop_iteration_one_does_not_start_second_iteration`
  - Given: a task with `stop_iteration = 1`
  - When: the loop runs
  - Then: it emits one progress update and stops at iteration 1
  - Assert: one progress update exists, stop reason is iteration limit, and iteration 2 was not started

Weak outputs to rewrite:

- Implement the fire grid.
- Test fire spread logic and wind.
- Make sure everything works.
- Use a map of cells to store state.
- The loop runs exactly one iteration.

### Step 4: Add Test-Making Guidance

After the concrete tests, add a concise "Test Plan" section:

- `Where`: exact file path(s), or assumption-based path(s) if no repo is available.
- `Style`: unit, component, integration, contract, or end-to-end, with a one-sentence reason.
- `Shape`: what each test should arrange, act on, and assert. Prefer Given/When/Then plus a named assertion.
- `Fixtures`: smallest useful fixtures or builders, avoiding elaborate setup.
- `Order`: the first 1-3 tests to write, chosen to create momentum and avoid overfitting.

If the feature probably needs randomness, time, network, concurrency, or external services, include how tests should control that source of nondeterminism, such as seeding randomness, using a fake clock, stubbing network boundaries, or using an in-memory adapter.

If the target is a Codex skill or other instruction artifact, start simpler than a full harness:

- Level 1: file existence tests.
- Level 2: instruction contract tests against required text or sections.
- Level 3: parser or fake-harness tests against the contract.
- Level 4: simulated behavior tests with fake outcomes.
- Level 5: actual agent-visible behavior tests.

For early skill TDD, recommend concrete tests like `test_skill_file_exists`, `test_skill_defines_required_loop_steps`, or `test_skill_requires_progress_update_per_iteration` before recommending a full fake harness.

### Step 5: Stop Before Implementation

End with a brief next step that identifies the first failing test to write. Do not include implementation code. Include test names, Given/When/Then descriptions, and assertion intent; include real test code only if the user explicitly asks for test skeletons.

## Output Shape

Use this structure by default:

```markdown
# Derive Tests: <Feature>

## Feature Frame
- ...

## First Tests to Write
1. ...

## Test Plan
- Where: ...
- Style: ...
- Shape: ...
- Fixtures: ...
- Order: ...

## Next Step
Start with [test name], because [short reason].
```

## Quality Bar

Before finalizing, check:

- Each test is understandable to someone who has not seen the implementation.
- Each behavior idea has been converted into a concrete test case with Given, When, Then, and Assert.
- Each test can plausibly fail for one focused reason.
- The first test is small enough to fail before implementation exists.
- Test locations match the repo's current conventions when repo context exists.
- The plan names test style, fixtures, and ordering.
- The response contains no implementation code.
