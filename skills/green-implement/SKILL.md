---
name: green-implement
description: Write the smallest production implementation needed to make a specific red test pass in a test-driven development workflow. Use after a focused failing test exists and the user asks for the green step, minimal implementation, or enough code to pass the current test without refactoring.
---

# Green Implement

## Overview

Make one red test pass with the least production code that honestly satisfies the tested behavior. Preserve the scope of the red test; do not refactor or generalize beyond what the current test demands.

## Operating Rules

- Start from a specific failing test. If no red test exists, stop and ask for one or use `$red-test` first.
- Change production code only as much as needed to pass the current red test.
- Keep test changes limited to fixing mistakes in the test itself, not weakening the assertion.
- Do not refactor while going green unless the code cannot compile or run without a tiny structural move.
- Prefer existing project APIs and patterns over new abstractions.
- Avoid speculative support for future cases. Let future red tests pull that behavior into existence.

## Workflow

### Step 1: Reproduce the Red Test

Run or inspect the failing test before editing:

- Confirm the test name and file path.
- Confirm the failure message.
- Confirm the failure represents missing behavior, not invalid setup.

If the red test is not failing, do not implement blindly. Explain whether it is already green, blocked, or failing for the wrong reason.

### Step 2: Locate the Smallest Production Boundary

Find the production code that should own the behavior:

- Use the public API, route, command, component, service, parser, or instruction file under test.
- Read nearby implementation patterns.
- Avoid adding private helpers unless the smallest honest change needs them.

### Step 3: Implement the Minimum

Make the smallest honest change:

- Prefer direct, readable code over premature abstraction.
- Add only data fields, branches, functions, or wiring required by the red test.
- Keep compatibility with existing tests.
- Do not broaden behavior unless the existing public contract already requires it.
- Leave ugly-but-contained code alone until `$refactor-implementation` runs with green tests.

### Step 4: Prove Green

Run:

- The focused red-test command until it passes.
- Any nearby tests likely affected by the production change.

If a broader suite is cheap and conventional, run it. If not, state what was and was not run.

### Step 5: Handoff

Report:

- Production files changed.
- Test command(s) run.
- The red test now passing.
- Any nearby tests run.
- The next obvious red test, if one emerged.
- Any refactor candidates to consider only after tests are green.

## Quality Bar

Before finalizing, check:

- The targeted red test passes.
- The implementation is no larger than the test requires.
- No assertion was weakened to get green.
- No broad refactor, cleanup, or future-proofing snuck into the green step.
- Any remaining risk is visible and tied to untested behavior.
