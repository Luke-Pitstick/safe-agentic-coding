# Agent Brief Template

Use this template for the Markdown artifact written by `$expand-task`.

```markdown
# <Task Title>

Generated: <YYYY-MM-DD>
Source prompt: <user's prompt or concise paraphrase>

## Narrow Scope

<One sentence describing the smallest useful implementation slice.>

## Grand-Scheme Fit

- User value: <why this matters to the user or operator>
- App area: <feature, workflow, route, module, service, or subsystem>
- Strategic fit: <how this supports the broader product/technical direction>
- Non-goal: <important thing this task will not solve>

## Current Context

- Relevant files or directories inspected:
  - `<path>`: <why it matters>
- Existing patterns to preserve:
  - <pattern>
- Assumptions:
  - <assumption>

## Technical Implementation Plan

1. Inspect <specific file/module/route/service>.
2. Change <specific behavior or interface>.
3. Follow <existing pattern/helper/style/test convention>.
4. Add or update <tests/docs/fixtures/config if applicable>.
5. Validate with <specific command, manual flow, screenshot, or check>.

## Delegate Agent Instructions

You may edit files within the scoped implementation area. Do not broaden scope without recording a follow-up task.

Owned scope:
- <files/directories/modules or "to be discovered from listed context">

Instructions:
1. Reconstruct context from the files listed above.
2. Implement only the narrow scope.
3. Preserve existing conventions.
4. Add validation or tests appropriate to the change.
5. Commit the completed, validated work if the parent prompt grants commit permission.

## Acceptance Criteria

- [ ] <observable behavior or artifact>
- [ ] <technical correctness condition>
- [ ] <validation command/check passes>

## Validation Plan

- Automated: `<command or "not identified yet">`
- Manual: <manual flow or "not needed">
- Review gate: <suggested skill such as `$review`, `$qa`, `$design-review`, or "none">

## Out of Scope

- <follow-up or tempting expansion>

## Follow-Up Task Ideas

- <optional next slice>
```

## Slug Guidance

- Use lowercase hyphen-case.
- Prefer 2-5 words.
- Name the outcome, not the implementation detail.
- Examples: `add-export-button.md`, `validate-import-csv.md`, `empty-state-copy.md`.
