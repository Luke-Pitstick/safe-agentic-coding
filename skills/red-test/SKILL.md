---
name: red-test
description: Write one highly scoped failing test before implementation in a test-driven development workflow. Use when the user asks for the red step, asks to write the first failing test, asks to turn a selected behavior into a concrete test, or wants one small test that proves desired behavior is currently missing.
---

# Red Test

## Overview

Write exactly one focused test that captures a desired behavior and fails for the right reason before implementation begins. Prefer the smallest public behavior boundary that makes the missing capability observable.

## Operating Rules

- Write one test by default. Add helper fixtures only when the test would otherwise be unclear or brittle.
- Do not implement production behavior in this skill.
- Do not broaden scope to cover adjacent behavior, cleanup, or refactoring.
- Prefer existing test conventions: framework, file layout, naming, fixtures, builders, assertion style, and command.
- Make the failing reason meaningful. A red test should fail because the behavior is missing, not because of syntax errors, missing imports that should exist, broken setup, or an unrealistic assertion.
- If the desired behavior is still abstract, first reduce it to a Given/When/Then/Assert shape. Use `$derive-tests` when the user needs multiple candidate tests before choosing one.

## Workflow

### Step 1: Select One Behavior

Identify the one behavior under test:

- Given: the smallest setup.
- When: the single action or trigger.
- Then: the observable outcome.
- Assert: the concrete value, state, event, output, rendered element, exception, file content, or call count.

If multiple behaviors are bundled together, pick the smallest valuable one and state what is left for later red tests.

### Step 2: Find the Test Location

Inspect the repo before writing:

- Locate nearby production code or the intended public boundary.
- Locate existing tests for similar code.
- Mirror local test naming, fixtures, and assertion patterns.
- If no test file exists for the boundary, create the narrowest conventional one.

Prefer public APIs, routes, commands, components, service methods, or instruction artifacts over private helper details.

### Step 3: Write the Red Test

Edit only the test file and any tiny test-only fixture needed for the red test.

The test should:

- Have a name that describes the behavior, not the implementation.
- Arrange the minimum state needed.
- Act once.
- Assert one primary outcome.
- Avoid snapshots unless snapshots are the local convention and the expected diff is small.
- Avoid sleeps, randomness, real network calls, and external services. Use fakes, seeds, fake clocks, in-memory adapters, or existing test utilities.

For instruction artifacts such as Codex skills, early red tests may assert file existence, required headings, required contract language, or golden-output behavior through a fake harness.

### Step 4: Prove It Is Red

Run the narrowest relevant test command:

- Prefer a single-test invocation when the framework supports it.
- If the command is unknown, infer it from package scripts, existing docs, or nearby CI config.
- Capture the failing assertion and confirm it matches the desired missing behavior.

If the failure is unrelated to the behavior, fix the test setup until the red failure is meaningful. Do not implement production behavior to make it pass.

### Step 5: Handoff

Report:

- Test file path.
- Test name.
- Command run.
- Failure observed.
- Why the failure is the right red failure.
- The minimal behavior the green step should implement next.

## Quality Bar

Before finalizing, check:

- The diff contains no production implementation for the behavior.
- The test fails for the desired missing behavior.
- The test is small enough that `$green-implement` can make it pass with a minimal change.
- The test does not lock in unnecessary internals.
- Any unrelated failures or setup gaps are called out clearly.
