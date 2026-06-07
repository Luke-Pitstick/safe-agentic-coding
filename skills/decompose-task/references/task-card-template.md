# Task Card Template

Use this template when the decomposition needs to be copied into issues, threads, or agent prompts.

## Compact Task Card

```markdown
### T<N>: <Verb-led title>

Outcome:
<One sentence describing the concrete result.>

Why it matters:
<One sentence tying the task to the broader goal.>

Scope:
- Include: <specific work>
- Exclude: <specific non-goals>

Context packet:
- Goal: <parent goal>
- Inputs: <files, links, docs, designs, logs, datasets, or user notes>
- Constraints: <technical, product, timeline, safety, style, or compatibility limits>
- Prior decisions: <decisions the agent should preserve>
- Reuse/library check: <whether to use $deep-dive to compare existing libraries, framework APIs, standard-library features, or existing project modules before building custom code>

Agent instructions:
1. Inspect <artifact or area>.
2. Determine <unknown, boundary, or approach>.
3. If this task builds common infrastructure, use $deep-dive to identify existing libraries or APIs that could reduce complexity, then decide build vs adopt.
4. Produce <deliverable>.
5. Validate with <test, review, screenshot, metric, or checklist>.

Expansion path:
- If you discover <condition>, split out a child task for <work>.
- Compare at least <N> viable approaches when <decision> is still open.
- Split out a library spike when an existing dependency may simplify the work but adoption risk is unclear.
- Record open questions only if they block implementation or change scope.

Acceptance criteria:
- [ ] <Observable completion condition>
- [ ] <Quality or coverage condition>
- [ ] <Handoff artifact exists and is named>

Dependencies:
- Blocks: <downstream task ids or "none">
- Blocked by: <upstream task ids or "none">

Handoff artifact:
<Expected final note, patch, issue, report, design, test result, or decision record.>
```

## Agent Dispatch Prompt

```markdown
Use the following task card as your full brief. Reconstruct context from the listed artifacts before acting. If you find missing information, make a reasonable assumption unless it changes scope, safety, or the user's stated goal.

Task:
<paste task card>

Return:
- What you changed or produced
- Evidence gathered
- Validation performed
- Remaining risks or child tasks
```

## Decomposition Sanity Checks

- Merge tasks when they share the same files, same acceptance criteria, and same agent context.
- Split tasks when they have different risks, owners, deliverables, or dependency timing.
- Promote an unknown to a discovery task when the answer changes architecture, schedule, or user-visible behavior.
- Put integration after parallel work whenever two agents can produce conflicting outputs.
- Keep follow-up ideas separate from the minimum complete path.
