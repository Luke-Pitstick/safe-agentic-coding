---
name: refactor-implementation
description: Refactor implementation code after tests are green while preserving behavior in a test-driven development workflow. Use after a red test has been made green and the user asks for the refactor step, cleanup, simplification, improved design, or removal of duplication without changing behavior.
---

# Refactor Implementation

## Overview

Improve code shape only after the relevant tests are green. Preserve observable behavior, keep the refactor small, and use the existing tests as the safety rail.

## Operating Rules

- Start by confirming the relevant tests are green or have recently passed.
- Do not add new behavior during refactor. If new behavior is needed, stop and create a new red test instead.
- Keep each refactor small enough to explain in one sentence.
- Prefer simplification, clearer names, reduced duplication, smaller functions, better boundaries, or removal of test-induced awkwardness.
- Preserve public APIs unless the user explicitly asked for a breaking cleanup.
- Do not rewrite working code for style preference alone.
- If tests are weak for the code being refactored, add or request a red test before making risky changes.

## Workflow

### Step 1: Establish the Green Baseline

Run or cite the relevant passing tests:

- The test that drove the implementation.
- Nearby tests for affected behavior.
- Broader suite only when cheap and appropriate.

If tests are failing, do not refactor. Return to `$green-implement` or diagnose the failing test first.

### Step 2: Choose One Refactor Goal

Pick the smallest useful cleanup:

- Remove duplication.
- Clarify names.
- Extract or inline a helper.
- Simplify conditionals.
- Move code to the boundary that owns it.
- Replace brittle setup with an existing fixture.
- Improve test readability without weakening assertions.

State the behavior that must remain unchanged.

### Step 3: Refactor in Small Moves

Edit conservatively:

- Keep behavior-preserving transformations mechanical where possible.
- Avoid mixing formatting churn with semantic cleanup.
- Keep tests green after each meaningful move when the refactor is non-trivial.
- Use existing formatters or linters only when they are standard for the repo and do not create broad unrelated diffs.

### Step 4: Prove Behavior Is Preserved

Run the same tests used for the green baseline:

- The targeted test.
- Nearby tests for affected code.
- Any cheap project-level check that is customary.

If a test fails after refactor, either fix the refactor without changing behavior or revert the refactor portion. Do not change expected behavior during this skill.

### Step 5: Handoff

Report:

- Refactor goal.
- Files changed.
- Behavior preserved.
- Tests run before and after, when available.
- Any cleanup deliberately deferred to keep the step small.
- The next red test to consider if new behavior is desired.

## Quality Bar

Before finalizing, check:

- Tests were green before refactor or the baseline status is explicitly documented.
- Tests are green after refactor.
- No new behavior was added.
- Public behavior and assertions are unchanged.
- The diff is smaller or clearer than the code it replaced.
- Any unrun tests or residual risk are stated plainly.
