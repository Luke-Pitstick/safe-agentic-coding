---
name: expand-task
description: Expand a simple task, prompt, rough idea, or one-line feature request into a narrow, app-aware implementation brief for a future delegate agent or skill. Use when the user asks Codex to flesh out an idea, clarify how a small task fits the app, connect a prompt to technical implementation, constrain scope, write an agent brief, or prepare a scoped task artifact in the workspace's agents folder.
---

# Expand Task

## Overview

Turn a small prompt into one narrow, delegate-ready implementation brief. The output should explain why the task matters, how it fits the app, what the technical plan is, what is out of scope, and how another agent should validate completion.

This skill does not implement the task and does not spawn subagents. It writes the final brief to the current workspace's `agents/` folder for use by `$delegate-agent-tasks` or another follow-up skill.

## Operating Rules

- Keep the scope narrow. Shape one coherent task, not an epic.
- Inspect enough of the workspace to connect the idea to the real app before writing the brief.
- Prefer existing architecture, conventions, files, components, APIs, styles, and test patterns.
- Include the grand-scheme context: user value, app area, affected flow, and why this belongs now.
- Convert ambiguity into assumptions, decision points, or explicit open questions.
- Cut attractive but nonessential follow-ups into out-of-scope or future tasks.
- Write the final artifact under `agents/` by default.
- Report the artifact path and the narrow scope summary in chat.

## Workflow

### Step 1: Understand the Prompt

Restate the user's prompt as:

- Intent: what the user wants.
- App fit: where this likely belongs.
- Smallest useful outcome: the minimum result worth delegating.
- Unknowns: what might change the plan.

If the prompt is too broad for one agent task, narrow it to the first useful slice and mention that broader work should use `$decompose-task`.

### Step 2: Inspect the App Context

Read only the relevant context needed to ground the task:

- `README`, `docs/`, `agents/`, `AGENTS.md`, or project notes.
- Package/build config and route/module structure.
- Existing components, services, APIs, models, schemas, tests, or styles near the likely change.
- Recent task briefs in `agents/` when they affect naming, scope, or conventions.

Use `rg --files` and `rg` first. Do not deeply inventory the whole repo unless the prompt is impossible to place.

### Step 3: Choose the Narrow Slice

Define the smallest coherent implementation unit:

- One user-visible behavior, workflow step, API behavior, component state, data path, doc improvement, or testable backend change.
- One primary owner area.
- A limited file/module surface.
- Clear acceptance criteria.
- Clear validation path.

If the natural task spans multiple independent outcomes, write only the first slice and add follow-up task ideas at the bottom of the brief.

### Step 4: Connect to Technical Implementation

Describe:

- Likely files or modules to inspect and edit.
- Data flow or control flow touched.
- Interfaces, APIs, events, routes, state, schemas, or styles involved.
- Existing patterns to follow.
- Test and validation strategy.
- Risks and constraints.

Make implementation guidance specific enough for a delegate agent to start, but avoid over-prescribing private implementation details that should be discovered from code.

### Step 5: Write the Agent Brief

Write the final Markdown brief to the current workspace:

- Resolve workspace root with `git rev-parse --show-toplevel` when inside a Git repo; otherwise use the current working directory.
- Create `<workspace-root>/agents/` if it does not exist.
- Default path: `agents/<task-slug>.md`.
- Use a short, stable slug derived from the narrow task, not the whole prompt.
- If that path exists, update it only when it is clearly the same task; otherwise create `agents/<task-slug>-2.md`, `agents/<task-slug>-3.md`, etc.
- If the user names a destination file, use that path.

Read `references/agent-brief-template.md` for the required artifact shape.

### Step 6: Final Response

Respond with:

- Artifact path.
- One-sentence narrow scope.
- Suggested next command, usually `$delegate-agent-tasks` with the new brief.
- Any assumptions or blockers that remain.

Do not paste the full artifact unless the user asks.

## Quality Bar

Before finalizing, check:

- The brief is narrow enough for one delegate agent.
- The brief explains how the task fits the app's larger product or technical direction.
- The technical plan names concrete likely files, modules, commands, or discovery targets.
- Out-of-scope items are explicit.
- Acceptance criteria and validation steps are testable.
- The artifact was written under `agents/` or the user explicitly requested another path.
- The brief can be used directly by `$delegate-agent-tasks`.

## Anti-Patterns

- Do not turn a simple prompt into a sprawling roadmap.
- Do not skip app-context inspection unless no workspace is available.
- Do not hide uncertainty inside vague implementation prose.
- Do not write a brief that requires the delegate agent to rediscover the user's intent.
- Do not use `docs/` for the final artifact unless the user asks; this skill writes agent handoff material.
