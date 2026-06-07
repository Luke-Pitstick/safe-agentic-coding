---
name: simplify-code
description: Analyze code complexity and identify simpler behavior-preserving ways to achieve the same goal, including whether existing libraries or framework features can replace custom code. Use when the user asks to reduce complexity, simplify implementation, review overengineering, find unnecessary abstractions, improve maintainability, lower cognitive/cyclomatic complexity, replace custom code with a library, or refactor code without changing behavior.
---

# Simplify Code

## Overview

Use this skill to find where code is more complicated than the problem requires, then recommend or implement simpler equivalent designs. Prefer evidence from the codebase, tests, call sites, and runtime behavior over broad style opinions.

## Operating Modes

- **Analysis mode**: Default when the user asks for a complexity review, audit, or recommendations. Produce ranked findings with file/line references and concrete simplification options.
- **Implementation mode**: Use when the user asks to simplify or refactor the code, or when the requested outcome clearly requires edits. Make narrowly scoped behavior-preserving changes and validate them.

## Workflow

1. **Establish the target**
   - Determine whether the user wants a whole-repo scan, a specific file/module, a diff review, or implementation.
   - If the scope is broad, sample from high-risk areas first: core workflows, recently changed files, large modules, complex conditionals, shared abstractions, and failing or flaky areas.

2. **Map behavior before judging structure**
   - Identify the user-visible behavior, public API, data flow, side effects, invariants, and tests.
   - Read call sites before changing abstractions.
   - Treat behavior preservation as the primary constraint.

3. **Find complexity hotspots**
   - Look for deeply nested conditionals, many flags/options, long functions, high fan-in/fan-out, duplicated branches, state machines without explicit states, clever generic utilities, premature extension points, indirection layers, and code paths only used once.
   - Use available tooling when present, such as coverage, type checks, linters, complexity metrics, dependency graphs, or test output.
   - Load [complexity-smells.md](references/complexity-smells.md) for a checklist.

4. **Classify the complexity**
   - **Essential complexity**: required by domain rules, platform constraints, performance, security, compatibility, or user behavior.
   - **Accidental complexity**: introduced by implementation choices, unclear boundaries, duplication, over-abstraction, premature generalization, or defensive code without a real case.
   - Do not recommend removing essential complexity unless you also preserve the real constraint another way.

5. **Design simpler equivalents**
   - Prefer local simplifications before architectural rewrites.
   - Collapse one-off abstractions, inline trivial wrappers, replace flag-heavy APIs with explicit flows, extract named helpers for repeated concepts, normalize data at boundaries, and remove dead or unreachable branches.
   - When custom code handles a common problem, invoke `$deep-dive` to find existing libraries, framework APIs, or standard-library features that can achieve the same goal with less maintenance burden.
   - Compare library adoption against keeping local code: dependency weight, maturity, security posture, API fit, license, integration cost, bundle/runtime impact, and testability.
   - Load [simplification-patterns.md](references/simplification-patterns.md) for options and tradeoffs.

6. **Choose build-vs-adopt deliberately**
   - Recommend a library only when it materially reduces complexity after integration costs are included.
   - Prefer built-in framework or standard-library capabilities over adding a dependency.
   - Keep custom code when the local behavior is small, domain-specific, security-sensitive, performance-critical, or easier to verify than a dependency.
   - If adoption is promising but uncertain, output a narrow spike task with acceptance criteria instead of forcing the dependency into the main refactor.

7. **Validate equivalence**
   - Find or add focused tests for changed behavior when implementation mode is active.
   - Run the smallest meaningful validation first, then broaden when the touched surface is shared.
   - Compare before/after behavior for edge cases, public APIs, generated outputs, and error handling.

8. **Report with judgment**
   - Rank findings by payoff and risk.
   - Include exact file/line references when possible.
   - For each recommendation, state the current complexity, why it is unnecessary or risky, the simpler approach, whether `$deep-dive` found viable existing libraries or APIs, expected benefit, validation needed, and migration risk.
   - Avoid vague advice like "make it cleaner" without an actionable equivalent.

## Implementation Guardrails

- Keep refactors small enough to review.
- Preserve public contracts unless the user explicitly asks for API changes.
- Do not remove duplication if shared abstraction would make the behavior harder to read or test.
- Do not add a new abstraction just to make code look tidy.
- Avoid broad rewrites when a local edit achieves the same goal.
- If tests are missing, add focused characterization tests before non-trivial behavior-preserving changes when feasible.

## Output Shape

For analysis mode:

```markdown
## Complexity Findings

1. [Impact: High | Risk: Medium] Title
   Location: path/to/file.ext:line
   Current complexity: ...
   Simpler equivalent: ...
   Existing library/API option: ...
   Why behavior stays the same: ...
   Validation: ...

## Quick Wins
...

## Leave As-Is
...
```

For implementation mode, summarize:

- Simplifications made.
- Libraries, framework APIs, or standard-library features adopted or rejected.
- Behavior preserved and how it was validated.
- Tests or checks run.
- Remaining complexity intentionally left alone.
