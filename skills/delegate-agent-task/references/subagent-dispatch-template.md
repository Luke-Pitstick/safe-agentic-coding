# Subagent Dispatch Template

Use this template to spawn a subagent from a task card.

## Spawn Prompt

```markdown
Parent goal:
<one-paragraph summary of the overall user goal>

Task ID:
<T<N>>

Task card:
<paste the complete task card>

Ownership:
- Write scope: <files/directories/modules/docs/design artifacts, or "read-only">
- Do not edit outside this scope unless you first explain why it is necessary.
- You are not alone in the codebase. Do not revert unrelated changes, and adapt to existing user or agent edits.

Edit permission:
- <"You may edit files within the write scope" OR "Read-only: do not modify files">
- If edits are allowed, implement the task directly in your workspace and report every changed file.

Commit instructions:
- <"Commit accepted work after targeted validation" OR "Do not commit; leave changes unstaged for parent integration">
- Before committing, run `git status --short` and stage only files inside your ownership scope.
- Do not stage unrelated user or agent changes.
- Use a concise commit message that includes the task ID when one exists.

Relevant skills:
- Use <skill names such as $write-tests, $investigate, $review, $qa, $qa-only, $design-review, or "none"> if available in your session.
- If a listed skill is unavailable, continue with the nearest equivalent workflow and report the gap.

Instructions:
1. Reconstruct context from the listed artifacts before acting.
2. Complete the task card within the ownership boundary.
3. Expand the task only when the expansion is required to satisfy acceptance criteria.
4. Record any child tasks instead of silently broadening scope.
5. Validate the result using the task card's validation method.
6. If blocked by another subagent's work, emit a blocker message naming the task, agent, or artifact you need, then wait for the parent to send that handoff. Do not broaden scope to bypass the dependency.
7. When complete, partial, blocked, or failed, emit the final handoff block exactly once so the parent can detect completion without interrupting you.

Return:
- Start with this final handoff block:
  ```text
  FINAL HANDOFF
  Task ID: <id>
  Status: complete | partial | blocked | failed
  Blocks/depends on: <none or task/agent/artifact>
  Ready for integration: yes | no
  ```
- Task ID and status: complete, partial, blocked, or failed
- Files or artifacts changed
- Commit hash and message, or "not committed" with reason
- Evidence gathered
- Validation performed
- Remaining risks
- Suggested child tasks
```

## Dispatch Board

```markdown
| Task | Agent | Type | Skills | Ownership | Commit | Status | Blocks | Handoff |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | local | parent | $review, $qa | integration | not ready | in progress | T3 | final merge |
| T2 | <agent-id> | worker | $write-tests | <scope> | committed: <hash> | complete | T4 | <artifact> |
```

## Agent Type Guide

- Use `worker` for bounded code, docs, migrations, tests, or artifact creation.
- Use `explorer` for read-only codebase questions with a precise answer shape.
- Use default for mixed reasoning, writing, planning, or general task execution.

## Review Agent Prompts

Use separate review agents when a gate can run independently from implementation.

```markdown
Parent goal:
<goal>

Review target:
<diff, files, URL, screenshots, task cards, or plan>

Relevant skill:
Use <$review | $qa | $qa-only | $design-review | $plan-eng-review | $plan-design-review | $autoplan> if available.

Review instructions:
1. Reconstruct the target context from the provided artifacts.
2. Run the named skill or its closest available equivalent.
3. Do not modify files unless the chosen skill explicitly requires fixing issues and the parent prompt grants write scope.
4. Return prioritized findings, validation evidence, and any follow-up task cards.
```

## Wait Strategy

- Wait immediately only for critical-path results.
- Keep working locally while sidecar agents run.
- Poll running agents in a loop and update the dispatch board instead of sending routine status-check messages.
- Treat each worker's `FINAL HANDOFF` block as its completion or blocker signal.
- If a worker is blocked by another subagent, keep it alive, record the dependency, and send the completed dependency artifact when available.
- Interrupt a worker only for explicit parent input requests, dependency handoffs, narrow repair requests, or returned incomplete work that the same worker can fix.
- Wait for all relevant agents or final handoff blocks before final integration.
- Prefer asking the original agent to repair narrow gaps.
- Prefer local integration for cross-task conflicts.
